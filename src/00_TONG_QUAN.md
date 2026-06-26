# TỔNG QUAN PHÂN CÔNG — Đồ án Spotify clone (5 người)

> Mục lục + bản đồ chung. Mỗi người đọc file riêng (01–05). Thống kê số file ở **`06_THONG_KE...md`**.
> Phân chia theo **chức năng**: mỗi người ôm 1 miền dữ liệu + toàn bộ màn hình/API liên quan (Android → Spring Boot). Cập nhật 2026-06-26 cho khớp code hiện tại.

---

## 1. Cách chia: LÁT CẮT DỌC theo "miền dữ liệu"

Mỗi người ôm trọn **1 nhóm entity** + tất cả màn hình/API liên quan, từ Android xuống Spring Boot. Nhờ vậy **mỗi bảng MySQL chỉ có đúng 1 chủ** → không giẫm chân nhau.

| Người | Miền dữ liệu (entity sở hữu) | Mảng tính năng |
|------|------------------------------|----------------|
| **1** | `User` + bảo mật | Đăng nhập, Đăng ký, Đa tài khoản, **Khung app (MainActivity)**, **Hồ sơ/Sửa hồ sơ** + **nền tảng mạng dùng chung** |
| **2** | `Genre`, `Mood` | Trang chủ (feed), Tìm kiếm, Thể loại, Bảng xếp hạng, Gợi ý |
| **3** | `Track`, `Artist`, `PlayHistory`, `LikedTrack`, `FollowedArtist` | **Trình phát**, Lời bài hát, **Chi tiết Nghệ sĩ**, Bài đã thích, Đang theo dõi, Nghe gần đây, **ShuffleController** |
| **4** | **`Album`**, `AlbumType`, `SavedAlbum`, `Playlist`, `PlaylistTrack` | **Chi tiết Album + lưu album**, **Danh sách phát (CRUD đầy đủ)**, màn **Thư viện** (hub) |
| **5** | `Subscription`, `UserSettings` | Premium, Thanh toán VNPay, **Quảng cáo AdMob**, Gate, Cài đặt, Thống kê |

> 📌 **Album, Playlist, album đã lưu** đều thuộc **Người 4** (cùng nhóm "bộ sưu tập" + chung `LibraryRepository`). Trang chủ/tìm kiếm (Người 2) chỉ *hiển thị thẻ* rồi bấm sang màn chi tiết của Người 4 — nhờ vậy Người 2 là mảng **nhẹ & dễ nhất**.

### 🧠 Câu thần chú để nhớ phần của mình (1 dòng/người)
- **Người 1** — *"Tôi lo cho user VÀO được app + hồ sơ."* (đăng nhập + khung app + hồ sơ)
- **Người 2** — *"Tôi lo cho user TÌM thấy nhạc."* (trang chủ + tìm kiếm + thể loại)
- **Người 3** — *"Tôi lo BÀI HÁT & NGHỆ SĨ: nghe, thích, theo dõi."*
- **Người 4** — *"Tôi lo ALBUM, DANH SÁCH PHÁT & THƯ VIỆN: mở, tạo, sửa, gom."*
- **Người 5** — *"Tôi lo TIỀN: Premium, thanh toán, quảng cáo, cài đặt."*

### 🎯 Mức độ khó từng mảng (nhóm tự phân theo năng lực)
Xếp từ **dễ học → khó** (chi tiết LOC ở `06` mục 7):
1. **Người 2** (trang chủ / tìm kiếm / thể loại) — 🟢 **dễ nhất + nhẹ nhất**: chủ yếu RecyclerView adapter lặp khuôn, phần ghép feed nặng đã nằm sẵn ở backend. *Quan trọng nhất cho demo* (bộ mặt & độ "sinh động" của app) → vừa quan trọng vừa dễ, hợp người ít kinh nghiệm.
2. **Người 4** (album + playlist + thư viện) — 🟡 **trung bình nhưng nhiều dòng nhất** (~7.000): CRUD nhiều thao tác, kéo-thả đổi thứ tự, ghép bìa 2x2 — hợp người làm nhanh.
3. **Người 1** (đăng nhập + khung app + hồ sơ) — 🟠 **TB-cao**: ít dòng nhưng khái niệm khó (JWT, Spring Security, nền tảng chung cả nhóm xài).
4. **Người 5** (premium + thanh toán + quảng cáo) — 🔴 **cao**: VNPay HMAC-SHA512, WebView deeplink, AdMob — logic tích hợp ngoài khó nhất.
5. **Người 3** (trình phát + nghệ sĩ + nhạc cá nhân) — 🔴 **nặng nhất**: ExoPlayer, đồng bộ lời LRC, Palette — cả lượng lẫn kỹ thuật.

