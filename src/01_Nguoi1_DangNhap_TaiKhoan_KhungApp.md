# NGƯỜI 1 — Đăng nhập, Tài khoản, Hồ sơ & Khung ứng dụng

> **Miền dữ liệu:** `User` + bảo mật. **Vai trò:** từ lúc bật app → đăng nhập/đăng ký → vào `MainActivity` (bottom-nav + drawer + đa tài khoản) + màn **Hồ sơ / Sửa hồ sơ** (dữ liệu user). Sở hữu **nền tảng mạng dùng chung** cả nhóm xài.

```
SplashActivity ─(có token?)─► MainActivity / LoginActivity ─► (login OK, lưu token) ─► MainActivity
MainActivity = bottom-nav (5 tab) + fragment_container + mini-player + adBanner + Drawer
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng (1 dòng) |
|------|------|--------------------|
| `SplashActivity.java` | Activity | Màn chờ 1.5s; có token → Main, không → Login |
| `LoginActivity.java` | Activity | Nhập user/pass, gọi đăng nhập |
| `RegisterActivity.java` | Activity | Đăng ký tài khoản mới |
| `AddAccountActivity.java` | Activity | Đăng nhập thêm 1 tài khoản (đa tài khoản) |
| `ProfileActivity.java` | Activity | Xem hồ sơ (tên, ảnh, playlist của user) |
| `EditProfileActivity.java` | Activity | Sửa tên + đổi avatar (upload) |
| `MainActivity.java` ⭐ | Activity | **Hub**: bottom-nav, fragment_container, mini-player, drawer, banner, `openDetail()`, `handleDetailIntent()` |
| `MusicApp.java` | Application | Chạy đầu tiên; khởi tạo AdMob (chạy nền) + theo dõi vòng đời Activity |
| `LoginViewModel.java` | ViewModel | State + sự kiện đăng nhập |
| `RegisterViewModel.java` | ViewModel | State + sự kiện đăng ký |
| `ProfileViewModel.java` / `EditProfileViewModel.java` | ViewModel | Xem & sửa hồ sơ |
| `MainViewModel.java` | ViewModel | Info user cho drawer + bài đang phát (mini-player), `refreshFromPlayer()` |
| `AuthRepository.java` | Repository | Gọi `login/register`, **lưu token** sau khi OK |
| `data/repository/UserRepository.java` | Repository | `getMe/getProfile/updateProfile/uploadAvatar/settings` (drawer + hồ sơ; Người 5 dùng `settings`) |
| `AccountAdapter.java` | Adapter | Danh sách avatar tài khoản trong drawer |
| `TokenManager.java` | Util | Đọc/ghi token + username (SharedPreferences) |
| `SessionManager.java` | Util | `markExpired()` → xoá token, bật Login (khi 401/403) |
| `AccountStore.java` | Util | Lưu nhiều tài khoản `{username,token,avatar}` |
| `LoginRequest/RegisterRequest/JwtResponse/User/UserMe/UserProfile/ProfileUpdateRequest` | Model | Body/kết quả auth + hồ sơ |

### 🔧 NỀN TẢNG DÙNG CHUNG (bạn quản lý — cả nhóm xài)
| File | Loại | Chức năng |
|------|------|-----------|
| `network/ApiService.java` | Interface | **Khai báo toàn bộ ~60 endpoint** |
| `network/RetrofitClient.java` | Singleton | Tạo Retrofit, `BASE_URL=10.0.2.2`, gắn interceptor, `GSON` chung |
| `network/AuthInterceptor.java` | Interceptor | Đính token vào request; 401/403 → `markExpired()` |
| `viewmodel/VmFactory.java` | Factory | Tạo mọi ViewModel + nhồi Repository |
| `data/RepoCallback.java` | Interface | Callback `onSuccess/onError` |
| `data/Resource.java` | Wrapper | Trạng thái Loading/Success/Error |
| `data/Event.java` | Wrapper | Sự kiện 1 lần (snackbar/điều hướng) |
| `util/NavHelper.java` | Util | Mở màn chi tiết (`openAlbum/openArtist/openPlaylist/openLiked`) — trong Main mở Fragment, ngoài Main relaunch Main |
| `fragment/BaseDetailFragment.java` | Fragment cha | Hoãn việc nặng tới sau animation (Album/Artist/Playlist kế thừa) |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/AuthController.java` | Controller | `/api/auth/register, login, refresh, logout` |
| `service/AuthService.java` | Service | Xác thực, sinh JWT, tạo user+settings khi đăng ký |
| `security/JwtUtil.java` | Security | Sinh & kiểm tra access/refresh token (HMAC, hạn) |
| `security/JwtFilter.java` | Security | Lọc mọi request, đọc Bearer → set "đã đăng nhập" |
| `security/UserDetailsServiceImpl.java` | Security | Nạp `User` từ DB cho Spring Security |
| `config/SecurityConfig.java` | Config | Endpoint nào public/cần auth, CORS, BCrypt |
| `config/DataSeeder.java` | Config | Seed dữ liệu mẫu khi khởi động |
| `OnlineMusicStreamingAppApplication.java` | Main | Hàm `main`, khởi động Spring Boot |
| `controller/UserController.java` | Controller | `/api/users/me`, `/me/profile`, `/me/avatar` (hồ sơ) |
| `service/UserService.java` | Service | Logic hồ sơ user |
| `entity/User.java` | Entity | Bảng `Users` |
| `repository/UserRepository.java` | Repository | `findByUsername`, `existsByEmail`… |
| `dto/LoginRequest, RegisterRequest, JwtResponse, RefreshTokenRequest, UserMeDto, UserProfileDto, ProfileUpdateRequest` | DTO | Auth + hồ sơ |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable / anim dùng |
|--------------|----------------------|
| `activity_splash/login/register` | `ic_music_note` |
| `activity_add_account` | `ic_close`, `bg_circle_dark`, `ic_music_note` |
| `activity_main` (MainActivity.java) | `ic_play`/`ic_pause`, `ic_avatar_placeholder`, `placeholder_gradient` · **anim:** `fade_in/out` (đổi tab), `slide_in/out_*` (mở chi tiết), `mini_player_slide_up/down` |
| `drawer_content` | `ic_avatar_placeholder`, `ic_add`, `ic_chart`, `ic_clock`, `ic_megaphone`, `ic_settings` |
| `item_drawer_account` / `item_drawer_add_account` | `ic_avatar_placeholder` / `bg_circle_dark`, `ic_add` |
| `layout_mini_player` | `bg_mini_player`, `mini_player_progress`, `ic_favorite`, `ic_play`, `ic_skip_next` (khung của bạn, logic phát Người 3) |
| `activity_profile` | `ic_arrow_back`, `ic_avatar_placeholder`, `ic_share`, `ic_more_vert` |
| `activity_edit_profile` | `ic_close`, `ic_avatar_placeholder` |

