# NGƯỜI 2 — Trang chủ, Tìm kiếm & Khám phá thể loại

> **Bạn là "mặt tiền" của app.** Đây là 2 tab đầu tiên người dùng nhìn thấy: **Trang chủ** (feed nhiều dải nội dung kiểu Spotify) và **Tìm kiếm** (gõ chữ ra bài hát + lưới thể loại "Duyệt tìm tất cả"). Kèm theo là màn **Chi tiết thể loại** và **Bảng xếp hạng**. Mảng này nặng về **RecyclerView nhiều kiểu item** và **feed động ghép từ nhiều nguồn**.

---

## 0. NỀN TẢNG DÙNG CHUNG (đọc nhanh — chi tiết ở file 01)
Mọi màn theo khuôn: **Fragment → ViewModel → Repository → ApiService → BE Controller → Service → Repository → Entity**.
- `network/ApiService.java` = nơi khai báo endpoint (đọc phần Home/Search/Genre/Charts).
- `viewmodel/VmFactory.java` = nơi tạo `HomeViewModel`, `SearchViewModel`, `GenreDetailViewModel`.
- `util/NavHelper.java` = mở màn chi tiết (bạn gọi `NavHelper.openAlbum/openArtist/openPlaylist`).
- Ảnh tải bằng Glide từ `RetrofitClient.BASE_MEDIA_URL + coverUrl`.

---

## 1. SƠ ĐỒ LUỒNG MẢNG CỦA BẠN

```
Tab Trang chủ ─► HomeFragment ─► HomeViewModel ─► HomeRepository ─► GET /api/home/feed?filter=
                     │ RecyclerView dọc, mỗi dòng là 1 "section" với kiểu khác nhau
                     │ (FEATURED, RECENTLY_PLAYED, CHART, MOOD_PLAYLIST, NEW_RELEASES...)
                     └─ bấm item ─► NavHelper.openAlbum/openArtist/openPlaylist hoặc mở Player

Tab Tìm kiếm ─► SearchFragment ─► SearchViewModel ─► SearchRepository ─► GET /api/search?q=
                     │ gõ chữ → hiện danh sách bài hát; chưa gõ → lưới thể loại "Duyệt tìm tất cả"
                     └─ bấm 1 thể loại ─► GenreDetailActivity ─► GET /api/genres/{id}/feed
```

---

## 2. CHI TIẾT TỪNG CHỨC NĂNG

### 2.1. TRANG CHỦ (Home feed) — phần khó nhất của bạn

**File cần đọc (FE):**
- `fragment/HomeFragment.java` — màn tab Trang chủ. Tạo `HomeFeedAdapter`, gắn các chip lọc (Tất cả / Nhạc / Đang theo dõi), mở drawer khi bấm avatar.
- `res/layout/fragment_home.xml` — thanh chip lọc + `rvHomeFeed` (RecyclerView dọc) + shimmer (khung xương lúc tải).
- `viewmodel/HomeViewModel.java` — gọi feed theo filter, giữ `LiveData<List<HomeSection>>`.
- `data/repository/HomeRepository.java` — gọi `api.homeFeed(filter)`.
- `model/HomeSection.java` — 1 dải nội dung: có `kind` (loại), `title`, `items` (list mix album/track/artist/playlist).

**File cần đọc (Adapter — đây là phần "nhiều kiểu item"):**
- `adapter/HomeFeedAdapter.java` — **adapter cha**: mỗi `HomeSection` render khác nhau tuỳ `kind`. Nó chứa các RecyclerView con nằm ngang. Có interface `OnHomeItemClick` (onTrack/onArtist/onAlbum/onPlaylist) để báo ngược về HomeFragment.
- `adapter/HomeTrackRowAdapter.java` — danh sách bài hát dạng hàng (dùng cả ở Search).
- `adapter/AlbumLabelAdapter.java` — thẻ album có nhãn "Album/Đĩa đơn/EP".
- `adapter/ArtistCircleAdapter.java` — avatar nghệ sĩ hình tròn ("Nghệ sĩ phổ biến").
- `adapter/RecentTileAdapter.java` — ô vuông "Gần đây".
- `adapter/GenreTileAdapter.java` — ô thể loại nhiều màu.
- `adapter/RadioCardAdapter.java` — thẻ radio.
- `adapter/ChartCardAdapter.java` — thẻ bảng xếp hạng.
- `adapter/TrackCardAdapter.java`, `AlbumCardAdapter.java`, `ArtistCardAdapter.java`, `PlaylistCardAdapter.java` — các thẻ vuông tái dùng nhiều nơi.
- **Layout item tương ứng:** `item_home_section.xml`, `item_home_featured.xml`, `item_home_chart.xml`, `item_home_recent_tile.xml`, `item_album_label_card.xml`, `item_artist_circle.xml`, `item_genre_tile.xml`, `item_radio_card.xml`, `item_track_card.xml`, `item_album_card.xml`, `item_artist_card.xml`, `item_playlist_card.xml`, `item_track_row.xml`.
- **Shimmer:** `layout_shimmer_home_section.xml` (khung xương xám lúc đang tải).

