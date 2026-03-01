---
summary: Thiết lập webhook Synology Chat và cấu hình OpenClaw
read_when:
  - Thiết lập Synology Chat với OpenClaw
  - Gỡ lỗi định tuyến webhook Synology Chat
title: Synology Chat
x-i18n:
  source_path: channels\synology-chat.md
  source_hash: 65cdf3be3fbf0652e201428d71e0ebd00dc6ab41f7e54c60b8f8019b4aeb12cf
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:26:05.020Z'
---

# Synology Chat (plugin)

Trạng thái: được hỗ trợ thông qua plugin như một kênh tin nhắn riêng sử dụng webhook của Synology Chat.
Plugin chấp nhận tin nhắn đến từ webhook gửi đi của Synology Chat và gửi phản hồi
thông qua webhook đến của Synology Chat.
## Plugin cần thiết

Synology Chat dựa trên plugin và không phải là một phần của cài đặt kênh cốt lõi mặc định.

Cài đặt từ bản sao cục bộ:

```bash
openclaw plugins install ./extensions/synology-chat
```

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập nhanh

1. Cài đặt và kích hoạt plugin Synology Chat.
2. Trong tích hợp Synology Chat:
   - Tạo một webhook đến và sao chép URL của nó.
   - Tạo một webhook đi với token bí mật của bạn.
3. Trỏ URL webhook đi đến gateway OpenClaw của bạn:
   - `https://gateway-host/webhook/synology` theo mặc định.
   - Hoặc `channels.synology-chat.webhookPath` tùy chỉnh của bạn.
4. Cấu hình `channels.synology-chat` trong OpenClaw.
5. Khởi động lại gateway và gửi tin nhắn riêng đến bot Synology Chat.

Cấu hình tối thiểu:

```json5
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```
## Biến môi trường

Đối với tài khoản mặc định, bạn có thể sử dụng biến môi trường:

- `SYNOLOGY_CHAT_TOKEN`
- `SYNOLOGY_CHAT_INCOMING_URL`
- `SYNOLOGY_NAS_HOST`
- `SYNOLOGY_ALLOWED_USER_IDS` (phân tách bằng dấu phẩy)
- `SYNOLOGY_RATE_LIMIT`
- `OPENCLAW_BOT_NAME`

Các giá trị cấu hình sẽ ghi đè biến môi trường.
## Chính sách tin nhắn riêng và kiểm soát truy cập

- `dmPolicy: "allowlist"` là mặc định được khuyến nghị.
- `allowedUserIds` chấp nhận một danh sách (hoặc chuỗi phân tách bằng dấu phẩy) các ID người dùng Synology.
- Trong chế độ `allowlist`, danh sách `allowedUserIds` trống được coi là cấu hình sai và tuyến webhook sẽ không khởi động (sử dụng `dmPolicy: "open"` để cho phép tất cả).
- `dmPolicy: "open"` cho phép bất kỳ người gửi nào.
- `dmPolicy: "disabled"` chặn tin nhắn riêng.
- Phê duyệt ghép nối hoạt động với:
  - `openclaw pairing list synology-chat`
  - `openclaw pairing approve synology-chat <CODE>`
## Gửi tin nhắn đi

Sử dụng ID người dùng Synology Chat dạng số làm đích đến.

Ví dụ:

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

Hỗ trợ gửi media thông qua phân phối tệp dựa trên URL.
## Nhiều tài khoản

Hỗ trợ nhiều tài khoản Synology Chat dưới `channels.synology-chat.accounts`.
Mỗi tài khoản có thể ghi đè token, URL đến, đường dẫn webhook, chính sách tin nhắn riêng và giới hạn.

```json5
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```
## Ghi chú bảo mật

- Giữ bí mật `token` và thay đổi nó nếu bị rò rỉ.
- Giữ `allowInsecureSsl: false` trừ khi bạn tin tưởng rõ ràng vào chứng chỉ NAS cục bộ tự ký.
- Các yêu cầu webhook đến được xác minh token và giới hạn tốc độ theo người gửi.
- Ưu tiên `dmPolicy: "allowlist"` cho môi trường sản xuất.