## 4. ⭐ THỨ TỰ ĐỌC FILE (hiểu luồng từ bật app → đăng nhập → khung)

1. `AndroidManifest.xml` → thấy `SplashActivity` là launcher, `MusicApp` là Application.
2. `MusicApp.java` → khởi tạo toàn cục (AdMob chạy nền, lifecycle callbacks).
3. `SplashActivity.java` → `util/TokenManager` (có token chưa?) → rẽ Login/Main.
4. **Luồng đăng nhập:** `LoginActivity` → `LoginViewModel` → `data/repository/AuthRepository` → `network/ApiService` (hàm `login`) → `RetrofitClient` + `AuthInterceptor`.
5. **BE đăng nhập:** `AuthController.login` → `AuthService` → `security/JwtUtil` → (mọi request sau qua) `JwtFilter` → `SecurityConfig` → `entity/User` + `UserRepository`.
6. **Khung app:** `MainActivity` (đọc kỹ: `setupBottomNav` / `setupDrawer` / `setupMiniPlayer` / `openDetail` / `handleDetailIntent`) → `MainViewModel` → `util/{AccountStore, AccountAdapter, SessionManager}`.
7. **Hồ sơ:** `ProfileActivity`/`EditProfileActivity` → `ProfileViewModel`/`EditProfileViewModel` → `data/repository/UserRepository` → BE `UserController` → `UserService` → `entity/User`.
8. **Hạ tầng chung (đọc cuối, tra khi cần):** `VmFactory` → `data/{Resource, Event, RepoCallback}` → `util/NavHelper` → `fragment/BaseDetailFragment`.

## 5. ENDPOINT · CHECKLIST · GIAO
- **Endpoint:** `POST /api/auth/{register,login,refresh,logout}` · `GET /api/users/me`, `PUT /me/profile`, `POST /me/avatar` (drawer + hồ sơ).
- **Khung 4 tab + chi tiết trong 1 Activity:** đổi tab → `popBackStack` + `replace`; mở chi tiết → `openDetail()` dùng `hide+add+addToBackStack` để bottom-nav/mini-player đứng yên.
- **Giao với người khác:** cả nhóm xài `ApiService/Retrofit/VmFactory/NavHelper/BaseDetailFragment` của bạn · mini-player khung của bạn, logic phát Người 3 · nút "Tạo" mở `CreateBottomSheet` (Người 4) · banner AdMob đặt trong `activity_main` nhưng điều khiển bởi `AdManager` (Người 5) · màn Hồ sơ bấm playlist → `NavHelper.openPlaylist` (chi tiết playlist của Người 4).
