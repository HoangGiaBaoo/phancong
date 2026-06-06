# NGƯỜI 4 — Thư viện, Danh sách phát & Yêu thích/Theo dõi

> **Bạn là "kho cá nhân" của người dùng.** Tab **Thư viện** gom: playlist của tôi, nghệ sĩ đang theo dõi, album đã lưu, bài hát đã thích, nghe gần đây. Bạn lo **toàn bộ CRUD playlist** (tạo, thêm/xóa bài, đổi thứ tự, sửa tên, đổi ảnh bìa, xóa) — phần thao tác nhiều nhất app. Bạn sở hữu `LibraryRepository` (repository to nhất) và 5 bảng quan hệ ở backend.

---

## 0. NỀN TẢNG DÙNG CHUNG (đọc nhanh — chi tiết ở file 01)
Khuôn MVVM như cả nhóm. Bạn sở hữu **`LibraryRepository`** — repository dùng chung nhiều nhất (Album/Artist/Track/Playlist/Genre đều có hàm ở đây). `VmFactory` tạo phần lớn ViewModel của bạn bằng `new XxxViewModel(new LibraryRepository(api))`.

---

## 1. SƠ ĐỒ LUỒNG MẢNG CỦA BẠN

```
Tab Thư viện ─► LibraryFragment ─► LibraryViewModel ─► LibraryRepository
     │ chip lọc: Playlist | Nghệ sĩ | Album | Đã thích
     │   ├─ Playlist  → GET /api/playlists
     │   ├─ Nghệ sĩ   → GET /api/artists/followed
     │   ├─ Album     → GET /api/albums/saved
     │   └─ Đã thích  → GET /api/tracks/liked
     └─ bấm item ─► NavHelper.openPlaylist/openArtist/openAlbum/openLiked

Tab "Tạo" (nút giữa bottom-nav) ─► CreateBottomSheet ─► tạo playlist mới

PlaylistDetail (1 playlist) ─► thêm bài / sửa thứ tự / đổi tên / đổi ảnh bìa / xóa
     ├─ AddTracksBottomSheet      → POST /api/playlists/{id}/tracks
     ├─ EditPlaylistActivity      → PUT  /api/playlists/{id}/tracks/order (kéo-thả đổi thứ tự)
     ├─ PlaylistEditBottomSheet   → PUT  /api/playlists/{id} (tên/mô tả)
     └─ PlaylistCoverPickerActivity → POST /api/playlists/{id}/cover (upload ảnh)
```

---

## 2. CHI TIẾT TỪNG CHỨC NĂNG

### 2.1. TAB THƯ VIỆN (màn gom tất cả)

**File cần đọc (FE):**
- `fragment/LibraryFragment.java` — **đọc kỹ**. Thanh chip lọc (Playlist/Nghệ sĩ/Album/Đã thích), 1 RecyclerView đa kiểu, item ghim "Bài hát đã thích" luôn ở đầu, các nút "Thêm" cuối danh sách. `onResume` gọi `vm.refresh()`.
- `res/layout/fragment_library.xml` + `item_library_row.xml`, `item_library_artist.xml`, `item_library_liked.xml`, `item_library_action.xml`.
- `viewmodel/LibraryViewModel.java` — **đọc kỹ**. Enum `Chip{ALL,PLAYLIST,ARTIST,ALBUM,LIKED}`. Mỗi chip `loadFor(chip)` gọi nguồn tương ứng. Giữ `playlists`, `artists`, `savedAlbums`, `liked`, `likedCount`.
- `adapter/LibraryAdapter.java` — adapter đa kiểu (playlist/artist/album/track/action/liked-songs). Có cache ảnh bìa playlist (ghép 4 ảnh bài).
- `adapter/LibraryPagerAdapter.java` — nếu Thư viện có dạng tab/pager.
- `ui/PlaylistCoverView.java` + `view_playlist_cover.xml` — view ảnh bìa playlist (1 ảnh hoặc ghép 2x2 từ ảnh các bài).

**Luồng:** mở tab → `LibraryViewModel` (constructor) `loadFor(ALL)` + `loadLikedCount()` → đổ vào `LibraryAdapter`. Bấm chip → `onChipSelected` → `loadFor(chip)`. Bấm item → `NavHelper.openXxx`.

