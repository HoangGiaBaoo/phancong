# TỔNG QUAN PHÂN CÔNG — Đồ án Spotify clone (5 người)

> Mục lục + bản đồ chung. Mỗi người đọc file riêng (01–05). Thống kê số file ở **`06_THONG_KE...md`**.
> **Phiên bản 2** — đã cân bằng lại theo nguyên tắc **"ai sở hữu dữ liệu (entity) thì sở hữu luôn màn hình của dữ liệu đó"**.

---

## 1. Cách chia: LÁT CẮT DỌC theo "miền dữ liệu"

Mỗi người ôm trọn **1 nhóm entity** + tất cả màn hình/API liên quan, từ Android xuống Spring Boot. Nhờ vậy **mỗi bảng MySQL chỉ có đúng 1 chủ** → không giẫm chân nhau.

| Người | Miền dữ liệu (entity sở hữu) | Mảng tính năng |
|------|------------------------------|----------------|
| **1** | `User` + bảo mật | Đăng nhập, Đăng ký, Đa tài khoản, **Khung app (MainActivity)** + **nền tảng mạng dùng chung** |
| **2** | `Genre`, `Mood` | Trang chủ (feed), Tìm kiếm, Thể loại, Bảng xếp hạng, Gợi ý |
| **3** | `Track`, `Artist`, `PlayHistory`, `LikedTrack`, `FollowedArtist` | **Trình phát**, Lời bài hát, **Chi tiết Nghệ sĩ**, Bài hát đã thích, Đang theo dõi, Nghe gần đây |
| **4** | `Album`, `Playlist`, `PlaylistTrack`, `SavedAlbum` | **Chi tiết Album**, Album đã lưu, **Danh sách phát (CRUD)**, màn Thư viện |
| **5** | `Subscription`, `UserSettings` | Premium, Thanh toán VNPay, Quảng cáo, Hồ sơ, Cài đặt, Thống kê |

> 🔄 **Thay đổi so với bản 1:** "Bài hát đã thích / Đang theo dõi / Nghe gần đây" chuyển từ Người 4 → **Người 3** (vì dữ liệu like/follow/history thuộc Người 3). Nhờ đó Người 4 nhẹ bớt (từ ~83 còn ~71 file).

### 🧠 Câu thần chú để nhớ phần của mình (1 dòng/người)
- **Người 1** — *"Tôi lo cho user VÀO được app."* (đăng nhập + khung app)
- **Người 2** — *"Tôi lo cho user TÌM thấy nhạc."* (trang chủ + tìm kiếm + thể loại)
- **Người 3** — *"Tôi lo BÀI HÁT & NGHỆ SĨ: nghe, thích, theo dõi."*
- **Người 4** — *"Tôi lo ALBUM & DANH SÁCH PHÁT: lưu, tạo, sửa."*
- **Người 5** — *"Tôi lo TIỀN & TÀI KHOẢN: Premium, hồ sơ, cài đặt."*

### 🗄️ 21 bảng MySQL → ai sở hữu (mỗi bảng đúng 1 chủ)
| Người | Bảng / enum sở hữu |
|------|--------------------|
| **1** | `User` |
| **2** | `Genre`, `Mood` |
| **3** | `Track`, `Artist`, `PlayHistory`, `LikedTrack`(+Id), `FollowedArtist`(+Id) |
| **4** | `Album`, `AlbumType`, `Playlist`, `PlaylistTrack`(+Id), `SavedAlbum`(+Id) |
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

- **Tổng ≈ 502 file** (FE 396 + BE 106). Code Java = 256 file.
- Bỏ ~132 file **tài nguyên dùng chung** (drawable, mipmap, values…) → còn **~360 file logic + giao diện** chia 5 người.

| Người | Đọc kỹ (FE Java + layout + BE Java) | Ghi chú |
|------|------------------------------------:|---------|
| 1 | **≈ 50** | nhẹ về tính năng nhưng **gánh nền tảng chung** (khó) |
| 2 | **≈ 67** | nhiều adapter (RecyclerView đa kiểu) |
| 3 | **≈ 76** | lớn nhất — gộp trình phát + nhạc cá nhân |
| 4 | **≈ 71** | album + playlist CRUD |
| 5 | **≈ 70** | tích hợp ngoài (VNPay, AdMob) |

> Ngoài phần riêng, **mọi người** đều phải nắm ~15 file **nền tảng chung** (xem mục 4).

---

## 4. File DÙNG CHUNG cả nhóm phải biết (Người 1 quản lý)

`network/ApiService.java` (mọi endpoint) · `RetrofitClient.java` · `AuthInterceptor.java` ·
`viewmodel/VmFactory.java` · `data/{Resource,Event,RepoCallback}.java` · `util/NavHelper.java` ·
`MainActivity.java` · BE `config/SecurityConfig.java` · `security/JwtFilter.java` ·
5 model gốc `Track/Album/Artist/Playlist/Genre`.

> Ai sửa các file này phải báo cả nhóm.

---

## 5. Cấu trúc mỗi file phân công (01–05)

1. Vai trò + sơ đồ luồng.
2. **BẢNG FILE ANDROID** — `File | Loại | Chức năng (1 dòng)`.
3. **BẢNG FILE BACKEND** — đủ controller/service/repository/entity/dto.
4. **BẢNG drawable/anim theo màn** — màn này dùng icon/hiệu ứng nào.
5. Luồng hoạt động chính (các bước).
6. Checklist + endpoint + điểm giao với người khác.

> 💡 Học theo luồng: *"bấm gì → ViewModel nào → Repository → endpoint → Controller/Service → bảng nào"*.
