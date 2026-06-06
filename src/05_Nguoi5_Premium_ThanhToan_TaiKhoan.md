# NGƯỜI 5 — Premium, Thanh toán, Hồ sơ, Cài đặt & Thống kê

> **Bạn lo "tiền & tài khoản cá nhân".** Mảng của bạn: gói **Premium** + **thanh toán VNPay** (WebView), khóa tính năng (**gate**) + **quảng cáo** cho user Free, và các màn cá nhân: **Hồ sơ, Sửa hồ sơ, Cài đặt, Thống kê nghe**. Đây là mảng có luồng **phức tạp nhất về tích hợp ngoài** (cổng thanh toán + AdMob).

---

## 0. NỀN TẢNG DÙNG CHUNG (đọc nhanh — chi tiết ở file 01)
Khuôn MVVM như cả nhóm. Bạn dùng `SubscriptionRepository` + `UserRepository`. Khái niệm trung tâm: **trạng thái Premium** — quyết định ẩn quảng cáo, mở khóa shuffle/tải xuống. Đọc bằng `util/PremiumChecker.isPremium(subscription)`.

---

## 1. SƠ ĐỒ LUỒNG MẢNG CỦA BẠN

```
Tab Premium ─► PremiumFragment ─► hiện các gói (Free/Cá nhân/Sinh viên/Gia đình)
     │ bấm "Nâng cấp"
     ▼
PremiumPlansActivity ─► chọn gói ─► SubscriptionViewModel.subscribe()
     │ POST /api/subscriptions/subscribe → tạo subscription PENDING + trả payUrl VNPay
     ▼
PaymentActivity (WebView mở payUrl VNPay sandbox)
     │ user thanh toán xong → VNPay redirect về backend → backend kích hoạt Premium
     │ → redirect deep-link  musicapp://payment/result?status=success
     ▼
quay lại app, trạng thái = Premium → ẩn quảng cáo, mở khóa tính năng

User Free chạm tính năng khóa → Gate bottom sheet → "Khám phá" → PremiumPlansActivity
   ├─ PremiumGateBottomSheet  (chung)
   ├─ ShuffleGateBottomSheet  (nút trộn bài ở Player/Album/Playlist)
   └─ DownloadGateBottomSheet (nút tải xuống ở Album/Playlist)

Drawer ─► Hồ sơ / Sửa hồ sơ / Cài đặt / Thống kê nghe
```

---

## 2. CHI TIẾT TỪNG CHỨC NĂNG

### 2.1. TAB PREMIUM & DANH SÁCH GÓI

**File cần đọc (FE):**
- `fragment/PremiumFragment.java` + `fragment_premium.xml` — tab Premium: giới thiệu quyền lợi + nút nâng cấp.
- `PremiumPlansActivity.java` + `activity_premium_plans.xml` — danh sách gói chi tiết để chọn mua.
- `viewmodel/SubscriptionViewModel.java` — **đọc kỹ**. Tải gói (`getPlans`), tải subscription hiện tại (`getMySubscription`), `subscribe(plan)`, `cancel()`.
- `data/repository/SubscriptionRepository.java` — gọi API subscription + payment.
- `util/PremiumChecker.java` — hàm tĩnh `isPremium(sub)` (đang ACTIVE & chưa hết hạn?).
- `adapter` gói + `item_premium_reason.xml` (lý do nên mua).
- models: `Subscription`, `PlanInfo`, `SubscribeRequest`, `SubscribeResponse`.

**File cần đọc (BE):**
- `controller/SubscriptionController.java` — `GET /api/subscriptions/me`, `/plans`, `POST /subscribe`, `/cancel`.
- `service/SubscriptionService.java` — tạo/hủy subscription, kiểm tra đang active.
- `service/SubscriptionAlreadyActiveException.java` — chặn mua khi đang có gói.
- `service/PremiumGuard.java` — chặn tính năng Premium ở backend (nếu có endpoint cần Premium).
- `entity/Subscription.java`, `entity/SubscriptionPlan.java` (enum FREE/INDIVIDUAL/STUDENT/FAMILY), `repository/SubscriptionRepository.java`.
- `dto/PlanInfoDto.java`, `dto/SubscribeRequest.java`.

