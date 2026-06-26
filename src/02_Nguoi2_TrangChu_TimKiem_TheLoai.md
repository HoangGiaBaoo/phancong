# NGƯỜI 2 — Trang chủ, Tìm kiếm, Thể loại & Album

> **Miền dữ liệu:** `Genre`, `Mood`, **`Album`**, `AlbumType`, `SavedAlbum`. **Vai trò:** 2 tab mặt tiền — **Trang chủ** (feed nhiều dải) + **Tìm kiếm** (gõ ra bài hát + lưới thể loại), **Chi tiết thể loại**, **Bảng xếp hạng**, và (mới nhận) **Chi tiết Album + lưu album vào thư viện**. Nặng về RecyclerView đa kiểu.

```
HomeFragment → HomeViewModel → HomeRepository → GET /api/home/feed?filter=
SearchFragment → SearchViewModel → SearchRepository → GET /api/search?q=
   (chưa gõ: lưới thể loại → GenreDetailActivity → GET /api/genres/{id}/feed)
NavHelper.openAlbum → AlbumDetailFragment → GET /api/albums/{id}/tracks (+ nút lưu thư viện)
```

> 📌 Miền **Album** thuộc bạn — vì luồng tự nhiên là *duyệt (home/search/genre) → mở album*. Album đã lưu vẫn **hiện trong Thư viện của Người 4** qua endpoint của bạn (giống cách Library hiện liked/followed của Người 3).

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `fragment/HomeFragment.java` | Fragment | Tab Trang chủ: chip lọc + feed dọc |
| `fragment/SearchFragment.java` | Fragment | Tab Tìm kiếm: ô tìm + lưới thể loại / kết quả |
| `GenreDetailActivity.java` | Activity | Màn 1 thể loại (dải bài/nghệ sĩ/playlist) |
| `fragment/AlbumDetailFragment.java` ⭐ | Fragment | Chi tiết album + nút [+thư viện][tải][⋮][shuffle][play] (kế thừa `BaseDetailFragment`) |
| `fragment/AlbumMenuBottomSheet.java` | BottomSheet | Menu ⋮ album: thêm/xoá thư viện, tải→gate, tới nghệ sĩ |
| `HomeViewModel.java` | ViewModel | Tải feed theo filter |
| `SearchViewModel.java` | ViewModel | `onQueryChanged`, kết quả + lưới thể loại |
| `GenreDetailViewModel.java` | ViewModel | Tải feed 1 thể loại |
| `AlbumDetailViewModel.java` ⭐ | ViewModel | Tải album + bài + trạng thái "đã lưu" |
| `HomeRepository.java` | Repository | `api.homeFeed(filter)` |
| `SearchRepository.java` | Repository | `api.search(q)` |
| `HomeFeedAdapter.java` | Adapter | **Nhạc trưởng**: render mỗi `HomeSection` theo `kind` |
| `GenreFeedAdapter.java` | Adapter | Render feed 1 thể loại (đa kiểu) |
| `HomeTrackRowAdapter.java` | Adapter | Danh sách bài (dùng cả Search) |
| `AlbumLabelAdapter.java` / `AlbumCardAdapter.java` | Adapter | Thẻ album có nhãn / thẻ album vuông |
| `ArtistCircleAdapter.java` | Adapter | Avatar nghệ sĩ tròn (discovery) |
| `RecentTileAdapter.java` | Adapter | Ô "Gần đây" |
| `GenreTileAdapter` / `GenreTileSmallAdapter` | Adapter | Ô thể loại nhiều màu |
| `RadioCardAdapter.java` | Adapter | Thẻ radio (dải nhạc theo nghệ sĩ) |
| `TrackCardAdapter` / `PlaylistCardAdapter` | Adapter | Thẻ vuông tái dùng (PlaylistCard ghép bìa 2x2 của Người 4) |
| `HomeSection`/`SearchResult`/`Genre`/`GenreFeedDto`/`GenreSectionDto`/**`Album`** | Model | DTO feed/tìm/thể loại/album |

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
| `controller/AlbumController.java` ⭐ | Controller | `/api/albums`, `/new`, `/{id}`, `/{id}/tracks`, `/saved`, `/{id}/saved`, `/{id}/save` |
| `service/AlbumService.java` ⭐ | Service | Logic album |
| `service/LibraryService.java` ⭐ | Service | Lưu album: `getSavedAlbums/toggleSaveAlbum/isAlbumSaved` (chỉ phần album) |
| `entity/Genre.java` / `Mood.java` | Entity/Enum | Bảng `Genres` / tâm trạng mood playlist |
| `entity/Album.java` / `AlbumType.java` ⭐ | Entity/Enum | Bảng `Albums` / ALBUM·SINGLE·EP·COMPILATION |
| `entity/SavedAlbum.java` + `SavedAlbumId.java` ⭐ | Entity | Album đã lưu (khoá kép user+album) |
| `repository/GenreRepository, AlbumRepository, SavedAlbumRepository` | Repository | Truy vấn |
| `dto/HomeSectionDto, GenreRequest/Response/SectionDto/FeedDto` + dto Album | DTO | Feed/thể loại/album |

> ⚠️ `GenreAdminController` (admin web) — xem mục admin ở `00`; chỉ cần nếu nhóm demo trang admin.

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_home` | `bg_avatar_orange`, `bg_chip_selector` (4 chip lọc) |
| `fragment_search` | `bg_avatar_orange`, `ic_camera`, `bg_search_bar`, `ic_search` |
| `activity_genre_detail` | `placeholder_gradient`, `ic_arrow_back` |
| `fragment_album_detail` | `placeholder_gradient`, `ic_arrow_back`, `ic_add_circle_outline`, `ic_download`, `ic_more_vert`, `ic_shuffle`, `ic_play` (Java đổi `ic_check_circle_green` khi đã lưu) |
| `bottom_sheet_album_menu` | `placeholder_gradient`, `ic_add_circle_outline`, `ic_download`, `ic_person` |
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

**Luồng Album (mới nhận):** 6. `NavHelper.openAlbum` → `fragment/AlbumDetailFragment` (kế thừa `BaseDetailFragment`) → `AlbumDetailViewModel` → `LibraryRepository` (hàm album, repo Người 4) → `AlbumController` → `AlbumService` → `entity/Album`; nút lưu → `LibraryService` + `SavedAlbum`; menu ⋮ → `AlbumMenuBottomSheet`.

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/home/feed`, `/api/search`, `/api/genres{,/{id}/tracks,/{id}/feed}`, `/api/charts/{tracks,artists}`, `/api/recommendations/{daily,mix}`, **`/api/albums/*` (gồm saved)**.
- **Giao:** bấm item → `NavHelper` (Người 1) mở chi tiết Nghệ sĩ (Người 3) / Player (Người 3) / Album (bạn) · `HomeFeedService` đọc `PlayHistory`/`Artist` (Người 3) + `Playlist` curated (Người 4) · `AlbumDetailViewModel` dùng `LibraryRepository` (Người 4 quản, dùng chung) · bài trong album bấm phát → `PlayerManager.play()` (Người 3) · nút "Tải" → gate (Người 5) · **Album đã lưu hiện ở Thư viện (Người 4)** qua `/api/albums/saved` của bạn.
