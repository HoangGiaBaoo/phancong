# NGƯỜI 5 — Premium, Thanh toán, Quảng cáo, Cài đặt & Thống kê

> **Miền dữ liệu:** `Subscription`, `UserSettings`. **Vai trò:** gói **Premium** + **thanh toán VNPay** (WebView), khóa tính năng (**gate**) + **quảng cáo AdMob** cho user Free, và các màn cá nhân: **Cài đặt, Thống kê nghe**. Luồng tích hợp ngoài phức tạp nhất.

```
Tab Premium → PremiumPlansActivity → subscribe → POST /api/subscriptions/subscribe (PENDING + payUrl)
   → PaymentActivity (WebView VNPay) → thanh toán → BE kích hoạt Premium → deep-link musicapp://payment/result
Free chạm tính năng khóa → Gate sheet → "Khám phá" → PremiumPlansActivity
Drawer → Cài đặt / Thống kê
```

---

## 1. BẢNG FILE ANDROID (FE)

| File | Loại | Chức năng |
|------|------|-----------|
| `fragment/PremiumFragment.java` | Fragment | Tab Premium: quyền lợi + nút nâng cấp |
| `PremiumPlansActivity.java` | Activity | Danh sách gói (Cá nhân/Sinh viên/Gia đình) để mua |
| `PaymentActivity.java` ⭐ | Activity | WebView VNPay; bắt deep-link kết quả + rewrite localhost→10.0.2.2 |
| `fragment/PremiumGateBottomSheet.java` | BottomSheet | Gate chung |
| `fragment/ShuffleGateBottomSheet.java` | BottomSheet | Gate khi Free chạm nút trộn bài (gọi từ `ShuffleController` của Người 3) |
| `fragment/DownloadGateBottomSheet.java` | BottomSheet | Gate khi Free chạm nút tải xuống |
| `SettingsActivity.java` | Activity | Công tắc cài đặt + Đăng xuất |
| `ListeningStatsActivity.java` | Activity | Thống kê: tổng thời lượng, top bài/nghệ sĩ |
| `SubscriptionViewModel.java` | ViewModel | Tải gói + subscription hiện tại; `subscribe/cancel` |
| `SettingsViewModel`/`ListeningStatsViewModel` | ViewModel | 2 màn cá nhân |
| `SubscriptionRepository.java` | Repository | API subscription + payment |
| `data/repository/UserRepository.java` | Repository | (của Người 1) — bạn chỉ dùng `settings()` cho màn Cài đặt |
| `util/PremiumChecker.java` ⭐ | Util | `isPremium(sub)` — quyết định ẩn QC, mở khóa |
| `util/AdManager.java` ⭐ | Util | Nạp banner + interstitial; đếm 3 bài/lần; `setPremium(true)`→tắt |
| `Subscription`/`PlanInfo`/`SubscribeRequest`/`SubscribeResponse` | Model | Gói & thanh toán |
| `UserSettings`/`ListeningStats` | Model | Cài đặt/thống kê |

## 2. BẢNG FILE BACKEND (BE)

| File | Loại | Chức năng |
|------|------|-----------|
| `controller/SubscriptionController.java` / `service/SubscriptionService.java` | Controller+Service | `/api/subscriptions/me\|plans\|subscribe\|cancel`; tạo/hủy gói |
| `service/SubscriptionAlreadyActiveException.java` | Exception | Chặn mua khi đang có gói |
| `controller/PaymentController.java` | Controller | `POST /api/payment/create`, `GET /api/payment/vnpay-return` |
| `service/VnpayService.java` ⭐ | Service | Sinh URL VNPay (HMAC-SHA512) + verify callback |
| `service/PremiumGuard.java` | Service | Chặn tính năng Premium ở BE |
| `controller/UserSettingsController.java` / `service/UserSettingsService.java` | Controller+Service | `GET/PUT /api/users/me/settings` |
| `controller/StatsController.java` / `service/ListeningStatsService.java` | Controller+Service | `GET /api/stats/listening?period=`; tính top từ `PlayHistory` |
| `controller/AdminController.java` | Controller | `/api/admin/**` CRUD bài hát + **upload bản HD** (admin web — xem mục admin `00`) |
| `entity/Subscription.java` / `SubscriptionPlan.java` | Entity/Enum | Bảng `Subscriptions` / FREE·INDIVIDUAL·STUDENT·FAMILY |
| `entity/UserSettings.java` / `StreamQuality.java` | Entity/Enum | Cài đặt user / chất lượng nhạc |
| `repository/SubscriptionRepository, UserSettingsRepository` | Repository | Truy vấn |
| `dto/PlanInfoDto, SubscribeRequest, UserSettingsDto, ListeningStatsDto, TopTrackDto, TopArtistDto` | DTO | Gói/cài đặt/thống kê |

## 3. DRAWABLE / ANIM THEO MÀN