> 💡 **Mẹo nhớ:** Library = "1 danh sách, lọc theo 4 chip, mỗi chip lấy từ 1 endpoint khác nhau". Toàn bộ nguồn dữ liệu nằm trong `LibraryViewModel.loadFor()`.

### 2.2. BÀI HÁT ĐÃ THÍCH / NGHỆ SĨ ĐANG THEO DÕI / NGHE GẦN ĐÂY

**File cần đọc (FE):**
- `fragment/LikedTracksFragment.java` + `fragment_liked_tracks.xml` ; `fragment/LikedSongsFragment.java` + `fragment_liked_songs.xml` ; `LikedSongsActivity.java` + `activity_liked_songs.xml` — màn "Bài hát đã thích".
- `viewmodel/LikedTracksViewModel.java` — `GET /api/tracks/liked`.
- `fragment/FollowingArtistsFragment.java` + `fragment_following_artists.xml` ; `viewmodel/FollowingArtistsViewModel.java` — `GET /api/artists/followed`.
- `fragment/RecentFragment.java` + `fragment_recent.xml` ; `RecentActivity.java` + `activity_recent.xml` ; `viewmodel/RecentViewModel.java` — `GET /api/history/recent`.
- `fragment/PlaylistsFragment.java` + `fragment_playlists.xml` ; `viewmodel/PlaylistsViewModel.java` — danh sách playlist của tôi.
- **Adapter:** `TrackListAdapter` (`item_track_list.xml`), `ArtistListAdapter` (`item_artist_list.xml`), `PlaylistAdapter` (`item_playlist_list.xml`), `RecentSectionAdapter` (`item_recent_header.xml`).
- `model/RecentItem.java`.

**Luồng:** đều giống nhau — Fragment → ViewModel → `LibraryRepository.getLikedTracks()/getFollowedArtists()/getRecentTracks()` → render list. Bấm bài → phát nhạc (PlayerManager — Người 3).

### 2.3. TẠO PLAYLIST MỚI

**File:** `fragment/CreateBottomSheet.java` + `bottom_sheet_create.xml` — sheet bật lên từ nút "Tạo" giữa bottom-nav: có "Danh sách phát". Dùng `dialog_create_playlist.xml` để nhập tên.
**BE:** `POST /api/playlists` (body `{name, isPublic}`).
**Luồng:** nút Tạo (ở MainActivity của Người 1) → `CreateBottomSheet` → nhập tên → `LibraryRepository.createPlaylist(name, isPublic)` → `POST /api/playlists` → mở luôn `PlaylistDetail` của playlist vừa tạo.

### 2.4. MÀN CHI TIẾT PLAYLIST (trung tâm thao tác)

**File cần đọc (FE):**
- `fragment/PlaylistDetailFragment.java` — **đọc kỹ**. Hiện bìa (ghép 2x2), danh sách bài, nút **[tải xuống] [⋮] ... [shuffle] [play]**, gợi ý thêm bài khi rỗng.
- `res/layout/fragment_playlist_detail.xml` ; `PlaylistDetailActivity.java` + `activity_playlist_detail.xml`.
- `viewmodel/PlaylistDetailViewModel.java` — tải playlist + bài + gợi ý; xử lý thêm/xóa/sửa/xóa playlist (enum `EditResult`).
- `fragment/PlaylistMenuBottomSheet.java` + `bottom_sheet_playlist_menu.xml` — menu ⋮ (Tải xuống→gate, Thêm bài, Chỉnh sửa, Tên & chi tiết, Tạo ảnh bìa, Xóa playlist).
- `adapter/TrackDetailAdapter.java` (dùng chung Người 3), `adapter/SuggestedTrackAdapter.java` + `item_suggested_track.xml` (gợi ý thêm bài).

**Luồng:** `NavHelper.openPlaylist` → `PlaylistDetailFragment` → `PlaylistDetailViewModel.loadIfNeeded()`: `GET /api/playlists/{id}` + `/{id}/tracks`. Rỗng → gọi gợi ý. Bấm bài → phát. Các nút mở các sheet/màn con bên dưới.

### 2.5. THÊM BÀI VÀO PLAYLIST (2 lối)