### 🗄️ 21 bảng MySQL → ai sở hữu (mỗi bảng đúng 1 chủ)
| Người | Bảng / enum sở hữu |
|------|--------------------|
| **1** | `User` |
| **2** | `Genre`, `Mood` |
| **3** | `Track`, `Artist`, `PlayHistory`, `LikedTrack`(+Id), `FollowedArtist`(+Id) |
| **4** | **`Album`**, **`AlbumType`**, **`SavedAlbum`**(+Id), `Playlist`, `PlaylistTrack`(+Id) |
| **5** | `Subscription`, `SubscriptionPlan`, `UserSettings`, `StreamQuality` |

> 📌 **Cách học nhanh nhất:** nhớ BẢNG của mình trước → rồi mọi màn hình/API/Controller/Service nào đụng tới bảng đó đều là của mình. Tra ngược trong `network/ApiService.java`: endpoint nào liên quan bảng của mình thì thuộc về mình.

---

## 2. Kiến trúc 1 dòng

```
Android:  View (Activity/Fragment) → ViewModel (LiveData) → Repository → ApiService (Retrofit)
                                                                              │ HTTP + JWT
Backend:  Controller → Service → Repository (JPA) → Entity ↔ MySQL
```

- **Stack:** Spring Boot 4 / Java 21 / MySQL 8 / JWT · Android (Retrofit 2 + ExoPlayer + Material 3, MVVM).
- BE context-path `/musicapp` cổng 8080. App gọi `http://10.0.2.2:8080/musicapp/`.
- Code BE: `G:\Download\online_music_streaming_app\online_music_streaming_app` (`com.huce.online_music_streaming_app`).
- Code FE: `C:\...\MusicStreamingApp\app` (`com.example.musicstreamingapp`).

---

## 3. Tổng số file & khối lượng mỗi người (chi tiết ở file 06)

- **Tổng ≈ 490 file** (FE ~387 + BE ~103). Code Java ≈ 245 file (FE 142 + BE 103).
- Bỏ ~130 file **tài nguyên dùng chung** (drawable, mipmap, values…) → còn **~345 file logic + giao diện** chia 5 người.

| Người | Đọc kỹ (FE Java + layout + BE Java) | LOC ước tính | Ghi chú |
|------|------------------------------------:|-------------:|---------|
| 1 | ≈ 57 | ~4.600 | đăng nhập + khung app + hồ sơ + **nền tảng chung** (JWT/Security — khó) |
| 2 | ≈ 67 | ~3.770 | trang chủ + tìm kiếm + thể loại (đa adapter — **dễ & nhẹ nhất**) |
| 3 | ≈ 72 | ~6.300 | **nặng nhất về kỹ thuật** — ExoPlayer, LRC, Palette |
| 4 | ≈ 71 | ~7.000 | album + playlist CRUD + thư viện (**nhiều dòng nhất**) |
| 5 | ≈ 62 | ~4.500 | tích hợp ngoài (VNPay, AdMob) — logic khó nhất |

> Người 2 (mảng dễ) nhẹ nhất ~3.770; Người 4 nhiều dòng nhất ~7.000 (độ khó TB). Xem **mức độ khó** bên dưới để cân người.

---

## 4. File DÙNG CHUNG cả nhóm phải biết (Người 1 quản lý)

`network/ApiService.java` (mọi endpoint) · `RetrofitClient.java` · `AuthInterceptor.java` ·
`viewmodel/VmFactory.java` · `data/{Resource,Event,RepoCallback}.java` · `util/NavHelper.java` ·
`MainActivity.java` · `fragment/BaseDetailFragment.java` (lớp cha 3 màn chi tiết) ·
BE `config/SecurityConfig.java` · `security/JwtFilter.java` ·
5 model gốc `Track/Album/Artist/Playlist/Genre`.

> Ai sửa các file này phải báo cả nhóm.

---

## 4b. ⭐ TÍNH NĂNG XUYÊN MÀN (không thuộc 1 người — dễ bị bỏ sót)

