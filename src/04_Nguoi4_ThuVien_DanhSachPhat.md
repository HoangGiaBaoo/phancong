# NGƯỜI 4 — Album, Danh sách phát & Thư viện

> **Miền dữ liệu:** `Album`, `Playlist`, `PlaylistTrack`, `SavedAlbum`. **Vai trò:** **Chi tiết Album** + lưu album vào thư viện, **toàn bộ CRUD Playlist** (tạo/thêm/xóa bài/đổi thứ tự/sửa tên/ảnh bìa/xóa), và **màn Thư viện** (hub gom playlist/nghệ sĩ/album/đã thích). Thao tác nhiều nhất app.

```
NavHelper.openAlbum → AlbumDetailFragment → GET /api/albums/{id}/tracks (+ nút lưu thư viện)
Tab Thư viện → LibraryFragment (4 chip) → playlist / album đã lưu / nghệ sĩ / đã thích
Nút "Tạo" → CreateBottomSheet → playlist mới → PlaylistDetail (thêm/sửa/đổi thứ tự/ảnh bìa/xóa)
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `fragment/AlbumDetailFragment.java` / `AlbumDetailActivity.java` | Fragment/Activity | Chi tiết album + nút [+thư viện][tải][⋮][shuffle][play] |
| `fragment/AlbumMenuBottomSheet.java` | BottomSheet | Menu ⋮ album: thêm/xoá thư viện, tải→gate, tới nghệ sĩ |
| `AlbumDetailViewModel.java` | ViewModel | Tải album + bài + trạng thái "đã lưu" |
| `fragment/LibraryFragment.java` | Fragment | **Tab Thư viện**: 4 chip lọc, item ghim "Đã thích" |
| `LibraryViewModel.java` | ViewModel | `loadFor(chip)` lấy playlist/nghệ sĩ/album/đã thích |
| `LibraryAdapter.java` / `LibraryPagerAdapter.java` | Adapter | Danh sách đa kiểu trong Thư viện |
| `ui/PlaylistCoverView.java` | View | Ảnh bìa playlist (1 ảnh hoặc ghép 2x2) |
| `fragment/PlaylistsFragment.java` | Fragment | Danh sách playlist của tôi |
| `PlaylistsViewModel.java` | ViewModel | Tải playlist |
| `fragment/PlaylistDetailFragment.java` / `PlaylistDetailActivity.java` | Fragment/Activity | Chi tiết playlist + nút [tải][⋮][shuffle][play] + gợi ý |
| `PlaylistDetailViewModel.java` | ViewModel | Tải + thêm/xoá/sửa/xoá playlist |
| `fragment/PlaylistMenuBottomSheet.java` | BottomSheet | Menu ⋮ playlist (tải/thêm/sửa/tên/ảnh bìa/xoá) |
| `fragment/CreateBottomSheet.java` | BottomSheet | Nút "Tạo" → tạo playlist |
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
| `data/repository/LibraryRepository.java` | Repository ⭐ | **Repo to nhất** — album/playlist/genre/liked/followed (dùng chung) |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/AlbumController.java` | Controller | `/api/albums`, `/new`, `/{id}`, `/{id}/tracks`, `/saved`, `/{id}/saved`, `/{id}/save` |
| `service/AlbumService.java` | Service | Logic album |
| `controller/PlaylistController.java` | Controller | CRUD playlist + tracks + order + cover |
| `service/PlaylistService.java` | Service | Logic playlist |
| `service/LibraryService.java` | Service | Album đã lưu: `getSavedAlbums/toggleSaveAlbum/isAlbumSaved` |
| `entity/Album.java` | Entity | Bảng `Albums` |
| `entity/AlbumType.java` | Enum | ALBUM/SINGLE/EP/COMPILATION |
| `entity/Playlist.java` | Entity | Bảng `Playlists` (có `coverUrl`,`description`,`isCurated`,`mood`) |
| `entity/PlaylistTrack.java` + `PlaylistTrackId.java` | Entity | Bài trong playlist (có cột `position`) |
| `entity/SavedAlbum.java` + `SavedAlbumId.java` | Entity | Album đã lưu (khoá kép user+album) |
| `repository/AlbumRepository, PlaylistRepository, PlaylistTrackRepository, SavedAlbumRepository` | Repository | Truy vấn 4 bảng |
| `dto/PlaylistRequest, PlaylistResponse, PlaylistUpdateRequest, PlaylistReorderRequest` | DTO | Body/kết quả playlist |

