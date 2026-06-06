# THỐNG KÊ FILE & TÀI NGUYÊN — toàn dự án

> (1) Tổng bao nhiêu file? (2) Mỗi người đọc bao nhiêu / tổng? (3) Mỗi màn dùng `anim`/`drawable` nào.

---

## 1. TỔNG KIỂM KÊ (đếm thực tế bằng lệnh)

### Android (FE)
| Loại | Số | | Loại | Số |
|------|--:|--|------|--:|
| Java | **154** | | drawable (.xml) | **103** |
| layout (.xml) | **93** | | anim | **11** |
| mipmap (icon launcher) | 12 | | values (colors/dimens/strings/themes…) | 6 |
| color | 4 | | font | 4 |
| xml (config) | 2 | | menu | 1 |
| Manifest + Gradle | ~6 | | **TỔNG FE** | **≈ 396** |

**Java FE chia theo gói:** Activity 23 · adapter 32 · viewmodel 25 · model 25 · fragment 23 · util 12 · data 10 · network 3 · ui 1.

### Spring Boot (BE)
| Loại | Số | | Loại | Số |
|------|--:|--|------|--:|
| controller | 19 | | service | 19 |
| dto | 23 | | entity | 21 |
| repository | 13 | | config | 3 |
| security | 3 | | main | 1 |
| application.yaml + pom | ~4 | | **TỔNG BE** | **≈ 106** |

### ➡️ TỔNG TOÀN DỰ ÁN: **≈ 502 file** (Java thuần = 256)

---

## 2. CHIA "PHẢI HIỂU LOGIC" vs "TÀI NGUYÊN DÙNG CHUNG"

| Nhóm | Gồm | Số | Cách xử lý |
|------|-----|--:|-----------|
| 🔵 **A — Logic + Giao diện** | Java (256) + layout (93) + anim (11) | **≈ 360** | **Chia cho 5 người** |
| ⚪ **B — Tài nguyên dùng chung** | drawable 103 + mipmap 12 + values 6 + color 4 + font 4 + xml 2 + menu 1 | **≈ 132** | KHÔNG chia. Đọc lướt 1 lần. Mỗi màn chỉ *tham chiếu* vài cái. |

> ⚪ **Drawable/màu/font không thuộc ai sở hữu.** Khi cần icon mới → thêm vào kho chung + báo nhóm tránh trùng tên.

---

## 3. MỖI NGƯỜI ĐỌC BAO NHIÊU / 360 (đã cân bằng lại)

| Người | Mảng | FE Java | Layout | BE Java | **Tổng cốt lõi** | % |
|------|------|--------:|-------:|--------:|----------------:|--:|
| **1** | Đăng nhập + Khung app + nền tảng chung | ~28 | ~8 | ~14 | **≈ 50** | 14% |
| **2** | Trang chủ + Tìm kiếm + Thể loại | ~27 | ~22 | ~18 | **≈ 67** | 19% |
| **3** | Trình phát + Nghệ sĩ + Đã thích/Theo dõi/Gần đây | ~36 | ~18 | ~22 | **≈ 76** | 21% |
| **4** | Album + Danh sách phát + Thư viện | ~33 | ~18 | ~20 | **≈ 71** | 20% |
| **5** | Premium + Thanh toán + Tài khoản | ~26 | ~16 | ~28 | **≈ 70** | 19% |

> Con số làm tròn (≈); file dùng chung tính 1 lần. Trước cân bằng Người 4 là ~83 (cao nhất) → nay ~71; Người 3 thành lớn nhất (~76) vì gộp "trình phát + nhạc cá nhân của tôi" — đây là vai trò trung tâm nhất nên hợp lý.

---

## 4. BẢN ĐỒ `anim` (11 file) → CHỨC NĂNG (đủ cả 11)

| File anim | Dùng ở (gọi trong code) | Người |
|-----------|--------------------------|-------|
| `fade_in`, `fade_out` | `MainActivity.loadFragment` — đổi 4 tab | 1 |
| `slide_in_right/left`, `slide_out_right/left` | `MainActivity.openDetail` — mở/đóng màn chi tiết | 1 (phục vụ 3,4) |
| `mini_player_slide_up/down` | `MainActivity.renderMiniPlayer` — hiện/ẩn mini-player | 1 (khung) / 3 (phát) |
| `player_slide_up/down`, `player_no_anim` | `PlayerActivity` mở/đóng | 3 |

> `anim` chỉ liên quan Người 1 (chuyển màn) và Người 3 (player). Người 2/4/5 hầu như không đụng.

---

## 5. CÁCH TRA `drawable`/`anim` CỦA 1 MÀN

1. Mở **file layout** màn đó → tìm `@drawable/` và `@anim/`.
2. Mở **file Java** màn đó → tìm `R.drawable.` và `R.anim.` (icon đổi lúc chạy, vd play↔pause).
3. Trong Android Studio: `Ctrl+Click` lên `@drawable/xxx` để nhảy tới file; chuột phải drawable → **Find Usages** xem màn nào dùng.

> Mỗi file phân công (01–05) đã có sẵn **bảng "drawable theo màn"** cho các màn của người đó.

---

## 7. ⭐ ĐÁNH GIÁ THEO CÔNG SỨC THẬT (số dòng code + độ khó) — quan trọng

> Đếm **số file** dễ đánh lừa: 1 file 400 dòng logic xoắn não nặng hơn 5 file 50 dòng. Dưới đây là đo **số dòng code (LOC)** thực + chấm độ khó logic.

**Tổng:** FE Java 14.119 · BE Java 4.689 · Layout 8.683 dòng.

