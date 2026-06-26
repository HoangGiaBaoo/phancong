# NGƯỜI 4 — Album, Danh sách phát & Thư viện

> **Miền dữ liệu:** `Album`, `AlbumType`, `SavedAlbum`, `Playlist`, `PlaylistTrack`. **Vai trò:** **Chi tiết Album** + lưu album vào thư viện, **toàn bộ CRUD Playlist** (tạo/thêm/xóa bài/đổi thứ tự/sửa tên/ảnh bìa/xóa), và **màn Thư viện** (hub gom playlist/nghệ sĩ/album đã lưu/đã thích). Thao tác nhiều nhất app. Sở hữu `LibraryRepository` (repo dùng chung) + `PlaylistCoverView` (ghép bìa 2x2).

```
NavHelper.openAlbum → AlbumDetailFragment → GET /api/albums/{id}/tracks (+ nút lưu thư viện)
Tab Thư viện → LibraryFragment (4 chip) → playlist / album đã lưu / nghệ sĩ / đã thích
Nút "Tạo" → CreateBottomSheet → playlist mới → PlaylistDetailFragment (thêm/sửa/đổi thứ tự/ảnh bìa/xóa)
```

> 📌 Album, Playlist, album đã lưu đều thuộc bạn (cùng nhóm "bộ sưu tập" + chung `LibraryRepository`). Trang chủ/tìm kiếm (Người 2) chỉ *hiển thị thẻ* rồi bấm sang màn chi tiết của bạn.

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `fragment/AlbumDetailFragment.java` ⭐ | Fragment | Chi tiết album + nút [+thư viện][tải][⋮][shuffle][play] (kế thừa `BaseDetailFragment`) |
| `fragment/AlbumMenuBottomSheet.java` | BottomSheet | Menu ⋮ album: thêm/xoá thư viện, tải→gate, tới nghệ sĩ |
| `AlbumDetailViewModel.java` | ViewModel | Tải album + bài + trạng thái "đã lưu" |
| `fragment/LibraryFragment.java` | Fragment | **Tab Thư viện**: 4 chip lọc, item ghim "Đã thích" |
| `LibraryViewModel.java` | ViewModel | `loadFor(chip)` lấy playlist/nghệ sĩ/album đã lưu/đã thích |
| `LibraryAdapter.java` | Adapter | Danh sách đa kiểu trong Thư viện |
| `ui/PlaylistCoverView.java` ⭐ | View | Ảnh bìa playlist: 1 ảnh hoặc **ghép 2x2** (Home/banner Người 2 cũng dùng) |
| `fragment/PlaylistsFragment.java` / `PlaylistsViewModel.java` | Fragment+VM | Danh sách playlist của tôi |
| `fragment/PlaylistDetailFragment.java` ⭐ | Fragment | Chi tiết playlist + [tải][⋮][shuffle][play] + gợi ý (kế thừa `BaseDetailFragment`, Palette) |
| `PlaylistDetailViewModel.java` | ViewModel | Tải + thêm/xoá/sửa/xoá playlist |
| `fragment/PlaylistMenuBottomSheet.java` | BottomSheet | Menu ⋮ playlist (tải/thêm/sửa/tên/ảnh bìa/xoá) |
| `fragment/CreateBottomSheet.java` | BottomSheet | Nút "Tạo" → tạo playlist (dùng layout `dialog_create_playlist`) |
| `fragment/AddTracksBottomSheet.java` + `AddTracksViewModel.java` | Sheet+VM | Thêm bài vào playlist (từ trong playlist) |
| `fragment/AddToPlaylistBottomSheet.java` + `AddToPlaylistViewModel.java` | Sheet+VM | Thêm 1 bài bất kỳ vào playlist (chọn đích) |
| `EditPlaylistActivity.java` + `EditPlaylistViewModel.java` | Activity+VM | Kéo-thả đổi thứ tự bài |
| `fragment/PlaylistEditBottomSheet.java` | BottomSheet | Sửa tên/mô tả/riêng tư |
| `PlaylistCoverPickerActivity.java` | Activity | Chọn ảnh máy → upload bìa |
| `TrackDetailAdapter` | Adapter | Danh sách bài trong album/playlist |
| `SuggestedTrackAdapter` | Adapter | Gợi ý thêm bài (playlist rỗng) |
| `PlaylistAdapter` | Adapter | Danh sách playlist |
| `AddTrackAdapter`/`PlaylistPickerAdapter`/`EditPlaylistTrackAdapter` | Adapter | Thêm bài / chọn playlist / kéo-thả sửa |
| `Album`/`Playlist`/`PlaylistRequest`/`PlaylistReorderRequest` | Model | DTO |
| `data/repository/LibraryRepository.java` ⭐ | Repository | **Repo to nhất** — album/playlist/genre/liked/followed (dùng chung) |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/AlbumController.java` | Controller | `/api/albums`, `/new`, `/{id}`, `/{id}/tracks`, `/saved`, `/{id}/saved`, `/{id}/save` |
| `service/AlbumService.java` | Service | Logic album |
| `service/LibraryService.java` | Service | Album đã lưu: `getSavedAlbums/toggleSaveAlbum/isAlbumSaved` |
| `controller/PlaylistController.java` | Controller | CRUD playlist + tracks + order + cover |
| `service/PlaylistService.java` | Service | Logic playlist (cập nhật `position` khi đổi thứ tự) |
| `entity/Album.java` / `AlbumType.java` | Entity/Enum | Bảng `Albums` / ALBUM·SINGLE·EP·COMPILATION |
| `entity/SavedAlbum.java` + `SavedAlbumId.java` | Entity | Album đã lưu (khoá kép user+album) |
| `entity/Playlist.java` | Entity | Bảng `Playlists` (có `coverUrl`,`description`,`isCurated`,`mood`,`coverColor`) |
| `entity/PlaylistTrack.java` + `PlaylistTrackId.java` | Entity | Bài trong playlist (có cột `position`) |
| `repository/AlbumRepository, SavedAlbumRepository, PlaylistRepository, PlaylistTrackRepository` | Repository | Truy vấn 4 bảng |
| `dto/PlaylistRequest, PlaylistResponse, PlaylistUpdateRequest, PlaylistReorderRequest` + dto Album | DTO | Body/kết quả album & playlist |

> ⚠️ Bảng nối có cột phụ (`position`, `savedAt`) là **Entity riêng** `@EmbeddedId` (khoá kép), KHÔNG dùng `@ManyToMany`.
> ⚠️ `PlaylistAdminController` (admin web — quản **curated playlist** + mood cho trang chủ) ghi vào entity của bạn nhưng phục vụ Người 2; xem mục admin ở `00`.

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_album_detail` | `placeholder_gradient`, `ic_arrow_back`, `ic_add_circle_outline`, `ic_download`, `ic_more_vert`, `ic_shuffle`, `ic_play` (Java đổi `ic_check_circle_green` khi đã lưu) |
| `bottom_sheet_album_menu` | `placeholder_gradient`, `ic_add_circle_outline`, `ic_download`, `ic_person` |
| `fragment_library` | `bg_avatar_orange`, `ic_search`, `ic_add`, `bg_chip_selector`×4, `ic_sort`, `ic_grid` |
| `fragment_playlist_detail` | `ic_arrow_back`, `ic_download`, `ic_more_vert`, `ic_shuffle`, `ic_play`, `ic_add`, `ic_drag_handle`, `ic_edit` |
| `bottom_sheet_playlist_menu` | `placeholder_gradient`, `ic_download`, `ic_plus_circle`, `ic_drag_handle`, `ic_edit`, `ic_camera`, `ic_delete` |
| `bottom_sheet_create` | `bg_circle_dark`, `ic_music_note`, `ic_collab`, `ic_blend`, `bg_close_circle`, `ic_close` |
| `dialog_create_playlist` | `bg_dialog_dark`, `ic_close` (ô nhập tên playlist) |
| `bottom_sheet_add_to_playlist` / `add_tracks` | `bg_search_field`, `ic_search` |
| `item_add_track` | `placeholder_gradient`, `ic_play`, `ic_add_circle_outline` |
| `item_edit_playlist_track` | `ic_remove_circle`, `placeholder_gradient`, `ic_drag_handle` |
| `sheet_playlist_edit` / `dialog_discard_changes` | `ic_edit`, `bg_edit_field`, `ic_lock_outline`, `ic_delete` / `bg_dialog_white` |
| `view_playlist_cover` | `placeholder_gradient` (×5 — ghép 2x2) |
| `item_library_*` (`row/artist/liked/action`) / `item_playlist_list/card` | `placeholder_gradient`, `ic_*` |

