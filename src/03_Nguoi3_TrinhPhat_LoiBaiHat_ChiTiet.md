# NGƯỜI 3 — Trình phát nhạc, Lời bài hát & Chi tiết Bài/Album/Nghệ sĩ

> **Bạn là "trái tim nghe nhạc" của app.** Bạn lo việc phát nhạc thật (ExoPlayer), màn Player full, lời bài hát chạy theo nhạc (LRC sync), và các màn chi tiết **Album / Nghệ sĩ** — nơi người dùng bấm "Phát". Backend của bạn gồm **stream file nhạc theo HTTP Range** (quan trọng để tua nhạc), Track/Album/Artist và lịch sử nghe. Mảng này ít màn nhưng **kỹ thuật sâu nhất**.

---

## 0. NỀN TẢNG DÙNG CHUNG (đọc nhanh — chi tiết ở file 01)
Khuôn MVVM như cả nhóm. Riêng bạn có thêm **1 thành phần đặc biệt không theo MVVM**: `PlayerManager` — một **singleton giữ ExoPlayer** sống xuyên suốt app (không chết khi đổi màn). Mọi nơi muốn phát nhạc đều gọi `PlayerManager.getInstance().play(...)`.

---

## 1. SƠ ĐỒ LUỒNG MẢNG CỦA BẠN

```
Bấm 1 bài (ở Home/Search/Album/Playlist...) 
        │ Intent("track") 
        ▼
  PlayerActivity ──► PlayerViewModel ──► PlayerManager (ExoPlayer singleton)
        │                  ▲ poll 500ms vị trí phát        │ phát URL nhạc
        │                  │                                ▼
        │            LiveData(track, playing, progress)   GET /audio/{file} (HTTP Range)
        │  bấm "Lời bài hát"
        ▼
  LyricsActivity (full lyrics, cuộn) ◄── LrcParser (tách timestamp [mm:ss.xx])

NavHelper.openAlbum  ──► AlbumDetailFragment ──► AlbumDetailViewModel ──► GET /api/albums/{id}/tracks
NavHelper.openArtist ──► ArtistDetailFragment ──► ArtistDetailViewModel ──► GET /api/artists/{id}/...
```

---

## 2. CHI TIẾT TỪNG CHỨC NĂNG

### 2.1. ENGINE PHÁT NHẠC — `PlayerManager` (đọc kỹ nhất)

**File:** `util/PlayerManager.java`
**Vai trò:** Singleton bọc **ExoPlayer**. Giữ: `player`, `currentTrack`, `queue` (hàng đợi), `currentIndex`, `shuffleEnabled` (mặc định BẬT — giống Spotify Free). Có 2 listener: `listener` (cho UI) và `adListener` (đếm quảng cáo — của Người 5).

**Hàm chính cần nhớ:**
- `init(ctx)` — tạo ExoPlayer 1 lần, cấu hình audio focus, khoá tốc độ 1x (tránh trôi tốc độ trên máy ảo).
- `play(ctx, track, trackList, index)` — nạp hàng đợi + phát bài, set `MediaItem` từ `BASE_MEDIA_URL + track.getAudioUrl()`.
- `playNext()` / `playPrevious()` — nếu shuffle thì chọn ngẫu nhiên; hết bài tự `playNext()` (qua listener `STATE_ENDED`).
- `togglePlayPause()`, `seekTo()`, `getCurrentPosition()`, `getDuration()`.
- `setListener()` / `setAdListener()` — đăng ký nghe sự kiện đổi bài / đổi trạng thái phát.

> ⚠️ Chỉ có **1 slot listener UI**. Màn nào đang hiện thì giành slot đó (Player giành khi mở, mini-player ở Main dùng cách poll). Đây là lý do có `MiniPlayerController`.

### 2.2. MÀN PLAYER (toàn màn hình)

