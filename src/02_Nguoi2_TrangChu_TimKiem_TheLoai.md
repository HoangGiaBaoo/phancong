# NGƯỜI 2 — Trang chủ, Tìm kiếm & Khám phá thể loại

> **Miền dữ liệu:** `Genre`, `Mood`. **Vai trò:** 2 tab mặt tiền — **Trang chủ** (feed nhiều dải) + **Tìm kiếm** (gõ ra bài hát + lưới thể loại), kèm **Chi tiết thể loại** và **Bảng xếp hạng**. Nặng về RecyclerView đa kiểu nhưng lặp khuôn → **mảng dễ học nhất**.

```
HomeFragment → HomeViewModel → HomeRepository → GET /api/home/feed?filter=
SearchFragment → SearchViewModel → SearchRepository → GET /api/search?q=
   (chưa gõ: lưới thể loại → GenreDetailActivity → GET /api/genres/{id}/feed)
```

> 📌 Các **thẻ trong feed** (album / nghệ sĩ / playlist) bấm vào → `NavHelper` mở màn chi tiết do người khác sở hữu (Album & Playlist = Người 4, Nghệ sĩ = Người 3). Bạn chỉ lo phần **hiển thị feed**, không lo màn chi tiết.

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
| `GenreFeedAdapter.java` | Adapter | Render feed 1 thể loại (đa kiểu) |
| `HomeTrackRowAdapter.java` | Adapter | Danh sách bài (dùng cả Search) |
| `AlbumLabelAdapter.java` / `AlbumCardAdapter.java` | Adapter | Thẻ album trong feed (bấm → Album detail của Người 4) |
| `ArtistCircleAdapter.java` | Adapter | Avatar nghệ sĩ tròn (discovery) |
| `RecentTileAdapter.java` | Adapter | Ô "Gần đây" |
| `GenreTileAdapter` / `GenreTileSmallAdapter` | Adapter | Ô thể loại nhiều màu |
| `RadioCardAdapter.java` | Adapter | Thẻ radio (dải nhạc theo nghệ sĩ) |
| `TrackCardAdapter` / `PlaylistCardAdapter` | Adapter | Thẻ vuông tái dùng (PlaylistCard ghép bìa 2x2 của Người 4) |
| `HomeSection`/`SearchResult`/`Genre`/`GenreFeedDto`/`GenreSectionDto` | Model | DTO feed/tìm/thể loại |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/HomeController.java` | Controller | `GET /api/home/feed?filter=all\|music\|following` |
| `service/HomeFeedService.java` | Service | Ghép feed: curated + gần đây + đề xuất + chart + 6 mood + album mới + nghệ sĩ |
| `controller/SearchController.java` | Controller | `GET /api/search?q=` |
| `controller/ChartController.java` | Controller | `GET /api/charts/tracks\|artists` (theo playCount) |
| `controller/GenreController.java` | Controller | `GET /api/genres`, `/{id}/tracks`, `/{id}/feed` |
| `service/GenreFeedService.java` / `GenreService.java` | Service | Ghép feed 1 thể loại / CRUD thể loại |
| `controller/RecommendationController.java` / `service/RecommendationService.java` | Controller+Service | `daily/mix`, nhạc/nghệ sĩ tương tự (Home gọi sang) |
| `entity/Genre.java` / `Mood.java` | Entity/Enum | Bảng `Genres` / tâm trạng mood playlist |
| `repository/GenreRepository.java` | Repository | Truy vấn thể loại |
| `dto/HomeSectionDto, GenreRequest/Response/SectionDto/FeedDto` | DTO | Feed/thể loại |

> ⚠️ `GenreAdminController` (admin web) — xem mục admin ở `00`; chỉ cần nếu nhóm demo trang admin.

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_home` | `bg_avatar_orange`, `bg_chip_selector` (4 chip lọc) |
| `fragment_search` | `bg_avatar_orange`, `ic_camera`, `bg_search_bar`, `ic_search` |
| `activity_genre_detail` | `placeholder_gradient`, `ic_arrow_back` |
| `item_home_featured` | `bg_card_dark`, `ic_more_vert` (bìa ghép 2x2 qua `PlaylistCoverView`) |
| `item_home_recent_tile` | `bg_recent_tile` |
| `item_genre_tile` / `_small` / `item_genre_section` | `bg_genre_tile`, `placeholder_gradient` |
| `item_radio_card` | `bg_radio_pastel` (Java đổi: orange/teal/purple) |
| `item_artist_circle` / `item_album_card` / `item_album_label_card` / `item_track_card` | `placeholder_gradient` |
| shimmer home/genre | `bg_shimmer_box` |

> Không màn nào của bạn dùng `anim` riêng (chuyển màn dùng anim chung Người 1).

## 4. ⭐ THỨ TỰ ĐỌC FILE

**Luồng Trang chủ (đọc trước):**
1. `fragment/HomeFragment` → `HomeViewModel` → `data/repository/HomeRepository` → `ApiService.homeFeed`.
2. `adapter/HomeFeedAdapter` (xem nó chọn adapter con theo `kind`: HomeTrackRow / AlbumLabel / ArtistCircle / PlaylistCard / RecentTile).
3. BE: `HomeController` → `HomeFeedService` (ghép dải; gọi `RecommendationService`, `ChartController` nguồn).

**Luồng Tìm kiếm:** 4. `fragment/SearchFragment` → `SearchViewModel` → `SearchRepository` → `SearchController`.

**Luồng Thể loại:** 5. `SearchFragment` (lưới thể loại) → `GenreDetailActivity` → `GenreDetailViewModel` → `LibraryRepository.getGenreFeed` (repo của Người 4) → `GenreController` → `GenreFeedService`; xem `adapter/GenreFeedAdapter`.

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/home/feed`, `/api/search`, `/api/genres{,/{id}/tracks,/{id}/feed}`, `/api/charts/{tracks,artists}`, `/api/recommendations/{daily,mix}`.
- **Giao:** bấm thẻ trong feed → `NavHelper` (Người 1) mở chi tiết Album (Người 4) / Nghệ sĩ (Người 3) / Playlist (Người 4) / Player (Người 3) · `HomeFeedService` đọc `PlayHistory`/`Artist` (Người 3) + `Album` & `Playlist` curated (Người 4) · `GenreDetailViewModel` dùng `LibraryRepository.getGenreFeed` (Người 4 quản repo, endpoint genre là của bạn).
