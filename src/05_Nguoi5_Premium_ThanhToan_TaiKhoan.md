# NGƯỜI 5 — Premium, Thanh toán, Hồ sơ, Cài đặt & Thống kê

> **Miền dữ liệu:** `Subscription`, `UserSettings`. **Vai trò:** gói **Premium** + **thanh toán VNPay** (WebView), khóa tính năng (**gate**) + **quảng cáo AdMob** cho user Free, và các màn cá nhân: **Hồ sơ, Sửa hồ sơ, Cài đặt, Thống kê nghe**. Luồng tích hợp ngoài phức tạp nhất.

```
Tab Premium → PremiumPlansActivity → subscribe → POST /api/subscriptions/subscribe (tạo PENDING + payUrl)
   → PaymentActivity (WebView VNPay) → thanh toán → BE kích hoạt Premium → deep-link musicapp://payment/result
Free chạm tính năng khóa → Gate sheet → "Khám phá" → PremiumPlansActivity
Drawer → Hồ sơ / Cài đặt / Thống kê
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `fragment/PremiumFragment.java` | Fragment | Tab Premium: quyền lợi + nút nâng cấp |
| `PremiumPlansActivity.java` | Activity | Danh sách gói (Cá nhân/Sinh viên/Gia đình) để mua |
| `PaymentActivity.java` | Activity ⭐ | WebView VNPay; bắt deep-link kết quả + rewrite localhost→10.0.2.2 |
| `fragment/PremiumGateBottomSheet.java` | BottomSheet | Gate chung |
| `fragment/ShuffleGateBottomSheet.java` | BottomSheet | Gate khi Free chạm nút trộn bài |
| `fragment/DownloadGateBottomSheet.java` | BottomSheet | Gate khi Free chạm nút tải xuống |
| `ProfileActivity.java` | Activity | Xem hồ sơ |
| `EditProfileActivity.java` | Activity | Sửa tên + đổi avatar (upload) |
| `SettingsActivity.java` | Activity | Công tắc cài đặt + Đăng xuất |
| `ListeningStatsActivity.java` | Activity | Thống kê: tổng thời lượng, top bài/nghệ sĩ |
| `SubscriptionViewModel.java` | ViewModel | Tải gói + subscription hiện tại; `subscribe/cancel` |
| `ProfileViewModel`/`EditProfileViewModel`/`SettingsViewModel`/`ListeningStatsViewModel` | ViewModel | 4 màn cá nhân |
| `SubscriptionRepository.java` | Repository | API subscription + payment |
| `data/repository/UserRepository.java` | Repository | `getMe/getMyProfile/updateProfile/uploadAvatar/settings` (chung Người 1) |
| `util/PremiumChecker.java` | Util ⭐ | `isPremium(sub)` — quyết định ẩn QC, mở khóa |
| `util/AdManager.java` | Util | Nạp banner + interstitial; `setPremium(true)`→tắt |
| `Subscription`/`PlanInfo`/`SubscribeRequest`/`SubscribeResponse` | Model | Gói & thanh toán |
| `UserProfile`/`ProfileUpdateRequest`/`UserSettings`/`ListeningStats` | Model | Hồ sơ/cài đặt/thống kê |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/SubscriptionController.java` | Controller | `/api/subscriptions/me\|plans\|subscribe\|cancel` |
| `service/SubscriptionService.java` | Service | Tạo/hủy gói, kiểm tra đang active |
| `service/SubscriptionAlreadyActiveException.java` | Exception | Chặn mua khi đang có gói |
| `controller/PaymentController.java` | Controller | `POST /api/payment/create`, `GET /api/payment/vnpay-return` |
| `service/VnpayService.java` | Service ⭐ | Sinh URL VNPay (HMAC-SHA512) + verify callback |
| `service/PremiumGuard.java` | Service | Chặn tính năng Premium ở BE |
| `controller/UserController.java` | Controller | `/api/users/me`, `/me/profile`, `/me/avatar` |
| `service/UserService.java` | Service | Logic hồ sơ |
| `controller/UserSettingsController.java` | Controller | `GET/PUT /api/users/me/settings` |
| `service/UserSettingsService.java` | Service | Đọc/ghi cài đặt |
| `controller/StatsController.java` | Controller | `GET /api/stats/listening?period=` |
| `service/ListeningStatsService.java` | Service | Tính top bài/nghệ sĩ/tổng từ `PlayHistory` |
| `controller/AdminController.java` | Controller | `/api/admin/**` CRUD (admin) — tham khảo, có thể đồng sở hữu |
| `entity/Subscription.java` | Entity | Bảng `Subscriptions` |
| `entity/SubscriptionPlan.java` | Enum | FREE/INDIVIDUAL/STUDENT/FAMILY |
| `entity/UserSettings.java` | Entity | Cài đặt user |
| `entity/StreamQuality.java` | Enum | Chất lượng nhạc |
| `repository/SubscriptionRepository, UserSettingsRepository` | Repository | Truy vấn |
| `dto/PlanInfoDto, SubscribeRequest, UserMeDto, UserProfileDto, ProfileUpdateRequest, UserSettingsDto, ListeningStatsDto, TopTrackDto, TopArtistDto` | DTO | Gói/hồ sơ/cài đặt/thống kê |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_premium` | `bg_card_pink`, `ic_premium`, `bg_reasons_card`, `bg_btn_white_pill` (Java: `ic_no_ads/download/shuffle/audio_quality/friends/queue_add`) |
| `activity_premium_plans` | `ic_arrow_back`, `bg_premium_individual/student/family`, `bg_pill_pink/purple/green`, `ic_premium`, `bg_btn_pill_pink/purple/green` |
| `activity_payment` | (chỉ WebView, không icon) |
| `bottom_sheet_premium_gate` | `ic_premium`, `bg_btn_white_pill` |
| `bottom_sheet_shuffle_gate` | `ic_premium`, `ic_shuffle`, `ic_queue`, `bg_btn_pill_green` |
| `bottom_sheet_download_gate` | `ic_premium`, `placeholder_gradient`, `bg_btn_pill_green` |
| `activity_profile` | `ic_arrow_back`, `ic_avatar_placeholder`, `ic_share`, `ic_more_vert` |
| `activity_edit_profile` | `ic_close`, `ic_avatar_placeholder` |
| `activity_settings` | `ic_arrow_back` |
| `activity_listening_stats` | `ic_arrow_back`, `ic_share` |
| `item_premium_reason` / `item_stats_section` | `ic_no_ads` / `bg_card_dark`, `ic_avatar_placeholder`, `placeholder_gradient` |

> Banner/interstitial AdMob hiển thị ở `MainActivity` (khung Người 1) — bạn điều khiển qua `AdManager`.

## 4. LUỒNG THANH TOÁN VNPAY (điểm nhấn demo)
1. Chọn gói → `SubscriptionViewModel.subscribe()` → `POST /api/subscriptions/subscribe` → BE tạo `Subscription` **PENDING** + `VnpayService.createPaymentUrl` (ký HMAC-SHA512) trả `payUrl`.
2. FE mở `PaymentActivity` (WebView) `loadUrl(payUrl)`.
3. User thanh toán → VNPay **redirect** `GET /api/payment/vnpay-return` → `VnpayService.verifyCallback` (ký lại, so hash, mã `00`) → **kích hoạt Premium** → redirect deep-link `musicapp://payment/result?status=success`.
4. `PaymentActivity` bắt deep-link → trả status → `loadCurrentSubscription()` → `PremiumChecker.isPremium`=true → ẩn QC, mở khóa.

> ⚠️ **Gotcha:** `vnp_ReturnUrl` là `localhost` → trên máy ảo phải rewrite host sang `10.0.2.2` (`PaymentActivity.rewriteLocalhost`), nếu không WebView lỗi `ERR_CONNECTION_REFUSED`.

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/subscriptions/*`, `/api/payment/{create,vnpay-return}`, `/api/users/me{,/profile,/avatar,/settings}`, `/api/stats/listening`.
- **Giao:** cung cấp 3 **gate sheet** + `PremiumChecker` cho Người 3 (Player) & Người 4 (Album/Playlist) gọi · banner+interstitial gắn vào `MainActivity` (Người 1) + `PlayerManager` (Người 3) · `UserRepository` & endpoint `/api/users/me` dùng chung Người 1 (drawer) · `ListeningStatsService` đọc `PlayHistory` (Người 3).