| Màn (layout) | drawable dùng |
|--------------|---------------|
| `fragment_premium` | `bg_card_pink`, `ic_premium`, `bg_reasons_card`, `bg_btn_white_pill` (Java: `ic_no_ads/download/shuffle/audio_quality/friends/queue_add`) |
| `activity_premium_plans` | `ic_arrow_back`, `bg_premium_individual/student/family`, `bg_pill_pink/purple/green`, `ic_premium`, `bg_btn_pill_pink/purple/green` |
| `activity_payment` | (chỉ WebView, không icon) |
| `bottom_sheet_premium_gate` | `ic_premium`, `bg_btn_white_pill` |
| `bottom_sheet_shuffle_gate` | `ic_premium`, `ic_shuffle`, `ic_queue`, `bg_btn_pill_green` |
| `bottom_sheet_download_gate` | `ic_premium`, `placeholder_gradient`, `bg_btn_pill_green` |
| `activity_settings` / `activity_listening_stats` | `ic_arrow_back` / `ic_arrow_back`, `ic_share` |
| `item_premium_reason` / `item_stats_section` | `ic_no_ads` / `bg_card_dark`, `ic_avatar_placeholder`, `placeholder_gradient` |

> Banner/interstitial AdMob hiển thị ở `MainActivity` (khung Người 1) — bạn điều khiển qua `AdManager`.

## 4. ⭐ THỨ TỰ ĐỌC FILE

**Nền tảng quyết định khóa (đọc trước):** 1. `util/PremiumChecker` — `isPremium(sub)` dùng khắp app.

**Luồng Premium → thanh toán:**
2. `fragment/PremiumFragment` → `PremiumPlansActivity` → `SubscriptionViewModel` → `data/repository/SubscriptionRepository` → `SubscriptionController` → `SubscriptionService` → `entity/Subscription`.
3. `PaymentActivity` (WebView) → `PaymentController` → `VnpayService` ⭐ (ký HMAC-SHA512, verify) — chú ý gotcha rewrite localhost→10.0.2.2.

**Luồng Gate (khóa tính năng Free):** 4. `PremiumGateBottomSheet` / `ShuffleGateBottomSheet` / `DownloadGateBottomSheet` — được gọi từ `ShuffleController` (Người 3), Player (3), Album (2), Playlist (4).

**Luồng Quảng cáo (xem 4b):** 5. `util/AdManager` ⭐ → `MusicApp` (cắm adListener) → `PlayerManager.setAdListener` (Người 3) → banner trong `MainActivity` (Người 1).

**Màn cá nhân:** 6. `SettingsActivity` → `SettingsViewModel` → `UserSettingsController` → `UserSettingsService`; `ListeningStatsActivity` → `ListeningStatsViewModel` → `StatsController` → `ListeningStatsService` (đọc `PlayHistory` của Người 3).

## 4b. ⭐ CHÈN QUẢNG CÁO (AdMob) — tính năng XUYÊN MÀN (chủ: Người 5)

Quảng cáo chỉ hiện cho user **Free**; Premium tắt sạch. 2 loại: **banner** (dải dưới Main) + **interstitial** (toàn màn, mỗi 3 bài).

| File | Người (vị trí) | Vai trò |
|------|----------------|---------|
| `util/AdManager.java` ⭐ | **5** | Lõi: nạp/hiện interstitial, đếm 3 bài/lần, pause nhạc khi hiện ad, `setPremium` |
| `util/PremiumChecker.java` | 5 | `isPremium` → bật/tắt quảng cáo |
| `MainActivity.java` (`b.adBanner`) + `activity_main.xml` (`<AdView>`) | 1 (khung) | Banner: hiện/ẩn theo Premium, load/resume/pause/destroy |
| `MusicApp.java` | 1 (khung) | `ActivityLifecycleCallbacks` + cắm `adListener` phụ vào PlayerManager · khởi tạo MobileAds (chạy nền) |
| `util/PlayerManager.java` (`setAdListener`) | 3 | Slot listener PHỤ (tách khỏi UI) để AdManager đếm bài |
| `build.gradle` + `AndroidManifest` (AdMob App ID test) | 1 | Thư viện `play-services-ads` |

**Luồng interstitial:** `MusicApp` cắm listener phụ → mỗi lần đổi bài gọi `AdManager.onTrackPlayed()` → đếm, đủ bội số 3 (Free) → cố hiện; hiện thì **pause nhạc**, đóng ad → **phát lại** + preload ad kế. Đang chuyển màn (activity null) → hoãn, lần resume kế `maybeShowPending()` xả ra.
**Luồng banner:** `MainActivity` quan sát subscription → `AdManager.setPremium()` → Free hiện `adBanner.loadAd()`, Premium ẩn.

> ⚠️ Quảng cáo **không thuộc trọn 1 người**: chủ là Người 5 (AdManager + PremiumChecker) nhưng điểm cắm ở file Người 1 (`MainActivity`, `MusicApp`) và Người 3 (`PlayerManager`). 3 bạn thống nhất khi sửa.
> Dùng **AdMob ID test của Google** (không tính tiền); đổi sang ID thật khi phát hành.

## 5. ENDPOINT · GIAO
- **Endpoint:** `/api/subscriptions/*`, `/api/payment/{create,vnpay-return}`, `/api/users/me/settings`, `/api/stats/listening`.
- **Giao:** cung cấp 3 **gate sheet** + `PremiumChecker` cho Người 3 (Player + `ShuffleController`), Người 2 (Album), Người 4 (Playlist) · banner+interstitial gắn vào `MainActivity` (Người 1) + `PlayerManager` (Người 3) · Hồ sơ + `/api/users/me` thuộc Người 1; bạn dùng `UserRepository.settings()` cho màn Cài đặt · `ListeningStatsService` đọc `PlayHistory` (Người 3).
