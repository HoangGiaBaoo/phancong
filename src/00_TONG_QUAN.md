# TỔNG QUAN PHÂN CÔNG — Đồ án Spotify clone (5 người)

> File này là **mục lục + bản đồ chung**. Mỗi người đọc file riêng của mình (01–05) để biết chi tiết file code phải nắm. Phần "Kiến trúc tổng quan" và "Nền tảng dùng chung" nằm đầy đủ trong **file 01**.

---

## 1. Cách chia việc: LÁT CẮT DỌC

Mỗi người **ôm trọn cả Android (FE) lẫn Spring Boot (BE)** của 1 mảng tính năng → tự demo được luồng xuyên suốt từ giao diện → API → database, không bị phụ thuộc người khác khi trình bày.

| Người | Mảng | File phân công | Điểm nhấn kỹ thuật |
|------|------|----------------|--------------------|
| **1** | Đăng nhập, Tài khoản & Khung app | `01_Nguoi1_DangNhap_TaiKhoan_KhungApp.md` | JWT, Spring Security, MainActivity hub, **nền tảng mạng dùng chung** |
| **2** | Trang chủ, Tìm kiếm & Thể loại | `02_Nguoi2_TrangChu_TimKiem_TheLoai.md` | RecyclerView đa kiểu, feed ghép nhiều nguồn |
| **3** | Trình phát, Lời bài hát & Chi tiết Bài/Album/Nghệ sĩ | `03_Nguoi3_TrinhPhat_LoiBaiHat_ChiTiet.md` | ExoPlayer, stream HTTP Range, LRC sync |
| **4** | Thư viện & Danh sách phát | `04_Nguoi4_ThuVien_DanhSachPhat.md` | CRUD playlist, 5 bảng quan hệ N-N |
| **5** | Premium, Thanh toán & Tài khoản cá nhân | `05_Nguoi5_Premium_ThanhToan_TaiKhoan.md` | VNPay (HMAC, WebView), AdMob, gate |

---

## 2. Kiến trúc 1 dòng (chi tiết ở file 01)

```
Android:  View (Activity/Fragment) → ViewModel (LiveData) → Repository → ApiService (Retrofit)
                                                                              │ HTTP + JWT
Backend:  Controller → Service → Repository (JPA) → Entity ↔ MySQL
```

- **Stack:** Spring Boot 4 / Java 21 / MySQL 8 / JWT · Android (Retrofit 2 + ExoPlayer + Material 3, MVVM).
- **Backend:** context-path `/musicapp`, port 8080. App gọi `http://10.0.2.2:8080/musicapp/`.
- **Thư mục code BE:** `G:\online_music_streaming_app\online_music_streaming_app` (package `com.huce.online_music_streaming_app`).
- **Thư mục code FE:** `C:\...\MusicStreamingApp` (package `com.example.musicstreamingapp`).

---

## 3. Bản đồ tính năng → người phụ trách

- Splash / Login / Register / đa tài khoản / Drawer / MainActivity ......... **Người 1**
- Trang chủ (feed) / Tìm kiếm / Thể loại / Bảng xếp hạng ................... **Người 2**
- Phát nhạc / Player / Lời bài hát / Chi tiết Album / Chi tiết Nghệ sĩ ..... **Người 3**
- Thư viện / Playlist (tạo, sửa, xóa, đổi thứ tự, ảnh bìa) / Đã thích / Theo dõi / Gần đây ... **Người 4**
- Premium / Thanh toán VNPay / Quảng cáo / Gate / Hồ sơ / Cài đặt / Thống kê ... **Người 5**

---

## 4. File DÙNG CHUNG cả nhóm phải biết (Người 1 quản lý)

`network/ApiService.java` (mọi endpoint) · `network/RetrofitClient.java` · `network/AuthInterceptor.java` ·
`viewmodel/VmFactory.java` (tạo mọi ViewModel) · `data/{Resource,Event,RepoCallback}.java` ·
`util/NavHelper.java` (mở màn chi tiết) · `MainActivity.java` (khung chứa mọi tab) ·
Backend: `config/SecurityConfig.java` · `security/JwtFilter.java`.

> Ai sửa các file này phải báo cả nhóm vì ảnh hưởng mọi người.

---

## 5. Cách đọc file của bạn

Mỗi file có cấu trúc giống nhau:
1. **Vai trò** của bạn (1 đoạn).
2. **Sơ đồ luồng** mảng của bạn.
3. **Chi tiết từng chức năng**: liệt kê *mọi file* (màn hình, ViewModel, Repository, Adapter, **layout XML**, Controller, Service, Entity) + giải thích vai trò + **luồng hoạt động từng bước**.
4. **Checklist "file của tôi"** — tick để nhớ file nào là của mình.
5. **Endpoint API** mảng bạn dùng.
6. **Điểm giao với người khác** — chỗ code bạn đụng vào người khác.

> 💡 Học theo luồng, đừng học theo từng file rời. Cứ bám: *"bấm nút gì → ViewModel nào → Repository nào → endpoint nào → Controller/Service nào → bảng nào"*. Mỗi file đã viết sẵn luồng đó để bạn đọc thuộc.