> Màn của bạn không dùng `anim` riêng (chuyển màn dùng anim chung Người 1).

## 4. ⭐ THỨ TỰ ĐỌC FILE

**Luồng Thư viện (đọc trước):**
1. `fragment/LibraryFragment` → `LibraryViewModel` → `data/repository/LibraryRepository` ⭐ (**đọc kỹ repo này** — nhiều người dùng chung).
2. `adapter/LibraryAdapter` + `ui/PlaylistCoverView` (ghép bìa 2x2).

**Luồng Album:**
3. `NavHelper.openAlbum` → `fragment/AlbumDetailFragment` (kế thừa `BaseDetailFragment`) → `AlbumDetailViewModel` → `LibraryRepository` (hàm album) → `AlbumController` → `AlbumService` → `entity/Album`; nút lưu → `LibraryService` + `SavedAlbum`; menu ⋮ → `AlbumMenuBottomSheet`.

**Luồng Playlist của tôi → chi tiết:**
4. `fragment/PlaylistsFragment` → `PlaylistsViewModel`; `adapter/PlaylistAdapter`.
5. `fragment/PlaylistDetailFragment` (kế thừa `BaseDetailFragment`) → `PlaylistDetailViewModel` → `PlaylistController` → `PlaylistService` → `entity/Playlist` + `PlaylistTrack`.

