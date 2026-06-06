# NGƯỜI 3 — Trình phát, Lời bài hát, Nghệ sĩ & Nhạc của tôi

> **Miền dữ liệu:** `Track`, `Artist`, `PlayHistory`, `LikedTrack`, `FollowedArtist`. **Vai trò:** phát nhạc (ExoPlayer), màn Player, lời bài hát LRC, **chi tiết Nghệ sĩ**, và 3 màn "của tôi": **Bài hát đã thích · Đang theo dõi · Nghe gần đây**. Backend gồm **stream nhạc HTTP Range**. Mảng trung tâm, kỹ thuật sâu nhất.

```
Bấm 1 bài → PlayerActivity → PlayerViewModel → PlayerManager (ExoPlayer) → GET /audio/{file} (Range)
NavHelper.openArtist → ArtistDetailFragment → GET /api/artists/{id}/...
Drawer/Library → LikedSongs · FollowingArtists · Recent
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `PlayerActivity.java` | Activity | Màn phát: bìa+Palette, seekbar, play/next/prev/shuffle/like/follow, teaser lời |
| `LyricsActivity.java` | Activity | Lời bài hát đầy đủ, cuộn |
| `ArtistDetailActivity.java` / `fragment/ArtistDetailFragment.java` | Activity/Fragment | Chi tiết nghệ sĩ: bài phổ biến + album + theo dõi |
| `LikedSongsActivity.java` / `fragment/LikedSongsFragment.java` / `fragment/LikedTracksFragment.java` | Activity/Fragment | "Bài hát đã thích" |
| `fragment/FollowingArtistsFragment.java` | Fragment | "Đang theo dõi" |
| `RecentActivity.java` / `fragment/RecentFragment.java` | Activity/Fragment | "Nghe gần đây" |
| `AddArtistActivity.java` | Activity | Tìm & theo dõi nghệ sĩ mới |
| `fragment/TrackMenuBottomSheet.java` | BottomSheet | Menu ⋮ 1 bài: thích / thêm playlist / tới album / xoá |
| `fragment/BaseDetailFragment.java` | Fragment (cha) | Lớp cha màn chi tiết (hoãn việc nặng cho mượt) |
| `PlayerViewModel.java` | ViewModel | `currentTrack/playing/progress/liked/following`; poll 500ms |
| `ArtistDetailViewModel.java` | ViewModel | Nghệ sĩ + bài + album + trạng thái follow |
| `LikedTracksViewModel`/`FollowingArtistsViewModel`/`RecentViewModel`/`AddArtistViewModel` | ViewModel | 4 màn danh sách |
| `PlayerRepository.java` | Repository | `toggleLike/toggleFollow/checkFollowState/recordPlay` |
| `util/PlayerManager.java` | Util ⭐ | **Singleton ExoPlayer**: hàng đợi, shuffle, next/prev, listener |
| `util/MiniPlayerController.java` | Util | Bơm mini-player vào Activity ngoài Main (poll 500ms) |
| `util/LrcParser.java` | Util | Tách LRC `[mm:ss.xx]` → dòng; `findActiveIndex` |
| `util/TimeUtil.java` | Util | Format mm:ss |
| `LyricLineAdapter` | Adapter | Từng dòng lời |
| `ArtistTrackAdapter`/`ArtistAlbumAdapter` | Adapter | Bài/album trong màn nghệ sĩ |
| `TrackListAdapter` | Adapter | Danh sách bài (đã thích) |
| `ArtistListAdapter` | Adapter | Danh sách nghệ sĩ (đang theo dõi) |
| `RecentSectionAdapter` | Adapter | Danh sách nghe gần đây |
| `SelectableArtistAdapter` | Adapter | Chọn nghệ sĩ để theo dõi |
| `ExploreArtistAdapter` | Adapter | Dải "Khám phá nghệ sĩ" dưới Player |
| `Track`/`Artist`/`RecentItem` | Model | DTO |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/TrackController.java` | Controller | `/api/tracks`, `/{id}`, `/{id}/related`, `/{id}/like`, `/tracks/liked` |
| `service/TrackService.java` | Service | Logic bài hát, like |
| `controller/ArtistController.java` | Controller | `/api/artists`, `/{id}`, `/{id}/albums`, `/{id}/tracks/popular`, `/{id}/related`, `/followed`, `/popular`, `/{id}/follow` |
| `service/ArtistService.java` | Service | Logic nghệ sĩ, follow |
| `controller/PlayHistoryController.java` | Controller | `POST /api/history`, `GET /history/recent` |
| `service/PlayHistoryService.java` | Service | Ghi & đọc lịch sử nghe |
| `controller/FileController.java` | Controller ⭐ | `GET /audio/{file}` (**Range 206**), `/images/{file}` |
| `service/FileStorageService.java` | Service | Đọc file nhạc/ảnh từ ổ đĩa |
| `config/FileStorageConfig.java` | Config | Thư mục lưu file |
| `entity/Track.java` | Entity | Bảng `Tracks` (có cột `lyrics`) |
| `entity/Artist.java` | Entity | Bảng `Artists` |
| `entity/PlayHistory.java` | Entity | Lịch sử nghe |
| `entity/LikedTrack.java` + `LikedTrackId.java` | Entity | Bài đã thích (khoá kép user+track) |
| `entity/FollowedArtist.java` + `FollowedArtistId.java` | Entity | Nghệ sĩ theo dõi (khoá kép) |
| `repository/TrackRepository, ArtistRepository, PlayHistoryRepository, LikedTrackRepository, FollowedArtistRepository` | Repository | Truy vấn 5 bảng trên |
| `dto/RecentItemDto.java` | DTO | 1 mục nghe gần đây |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable / anim |
|--------------|-----------------|
| `activity_player` | `ic_chevron_down`, `ic_more_vert`, `bg_hq_badge`, `ic_check`, `ic_plus_circle`, `ic_shuffle`, `ic_skip_previous/next`, `bg_play_btn`, `ic_play`(→`ic_pause` Java), `ic_timer/cast/share/queue`, `bg_lyrics_card/button`, `bg_following_outline`, `placeholder_gradient` · **anim:** `player_slide_down`, `player_no_anim` |
| `activity_lyrics` | `ic_chevron_down` |
| `fragment_artist_detail` / `activity_artist_detail` | `placeholder_gradient`, `bg_artist_header_gradient`, `ic_arrow_back`, `ic_shuffle`, `bg_play_btn`, `ic_play` |
| `fragment_liked_songs` / `activity_liked_songs` | `bg_liked_gradient`, `ic_favorite_filled`, `ic_arrow_back`, `ic_shuffle`, `ic_play` |
| `fragment_following_artists` / `fragment_recent` / `activity_recent` | `ic_add` / `ic_arrow_back` |
| `activity_add_artist` | `ic_close`, `bg_search_field`, `ic_search` |
| `bottom_sheet_track_menu` | `placeholder_gradient`, `ic_favorite`, `ic_no_ads`, `ic_plus_circle`, `ic_delete`, `ic_music_note`, `ic_person` |
| `item_artist_track` / `item_selectable_artist` | `ic_check_circle_green`, `ic_more_vert` / `bg_check_badge`, `ic_check` |