### 2.2. THANH TOÁN VNPAY (luồng phức tạp nhất — đọc kỹ)

**File cần đọc (FE):**
- `PaymentActivity.java` — **đọc rất kỹ**. WebView mở trang VNPay. Bắt deep-link kết quả + **rewrite host localhost → 10.0.2.2** (nếu không sẽ `ERR_CONNECTION_REFUSED`).
- `res/layout/activity_payment.xml` — WebView + progress.

**File cần đọc (BE):**
- `controller/PaymentController.java` — `POST /api/payment/create` (tạo URL thanh toán), `GET /api/payment/vnpay-return` (VNPay gọi về sau khi thanh toán).
- `service/VnpayService.java` — **đọc kỹ**. Sinh URL thanh toán có chữ ký HMAC-SHA512 (`createPaymentUrl`) và kiểm tra chữ ký khi VNPay callback (`verifyCallback`).

**Luồng thanh toán (học thuộc — đây là điểm nhấn demo):**
1. User chọn gói → `SubscriptionViewModel.subscribe()` → `POST /api/subscriptions/subscribe` → BE tạo `Subscription` trạng thái **PENDING**, rồi `VnpayService.createPaymentUrl(subscriptionId, amount, ip)` sinh URL VNPay (kèm `vnp_SecureHash`).
2. FE nhận `payUrl` → mở `PaymentActivity` (WebView) → `loadUrl(payUrl)`.
3. User nhập thẻ test trên VNPay sandbox → thanh toán.
4. VNPay **redirect 302** về `vnp_ReturnUrl` = `GET /api/payment/vnpay-return?...` (kèm mã kết quả + chữ ký).
5. BE `PaymentController.vnpayReturn()` → `VnpayService.verifyCallback(params)`:
   - Dựng lại chuỗi hash từ tham số, ký lại HMAC-SHA512, so với `vnp_SecureHash`.
   - Khớp + `vnp_ResponseCode == "00"` → **kích hoạt Premium** (đổi Subscription sang ACTIVE).
   - Rồi redirect về deep-link `musicapp://payment/result?status=success` (hoặc `failed`).
6. `PaymentActivity` bắt deep-link đó trong `shouldOverrideUrlLoading` → `setResult(status)` → `finish()` → về `PremiumPlansActivity`.
7. App `loadCurrentSubscription()` lại → `PremiumChecker.isPremium` = true → ẩn quảng cáo, mở khóa.

> ⚠️ **Gotcha quan trọng (đã ghi trong memory dự án):** backend cấu hình `vnp_ReturnUrl` là `localhost` → trên máy ảo "localhost" trỏ về chính máy ảo → WebView lỗi kết nối. `PaymentActivity.rewriteLocalhost()` đổi host sang `10.0.2.2` (host trong `RetrofitClient.BASE_URL`). Nhớ điều này khi demo.

### 2.3. KHÓA TÍNH NĂNG (Gate) cho user Free

**File cần đọc (FE):**
- `fragment/PremiumGateBottomSheet.java` + `bottom_sheet_premium_gate.xml` — gate chung.
- `fragment/ShuffleGateBottomSheet.java` + `bottom_sheet_shuffle_gate.xml` — bật khi Free chạm nút trộn bài (ở Player/Album/Playlist của Người 3/4).
- `fragment/DownloadGateBottomSheet.java` + `bottom_sheet_download_gate.xml` — bật khi Free chạm nút tải xuống (Album/Playlist).

**Luồng:** màn của Người 3/4 kiểm tra `isPremium`; nếu Free → `new ShuffleGateBottomSheet().show(...)`. Trong sheet, nút "Khám phá" → mở `PremiumPlansActivity` (màn của bạn). **Bạn cung cấp các sheet này, người khác chỉ gọi.**

