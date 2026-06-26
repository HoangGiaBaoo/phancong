# THỐNG KÊ FILE & TÀI NGUYÊN — toàn dự án

> **Cập nhật 2026-06-26** theo code thực tế (sau gộp màn chi tiết Activity→Fragment, tách `ShuffleController`, chuyển miền Album sang Người 2).
> (1) Tổng bao nhiêu file? (2) Mỗi người đọc bao nhiêu / tổng? (3) Mỗi màn dùng `anim`/`drawable` nào.

---

## 1. TỔNG KIỂM KÊ (đếm thực tế)

### Android (FE)
| Loại | Số | | Loại | Số |
|------|--:|--|------|--:|
| Java | **142** | | drawable (.xml) | ~100 |
| layout (.xml) | **85** | | anim | 11 |
| mipmap (icon launcher) | 12 | | values | 6 |
| color/font/xml/menu | ~11 | | **TỔNG FE** | **≈ 387** |

**Java FE chia theo gói:** Activity 17 · adapter 28 · viewmodel 25 · model 25 · fragment 22 · util 11 · data 10 (3 + repository 7) · network 3 · ui 1.

### Spring Boot (BE)
| Loại | Số | | Loại | Số |
|------|--:|--|------|--:|
| controller | 20 | | service | 19 |
| dto | ~23 | | entity | 21 |
| repository | 13 | | config | 3 |
| security | 3 | | main | 1 |
| **TỔNG BE** | **≈ 103** | | | |

### ➡️ TỔNG TOÀN DỰ ÁN: **≈ 490 file** (Java thuần ≈ 245)

---

## 2. CHIA "PHẢI HIỂU LOGIC" vs "TÀI NGUYÊN DÙNG CHUNG"

| Nhóm | Gồm | Số | Cách xử lý |
|------|-----|--:|-----------|
| 🔵 **A — Logic + Giao diện** | Java (245) + layout (85) + anim (11) | **≈ 340** | **Chia cho 5 người** |
| ⚪ **B — Tài nguyên dùng chung** | drawable ~100 + mipmap 12 + values 6 + color/font/xml/menu ~11 | **≈ 130** | KHÔNG chia. Đọc lướt 1 lần. |

> ⚪ **Drawable/màu/font không thuộc ai sở hữu.** Cần icon mới → thêm vào kho chung + báo nhóm tránh trùng tên.

---

## 3. MỖI NGƯỜI ĐỌC BAO NHIÊU / 340 (chia theo chức năng)

| Người | Mảng | FE Java | Layout | BE Java | **Tổng cốt lõi** | % |
|------|------|--------:|-------:|--------:|----------------:|--:|
| **1** | Đăng nhập + Khung app + nền tảng chung + Hồ sơ | ~32 | ~10 | ~16 | **≈ 57** | 17% |
| **2** | Trang chủ + Tìm kiếm + Thể loại | ~27 | ~22 | ~18 | **≈ 67** | 19% |
| **3** | Trình phát + Nghệ sĩ + Đã thích/Theo dõi/Gần đây | ~34 | ~16 | ~22 | **≈ 72** | 21% |
| **4** | Album + Danh sách phát + Thư viện | ~33 | ~18 | ~20 | **≈ 71** | 21% |
| **5** | Premium + Thanh toán + Quảng cáo + Cài đặt + Thống kê | ~24 | ~14 | ~25 | **≈ 62** | 18% |

> Người 2 nhiều adapter (RecyclerView đa kiểu) nhưng lặp khuôn → **độ khó thấp nhất** + **khối lượng nhẹ nhất** (~67 file, ~3.770 dòng). Người 4 ôm thêm Album nên **nhiều dòng nhất** (~7.000) dù độ khó chỉ TB — xem mục 7.

---

## 4. BẢN ĐỒ `anim` (11 file) → CHỨC NĂNG

| File anim | Dùng ở (gọi trong code) | Người |
|-----------|--------------------------|-------|
| `fade_in`, `fade_out` | `MainActivity.loadFragment` — đổi 4 tab | 1 |
| `slide_in_right/left`, `slide_out_right/left` | `MainActivity.openDetail` — mở/đóng màn chi tiết | 1 (phục vụ 2,3,4) |
| `mini_player_slide_up/down` | `MainActivity.renderMiniPlayer` — hiện/ẩn mini-player | 1 (khung) / 3 (phát) |
| `player_slide_up/down`, `player_no_anim` | `PlayerActivity` mở/đóng | 3 |

> `anim` chỉ liên quan Người 1 (chuyển màn) và Người 3 (player). Người 2/4/5 hầu như không đụng.

---

## 5. CÁCH TRA `drawable`/`anim` CỦA 1 MÀN

1. Mở **file layout** màn đó → tìm `@drawable/` và `@anim/`.
2. Mở **file Java** màn đó → tìm `R.drawable.` và `R.anim.` (icon đổi lúc chạy, vd play↔pause).
3. Android Studio: `Ctrl+Click` lên `@drawable/xxx` để nhảy tới file; chuột phải → **Find Usages**.

---

## 7. ⭐ ĐÁNH GIÁ THEO CÔNG SỨC THẬT (LOC + độ khó) — sau cân bằng