**File cần đọc (FE):**
- `PlayerActivity.java` — **đọc kỹ**. Bind: ảnh bìa (+ đổi màu nền theo bìa bằng Palette), tên bài/nghệ sĩ, seekbar, nút play/next/prev/shuffle/like/follow, "teaser" lời bài hát chạy theo nhạc.
- `res/layout/activity_player.xml` — bố cục màn phát.
- `viewmodel/PlayerViewModel.java` — **đọc kỹ**. Implements `PlayerManager.OnTrackChangeListener`. Giữ `currentTrack`, `playing`, `liked`, `following`, `progress`. Có `poller` (Handler 500ms) cập nhật vị trí phát ra seekbar.
- `data/repository/PlayerRepository.java` — gọi `toggleLike`, `toggleFollow`, `checkFollowState`, `recordPlay`.
- `adapter/ExploreArtistAdapter.java` — dải "Khám phá nghệ sĩ" dưới màn Player.
- `util/TimeUtil.java` — format mm:ss.

**Luồng mở Player (học thuộc):**
1. Nơi khác `startActivity(PlayerActivity, extra "track")`.
2. `PlayerActivity.onCreate`: nếu bài này chưa phát → `PlayerManager.play(...)`. Tạo `PlayerViewModel` + gọi `vm.recordPlay(trackId)` (ghi lịch sử nghe).
3. `PlayerViewModel` (constructor): `PlayerManager.setListener(this)`, lấy bài hiện tại, **bật poller 500ms**.
4. Poller mỗi 500ms đọc `getCurrentPosition()/getDuration()` → cập nhật `progress` LiveData → `PlayerActivity.renderProgress()` kéo seekbar + đồng hồ.
5. Bấm play/pause → `vm.onPlayPauseClicked()` → `PlayerManager.togglePlayPause()`.
6. Khi nhạc tự sang bài mới → `PlayerManager` gọi `onTrackChanged()` → `PlayerViewModel` cập nhật `currentTrack` → màn vẽ lại.
7. Nút **Like** → `vm.onLikeClicked()` → `PlayerRepository.toggleLike(trackId)` → `POST /api/tracks/{id}/like` (ghi vào bảng LikedTrack của Người 4).
8. Nút **Theo dõi nghệ sĩ** → `vm.onFollowArtistClicked()` → `POST /api/artists/{id}/follow`.
9. Nút **Shuffle**: nếu Free → mở `ShuffleGateBottomSheet` (Người 5); nếu Premium → bật/tắt thật. Nút **Prev** bị mờ với Free (ép trộn bài).

### 2.3. LỜI BÀI HÁT (Lyrics + LRC sync)

**File cần đọc (FE):**
- `LyricsActivity.java` + `activity_lyrics.xml` — màn lời đầy đủ, cuộn, tô sáng dòng đang hát.
- `util/LrcParser.java` — **đọc kỹ**. Tách chuỗi LRC `[mm:ss.xx] lời...` thành list `LrcLine{timeMs, text}`. Hàm `findActiveIndex(lines, posMs)` tìm dòng đang hát; `toCleanText()` bỏ timestamp.
- `adapter/LyricLineAdapter.java` + `item_lyric_line.xml` — từng dòng lời.

**Luồng:** Track có cột `lyrics` (text LRC). `PlayerActivity.renderTrack()` parse bằng `LrcParser.parse()`; "teaser" 1 dòng chạy theo nhạc nhờ Handler 100ms gọi `findActiveIndex`. Bấm "Lời bài hát" → mở `LyricsActivity` xem full.

### 2.4. CHI TIẾT ALBUM