### 2.4. QUẢNG CÁO (AdMob) cho user Free

**File cần đọc (FE):**
- `util/AdManager.java` — **đọc kỹ**. Nạp banner + quảng cáo xen kẽ (interstitial). `setPremium(true)` → tắt quảng cáo.
- Banner hiển thị ở `MainActivity` (khung của Người 1): `renderAdsForSubscription()` — Premium ẩn, Free hiện.
- Interstitial: thường bật sau vài lần đổi bài (qua `PlayerManager.setAdListener` của Người 3).

**Luồng:** `MainActivity` quan sát subscription → `AdManager.setPremium(isPremium)` → quyết định hiện/ẩn quảng cáo. Đây là lý do quảng cáo gắn chặt với trạng thái Premium của bạn.

### 2.5. HỒ SƠ & SỬA HỒ SƠ

**File cần đọc (FE):**
- `ProfileActivity.java` + `activity_profile.xml` — xem hồ sơ (avatar, tên hiển thị, số playlist/đang theo dõi...).
- `EditProfileActivity.java` + `activity_edit_profile.xml` — sửa tên hiển thị + đổi avatar (upload ảnh).
- `viewmodel/ProfileViewModel.java`, `viewmodel/EditProfileViewModel.java`.
- `data/repository/UserRepository.java` — `getMe`, `getMyProfile`, `updateProfile`, `uploadAvatar` (dùng chung với drawer của Người 1).
- models: `UserProfile`, `ProfileUpdateRequest`, `UserMe`.

**File cần đọc (BE):**
- `controller/UserController.java` + `service/UserService.java` — `GET /api/users/me`, `/me/profile`, `PUT /me/profile`, `POST /me/avatar`.
- `dto/UserMeDto.java`, `dto/UserProfileDto.java`, `dto/ProfileUpdateRequest.java`.

**Luồng sửa hồ sơ:** nhập tên/chọn ảnh → `EditProfileViewModel` → `UserRepository.updateProfile()` (+ `uploadAvatar` multipart) → `PUT /api/users/me/profile` → về cập nhật lại drawer/Profile.

### 2.6. CÀI ĐẶT

**File cần đọc (FE):**
- `SettingsActivity.java` + `activity_settings.xml` — các công tắc (phiên riêng tư, chất lượng nhạc...), nút Đăng xuất.
- `viewmodel/SettingsViewModel.java` — đọc/ghi settings (thường debounce khi đổi công tắc).
- `data/repository/UserRepository.java` (phần settings) + `model/UserSettings.java`.

**File cần đọc (BE):**
- `controller/UserSettingsController.java` + `service/UserSettingsService.java` — `GET/PUT /api/users/me/settings`.
- `entity/UserSettings.java`, `entity/StreamQuality.java`, `repository/UserSettingsRepository.java`, `dto/UserSettingsDto.java`.

**Luồng:** mở → `GET /api/users/me/settings` → đổ vào công tắc. Đổi công tắc → `vm.onSwitchChanged()` → (debounce) → `PUT /api/users/me/settings`. Nút Đăng xuất → xóa token (qua `SessionManager`/`TokenManager` của Người 1) → về Login.

### 2.7. THỐNG KÊ NGHE

**File cần đọc (FE):**
- `ListeningStatsActivity.java` + `activity_listening_stats.xml` — biểu đồ/danh sách: tổng thời lượng, top bài, top nghệ sĩ theo tuần/tháng/năm.
- `viewmodel/ListeningStatsViewModel.java` — `GET /api/stats/listening?period=&offset=`.
- `adapter` + `item_stats_section.xml`.
- `model/ListeningStats.java`.

**File cần đọc (BE):**
- `controller/StatsController.java` + `service/ListeningStatsService.java` — tính từ bảng `PlayHistory` (của Người 3).
- `dto/ListeningStatsDto.java`, `dto/TopTrackDto.java`, `dto/TopArtistDto.java`.