| Người | Tổng LOC (ước) | Độ khó logic | Kết luận công sức |
|------|---------------:|--------------|-------------------|
| 1 | **~4.600** | 🔴 Cao — JWT, Spring Security, bug 404→403 | TB-cao (khái niệm khó) |
| 2 | **~3.770** | 🟢 Thấp — nhiều adapter nhưng lặp khuôn | **Nhẹ & dễ nhất** |
| 3 | **~6.300** | 🔴 Cao — ExoPlayer, LRC sync, Palette, ShuffleController | Nặng nhất về kỹ thuật |
| 4 | **~7.000** | 🟡 TB — album + kéo-thả đổi thứ tự, ghép bìa 2x2, CRUD đa màn | **Nhiều dòng nhất** |
| 5 | **~4.500** | 🔴 Rất cao — VNPay HMAC-SHA512, WebView deeplink, AdMob | TB-cao (logic khó nhất) |

> Người 2 (mảng dễ) nhẹ nhất ~3.770; Người 4 nhiều dòng nhất ~7.000 (độ khó TB). Chủ ý: mảng dễ + nhẹ cho người ít kinh nghiệm, mảng nhiều dòng cho người làm nhanh.

### File "nặng đô" cần lưu ý (giao cho người cứng tay)
| File | Dòng | Người | Vì sao nặng |
|------|----:|------|-------------|
| `PlaylistDetailFragment` | ~386 | 4 | Palette, bìa 2x2, gợi ý, menu |
| `AddToPlaylistViewModel` | ~333 | 4 | Tạo playlist nội tuyến + multi-select |
| `PlayerActivity` (+ `activity_player.xml` 433) | ~346 | 3 | Palette, lời chạy, seekbar, animate bìa |
| `PlayerManager` | ~179 | 3 | Máy trạng thái ExoPlayer |
| `ShuffleController` | — | 3 | Gộp trộn bài + gate (dùng 4 màn) |
| `MainActivity` | ~398 | 1 | Hub: nav + drawer + mini-player + QC + handleDetailIntent |
| `AlbumDetailViewModel` / `AlbumDetailFragment` | — | 4 | Album + lưu thư viện |
| `VnpayService` + `PaymentActivity` | 122+158 | 5 | **Ngắn nhưng khó nhất**: HMAC + deeplink |
| `AdminController` / `DataSeeder` | ~291 / 235 | 5 / 1 | Dài **nhưng dễ** (CRUD/seed máy móc) |

### 📌 Khuyến nghị
1. **Người 3 nặng kỹ thuật nhất** (ExoPlayer) → giao cho bạn code cứng nhất.
2. **Người 5 & 1 logic khó** (VNPay, bảo mật) → bạn chịu khó đọc tài liệu.
3. **Người 2 nhẹ & dễ nhất** → người ít kinh nghiệm; **Người 4 nhiều dòng nhất** (Album + Playlist) độ khó TB → người làm nhanh.
4. Đừng sợ `AdminController`/`DataSeeder` dài — CRUD/seed máy móc, đọc 1 lần là hiểu.

---

## 7b. ENDPOINT KHAI BÁO MÀ CHƯA DÙNG (app Android không gọi)

> Biết để khỏi tưởng có màn mà đi tìm; cũng là "đất" làm dày nếu mảng ai mỏng.

| Endpoint | Vì sao không dùng |
|----------|-------------------|
| `/api/charts/tracks` · `/api/charts/artists` | Chart chỉ hiện trong Home feed, không màn riêng |
| `/api/subscriptions/plans` | 3 gói **hard-code** trong `activity_premium_plans.xml` |
| `/api/albums/{id}` (getAlbum) | Màn Album lấy `getAlbums()` rồi lọc |
| `/api/albums/new` · `/api/playlists/curated` · `/api/artists/popular` | Đã gộp trong Home feed |
| `/api/recommendations/mix` | Chỉ dùng `/daily` |
| `/api/genres/{id}/tracks` | Màn Thể loại dùng `/feed` |
| `/api/tracks/{id}/related` · `/api/artists/{id}/related` | Thẻ "Khám phá" ở Player là placeholder, chưa nối |
| `/api/tracks/{id}` (getTrack) · `/api/history` (full) | Đã có list / chỉ dùng `/recent` |
| `POST /api/auth/refresh` | FE chưa dùng — gặp 401/403 là đăng xuất luôn |

---

## 8. DRAWABLE DÙNG CHUNG KHẮP NƠI (nhớ để khỏi nhầm của riêng ai)
- `placeholder_gradient` — ảnh chờ khi load bìa (≈ 40 màn).
- `ic_arrow_back` / `ic_chevron_down` / `ic_close` — quay lại/đóng. `ic_more_vert` — 3 chấm.
- `ic_play` / `ic_pause` / `ic_shuffle` / `ic_skip_next/previous` — điều khiển phát.
- `bg_shimmer_box` — khung xương. `bg_chip_selector` — chip lọc (Home, Library).
- `bg_btn_pill_green` / `ic_premium` — nút & icon Premium (gate).

> 🔧 **util dùng chung không thuộc ai:** `ItemAnim` (hiệu ứng item RecyclerView) — adapter Người 2/3/4 đều dùng.
