# NGƯỜI 2 — Trang chủ, Tìm kiếm & Khám phá thể loại

> **Miền dữ liệu:** `Genre`, `Mood`. **Vai trò:** 2 tab mặt tiền — **Trang chủ** (feed nhiều dải) + **Tìm kiếm** (gõ ra bài hát + lưới thể loại), kèm **Chi tiết thể loại** và **Bảng xếp hạng**. Nặng về RecyclerView đa kiểu.

```
HomeFragment → HomeViewModel → HomeRepository → GET /api/home/feed?filter=
SearchFragment → SearchViewModel → SearchRepository → GET /api/search?q=
   (chưa gõ: lưới thể loại → GenreDetailActivity → GET /api/genres/{id}/feed)
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `fragment/HomeFragment.java` | Fragment | Tab Trang chủ: chip lọc + feed dọc |
| `fragment/SearchFragment.java` | Fragment | Tab Tìm kiếm: ô tìm + lưới thể loại / kết quả |
| `GenreDetailActivity.java` | Activity | Màn 1 thể loại (dải bài/nghệ sĩ/playlist) |
| `HomeViewModel.java` | ViewModel | Tải feed theo filter |
| `SearchViewModel.java` | ViewModel | `onQueryChanged`, kết quả + lưới thể loại |
| `GenreDetailViewModel.java` | ViewModel | Tải feed 1 thể loại |
| `HomeRepository.java` | Repository | `api.homeFeed(filter)` |
| `SearchRepository.java` | Repository | `api.search(q)` |
| `HomeFeedAdapter.java` | Adapter | **Nhạc trưởng**: render mỗi `HomeSection` theo `kind` |
| `HomeTrackRowAdapter.java` | Adapter | Danh sách bài (dùng cả Search) |
| `AlbumLabelAdapter.java` | Adapter | Thẻ album có nhãn Album/EP/Đĩa đơn |
| `ArtistCircleAdapter.java` | Adapter | Avatar nghệ sĩ tròn |
| `RecentTileAdapter.java` | Adapter | Ô "Gần đây" |
| `GenreTileAdapter` / `GenreTileSmallAdapter` | Adapter | Ô thể loại nhiều màu |
| `RadioCardAdapter.java` | Adapter | Thẻ radio (4 màu nền) |
| `ChartCardAdapter.java` | Adapter | Thẻ bảng xếp hạng |
| `TrackCardAdapter`/`AlbumCardAdapter`/`ArtistCardAdapter`/`PlaylistCardAdapter` | Adapter | Thẻ vuông tái dùng |
| `ExploreCardAdapter.java` | Adapter | Thẻ hashtag "#pop…" ở Search |
| `HomeSection`/`SearchResult`/`Genre`/`GenreFeedDto`/`GenreSectionDto` | Model | DTO feed/tìm/thể loại |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/HomeController.java` | Controller | `GET /api/home/feed?filter=all\|music\|following` |
| `service/HomeFeedService.java` | Service | Ghép feed: curated + gần đây + đề xuất + chart + 6 mood + album mới + nghệ sĩ |
| `controller/SearchController.java` | Controller | `GET /api/search?q=` |
| `controller/ChartController.java` | Controller | `GET /api/charts/tracks\|artists` (theo playCount) |
| `controller/GenreController.java` | Controller | `GET /api/genres`, `/{id}/tracks`, `/{id}/feed` |
| `service/GenreFeedService.java` | Service | Ghép feed cho 1 thể loại |
| `service/GenreService.java` | Service | CRUD thể loại |
| `controller/GenreAdminController.java` | Controller | Admin thêm/sửa thể loại |
| `controller/RecommendationController.java` | Controller | `GET /api/recommendations/daily\|mix` |
| `service/RecommendationService.java` | Service | `dailyMix`, nhạc/nghệ sĩ tương tự (Home gọi sang) |
| `entity/Genre.java` | Entity | Bảng `Genres` |
| `entity/Mood.java` | Enum | Tâm trạng (WORKOUT/PARTY/RELAX…) cho mood playlist |
| `repository/GenreRepository.java` | Repository | Truy vấn thể loại |
| `dto/HomeSectionDto` | DTO | 1 dải feed (`kind,title,items`) |
| `dto/GenreRequest, GenreResponse, GenreSectionDto, GenreFeedDto` | DTO | Thể loại |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_home` | `bg_avatar_orange`, `bg_chip_selector` (4 chip lọc) |
| `fragment_search` | `bg_avatar_orange`, `ic_camera`, `bg_search_bar`, `ic_search` |
| `activity_genre_detail` | `placeholder_gradient`, `ic_arrow_back` |
| `item_home_featured` | `bg_card_dark`, `ic_more_vert` |
| `item_home_chart` | `bg_card_top50` |
| `item_home_recent_tile` | `bg_recent_tile` |
| `item_genre_tile` / `_small` | `bg_genre_tile`, `placeholder_gradient` |
| `item_radio_card` | `bg_radio_pastel` (Java đổi: `bg_radio_orange/teal/purple`) |
| `item_artist_circle` / `item_explore_artist_card` | `placeholder_gradient` / `bg_card_dark` |
| shimmer home/genre | `bg_shimmer_box` |

> Không màn nào của bạn dùng `anim` riêng.

## 4. LUỒNG HOME FEED
1. `HomeFragment` → `vm.loadIfNeeded()` → `HomeRepository.getFeed(filter)` → `GET /api/home/feed`.
2. BE `HomeFeedService.buildFeed(userId, filter)` ghép nhiều dải (FEATURED, RECENTLY_PLAYED, RECOMMENDED→gọi `RecommendationService`, CHART, 6×MOOD, NEW_RELEASES, POPULAR_ARTISTS), bỏ dải rỗng.
3. `HomeFeedAdapter` render mỗi `kind` thành 1 RecyclerView ngang. Bấm thẻ → `NavHelper.openAlbum/openArtist/openPlaylist` hoặc mở `PlayerActivity`.

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/home/feed`, `/api/search`, `/api/genres{,/{id}/tracks,/{id}/feed}`, `/api/charts/{tracks,artists}`, `/api/recommendations/{daily,mix}`.
- **Giao:** bấm item → `NavHelper` (Người 1) mở chi tiết Album (Người 4) / Nghệ sĩ (Người 3) / Player (Người 3). `HomeFeedService` đọc `PlayHistory`/`Album`/`Artist` (entity của Người 3/4). `GenreDetailViewModel` dùng `LibraryRepository.getGenreFeed` (Người 4 quản repo, nhưng endpoint genre là của bạn).
