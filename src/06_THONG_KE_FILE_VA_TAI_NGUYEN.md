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

## 6. DRAWABLE DÙNG CHUNG XUẤT HIỆN KHẮP NƠI (nhớ để khỏi nhầm là của riêng ai)
- `placeholder_gradient` — ảnh chờ khi load bìa (≈ 40 màn dùng).
- `ic_arrow_back` / `ic_chevron_down` / `ic_close` — nút quay lại/đóng.
- `ic_more_vert` — nút 3 chấm.
- `ic_play` / `ic_pause` / `ic_shuffle` / `ic_skip_next/previous` — điều khiển phát.
- `bg_shimmer_box` — ô xám khung xương lúc tải.
- `bg_chip_selector` — chip lọc (Home, Library).
- `bg_btn_pill_green` / `ic_premium` — nút & icon Premium (gate).
