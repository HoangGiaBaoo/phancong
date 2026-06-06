# NGƯỜI 1 — Đăng nhập, Tài khoản & Khung ứng dụng

> **Bạn là "người gác cổng" của app.** Bạn lo từ lúc bật app (Splash) → đăng nhập/đăng ký → vào màn chính (MainActivity) với thanh điều hướng dưới + ngăn kéo (drawer) + chuyển đổi nhiều tài khoản. Bạn cũng **sở hữu nền tảng mạng dùng chung** mà cả 4 người còn lại đều xài (Retrofit, JWT, VmFactory). Hiểu phần này tức là hiểu "xương sống" của cả app.

---

## 0. KIẾN TRÚC TỔNG QUAN (cả nhóm đọc 1 lần)

App Android viết theo **MVVM**. Mọi màn hình đều chạy theo đúng 1 khuôn:

```
Activity/Fragment  →  ViewModel  →  Repository  →  ApiService (Retrofit)
   (View: chỉ vẽ)     (giữ state     (gói gọi mạng)   (khai báo endpoint)
                       bằng LiveData)                          │
                                                               ▼  HTTP + JWT
Backend:  Controller  →  Service  →  Repository (JPA)  →  Entity ↔ MySQL
```

- **View** (Activity/Fragment): chỉ bind dữ liệu ra giao diện, bắt sự kiện bấm nút, **không gọi mạng trực tiếp**.
- **ViewModel**: giữ trạng thái màn hình bằng `LiveData`, sống sót khi xoay máy, gọi Repository.
- **Repository**: bọc Retrofit, trả kết quả về qua `RepoCallback` (success/error).
- **ApiService**: 1 interface khai báo tất cả endpoint (file `network/ApiService.java`).
- Backend Spring Boot: `Controller` (nhận HTTP) → `Service` (logic) → `Repository` (JPA) → `Entity` (bảng MySQL).

**Quy ước backend:** context-path `/musicapp`, port 8080. URL đầy đủ ví dụ `http://localhost:8080/musicapp/api/auth/login`. App Android gọi qua `http://10.0.2.2:8080/musicapp/` (10.0.2.2 = "localhost của máy tính" nhìn từ máy ảo).

---

## 1. NỀN TẢNG DÙNG CHUNG — bạn là người quản lý (cả nhóm phải hiểu)

Đây là các file **không thuộc riêng tính năng nào** nhưng mọi tính năng đều đi qua. Bạn (Người 1) chịu trách nhiệm chính:

| File | Vai trò | Luồng |
|------|---------|-------|
| `network/RetrofitClient.java` | Tạo đối tượng Retrofit (singleton). Khai báo `BASE_URL = 10.0.2.2:8080/musicapp/` và `BASE_MEDIA_URL` (dùng để load ảnh/nhạc bằng Glide/ExoPlayer). Gắn `AuthInterceptor` + log. | `getApiService(prefs)` trả về 1 `ApiService` đã sẵn token. `reset()` xoá cache khi đổi tài khoản. |
| `network/ApiService.java` | **Bản hợp đồng API** — 1 interface Retrofit liệt kê toàn bộ ~60 endpoint (auth, track, album, playlist, search, subscription…). Mỗi người đọc phần endpoint của mình ở đây. | Retrofit tự sinh code gọi HTTP từ các annotation `@GET/@POST/...`. |
| `network/AuthInterceptor.java` | Tự động **đính kèm header `Authorization: Bearer <token>`** vào mọi request (trừ `/api/auth/`). Nếu server trả **401/403** → gọi `SessionManager.markExpired()` để đá về Login. | Chạy với MỌI request của app. ⚠️ Đây là chỗ từng gây bug "đang xem album bị văng ra Login" (xem `memory` của dự án). |
| `viewmodel/VmFactory.java` | **Nhà máy tạo ViewModel.** Vì không dùng Hilt/Dagger, mỗi khi 1 màn cần ViewModel thì `VmFactory` `new` ra đúng ViewModel + nhồi Repository tương ứng. | Có 1 dãy `if (modelClass == XxxViewModel) return new XxxViewModel(new XxxRepository(api))`. Muốn biết VM nào dùng Repository nào → đọc file này. |
| `data/RepoCallback.java` | Interface callback đơn giản: `onSuccess(T)` / `onError(String)`. Repository trả kết quả về ViewModel qua đây. | Thay cho RxJava/coroutine — giữ đơn giản cho đồ án. |
| `data/Resource.java` | Bọc trạng thái `Loading / Success / Error` (dùng khi UI cần phân biệt đang tải). | Một số màn dùng, một số trả thẳng LiveData. |
| `data/Event.java` | Bọc sự kiện "1 lần" (snackbar, điều hướng) để không bị phát lại khi xoay máy (`consume(...)`). | ViewModel `postValue(new Event<>(msg))`, View `e.consume(msg -> ...)`. |
| `util/NavHelper.java` | **Bộ định tuyến mở màn chi tiết.** `openAlbum/openArtist/openPlaylist/openLiked/openRecent`. Nếu đang trong MainActivity → mở dạng **Fragment** (bottom-nav đứng yên); nếu ở Activity riêng → mở Activity. | Cả nhóm gọi `NavHelper.openXxx(context, id)` thay vì tự tạo Intent. |