**Luồng Tạo/Sửa (thao tác phức tạp):**
6. Nút "Tạo" (ở `MainActivity` của Người 1) → `fragment/CreateBottomSheet` (+ layout `dialog_create_playlist`).
7. Thêm bài: `AddTracksBottomSheet` (trong playlist) **hoặc** `AddToPlaylistBottomSheet` (từ menu ⋮ 1 bài của Người 3).
8. Đổi thứ tự: `EditPlaylistActivity` + `EditPlaylistTrackAdapter` + `ItemTouchHelper` → `LibraryRepository.reorderPlaylistTracks` → `PUT /api/playlists/{id}/tracks/order`.
9. Bìa: `PlaylistCoverPickerActivity` → `POST /api/playlists/{id}/cover`; Sửa tên: `PlaylistEditBottomSheet`.

## 4c. ⭐ MA TRẬN THÊM / SỬA / XÓA (Thư viện, Album & Playlist)

| Hành vi | Endpoint | Bấm từ đâu | Chủ |
|---------|----------|-----------|-----|
| Lưu/bỏ **album** khỏi Thư viện | `POST /api/albums/{id}/save` (toggle) | Album detail (+/✓) + menu ⋮ album | **4** |
| Tạo **playlist** | `POST /api/playlists` | Nút "Tạo" (`CreateBottomSheet`) | **4** |
| Sửa tên/mô tả playlist | `PUT /api/playlists/{id}` | `PlaylistEditBottomSheet` | **4** |
| Đổi ảnh bìa playlist | `POST /api/playlists/{id}/cover` | `PlaylistCoverPickerActivity` | **4** |
| Xóa playlist | `DELETE /api/playlists/{id}` | menu ⋮ playlist → xác nhận | **4** |
| **Thêm bài** vào playlist | `POST /api/playlists/{id}/tracks` | `AddTracksBottomSheet` **hoặc** menu ⋮ 1 bài → `AddToPlaylistBottomSheet` | **4** |
| **Xóa bài** khỏi playlist | `DELETE /api/playlists/{id}/tracks/{trackId}` | `EditPlaylistActivity` **hoặc** menu ⋮ bài trong playlist | **4** |
| Đổi thứ tự bài | `PUT /api/playlists/{id}/tracks/order` | `EditPlaylistActivity` (kéo-thả) | **4** |
| Thích bài (→ "Đã thích") | `POST /api/tracks/{id}/like` | Player + menu ⋮ bài | **3** |
| Theo dõi nghệ sĩ (→ Thư viện) | `POST /api/artists/{id}/follow` | Artist detail + Player | **3** |

> ⚠️ menu ⋮ 1 bài (`TrackMenuBottomSheet`) là của **Người 3**, nhưng mục "Thêm vào playlist" gọi `AddToPlaylistBottomSheet` + `addTrackToPlaylist` của **bạn**. Khi 1 trong 2 sửa, báo người kia.
> 📌 **Album không có "sửa"** cho user thường (album là nội dung catalog; chỉ admin sửa qua AdminController — ngoài phạm vi).

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/albums/*` (gồm saved), `/api/playlists/*` (CRUD + order + cover).
- **Giao:** thẻ album/playlist trong feed của **Người 2** bấm vào → mở màn chi tiết của bạn · Library hub gọi `/api/tracks/liked` + `/api/artists/followed` (Người 3) cho 2 chip "Đã thích/Nghệ sĩ" · bấm bài → `PlayerManager.play()` (Người 3) · nút "Tải xuống" → gate Premium (Người 5) · `LibraryRepository` (của bạn) được Người 2 (genre feed) & Người 3 dùng chung · `ui/PlaylistCoverView` (của bạn) dùng ở Home/banner (Người 2) · nút "Tạo" nằm ở `MainActivity` (Người 1).