| Người | Java LOC | Layout LOC | **Tổng LOC** | Độ khó logic | Kết luận công sức |
|------|--------:|----------:|------------:|--------------|-------------------|
| 1 | ~3.300 | ~700 | **~4.000** | 🔴 Cao — JWT, Spring Security, bug 404→403 | TB-cao (ít dòng, khái niệm khó) |
| 2 | ~2.870 | ~900 | **~3.770** | 🟢 Thấp — nhiều adapter nhưng lặp khuôn | **Nhẹ nhất** |
| 3 | ~4.300 | ~2.000 | **~6.300** | 🔴 Cao — ExoPlayer, LRC sync, Palette | Nặng (cả lượng lẫn khó) |
| 4 | ~5.200 | ~1.800 | **~7.000** | 🟡 TB — kéo-thả đổi thứ tự, ghép bìa 2x2, adapter đa kiểu | **Nặng nhất về lượng** |
| 5 | ~3.560 | ~1.270 | **~4.830** | 🔴 Rất cao — VNPay HMAC-SHA512, WebView deeplink | TB-cao (logic khó nhất app) |

> ⚠️ **Chênh lệch thật:** Người 4 (~7.000 dòng) gần **GẤP ĐÔI** Người 2 (~3.770). Cách chia theo % file (P4≈71, P2≈67) đã che mất điều này.

### File "nặng đô" cần lưu ý (giao cho người cứng tay)
| File | Dòng | Người | Vì sao nặng |
|------|----:|------|-------------|
| `PlaylistDetailFragment` + `Activity` | 386+341 | 4 | Palette, bìa 2x2, gợi ý, menu |
| `AddToPlaylistViewModel` | 333 | 4 | Tạo playlist nội tuyến + multi-select |
| `PlayerActivity` (+ `activity_player.xml` 433) | 346 | 3 | Palette, lời chạy, seekbar, animate bìa |
| `MainActivity` | 363 | 1 | Hub: nav + drawer + mini-player + QC |
| `PlayerManager` | 179 | 3 | Máy trạng thái ExoPlayer |
| `VnpayService` + `PaymentActivity` | 122+158 | 5 | **Ngắn nhưng khó nhất**: mã hoá HMAC + deeplink |
| `AdminController` / `DataSeeder` | 291 / 235 | 5 / 1 | Dài **nhưng dễ** (CRUD/seed máy móc) — đừng sợ con số |

### 📌 Khuyến nghị (giữ cách chia dễ học, cân bằng bằng PHÂN CÔNG NGƯỜI)
1. **Người 3 & 4 là 2 mảng nặng nhất** → giao cho 2 bạn code cứng nhất nhóm.
2. **Người 5 & 1 ít dòng hơn nhưng logic khó nhất** (VNPay, bảo mật) → giao cho bạn chịu khó đọc tài liệu.
3. **Người 2 nhẹ nhất** → bạn này nên **hỗ trợ Người 4** (vd nhận phần màn Album: `AlbumDetail` + album đã lưu) nếu muốn cân chính xác. Đây là 1 thao tác chuyển gọn, mình làm trong 5 phút nếu nhóm đồng ý.
4. Đừng sợ `AdminController` 291 hay `DataSeeder` 235 dòng — chúng dài nhưng là CRUD/seed máy móc, đọc 1 lần là hiểu.

---

## 7b. ENDPOINT KHAI BÁO MÀ CHƯA DÙNG (13 cái — app Android không gọi)

> Backend có sẵn + `ApiService` khai báo, nhưng **không màn nào gọi**. Biết để khỏi tưởng có màn mà đi tìm; cũng là "đất" làm thêm nếu mảng ai mỏng.

| Endpoint | Vì sao không dùng |
|----------|-------------------|
| `/api/charts/tracks` · `/api/charts/artists` | Chart chỉ hiện trong **Home feed** (BE dựng sẵn), không có màn riêng |
| `/api/subscriptions/plans` | 3 gói **hard-code** trong `activity_premium_plans.xml` |
| `/api/albums/{id}` (getAlbum) | Màn Album lấy `getAlbums()` rồi lọc |
| `/api/albums/new` · `/api/playlists/curated` · `/api/artists/popular` | Đã gộp trong Home feed |
| `/api/recommendations/mix` | Chỉ dùng `/daily` |
| `/api/genres/{id}/tracks` | Màn Thể loại dùng `/feed` |
| `/api/tracks/{id}/related` · `/api/artists/{id}/related` | Thẻ "Khám phá" ở Player là placeholder, chưa nối |
| `/api/tracks/{id}` (getTrack) · `/api/history` (full) | Không cần — đã có list / chỉ dùng `/recent` |

> 📌 **Hệ quả phân công:** "Bảng xếp hạng" (Người 2) và "danh sách gói" (Người 5) **nhẹ hơn** con số thoạt nhìn. Ngược lại, nối các endpoint trên vào UI là cách **làm dày** mảng nếu cần.

> 🔧 **util dùng chung:** `ItemAnim` (hiệu ứng item RecyclerView) dùng bởi adapter của Người 2/3/4 — không thuộc riêng ai.

---

## 8. DRAWABLE DÙNG CHUNG XUẤT HIỆN KHẮP NƠI (nhớ để khỏi nhầm là của riêng ai)
- `placeholder_gradient` — ảnh chờ khi load bìa (≈ 40 màn dùng).
- `ic_arrow_back` / `ic_chevron_down` / `ic_close` — nút quay lại/đóng.
- `ic_more_vert` — nút 3 chấm.
- `ic_play` / `ic_pause` / `ic_shuffle` / `ic_skip_next/previous` — điều khiển phát.
- `bg_shimmer_box` — ô xám khung xương lúc tải.
- `bg_chip_selector` — chip lọc (Home, Library).
- `bg_btn_pill_green` / `ic_premium` — nút & icon Premium (gate).