**File cần đọc (BE):**
- `controller/HomeController.java` — `GET /api/home/feed?filter=all|music|following`.
- `service/HomeFeedService.java` — **đọc kỹ**: nơi "lắp ráp" feed từ nhiều nguồn.
- `dto/HomeSectionDto.java` — cấu trúc 1 dải trả về (`kind`, `title`, `subtitle`, `items`).
- `service/RecommendationService.java` + `controller/RecommendationController.java` — đề xuất hằng ngày (`dailyMix`) + nhạc/nghệ sĩ tương tự (`/api/recommendations/daily`, `/recommendations/mix`). Home gọi sang đây.

**Luồng Home feed (học thuộc):**
1. `HomeFragment.onViewCreated` → tạo `HomeViewModel` → `vm.loadIfNeeded()`.
2. `HomeViewModel.load(filter)` → `HomeRepository.getFeed(filter)` → `api.homeFeed(filter)` → `GET /api/home/feed`.
3. BE `HomeController` lấy userId từ JWT → `HomeFeedService.buildFeed(userId, filter)`:
   - Lấy playlist **curated** (do admin tạo) → tạo dải `FEATURED` + `TOP_PICKS`.
   - Lấy lịch sử nghe gần đây (`PlayHistory`) → dải `RECENTLY_PLAYED`.
   - Gọi `RecommendationService.dailyMix(userId, 10)` → dải `RECOMMENDED`.
   - Top theo `playCount` → dải `CHART`.
   - Lần lượt 6 tâm trạng (Mood): Hoài niệm, Tập luyện, Tiệc tùng, Vui vẻ, Thư giãn, Hát theo → các dải `MOOD_PLAYLIST`.
   - Album mới → `NEW_RELEASES`; nghệ sĩ nhiều follower → `POPULAR_ARTISTS`.
   - Lọc bỏ dải rỗng. (filter=`following` chỉ trả nghệ sĩ đang theo dõi; `music` bỏ FEATURED.)
4. Về FE: `HomeViewModel.sections` cập nhật → `HomeFragment` đổ vào `HomeFeedAdapter` → mỗi `kind` render 1 kiểu RecyclerView con.
5. Bấm 1 thẻ → `HomeFeedAdapter` gọi callback → `HomeFragment.onAlbum/onArtist/...` → `NavHelper.openXxx` (Người 3/4) hoặc mở `PlayerActivity` (Người 3).

> 💡 **Mẹo nhớ:** Home = "1 RecyclerView dọc, mỗi dòng lại là 1 RecyclerView ngang". `HomeFeedAdapter` là "nhạc trưởng" phân loại theo `kind`.

### 2.2. TÌM KIẾM

**File cần đọc (FE):**
- `fragment/SearchFragment.java` — ô tìm kiếm + 2 chế độ hiển thị: (a) chưa gõ → lưới thể loại "Duyệt tìm tất cả" + các thẻ "Khám phá"; (b) đang gõ → danh sách bài hát kết quả.
- `res/layout/fragment_search.xml` — `etSearch`, `defaultContainer` (lưới thể loại), `resultsContainer` (kết quả), shimmer.
- `viewmodel/SearchViewModel.java` — nhận `onQueryChanged`, gọi search, giữ `trackResults`, `genres`, `showingResults` (cờ chuyển chế độ).
- `data/repository/SearchRepository.java` — gọi `api.search(q)`.
- `model/SearchResult.java` — kết quả `{tracks, artists, albums...}`.
- **Adapter:** `ExploreCardAdapter` (thẻ hashtag "#pop đương đại"...), `GenreTileAdapter` (lưới thể loại), `HomeTrackRowAdapter` (danh sách kết quả — dùng lại của Home).
- **Layout:** `item_explore_card.xml`, `item_genre_tile.xml`, `layout_shimmer_genre_grid.xml`.

**File cần đọc (BE):**
- `controller/SearchController.java` — `GET /api/search?q=` → tìm theo tên bài/nghệ sĩ/album.
- (dùng `TrackRepository`, `ArtistRepository`, `AlbumRepository` — thuộc Người 3, chỉ đọc tham khảo).

**Luồng:** gõ chữ → `TextWatcher` → `vm.onQueryChanged(q)` → (thường có debounce/đợi) → `SearchRepository.search(q)` → `GET /api/search` → kết quả về → `renderResults()` + bật `resultsContainer`, ẩn `defaultContainer`. Xoá hết chữ → quay lại lưới thể loại.

### 2.3. CHI TIẾT THỂ LOẠI (bấm 1 ô thể loại)