> 📌 **Lời khuyên:** In riêng bảng này ra. Khi bất kỳ ai trong nhóm hỏi "dữ liệu đi từ đâu tới đâu", câu trả lời luôn nằm trong `ApiService` (gọi gì) + `VmFactory` (ai tạo ai) + `RetrofitClient` (gọi tới đâu).

---

## 2. SƠ ĐỒ LUỒNG MẢNG CỦA BẠN

```
SplashActivity ──(đã có token?)──► MainActivity (Trang chủ)
       │  chưa có token
       ▼
  LoginActivity ──► (login OK, lưu token) ──► MainActivity
       │  bấm "Đăng ký"
       ▼
  RegisterActivity ──► (đăng ký OK) ──► quay lại Login

MainActivity (khung cố định, luôn hiện):
   ├─ BottomNav: Trang chủ | Tìm kiếm | Thư viện | Premium | (Tạo)
   ├─ fragment_container: nơi nạp 4 tab + mọi màn chi tiết
   ├─ mini-player (dải nhạc đang phát, nổi trên bottom-nav)
   ├─ adBanner (quảng cáo cho user Free)
   └─ Drawer (ngăn kéo trái): avatar, đổi tài khoản, Hồ sơ, Cài đặt, Thống kê...
```

---

## 3. CHI TIẾT TỪNG CHỨC NĂNG

### 3.1. Màn khởi động (Splash) — quyết định vào đâu

**File cần đọc:**
- `SplashActivity.java` — màn chờ ~1-2s khi mở app.
- `res/layout/activity_splash.xml` — logo giữa màn.
- `util/TokenManager.java` — đọc/ghi token + username vào `SharedPreferences`.
- `MusicApp.java` — lớp `Application`, chạy **trước** mọi Activity (khởi tạo SDK quảng cáo, ExoPlayer…).

**Luồng:** App mở → `MusicApp.onCreate()` (khởi tạo toàn cục) → `SplashActivity` hỏi `TokenManager.getToken()`:
- Có token → `startActivity(MainActivity)` → `finish()`.
- Không có → `startActivity(LoginActivity)`.

### 3.2. Đăng nhập

**File cần đọc (FE):**
- `LoginActivity.java` — màn nhập username/password.
- `res/layout/activity_login.xml` — ô nhập + nút Đăng nhập + link Đăng ký.
- `viewmodel/LoginViewModel.java` — giữ state, nhận sự kiện bấm "Đăng nhập".
- `data/repository/AuthRepository.java` — gọi API login, **lưu token** sau khi thành công.
- `model/LoginRequest.java` — body gửi đi `{username, password}`.
- `model/JwtResponse.java` — kết quả trả về `{accessToken, refreshToken, username, role...}`.
- `util/TokenManager.java` — lưu token/username.
- `util/AccountStore.java` — lưu danh sách nhiều tài khoản (cho tính năng đổi tài khoản).