**File:**
- `fragment/AddTracksBottomSheet.java` + `bottom_sheet_add_tracks.xml` + `viewmodel/AddTracksViewModel.java` + `item_add_track.xml` — từ trong 1 playlist, mở sheet chọn bài để thêm.
- `fragment/AddToPlaylistBottomSheet.java` + `bottom_sheet_add_to_playlist.xml` + `viewmodel/AddToPlaylistViewModel.java` + `item_playlist_picker.xml`, `item_playlist_picker_section.xml` — từ 1 bài bất kỳ (menu ⋮), chọn playlist đích (hoặc tạo mới).
**BE:** `POST /api/playlists/{id}/tracks?trackId=`.
**Luồng:** chọn bài/playlist → `LibraryRepository.addTrackToPlaylist(playlistId, trackId)` → reload playlist.

### 2.6. CHỈNH SỬA PLAYLIST (đổi thứ tự kéo-thả)

**File:** `EditPlaylistActivity.java` + `activity_edit_playlist.xml` + `viewmodel/EditPlaylistViewModel.java` + `adapter` kéo-thả (`item_edit_playlist_track.xml`), `dialog_discard_changes.xml`.
**BE:** `PUT /api/playlists/{id}/tracks/order` (body `{trackIds:[...]}`), `DELETE /api/playlists/{id}/tracks/{trackId}`.
**Luồng:** kéo-thả đổi vị trí (ItemTouchHelper) → lưu danh sách `trackIds` theo thứ tự mới → `reorderPlaylistTracks(...)`.

### 2.7. SỬA TÊN/MÔ TẢ & ĐỔI ẢNH BÌA

**File:**
- `fragment/PlaylistEditBottomSheet.java` + `sheet_playlist_edit.xml` — sửa tên + mô tả + quyền riêng tư. BE: `PUT /api/playlists/{id}`.
- `PlaylistCoverPickerActivity.java` + `activity_playlist_cover_picker.xml` — chọn ảnh từ máy upload. BE: `POST /api/playlists/{id}/cover` (multipart).
**Luồng upload bìa:** chọn ảnh → đọc bytes → `LibraryRepository.uploadPlaylistCover(id, bytes, mime)` → BE lưu file, trả `coverUrl` → reload.

### 2.8. XÓA PLAYLIST
Menu ⋮ → "Xóa" → `AlertDialog` xác nhận → `LibraryRepository.deletePlaylist(id)` → `DELETE /api/playlists/{id}` → `popBackStack`.

### 2.9. THÊM NGHỆ SĨ ĐỂ THEO DÕI
**File:** `AddArtistActivity.java` + `activity_add_artist.xml` + `viewmodel/AddArtistViewModel.java` + `adapter/SelectableArtistAdapter.java` + `item_selectable_artist.xml`. BE: `POST /api/artists/{id}/follow`.

### 2.10. BACKEND của bạn (5 bảng quan hệ)

**File cần đọc (BE):**
- `controller/PlaylistController.java` + `service/PlaylistService.java` — toàn bộ CRUD playlist + reorder + cover. **Đọc kỹ.**
- `service/LibraryService.java` — album đã lưu (`getSavedAlbums`, `toggleSaveAlbum`, `isAlbumSaved`) + tổng hợp thư viện.
- **Entity + Repository (bảng nối N-N có cột phụ → entity riêng):**
  - `Playlist` + `PlaylistRepository`
  - `PlaylistTrack` + `PlaylistTrackId` + `PlaylistTrackRepository` (có cột `position` để sắp thứ tự).
  - `LikedTrack` + `LikedTrackId` + `LikedTrackRepository` (bài đã thích).
  - `FollowedArtist` + `FollowedArtistId` + `FollowedArtistRepository` (nghệ sĩ theo dõi).
  - `SavedAlbum` + `SavedAlbumId` + `SavedAlbumRepository` (album lưu thư viện).
- **DTO:** `PlaylistRequest`, `PlaylistResponse`, `PlaylistUpdateRequest`, `PlaylistReorderRequest`.

> ⚠️ **Lưu ý kỹ thuật quan trọng:** các bảng nối có cột phụ (như `position`, `savedAt`) phải là **Entity riêng** với `@EmbeddedId` (khóa kép userId+itemId), KHÔNG dùng `@ManyToMany`. Đây là lý do mỗi quan hệ có 2 file (`Xxx` + `XxxId`).