**File cần đọc (FE):**
- `fragment/AlbumDetailFragment.java` — bản Fragment (mở trong Main, bottom-nav đứng yên). Hiện bìa, tên, nghệ sĩ, danh sách bài; hàng nút **[+ thư viện] [tải xuống] [⋮] ... [shuffle] [play]**.
- `res/layout/fragment_album_detail.xml`.
- `AlbumDetailActivity.java` + `activity_album_detail.xml` — bản Activity (mở khi không ở trong Main, vd từ Profile).
- `viewmodel/AlbumDetailViewModel.java` — tải album + tracks + trạng thái "đã lưu thư viện".
- `data/repository/LibraryRepository.java` (phần Albums) — dùng chung với Người 4.
- `fragment/AlbumMenuBottomSheet.java` + `bottom_sheet_album_menu.xml` — menu ⋮ (Thêm/Xóa thư viện, Tải xuống→gate, Tới nghệ sĩ).
- `fragment/BaseDetailFragment.java` — **lớp cha** của các màn chi tiết Fragment (hoãn việc nặng tới sau khi animation trượt xong → mượt). Album/Artist/Playlist detail đều kế thừa.
- `adapter/TrackDetailAdapter.java` + `item_track_detail.xml` — danh sách bài trong album/playlist (dùng chung Người 4).

**Luồng:** `NavHelper.openAlbum` → `AlbumDetailFragment` → `AlbumDetailViewModel.loadIfNeeded()` gọi song song: `checkAlbumSaved` (`GET /api/albums/{id}/saved`), lấy album (`GET /api/albums`), lấy bài (`GET /api/albums/{id}/tracks`). Bấm 1 bài → `PlayerManager.play(track, danh sách, index)` + mở `PlayerActivity`. Nút **+** → lưu album (gọi sang LibraryService — xem Người 4).

### 2.5. CHI TIẾT NGHỆ SĨ

**File cần đọc (FE):**
- `fragment/ArtistDetailFragment.java` + `fragment_artist_detail.xml` (và `ArtistDetailActivity` + `activity_artist_detail.xml`).
- `viewmodel/ArtistDetailViewModel.java` — tải nghệ sĩ + bài phổ biến + album + trạng thái theo dõi.
- `adapter/ArtistTrackAdapter.java` (`item_artist_track.xml`), `adapter/ArtistAlbumAdapter.java` (`item_artist_album_vertical.xml`).

**Luồng:** `NavHelper.openArtist` → tải `GET /api/artists/{id}`, `/{id}/tracks/popular`, `/{id}/albums`, `/{id}/related`. Nút **Theo dõi** → `POST /api/artists/{id}/follow`.

### 2.6. MENU 3 CHẤM CỦA 1 BÀI HÁT

**File:** `fragment/TrackMenuBottomSheet.java` + `bottom_sheet_track_menu.xml` — menu khi bấm ⋮ ở 1 bài: Thích, Thêm vào playlist (mở sheet của Người 4), Tới album (NavHelper), Xóa khỏi playlist (nếu trong playlist).

### 2.7. STREAM FILE NHẠC (Backend — rất quan trọng)

**File cần đọc (BE):**
- `controller/FileController.java` — `GET /audio/{filename}` và `GET /images/{filename}`. **Xử lý header `Range`** → trả `206 Partial Content` để ExoPlayer tua được; không có Range → `200 OK`.
- `service/FileStorageService.java` — đọc file từ ổ đĩa (`D:/music-files/audio`, `/images`).
- `config/FileStorageConfig.java` — cấu hình thư mục lưu file.
- `controller/TrackController.java` — `GET /api/tracks`, `/{id}`, `/{id}/related`, `POST /{id}/like`, `GET /tracks/liked`.
- `service/TrackService.java` — logic bài hát.
- `controller/ArtistController.java` + `service/ArtistService.java` — nghệ sĩ, follow, popular, related.
- `controller/AlbumController.java` + `service/AlbumService.java` — album (lưu ý: phần "lưu thư viện" gọi `LibraryService` của Người 4).
- `controller/PlayHistoryController.java` + `service/PlayHistoryService.java` — `POST /api/history` (ghi lượt nghe), `GET /api/history/recent`.
- `controller/RecommendationController` + `service/RecommendationService` — nếu nhóm xếp vào Người 2 thì bỏ; ở đây nêu vì "related" liên quan Player.
- **Entity/Repo:** `Track`, `Album`, `Artist`, `PlayHistory`, `AlbumType` + `TrackRepository`, `AlbumRepository`, `ArtistRepository`, `PlayHistoryRepository`.