**File cần đọc (FE):**
- `GenreDetailActivity.java` + `activity_genre_detail.xml` — màn 1 thể loại (vd "Pop"): có dải bài hát, nghệ sĩ, playlist theo thể loại.
- `viewmodel/GenreDetailViewModel.java` — gọi feed thể loại.
- `data/repository/LibraryRepository.java` (hàm `getGenreFeed`) — dùng chung với Người 4.
- `model/GenreFeedDto.java`, `model/GenreSectionDto.java`, `model/Genre.java`.
- **Adapter:** `GenreTileSmallAdapter`, các adapter dải tái dùng.
- **Layout:** `item_genre_tile_small.xml`, `item_genre_section.xml`, `layout_shimmer_genre_detail.xml`.

**File cần đọc (BE):**
- `controller/GenreController.java` — `GET /api/genres`, `/genres/{id}/tracks`, `/genres/{id}/feed`.
- `service/GenreFeedService.java` — ghép feed cho 1 thể loại.
- `service/GenreService.java` — CRUD thể loại cơ bản.
- `entity/Genre.java`, `entity/Mood.java`, `entity/AlbumType.java`, `repository/GenreRepository.java`.
- `dto/GenreFeedDto.java`, `dto/GenreSectionDto.java`, `dto/GenreResponse.java`, `dto/GenreRequest.java`.
- `controller/GenreAdminController.java` — admin thêm/sửa thể loại (tham khảo).

**Luồng:** bấm ô thể loại trong Search → `Intent` kèm `genreId/genreName` → `GenreDetailActivity` → `GenreDetailViewModel` → `LibraryRepository.getGenreFeed(id)` → `GET /api/genres/{id}/feed` → `GenreFeedService` ghép các dải → render.

### 2.4. BẢNG XẾP HẠNG (Charts)

**File (BE):** `controller/ChartController.java` — `GET /api/charts/tracks?limit=`, `/charts/artists?limit=` (sắp theo `playCount`).
**File (FE):** hiển thị qua dải `CHART` trong Home (`ChartCardAdapter` + `item_home_chart.xml`). Nếu có màn chart riêng thì cũng thuộc bạn.

---

## 3. ✅ CHECKLIST "FILE CỦA TÔI" (Người 2)

**Android — màn hình & layout:**
- [ ] `fragment/HomeFragment.java` + `fragment_home.xml`
- [ ] `fragment/SearchFragment.java` + `fragment_search.xml`
- [ ] `GenreDetailActivity.java` + `activity_genre_detail.xml`

**Android — ViewModel / Repository:**
- [ ] `HomeViewModel`, `SearchViewModel`, `GenreDetailViewModel`
- [ ] `HomeRepository`, `SearchRepository` (+ dùng chung `LibraryRepository.getGenreFeed`)

**Android — Adapter (nhiều!):**
- [ ] `HomeFeedAdapter`, `HomeTrackRowAdapter`, `AlbumLabelAdapter`, `ArtistCircleAdapter`, `RecentTileAdapter`, `GenreTileAdapter`, `GenreTileSmallAdapter`, `RadioCardAdapter`, `ChartCardAdapter`
- [ ] `TrackCardAdapter`, `AlbumCardAdapter`, `ArtistCardAdapter`, `PlaylistCardAdapter`, `ExploreCardAdapter`
- [ ] models: `HomeSection`, `SearchResult`, `Genre`, `GenreFeedDto`, `GenreSectionDto`
- [ ] các `item_*.xml` + shimmer tương ứng (liệt kê ở mục 2)

**Backend:**
- [ ] `controller/HomeController`, `service/HomeFeedService`, `dto/HomeSectionDto`
- [ ] `controller/SearchController`
- [ ] `controller/ChartController`
- [ ] `controller/GenreController`, `service/GenreFeedService`, `service/GenreService`, `controller/GenreAdminController`
- [ ] `service/RecommendationService`, `controller/RecommendationController`
- [ ] `entity/Genre`, `entity/Mood`, `entity/AlbumType`, `repository/GenreRepository`
- [ ] `dto/GenreFeedDto`, `dto/GenreSectionDto`, `dto/GenreResponse`, `dto/GenreRequest`

---

## 4. ENDPOINT API MẢNG BẠN DÙNG
- `GET /api/home/feed?filter=all|music|following`
- `GET /api/search?q=`
- `GET /api/genres`, `GET /api/genres/{id}/tracks`, `GET /api/genres/{id}/feed`
- `GET /api/charts/tracks?limit=`, `GET /api/charts/artists?limit=`
- `GET /api/recommendations/daily`, `GET /api/recommendations/mix`

## 5. ĐIỂM GIAO VỚI NGƯỜI KHÁC
- Bấm item trên Home/Search → gọi `NavHelper` (Người 1) để mở **màn chi tiết Album/Nghệ sĩ** (Người 3) hoặc **Playlist** (Người 4), hoặc mở **Player** (Người 3).
- `HomeFeedService` (của bạn) gọi `RecommendationService` + đọc `PlayHistory`/`Album`/`Artist` repository (entity do Người 3/4 sở hữu) → khi họ đổi entity, feed có thể ảnh hưởng.
- `HomeTrackRowAdapter` dùng chung giữa Home và Search — sửa cẩn thận.