**File cần đọc (BE):**
- `controller/AuthController.java` — `POST /api/auth/login`, `/register`, `/refresh`, `/logout`.
- `service/AuthService.java` — logic: xác thực username/password, sinh JWT.
- `security/JwtUtil.java` — sinh & kiểm tra token (HMAC + hạn dùng).
- `security/UserDetailsServiceImpl.java` — nạp user từ DB cho Spring Security.
- `security/JwtFilter.java` — lọc mọi request, đọc Bearer token, set "đã đăng nhập".
- `config/SecurityConfig.java` — khai báo endpoint nào public, endpoint nào cần đăng nhập.
- `entity/User.java` + `repository/UserRepository.java` — bảng `Users`.
- `dto/LoginRequest.java`, `dto/JwtResponse.java`.

**Luồng end-to-end (học thuộc luồng này, các luồng khác tương tự):**
1. User nhập → `LoginActivity` bắt sự kiện nút → gọi `vm.login(user, pass)`.
2. `LoginViewModel` gọi `authRepository.login(...)`.
3. `AuthRepository` gọi `api.login(new LoginRequest(...))` (Retrofit → HTTP POST).
4. Bên BE: `AuthController.login()` → `AuthService.login()`:
   - `authenticationManager.authenticate(...)` kiểm tra mật khẩu (so với `passwordHash` bcrypt trong DB).
   - Đúng → `JwtUtil.generateToken(username)` sinh access token + refresh token → trả `JwtResponse`.
   - Sai → trả **401** `{error: "Invalid username or password"}`.
5. Về FE: `AuthRepository` nhận `JwtResponse` → `TokenManager.saveToken()` + `AccountStore.addOrUpdate()` → callback `onSuccess`.
6. `LoginViewModel` báo thành công → `LoginActivity` mở `MainActivity`, `finish()`.

### 3.3. Đăng ký

**File:** `RegisterActivity.java` + `activity_register.xml` + `viewmodel/RegisterViewModel.java` + `AuthRepository` (hàm `register`) + `model/RegisterRequest.java`.
**BE:** `AuthController.register()` → `AuthService.register()`:
- Kiểm tra trùng username/email → nếu trùng trả **400**.
- Mã hoá mật khẩu bằng `passwordEncoder.encode(...)` (bcrypt) → lưu `User` → tạo `UserSettings` mặc định.

**Luồng:** giống Login nhưng kết thúc bằng việc quay lại màn Login để đăng nhập.

### 3.4. MainActivity — khung ứng dụng (TRÁI TIM của mảng bạn)

**File cần đọc:**
- `MainActivity.java` — **đọc kỹ nhất**. Quản lý: bottom-nav, fragment_container, mini-player, drawer, banner quảng cáo.
- `res/layout/activity_main.xml` — DrawerLayout bọc: AppBar + `fragment_container` + `bottom_nav` + `mini_player` + `adBanner`.
- `res/layout/drawer_content.xml` — nội dung ngăn kéo (avatar, các mục Hồ sơ/Cài đặt/Thống kê/Gần đây).
- `viewmodel/MainViewModel.java` — giữ thông tin user (avatar, tên) cho drawer + trạng thái mini-player (bài đang phát).
- `data/repository/UserRepository.java` — lấy thông tin `getMe()` (dùng chung với Người 5).
- `util/BottomNavHelper.java` — tiện ích cho bottom-nav (nếu có).

**Luồng quan trọng cần nhớ — cách 4 tab + màn chi tiết cùng sống trong 1 Activity:**
- `setupBottomNav()`: bấm tab → `popBackStack` (gỡ hết màn chi tiết đang chồng) → `loadFragment(new HomeFragment()/SearchFragment()/...)` bằng `replace()`.
- Tab "Tạo" (nav_create) không phải tab thật → mở `CreateBottomSheet` (của Người 4).
- `openDetail(fragment)`: khi mở Album/Nghệ sĩ/Playlist, dùng `hide(current) + add(...)` + `addToBackStack` → **bottom-nav và mini-player đứng yên**, chỉ phần giữa đổi. Bấm Back → trả về tab cũ nguyên trạng. **Đây là lý do NavHelper mở Fragment thay vì Activity khi đang ở Main.**

**Mini-player** (dải nhạc nổi ở dưới — logic phát thuộc Người 3, nhưng "cái khung" nằm ở đây):
- `MainViewModel.currentTrack()` đổi → `renderMiniPlayer()` hiện/ẩn dải + load ảnh bìa.
- `progressTick` (Handler 500ms) cập nhật thanh tiến trình.
- Bấm vào dải → mở `PlayerActivity` (Người 3).