**Luồng stream:** ExoPlayer mở URL `http://10.0.2.2:8080/musicapp/audio/abc.mp3` kèm header `Range: bytes=...` → `FileController` đọc file qua `FileStorageService`, trả đúng đoạn byte (206). Nhờ vậy người dùng tua/seek được mà không tải cả file.

---

## 3. ✅ CHECKLIST "FILE CỦA TÔI" (Người 3)

**Android — màn hình & layout:**
- [ ] `PlayerActivity` + `activity_player.xml`
- [ ] `LyricsActivity` + `activity_lyrics.xml`
- [ ] `AlbumDetailFragment` + `fragment_album_detail.xml` ; `AlbumDetailActivity` + `activity_album_detail.xml`
- [ ] `ArtistDetailFragment` + `fragment_artist_detail.xml` ; `ArtistDetailActivity` + `activity_artist_detail.xml`
- [ ] `BaseDetailFragment` (lớp cha chi tiết)
- [ ] `TrackMenuBottomSheet` + `bottom_sheet_track_menu.xml`
- [ ] `AlbumMenuBottomSheet` + `bottom_sheet_album_menu.xml`

**Android — ViewModel / Repository / Util / Adapter:**
- [ ] `PlayerViewModel`, `AlbumDetailViewModel`, `ArtistDetailViewModel`
- [ ] `PlayerRepository` (+ dùng chung `LibraryRepository`)
- [ ] `util/PlayerManager`, `util/MiniPlayerController`, `util/LrcParser`, `util/TimeUtil`
- [ ] `TrackDetailAdapter`, `LyricLineAdapter`, `ArtistTrackAdapter`, `ArtistAlbumAdapter`, `ExploreArtistAdapter`
- [ ] models: `Track`, `Album`, `Artist`
- [ ] layout item: `item_track_detail.xml`, `item_lyric_line.xml`, `item_artist_track.xml`, `item_artist_album_vertical.xml`, `layout_mini_player.xml` (khung ở Main do Người 1 giữ, logic phát của bạn)

**Backend:**
- [ ] `controller/FileController`, `service/FileStorageService`, `config/FileStorageConfig`
- [ ] `controller/TrackController`, `service/TrackService`
- [ ] `controller/AlbumController`, `service/AlbumService`
- [ ] `controller/ArtistController`, `service/ArtistService`
- [ ] `controller/PlayHistoryController`, `service/PlayHistoryService`
- [ ] entity `Track`, `Album`, `Artist`, `PlayHistory`, `AlbumType`
- [ ] repository `TrackRepository`, `AlbumRepository`, `ArtistRepository`, `PlayHistoryRepository`
- [ ] dto `RecentItemDto`

---

## 4. ENDPOINT API MẢNG BẠN DÙNG
- `GET /audio/{filename}` (stream, Range), `GET /images/{filename}`
- `GET /api/tracks`, `/api/tracks/{id}`, `/api/tracks/{id}/related`, `POST /api/tracks/{id}/like`, `GET /api/tracks/liked`
- `GET /api/albums`, `/api/albums/{id}`, `/api/albums/{id}/tracks`
- `GET /api/artists/{id}`, `/api/artists/{id}/albums`, `/api/artists/{id}/tracks/popular`, `/api/artists/{id}/related`, `POST /api/artists/{id}/follow`
- `POST /api/history`, `GET /api/history/recent?limit=`

## 5. ĐIỂM GIAO VỚI NGƯỜI KHÁC
- **Mọi người** muốn phát nhạc đều gọi `PlayerManager.play(...)` của bạn → API này phải ổn định.
- Mini-player ở MainActivity (Người 1) hiển thị bài bạn đang phát (qua `MainViewModel`/`MiniPlayerController`).
- Nút **Like / Thêm thư viện / Theo dõi** trên màn của bạn ghi vào bảng `LikedTrack`/`SavedAlbum`/`FollowedArtist` (Người 4 sở hữu); danh sách "Đã thích / Thư viện / Đang theo dõi" do Người 4 hiển thị.
- Nút **Shuffle/Tải xuống** mở gate Premium (Người 5).