> ⚠️ Bảng nối có cột phụ (`position`, `savedAt`) là **Entity riêng** với `@EmbeddedId` (khoá kép), KHÔNG dùng `@ManyToMany` → vì vậy mỗi quan hệ có 2 file (`Xxx`+`XxxId`).

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_album_detail` | `placeholder_gradient`, `ic_arrow_back`, `ic_add_circle_outline`, `ic_download`, `ic_more_vert`, `ic_shuffle`, `ic_play` (Java đổi `ic_check_circle_green` khi đã lưu) |
| `bottom_sheet_album_menu` | `placeholder_gradient`, `ic_add_circle_outline`, `ic_download`, `ic_person` |
| `fragment_library` | `bg_avatar_orange`, `ic_search`, `ic_add`, `bg_chip_selector`×4, `ic_sort`, `ic_grid` |
| `fragment_playlist_detail` | `ic_arrow_back`, `ic_download`, `ic_more_vert`, `ic_shuffle`, `ic_play`, `ic_add`, `ic_drag_handle`, `ic_edit` |
| `bottom_sheet_playlist_menu` | `placeholder_gradient`, `ic_download`, `ic_plus_circle`, `ic_drag_handle`, `ic_edit`, `ic_camera`, `ic_delete` |
| `bottom_sheet_create` | `bg_circle_dark`, `ic_music_note`, `ic_collab`, `ic_blend`, `bg_close_circle`, `ic_close` |
| `bottom_sheet_add_to_playlist` / `add_tracks` | `bg_search_field`, `ic_search` |
| `item_add_track` | `placeholder_gradient`, `ic_play`, `ic_add_circle_outline` |
| `item_edit_playlist_track` | `ic_remove_circle`, `placeholder_gradient`, `ic_drag_handle` |
| `sheet_playlist_edit` | `ic_edit`, `bg_edit_field`, `ic_lock_outline`, `ic_delete` |
| `view_playlist_cover` | `placeholder_gradient` (×5 — ghép 2x2) |
| `dialog_create_playlist` / `dialog_discard_changes` | `bg_dialog_dark`,`ic_close` / `bg_dialog_white` |

> Màn của bạn không dùng `anim` riêng (chuyển màn dùng anim chung của Người 1).

## 4. LUỒNG ĐỔI THỨ TỰ BÀI (ví dụ thao tác phức tạp)
1. `EditPlaylistActivity` → `EditPlaylistTrackAdapter` + `ItemTouchHelper` kéo-thả.
2. Lưu danh sách `trackIds` theo thứ tự mới → `LibraryRepository.reorderPlaylistTracks` → `PUT /api/playlists/{id}/tracks/order`.
3. BE `PlaylistService` cập nhật cột `position` trong `PlaylistTrack`.

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/albums/*` (gồm saved), `/api/playlists/*` (CRUD + order + cover).
- **Giao:** Library hub gọi `/api/tracks/liked` + `/api/artists/followed` (endpoint của **Người 3**) cho 2 chip "Đã thích/Nghệ sĩ" · bấm bài → `PlayerManager.play()` (Người 3) · nút "Tải xuống" → gate Premium (Người 5) · `LibraryRepository` (của bạn) được Người 2 (genre feed) & Người 3 (chi tiết) dùng chung · nút "Tạo" nằm ở `MainActivity` (Người 1).