## 4. LUỒNG PHÁT NHẠC (cốt lõi)
1. Nơi khác `startActivity(PlayerActivity, "track")` → nếu chưa phát thì `PlayerManager.play(track, list, idx)`; gọi `vm.recordPlay()` (ghi history).
2. `PlayerViewModel` đăng ký `PlayerManager.setListener(this)` + poll 500ms cập nhật seekbar.
3. ExoPlayer mở `BASE_MEDIA_URL + audioUrl` → `FileController` trả **206 Partial** theo header `Range` → tua được.
4. Hết bài → `PlayerManager.playNext()` (ngẫu nhiên nếu shuffle) → `onTrackChanged` → UI vẽ lại.
5. Like → `POST /api/tracks/{id}/like`; Follow → `POST /api/artists/{id}/follow`; Shuffle Free → gate (Người 5).

## 5. ENDPOINT · GIAO
- **Endpoint:** `/audio/{file}`, `/images/{file}`, `/api/tracks/*`, `/api/artists/*`, `/api/history/*`.
- **Giao:** **mọi người** phát nhạc qua `PlayerManager.play()` của bạn · mini-player ở Main (Người 1) hiển thị bài bạn phát · Like/Follow ghi vào bảng của bạn nhưng nút Like cũng có ở Player; nút "Thêm vào playlist" mở sheet của Người 4 · màn Thư viện (Người 4) gọi `/api/tracks/liked` + `/api/artists/followed` của bạn cho 2 chip "Đã thích/Nghệ sĩ".
