---
summary: Cách OpenClaw xoay vòng các hồ sơ xác thực và dự phòng trên các mô hình
read_when:
  - >-
    Chẩn đoán xoay vòng hồ sơ xác thực, thời gian chờ hoặc hành vi dự phòng mô
    hình
  - Cập nhật các quy tắc failover cho các hồ sơ xác thực hoặc mô hình
title: Chuyển đổi dự phòng Mô hình
x-i18n:
  source_path: concepts\model-failover.md
  source_hash: eab7c0633824d941cf0d6ce4294f0bc8747fbba2ce93650e9643eca327cd04a9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:39:38.317Z'
---

# Chuyển đổi dự phòng mô hình

OpenClaw xử lý các lỗi trong hai giai đoạn:

1. **Xoay vòng hồ sơ xác thực** trong nhà cung cấp hiện tại.
2. **Quay lại mô hình** sang mô hình tiếp theo trong `agents.defaults.model.fallbacks`.

Tài liệu này giải thích các quy tắc thời gian chạy và dữ liệu hỗ trợ chúng.
## Xác thực: lưu trữ (khóa + OAuth)

OpenClaw sử dụng **auth profiles** cho cả khóa API và token OAuth.

- Secrets được lưu trong `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (cũ: `~/.openclaw/agent/auth-profiles.json`).
- Config `auth.profiles` / `auth.order` là **chỉ metadata + định tuyến** (không có secrets).
- Tệp OAuth chỉ nhập cũ: `~/.openclaw/credentials/oauth.json` (được nhập vào `auth-profiles.json` khi sử dụng lần đầu).

Chi tiết hơn: [/concepts/oauth](/concepts/oauth)

Loại thông tin xác thực:

- `type: "api_key"` → `{ provider, key }`
- `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` cho một số nhà cung cấp)
## ID Hồ sơ

Các lần đăng nhập OAuth tạo ra các hồ sơ riêng biệt để nhiều tài khoản có thể tồn tại cùng nhau.

- Mặc định: `provider:default` khi không có email khả dụng.
- OAuth với email: `provider:<email>` (ví dụ `google-antigravity:user@gmail.com`).

Các hồ sơ nằm trong `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` dưới `profiles`.
## Thứ tự xoay vòng

Khi một nhà cung cấp có nhiều hồ sơ, OpenClaw chọn một thứ tự như sau:

1. **Cấu hình rõ ràng**: `auth.order[provider]` (nếu được đặt).
2. **Hồ sơ được cấu hình**: `auth.profiles` được lọc theo nhà cung cấp.
3. **Hồ sơ được lưu trữ**: các mục trong `auth-profiles.json` cho nhà cung cấp.

Nếu không có thứ tự rõ ràng nào được cấu hình, OpenClaw sử dụng thứ tự round‑robin:

- **Khóa chính:** loại hồ sơ (**OAuth trước các khóa API**).
- **Khóa phụ:** `usageStats.lastUsed` (cũ nhất trước, trong mỗi loại).
- **Hồ sơ cooldown/bị vô hiệu hóa** được chuyển đến cuối, được sắp xếp theo thời gian hết hạn sớm nhất.

### Độ dính phiên (thân thiện với bộ nhớ đệm)

OpenClaw **ghim hồ sơ xác thực được chọn cho mỗi phiên** để giữ bộ nhớ đệm nhà cung cấp ấm áp.
Nó **không** xoay vòng trên mỗi yêu cầu. Hồ sơ được ghim được sử dụng lại cho đến khi:

- phiên được đặt lại (`/new` / `/reset`)
- một nén hoàn tất (số lượng nén tăng lên)
- hồ sơ ở trạng thái cooldown/bị vô hiệu hóa

Lựa chọn thủ công thông qua `/model …@<profileId>` đặt một **ghi đè do người dùng** cho phiên đó
và không được xoay vòng tự động cho đến khi một phiên mới bắt đầu.

Hồ sơ được ghim tự động (được chọn bởi bộ định tuyến phiên) được coi là một **tùy chọn**:
chúng được thử trước tiên, nhưng OpenClaw có thể xoay vòng sang hồ sơ khác khi giới hạn tốc độ/hết thời gian chờ.
Hồ sơ được ghim bởi người dùng vẫn bị khóa vào hồ sơ đó; nếu nó không thành công và các dự phòng mô hình
được cấu hình, OpenClaw chuyển sang mô hình tiếp theo thay vì chuyển đổi hồ sơ.

### Tại sao OAuth có thể "trông bị mất"

Nếu bạn có cả hồ sơ OAuth và hồ sơ khóa API cho cùng một nhà cung cấp, round‑robin có thể chuyển đổi giữa chúng trên các tin nhắn trừ khi được ghim. Để buộc một hồ sơ duy nhất:

- Ghim với `auth.order[provider] = ["provider:profileId"]`, hoặc
- Sử dụng ghi đè cho mỗi phiên thông qua `/model …` với ghi đè hồ sơ (khi được hỗ trợ bởi giao diện người dùng/bề mặt trò chuyện của bạn).
## Thời gian chờ

Khi một hồ sơ không thành công do lỗi xác thực/giới hạn tốc độ (hoặc hết thời gian chờ có vẻ như giới hạn tốc độ), OpenClaw đánh dấu nó trong thời gian chờ và chuyển sang hồ sơ tiếp theo.
Lỗi định dạng/yêu cầu không hợp lệ (ví dụ: lỗi xác thực ID lệnh gọi công cụ Cloud Code Assist) được coi là đáng chuyển đổi dự phòng và sử dụng cùng một thời gian chờ.

Thời gian chờ sử dụng backoff theo cấp số nhân:

- 1 phút
- 5 phút
- 25 phút
- 1 giờ (giới hạn)

Trạng thái được lưu trữ trong `auth-profiles.json` dưới `usageStats`:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```
## Vô hiệu hóa thanh toán