---

## 3. ✅ CHECKLIST "FILE CỦA TÔI" (Người 4)

**Android — màn hình & sheet & layout:**
- [ ] `LibraryFragment` + `fragment_library.xml` (+ item_library_*.xml)
- [ ] `PlaylistsFragment` + `fragment_playlists.xml`
- [ ] `FollowingArtistsFragment` + `fragment_following_artists.xml`
- [ ] `LikedTracksFragment`/`LikedSongsFragment`/`LikedSongsActivity` + layout
- [ ] `RecentFragment`/`RecentActivity` + layout
- [ ] `PlaylistDetailFragment`/`PlaylistDetailActivity` + layout
- [ ] `CreateBottomSheet`, `AddTracksBottomSheet`, `AddToPlaylistBottomSheet`, `PlaylistEditBottomSheet`, `PlaylistMenuBottomSheet` + layout
- [ ] `EditPlaylistActivity`, `PlaylistCoverPickerActivity`, `AddArtistActivity` + layout

**Android — ViewModel / Repository / Adapter / UI:**
- [ ] `LibraryViewModel`, `PlaylistsViewModel`, `FollowingArtistsViewModel`, `LikedTracksViewModel`, `RecentViewModel`, `PlaylistDetailViewModel`, `AddToPlaylistViewModel`, `AddTracksViewModel`, `EditPlaylistViewModel`, `AddArtistViewModel`
- [ ] `LibraryRepository` (repository to nhất — bạn sở hữu)
- [ ] `LibraryAdapter`, `LibraryPagerAdapter`, `PlaylistAdapter`, `TrackListAdapter`, `ArtistListAdapter`, `SelectableArtistAdapter`, `RecentSectionAdapter`, `SuggestedTrackAdapter`
- [ ] `ui/PlaylistCoverView` + `view_playlist_cover.xml`
- [ ] models: `Playlist`, `RecentItem`, `PlaylistRequest`, `PlaylistReorderRequest`

**Backend:**
- [ ] `controller/PlaylistController`, `service/PlaylistService`, `service/LibraryService`
- [ ] `entity/Playlist` + `PlaylistRepository`
- [ ] `entity/PlaylistTrack` + `PlaylistTrackId` + `PlaylistTrackRepository`
- [ ] `entity/LikedTrack` + `LikedTrackId` + `LikedTrackRepository`
- [ ] `entity/FollowedArtist` + `FollowedArtistId` + `FollowedArtistRepository`
- [ ] `entity/SavedAlbum` + `SavedAlbumId` + `SavedAlbumRepository`
- [ ] `dto/PlaylistRequest`, `PlaylistResponse`, `PlaylistUpdateRequest`, `PlaylistReorderRequest`

---

## 4. ENDPOINT API MẢNG BẠN DÙNG
- `GET /api/playlists`, `GET /api/playlists/curated?mood=`, `POST /api/playlists`, `GET /api/playlists/{id}`, `GET /api/playlists/{id}/tracks`
- `POST /api/playlists/{id}/tracks?trackId=`, `DELETE /api/playlists/{id}/tracks/{trackId}`, `PUT /api/playlists/{id}/tracks/order`
- `PUT /api/playlists/{id}`, `DELETE /api/playlists/{id}`, `POST /api/playlists/{id}/cover`
- `GET /api/tracks/liked`, `GET /api/artists/followed`, `GET /api/albums/saved`, `GET /api/history/recent?limit=`

## 5. ĐIỂM GIAO VỚI NGƯỜI KHÁC
- `LibraryRepository` (của bạn) được Người 2 (genre feed) và Người 3 (album/artist detail) dùng chung — sửa hàm phải báo.
- Nút Like/Follow/Save nằm ở màn của Người 3 nhưng **ghi vào bảng của bạn** (LikedTrack/FollowedArtist/SavedAlbum). Bạn lo phần đọc lại (hiển thị trong Thư viện).
- Nút "Tải xuống" trong playlist mở gate Premium (Người 5).
- Mở Create từ nút "Tạo" của MainActivity (Người 1).
