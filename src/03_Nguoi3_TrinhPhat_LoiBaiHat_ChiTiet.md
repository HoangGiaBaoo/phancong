# NGƯỜI 3 — Trình phát, Lời bài hát, Nghệ sĩ & Nhạc của tôi

> **Miền dữ liệu:** `Track`, `Artist`, `PlayHistory`, `LikedTrack`, `FollowedArtist`. **Vai trò:** phát nhạc (ExoPlayer), màn Player, lời bài hát LRC, **chi tiết Nghệ sĩ**, và 3 màn "của tôi": **Bài hát đã thích · Đang theo dõi · Nghe gần đây**. Sở hữu **stream nhạc HTTP Range** + **ShuffleController**. Mảng trung tâm, kỹ thuật sâu nhất.

```
Bấm 1 bài → PlayerActivity → PlayerViewModel → PlayerManager (ExoPlayer) → GET /audio/{file} (Range)
NavHelper.openArtist → ArtistDetailFragment → GET /api/artists/{id}/...
Drawer/Library → LikedSongsFragment · FollowingArtistsFragment · RecentFragment
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `PlayerActivity.java` ⭐ | Activity | Màn phát: bìa+Palette, seekbar, play/next/prev/shuffle/like/follow, teaser lời, badge HQ |
| `LyricsActivity.java` | Activity | Lời bài hát đầy đủ, cuộn |
| `fragment/ArtistDetailFragment.java` | Fragment | Chi tiết nghệ sĩ: bài phổ biến + album + theo dõi (kế thừa `BaseDetailFragment`) |
| `fragment/LikedSongsFragment.java` | Fragment | "Bài hát đã thích" |
| `fragment/FollowingArtistsFragment.java` | Fragment | "Đang theo dõi" |
| `fragment/RecentFragment.java` | Fragment | "Nghe gần đây" |
| `AddArtistActivity.java` | Activity | Tìm & theo dõi nghệ sĩ mới |
| `fragment/TrackMenuBottomSheet.java` | BottomSheet | Menu ⋮ 1 bài: thích / thêm playlist / tới album / xoá |
| `PlayerViewModel.java` | ViewModel | `currentTrack/playing/progress/liked/following`; poll 500ms |
| `ArtistDetailViewModel.java` | ViewModel | Nghệ sĩ + bài + album + trạng thái follow |
| `LikedTracksViewModel`/`FollowingArtistsViewModel`/`RecentViewModel`/`AddArtistViewModel` | ViewModel | 4 màn danh sách |
| `PlayerRepository.java` | Repository | `toggleLike/toggleFollow/checkFollowState/recordPlay` |
| `util/PlayerManager.java` ⭐ | Util | **Singleton ExoPlayer**: hàng đợi, shuffle, next/prev, 1 listener UI + 1 adListener |
| `util/ShuffleController.java` ⭐ | Util **(mới)** | Gộp logic trộn bài + gate: `onShuffleClicked / setPremium / startIndex / Renderer` (dùng Player+Album+Artist+Playlist) |
| `util/LrcParser.java` | Util | Tách LRC `[mm:ss.xx]` → dòng; `findActiveIndex` |
| `util/TimeUtil.java` | Util | Format mm:ss |
| `LyricLineAdapter` | Adapter | Từng dòng lời |
| `ArtistTrackAdapter`/`ArtistAlbumAdapter` | Adapter | Bài/album trong màn nghệ sĩ |
| `TrackListAdapter` | Adapter | Danh sách bài (đã thích) |
| `ArtistListAdapter` | Adapter | Danh sách nghệ sĩ (đang theo dõi) |
| `RecentSectionAdapter` | Adapter | Danh sách nghe gần đây |
| `SelectableArtistAdapter` | Adapter | Chọn nghệ sĩ để theo dõi |
| `ExploreArtistAdapter` | Adapter | Dải "Khám phá nghệ sĩ" dưới Player (placeholder — chưa nối related) |
| `Track`/`Artist`/`RecentItem` | Model | DTO |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/TrackController.java` | Controller | `/api/tracks`, `/{id}`, `/{id}/related`, `/{id}/like`, `/tracks/liked` |
| `service/TrackService.java` | Service | Logic bài hát, like |
| `controller/ArtistController.java` | Controller | `/api/artists`, `/{id}`, `/{id}/albums`, `/{id}/tracks/popular`, `/{id}/related`, `/followed`, `/popular`, `/{id}/follow` |
| `service/ArtistService.java` | Service | Logic nghệ sĩ, follow |
| `controller/PlayHistoryController.java` / `service/PlayHistoryService.java` | Controller+Service | `POST /api/history`, `GET /history/recent`; ghi & đọc lịch sử |
| `controller/FileController.java` ⭐ | Controller | `GET /audio/{file}` (**Range 206**), `/images/{file}` |
| `service/FileStorageService.java` / `config/FileStorageConfig.java` | Service/Config | Đọc file nhạc/ảnh từ ổ đĩa; thư mục lưu |
| `entity/Track.java` | Entity | Bảng `Tracks` (có `lyrics`, `audioUrl`, `audioUrlHigh`) |
| `entity/Artist.java` | Entity | Bảng `Artists` |
| `entity/PlayHistory.java` | Entity | Lịch sử nghe |
| `entity/LikedTrack.java` + `LikedTrackId.java` | Entity | Bài đã thích (khoá kép) |
| `entity/FollowedArtist.java` + `FollowedArtistId.java` | Entity | Nghệ sĩ theo dõi (khoá kép) |
| `repository/TrackRepository, ArtistRepository, PlayHistoryRepository, LikedTrackRepository, FollowedArtistRepository` | Repository | Truy vấn 5 bảng |
| `dto/RecentItemDto.java` | DTO | 1 mục nghe gần đây |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable / anim |
|--------------|-----------------|
| `activity_player` | `ic_chevron_down`, `ic_more_vert`, `bg_hq_badge`, `ic_check`, `ic_plus_circle`, `ic_shuffle`, `ic_skip_previous/next`, `bg_play_btn`, `ic_play`(→`ic_pause`), `ic_timer/cast/share/queue`, `bg_lyrics_card/button`, `bg_following_outline`, `placeholder_gradient` · **anim:** `player_slide_up/down`, `player_no_anim` |
| `activity_lyrics` | `ic_chevron_down` |
| `fragment_artist_detail` | `placeholder_gradient`, `bg_artist_header_gradient`, `ic_arrow_back`, `ic_shuffle`, `bg_play_btn`, `ic_play` |
| `fragment_liked_songs` | `bg_liked_gradient`, `ic_favorite_filled`, `ic_arrow_back`, `ic_shuffle`, `ic_play` |
| `fragment_following_artists` / `fragment_recent` | `ic_add` / `ic_arrow_back`, `item_recent_header` |
| `activity_add_artist` | `ic_close`, `bg_search_field`, `ic_search` |
| `bottom_sheet_track_menu` | `placeholder_gradient`, `ic_favorite`, `ic_no_ads`, `ic_plus_circle`, `ic_delete`, `ic_music_note`, `ic_person` |
| `item_artist_track` / `item_selectable_artist` | `ic_check_circle_green`, `ic_more_vert` / `bg_check_badge`, `ic_check` |
| `item_explore_artist_card` | `bg_card_dark` |