| Tính năng | File chính | Chủ | Cắm vào file của ai |
|-----------|-----------|-----|----------------------|
| **Quảng cáo AdMob** (banner + interstitial mỗi 3 bài) | `AdManager`, `PremiumChecker` | **5** | `MainActivity`+`MusicApp` (1) · `PlayerManager` (3) |
| Mini-player (dải nhạc nổi) | `layout_mini_player` + logic trong `MainActivity` | khung **1** / phát **3** | chỉ hiển thị ở MainActivity |
| **Trộn bài + gate** | `util/ShuffleController` | **3** | Player(3), Album(2), Artist(3), Playlist(4) |
| Lớp cha màn chi tiết (hoãn việc nặng cho mượt) | `fragment/BaseDetailFragment` | **3** (hạ tầng) | Album(2), Artist(3), Playlist(4) kế thừa |
| Khung xương Shimmer (lúc tải) | `layout_shimmer_*` + thư viện Shimmer | mỗi màn tự lo | Home(2), Library(4), Album(2), Playlist(4), Genre(2) |
| Đổi màu nền theo ảnh bìa (Palette) | androidx.palette | mỗi màn tự lo | Player(3), Playlist(4) |
| 3 gate Premium (tải/trộn/chung) | `*GateBottomSheet` | **5** | gọi từ Player(3), Album(2), Playlist(4) |
| Ghi lịch sử nghe (mỗi lần phát) | `recordPlay`→`PlayHistory` | **3** (ghi) | dùng bởi Home(2), Recent(3), Stats(5) |
| Badge "HQ" (nhạc chất lượng cao) | `updateHqBadge`, `StreamQuality` | 3 hiện / 5 premium | bản thường `audioUrl` vs HD `audioUrlHigh` |
| Đa tài khoản | `AccountStore`, `AccountAdapter` | **1** | — |
| Ghép bìa 2x2 (collage) | `ui/PlaylistCoverView` | **4** | dùng ở Home banner/card (2) qua adapter |

**Hạ tầng/cấu hình (chủ: Người 1):** edge-to-edge opt-out (`targetSdk 36` + `values-v35`) · dark theme (`values-night`) · cho phép HTTP (`usesCleartextTraffic`) · khởi tạo MobileAds **chạy nền** + theo dõi vòng đời Activity (`MusicApp`).

---

## 4c. 🌐 TRANG ADMIN WEB (tuỳ chọn — quản nội dung trang chủ)

`admin-panel/admin.html` (React 1 file, chạy bằng `localhost:5173`) + 3 controller BE: `PlaylistAdminController` (curated playlist + mood + bìa), `GenreAdminController` (thể loại), `AdminController` (bài hát + upload **bản HD**). **App Android KHÔNG gọi** các endpoint này.

> 🎯 **Mục đích:** đổ dữ liệu cho **trang chủ sinh động** (banner FEATURED, dải mood) mà không phải sửa code. Vì nội dung phục vụ Home (Người 2) nhưng ghi bảng `Playlist` (Người 4) + `Track`/HD (Người 5) → **đề xuất giao trang admin cho Người 2** (chủ trang chủ), 4 & 5 hỗ trợ phần entity của mình. Nếu nhóm KHÔNG demo web admin thì cả khối này ngoài phạm vi.

### ⚠️ Khoảng trống đã biết (không phải lỗi quên)
- **`POST /api/auth/refresh`** có ở BE nhưng FE chưa dùng (gặp 401/403 là đăng xuất luôn).
- **13 endpoint khai báo mà chưa nối UI** (chart màn riêng, gói từ API, related/“Khám phá”, recommendations/mix…) — liệt kê ở `06` mục 7b. Đây là "đất" làm dày nếu mảng ai mỏng.
- Tải xuống chỉ là **gate** (chưa lưu file offline thật). Tier 2/3 (collab playlist, podcast, nghe cùng bạn) cố ý bỏ.

---

## 5. Cấu trúc mỗi file phân công (01–05)

1. Vai trò + sơ đồ luồng.
2. **BẢNG FILE ANDROID** + **BẢNG FILE BACKEND**.
3. **BẢNG drawable/anim theo màn**.
4. **⭐ THỨ TỰ ĐỌC FILE** — đọc từ đâu tới đâu để hiểu trọn 1 luồng.
5. Luồng hoạt động chính + checklist + điểm giao với người khác.

> 💡 Học theo luồng: *"bấm gì → ViewModel nào → Repository → endpoint → Controller/Service → bảng nào"*.