Các lỗi thanh toán/tín dụng (ví dụ "không đủ tín dụng" / "số dư tín dụng quá thấp") được coi là đáng chuyển đổi dự phòng, nhưng chúng thường không phải là tạm thời. Thay vì thời gian chờ ngắn, OpenClaw đánh dấu hồ sơ là **bị vô hiệu hóa** (với thời gian chờ lâu hơn) và chuyển sang hồ sơ/nhà cung cấp tiếp theo.

Trạng thái được lưu trữ trong `auth-profiles.json`:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Giá trị mặc định:

- Thời gian chờ thanh toán bắt đầu ở **5 giờ**, tăng gấp đôi cho mỗi lỗi thanh toán, và giới hạn ở **24 giờ**.
- Bộ đếm thời gian chờ được đặt lại nếu hồ sơ không bị lỗi trong **24 giờ** (có thể cấu hình).
## Dự phòng mô hình

Nếu tất cả các hồ sơ cho một nhà cung cấp không thành công, OpenClaw sẽ chuyển sang mô hình tiếp theo trong
`agents.defaults.model.fallbacks`. Điều này áp dụng cho các lỗi xác thực, giới hạn tốc độ và
hết thời gian chờ đã làm cạn kiệt vòng xoay hồ sơ (các lỗi khác không kích hoạt dự phòng).

Khi một lần chạy bắt đầu với ghi đè mô hình (hooks hoặc CLI), dự phòng vẫn kết thúc tại
`agents.defaults.model.primary` sau khi thử bất kỳ dự phòng nào được cấu hình.
## Cấu hình liên quan

Xem [Cấu hình Gateway](/gateway/configuration) để biết:

- `auth.profiles` / `auth.order`
- `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
- `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
- `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel` định tuyến

Xem [Mô hình](/concepts/models) để biết tổng quan về lựa chọn mô hình và dự phòng rộng hơn.