# NGƯỜI 1 — Đăng nhập, Tài khoản & Khung ứng dụng

> **Miền dữ liệu:** `User` + bảo mật. **Vai trò:** từ lúc bật app → đăng nhập/đăng ký → vào `MainActivity` (bottom-nav + drawer + đa tài khoản). Sở hữu **nền tảng mạng dùng chung** cả nhóm xài.

```
SplashActivity ─(có token?)─► MainActivity / LoginActivity ─► (login OK, lưu token) ─► MainActivity
MainActivity = bottom-nav (5 tab) + fragment_container + mini-player + adBanner + Drawer
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng (1 dòng) |
|------|------|--------------------|
| `SplashActivity.java` | Activity | Màn chờ; có token → Main, không → Login |
| `LoginActivity.java` | Activity | Nhập user/pass, gọi đăng nhập |
| `RegisterActivity.java` | Activity | Đăng ký tài khoản mới |
| `AddAccountActivity.java` | Activity | Đăng nhập thêm 1 tài khoản (đa tài khoản) |
| `MainActivity.java` | Activity | **Hub**: bottom-nav, fragment_container, mini-player, drawer, banner |
| `MusicApp.java` | Application | Chạy đầu tiên; khởi tạo AdMob/ExoPlayer toàn cục |
| `LoginViewModel.java` | ViewModel | State + sự kiện đăng nhập |
| `RegisterViewModel.java` | ViewModel | State + sự kiện đăng ký |
| `MainViewModel.java` | ViewModel | Thông tin user cho drawer + bài đang phát (mini-player) |
| `AuthRepository.java` | Repository | Gọi `login/register`, **lưu token** sau khi OK |
| `AccountAdapter.java` | Adapter | Danh sách avatar tài khoản trong drawer |
| `TokenManager.java` | Util | Đọc/ghi token + username (SharedPreferences) |
| `SessionManager.java` | Util | `markExpired()` → xoá token, bật Login (khi 401/403) |
| `AccountStore.java` | Util | Lưu nhiều tài khoản `{username,token,avatar}` |
| `BottomNavHelper.java` | Util | Tiện ích cho bottom-nav |
| `LoginRequest/RegisterRequest/JwtResponse/User/UserMe` | Model | Body/kết quả auth + user |

### 🔧 NỀN TẢNG DÙNG CHUNG (bạn quản lý — cả nhóm xài)
| File | Loại | Chức năng |
|------|------|-----------|
| `network/ApiService.java` | Interface | **Khai báo toàn bộ ~60 endpoint** |
| `network/RetrofitClient.java` | Singleton | Tạo Retrofit, `BASE_URL=10.0.2.2`, gắn interceptor |
| `network/AuthInterceptor.java` | Interceptor | Đính token vào request; 401/403 → `markExpired()` |
| `viewmodel/VmFactory.java` | Factory | Tạo mọi ViewModel + nhồi Repository |
| `data/RepoCallback.java` | Interface | Callback `onSuccess/onError` |
| `data/Resource.java` | Wrapper | Trạng thái Loading/Success/Error |
| `data/Event.java` | Wrapper | Sự kiện 1 lần (snackbar/điều hướng) |
| `util/NavHelper.java` | Util | Mở màn chi tiết (`openAlbum/openArtist/...`) |

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
| `entity/User.java` | Entity | Bảng `Users` |
| `repository/UserRepository.java` | Repository | `findByUsername`, `existsByEmail`… |
| `dto/LoginRequest, RegisterRequest, JwtResponse, RefreshTokenRequest` | DTO | Body/kết quả auth |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable / anim dùng |
|--------------|----------------------|
| `activity_splash/login/register` | `ic_music_note` |
| `activity_add_account` | `ic_close`, `bg_circle_dark`, `ic_music_note` |
| `activity_main` (MainActivity.java) | `ic_play`/`ic_pause`, `ic_avatar_placeholder`, `placeholder_gradient` · **anim:** `fade_in/out` (đổi tab), `slide_in/out_*` (mở chi tiết), `mini_player_slide_up/down` |
| `drawer_content` | `ic_avatar_placeholder`, `ic_add`, `ic_chart`, `ic_clock`, `ic_megaphone`, `ic_settings` |
| `item_drawer_account` / `item_drawer_add_account` | `ic_avatar_placeholder` / `bg_circle_dark`, `ic_add` |
| `layout_mini_player` ⚠️ | `bg_mini_player`, `mini_player_progress`, `ic_favorite`, `ic_play`, `ic_skip_next` (khung của bạn, logic phát của Người 3) |

## 4. LUỒNG ĐĂNG NHẬP (mẫu — các luồng khác tương tự)
1. `LoginActivity` → `vm.login(user,pass)` → `AuthRepository.login()` → `api.login()` (POST).
2. BE `AuthController.login` → `AuthService.login`: `authenticationManager.authenticate` (so bcrypt) → `JwtUtil.generateToken` → trả `JwtResponse`. Sai → 401.
3. FE: `AuthRepository` lưu token (`TokenManager`+`AccountStore`) → `onSuccess` → mở `MainActivity`.

**Khung 4 tab + chi tiết trong 1 Activity:** đổi tab → `popBackStack` + `replace`; mở chi tiết → `openDetail()` dùng `hide+add+addToBackStack` để bottom-nav/mini-player đứng yên.

## 5. ENDPOINT · CHECKLIST · GIAO
- **Endpoint:** `POST /api/auth/{register,login,refresh,logout}` · `GET /api/users/me` (drawer — endpoint do Người 5 viết).
- **Checklist:** xem 2 bảng FILE ở trên (FE + nền tảng chung + BE).
- **Giao với người khác:** cả nhóm xài `ApiService/Retrofit/VmFactory/NavHelper` của bạn · mini-player khung của bạn, logic phát Người 3 · drawer gọi `GET /api/users/me` (Người 5) · nút "Tạo" mở `CreateBottomSheet` (Người 4).