## 4. ⭐ THỨ TỰ ĐỌC FILE

**Luồng phát nhạc (cốt lõi — đọc trước):**
1. `util/PlayerManager` ⭐ — **đọc đầu tiên**, mọi thứ phụ thuộc máy trạng thái ExoPlayer này (hàng đợi, shuffle, listener).
2. `PlayerActivity` → `PlayerViewModel` → `data/repository/PlayerRepository`; kèm `util/ShuffleController` (nút trộn + gate) + `util/TimeUtil`.
3. BE stream: `FileController` (trả 206 theo `Range`) → `FileStorageService`.

**Luồng lời bài hát:** 4. `LyricsActivity` → `util/LrcParser` → `LyricLineAdapter` (teaser lời nằm trong `PlayerActivity`).

**Luồng Nghệ sĩ:** 5. `NavHelper.openArtist` → `fragment/ArtistDetailFragment` (kế thừa `BaseDetailFragment`) → `ArtistDetailViewModel` → `ArtistController` → `ArtistService` → `entity/Artist`; xem `ArtistTrackAdapter`/`ArtistAlbumAdapter`.

**Luồng "Nhạc của tôi":** 6. `LikedSongsFragment` → `LikedTracksViewModel` → `/api/tracks/liked`; `FollowingArtistsFragment` → `/api/artists/followed`; `RecentFragment` → `/api/history/recent` → `PlayHistoryController`.

**Bổ trợ:** 7. `fragment/TrackMenuBottomSheet` (menu ⋮ 1 bài — mục "Thêm playlist" gọi sang Người 4).

## 5. ENDPOINT · GIAO
- **Endpoint:** `/audio/{file}`, `/images/{file}`, `/api/tracks/*`, `/api/artists/*`, `/api/history/*`.
- **Giao:** **mọi người** phát nhạc qua `PlayerManager.play()` của bạn · mini-player ở Main (Người 1) hiển thị bài bạn phát · `ShuffleController` của bạn gọi `ShuffleGateBottomSheet` (Người 5) khi Free · nút "Thêm vào playlist" trong `TrackMenuBottomSheet` mở sheet của Người 4 · màn Thư viện (Người 4) gọi `/api/tracks/liked` + `/api/artists/followed` của bạn · `PlayerManager.setAdListener` cho `AdManager` (Người 5) đếm bài.