**Luồng:** chọn kỳ (tuần/tháng/năm) → `ListeningStatsViewModel` → `GET /api/stats/listening` → `ListeningStatsService` gom `PlayHistory` theo thời gian → trả top + tổng → render.

---

## 3. ✅ CHECKLIST "FILE CỦA TÔI" (Người 5)

**Android — màn hình & sheet & layout:**
- [ ] `PremiumFragment` + `fragment_premium.xml`
- [ ] `PremiumPlansActivity` + `activity_premium_plans.xml`
- [ ] `PaymentActivity` + `activity_payment.xml`
- [ ] `PremiumGateBottomSheet`/`ShuffleGateBottomSheet`/`DownloadGateBottomSheet` + 3 layout `bottom_sheet_*_gate.xml`
- [ ] `ProfileActivity` + `activity_profile.xml`
- [ ] `EditProfileActivity` + `activity_edit_profile.xml`
- [ ] `SettingsActivity` + `activity_settings.xml`
- [ ] `ListeningStatsActivity` + `activity_listening_stats.xml` (+ `item_stats_section.xml`, `item_premium_reason.xml`)

**Android — ViewModel / Repository / Util:**
- [ ] `SubscriptionViewModel`, `ProfileViewModel`, `EditProfileViewModel`, `SettingsViewModel`, `ListeningStatsViewModel`
- [ ] `SubscriptionRepository` (+ dùng chung `UserRepository`)
- [ ] `util/PremiumChecker`, `util/AdManager`
- [ ] models: `Subscription`, `PlanInfo`, `SubscribeRequest`, `SubscribeResponse`, `UserProfile`, `ProfileUpdateRequest`, `UserSettings`, `ListeningStats`

**Backend:**
- [ ] `controller/SubscriptionController`, `service/SubscriptionService`, `service/SubscriptionAlreadyActiveException`, `service/PremiumGuard`
- [ ] `controller/PaymentController`, `service/VnpayService`
- [ ] `entity/Subscription`, `entity/SubscriptionPlan`, `repository/SubscriptionRepository`
- [ ] `controller/UserController`, `service/UserService`
- [ ] `controller/UserSettingsController`, `service/UserSettingsService`, `entity/UserSettings`, `entity/StreamQuality`, `repository/UserSettingsRepository`
- [ ] `controller/StatsController`, `service/ListeningStatsService`
- [ ] dto: `PlanInfoDto`, `SubscribeRequest`, `UserMeDto`, `UserProfileDto`, `ProfileUpdateRequest`, `UserSettingsDto`, `ListeningStatsDto`, `TopTrackDto`, `TopArtistDto`

---

## 4. ENDPOINT API MẢNG BẠN DÙNG
- `GET /api/subscriptions/me`, `/plans`, `POST /api/subscriptions/subscribe`, `/cancel`
- `POST /api/payment/create`, `GET /api/payment/vnpay-return` (VNPay gọi về)
- `GET /api/users/me`, `/me/profile`, `PUT /me/profile`, `POST /me/avatar`
- `GET /api/users/me/settings`, `PUT /api/users/me/settings`
- `GET /api/stats/listening?period=week|month|year&offset=`

## 5. ĐIỂM GIAO VỚI NGƯỜI KHÁC
- Bạn cung cấp 3 **gate sheet** + `PremiumChecker` cho Người 3 (Player/Album) và Người 4 (Playlist) gọi khi user Free chạm tính năng khóa.
- `AdManager` + banner hiển thị trong **MainActivity** (khung Người 1); interstitial gắn vào `PlayerManager` (Người 3).
- `UserRepository` dùng chung với Người 1 (drawer header).
- `ListeningStatsService` đọc bảng `PlayHistory` (Người 3) → khi họ đổi cách ghi lịch sử, thống kê bị ảnh hưởng.
- Khi mua Premium xong, app gọi lại `loadCurrentSubscription()` ở MainActivity (Người 1) để ẩn quảng cáo.
