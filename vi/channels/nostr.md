---
summary: Kênh tin nhắn riêng Nostr thông qua tin nhắn mã hóa NIP-04
read_when:
  - Bạn muốn OpenClaw nhận tin nhắn riêng qua Nostr
  - Bạn đang thiết lập hệ thống nhắn tin phi tập trung
title: Nostr
x-i18n:
  source_path: channels\nostr.md
  source_hash: 6b9fe4c74bf5e7c0f59bbaa129ec5270fd29a248551a8a9a7dde6cff8fb46111
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:24:58.997Z'
---

# Nostr

**Trạng thái:** Plugin tùy chọn (tắt theo mặc định).

Nostr là một giao thức phi tập trung cho mạng xã hội. Kênh này cho phép OpenClaw nhận và phản hồi các tin nhắn riêng được mã hóa (DMs) thông qua NIP-04.
## Cài đặt (theo yêu cầu)

### Thiết lập ban đầu (khuyến nghị)

- Trình hướng dẫn thiết lập ban đầu (`openclaw onboard`) và `openclaw channels add` liệt kê các plugin kênh tùy chọn.
- Chọn Nostr sẽ nhắc bạn cài đặt plugin theo yêu cầu.

Cài đặt mặc định:

- **Kênh Dev + git checkout có sẵn:** sử dụng đường dẫn plugin cục bộ.
- **Stable/Beta:** tải xuống từ npm.

Bạn luôn có thể ghi đè lựa chọn trong lời nhắc.

### Cài đặt thủ công

```bash
openclaw plugins install @openclaw/nostr
```

Use a local checkout (dev workflows):

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Khởi động lại Gateway sau khi cài đặt hoặc kích hoạt plugin.
## Thiết lập nhanh

1. Tạo cặp khóa Nostr (nếu cần):

```bash
# Using nak
nak key generate
```

2. Add to config:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. Export the key:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Khởi động lại Gateway.
## Tham chiếu cấu hình

| Khóa          | Loại     | Mặc định                                     | Mô tả                         |
| ------------ | -------- | ------------------------------------------- | ----------------------------------- |
| `privateKey` | string   | bắt buộc                                    | Khóa riêng tư ở định dạng `nsec` hoặc hex |
| `relays`     | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | URL relay (WebSocket)              |
| `dmPolicy`   | string   | `pairing`                                   | Chính sách truy cập tin nhắn riêng                    |
| `allowFrom`  | string[] | `[]`                                        | Khóa công khai người gửi được phép              |
| `enabled`    | boolean  | `true`                                      | Bật/tắt kênh              |
| `name`       | string   | -                                           | Tên hiển thị                        |
| `profile`    | object   | -                                           | Metadata hồ sơ NIP-01             |
## Metadata hồ sơ

Dữ liệu hồ sơ được xuất bản dưới dạng sự kiện NIP-01 `kind:0`. Bạn có thể quản lý nó từ giao diện điều khiển (Kênh -> Nostr -> Hồ sơ) hoặc đặt trực tiếp trong cấu hình.

Ví dụ:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Personal assistant DM bot",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Notes:

- Profile URLs must use `https://`.
- Nhập từ relay sẽ hợp nhất các trường và bảo toàn các ghi đè cục bộ.
## Kiểm soát truy cập

### Chính sách tin nhắn riêng

- **pairing** (mặc định): người gửi không xác định sẽ nhận được mã ghép nối.
- **allowlist**: chỉ các pubkey trong `allowFrom` mới có thể gửi tin nhắn riêng.
- **open**: tin nhắn riêng đến công khai (yêu cầu `allowFrom: ["*"]`).
- **disabled**: bỏ qua tin nhắn riêng đến.

### Ví dụ allowlist

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```
## Định dạng khóa

Các định dạng được chấp nhận:

- **Khóa riêng:** `nsec...` hoặc hex 64 ký tự
- **Khóa công khai (`allowFrom`):** `npub...` hoặc hex
## Relays

Mặc định: `relay.damus.io` và `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

Tips:

- Use 2-3 relays for redundancy.
- Avoid too many relays (latency, duplication).
- Paid relays can improve reliability.
- Local relays are fine for testing (`ws://localhost:7777`).
## Hỗ trợ giao thức

| NIP    | Trạng thái | Mô tả                                 |
| ------ | ---------- | ------------------------------------- |
| NIP-01 | Được hỗ trợ | Định dạng sự kiện cơ bản + metadata hồ sơ |
| NIP-04 | Được hỗ trợ | Tin nhắn riêng được mã hóa (`kind:4`) |
| NIP-17 | Đã lên kế hoạch | Tin nhắn riêng được bọc quà |
| NIP-44 | Đã lên kế hoạch | Mã hóa có phiên bản |
## Kiểm thử

### Relay cục bộ

```bash
# Start strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### Kiểm thử thủ công

1. Ghi chú pubkey của bot (npub) từ logs.
2. Mở một ứng dụng Nostr (Damus, Amethyst, v.v.).
3. Gửi tin nhắn riêng đến pubkey của bot.
4. Xác minh phản hồi.
## Khắc phục sự cố

### Không nhận được tin nhắn

- Xác minh khóa riêng tư hợp lệ.
- Đảm bảo các URL relay có thể truy cập và sử dụng `wss://` (hoặc `ws://` cho local).
- Xác nhận `enabled` không phải là `false`.
- Kiểm tra nhật ký Gateway để tìm lỗi kết nối relay.

### Không gửi phản hồi

- Kiểm tra relay chấp nhận ghi.
- Xác minh kết nối đầu ra.
- Theo dõi giới hạn tốc độ của relay.

### Phản hồi trùng lặp

- Điều này được mong đợi khi sử dụng nhiều relay.
- Tin nhắn được loại bỏ trùng lặp theo ID sự kiện; chỉ lần gửi đầu tiên mới kích hoạt phản hồi.
## Bảo mật

- Không bao giờ commit các khóa riêng tư.
- Sử dụng biến môi trường cho các khóa.
- Cân nhắc `allowlist` cho các bot sản xuất.
## Hạn chế (MVP)

- Chỉ tin nhắn riêng (không có trò chuyện nhóm).
- Không có tệp đính kèm phương tiện.
- Chỉ NIP-04 (NIP-17 gift-wrap đã lên kế hoạch).