**Banner quảng cáo:** `renderAdsForSubscription()` — Premium thì ẩn banner, Free thì hiện (liên quan Người 5).

### 3.5. Đa tài khoản (đổi/thêm tài khoản trong Drawer)

**File cần đọc:**
- `AddAccountActivity.java` + `activity_add_account.xml` — đăng nhập thêm 1 tài khoản khác.
- `util/AccountStore.java` — lưu danh sách `{username, token, displayName, avatar}` nhiều tài khoản.
- `adapter/AccountAdapter.java` — danh sách avatar tài khoản nằm ngang trong drawer.
- `res/layout/item_drawer_account.xml`, `item_drawer_add_account.xml` — item từng tài khoản / nút "Thêm".

**Luồng đổi tài khoản:** bấm avatar trong drawer → `accountStore.switchTo(username)` (đổi token đang dùng) → `RetrofitClient.reset()` (xoá Retrofit cũ để dùng token mới) → khởi động lại `MainActivity` với `CLEAR_TASK`.

### 3.6. Đăng xuất & hết phiên

**File:** `util/SessionManager.java` — giữ 1 cờ "đã hết hạn", khi `markExpired()` thì xoá token + bật `LoginActivity` (CLEAR_TASK). Được gọi từ `AuthInterceptor` khi gặp 401/403.

---

## 4. ✅ CHECKLIST "FILE CỦA TÔI" (Người 1)

**Android — màn hình & layout:**
- [ ] `SplashActivity.java` + `activity_splash.xml`
- [ ] `LoginActivity.java` + `activity_login.xml`
- [ ] `RegisterActivity.java` + `activity_register.xml`
- [ ] `AddAccountActivity.java` + `activity_add_account.xml`
- [ ] `MainActivity.java` + `activity_main.xml` + `drawer_content.xml` + `item_drawer_account.xml` + `item_drawer_add_account.xml`
- [ ] `MusicApp.java`

**Android — ViewModel / Repository / Adapter / Util:**
- [ ] `LoginViewModel`, `RegisterViewModel`, `MainViewModel`
- [ ] `AuthRepository` (+ `UserRepository` dùng chung)
- [ ] `AccountAdapter`
- [ ] `TokenManager`, `SessionManager`, `AccountStore`, `BottomNavHelper`

**Android — NỀN TẢNG DÙNG CHUNG (bạn quản lý):**
- [ ] `RetrofitClient`, `ApiService`, `AuthInterceptor`
- [ ] `VmFactory`
- [ ] `data/Resource`, `data/Event`, `data/RepoCallback`
- [ ] `util/NavHelper`
- [ ] models: `LoginRequest`, `RegisterRequest`, `JwtResponse`, `User`, `UserMe`

**Backend:**
- [ ] `controller/AuthController`, `service/AuthService`
- [ ] `security/JwtUtil`, `security/JwtFilter`, `security/UserDetailsServiceImpl`
- [ ] `config/SecurityConfig`, `config/DataSeeder` (seed dữ liệu mẫu), `OnlineMusicStreamingAppApplication` (hàm main)
- [ ] `entity/User`, `repository/UserRepository`
- [ ] `dto/LoginRequest`, `dto/RegisterRequest`, `dto/JwtResponse`, `dto/RefreshTokenRequest`

---

## 5. ENDPOINT API MẢNG BẠN DÙNG
- `POST /api/auth/register` — đăng ký
- `POST /api/auth/login` — đăng nhập (trả JWT)
- `POST /api/auth/refresh` — làm mới token
- `POST /api/auth/logout` — đăng xuất
- `GET /api/users/me` — thông tin user cho drawer (dùng chung Người 5)

## 6. ĐIỂM GIAO VỚI NGƯỜI KHÁC
- **Cả 4 người** đều xài `ApiService`, `RetrofitClient`, `VmFactory`, `NavHelper` của bạn → khi bạn sửa, báo cả nhóm.
- **Người 3** đặt logic mini-player vào khung MainActivity của bạn (`MainViewModel.currentTrack`).
- **Người 5** dùng chung `UserRepository` (drawer header) và đọc trạng thái Premium để ẩn banner.
- **Người 4** mở `CreateBottomSheet` từ tab "Tạo" của bạn.
