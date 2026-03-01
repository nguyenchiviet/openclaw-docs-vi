---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình của Tlon/Urbit'
read_when:
  - Đang làm việc trên các tính năng kênh Tlon/Urbit
title: Tlon
x-i18n:
  source_path: channels\tlon.md
  source_hash: dcf889dcd567a870ce08873f5afd9cee63fbb67e22509e181c0cc4bff78aadd4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:28:28.888Z'
---

# Tlon (plugin)

Tlon là một ứng dụng nhắn tin phi tập trung được xây dựng trên Urbit. OpenClaw kết nối với Urbit ship của bạn và có thể
phản hồi tin nhắn riêng và tin nhắn nhóm. Phản hồi nhóm yêu cầu @ mention theo mặc định và có thể
được hạn chế thêm thông qua danh sách cho phép.

Trạng thái: được hỗ trợ qua plugin. Tin nhắn riêng, mention nhóm, phản hồi thread, và dự phòng media chỉ văn bản
(URL được thêm vào chú thích). Reactions, bình chọn, và tải lên media gốc không được hỗ trợ.
## Plugin cần thiết

Tlon được cung cấp dưới dạng plugin và không được tích hợp sẵn trong bản cài đặt cốt lõi.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/tlon
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/tlon
```

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập

1. Cài đặt plugin Tlon.
2. Thu thập URL ship và mã đăng nhập của bạn.
3. Cấu hình `channels.tlon`.
4. Khởi động lại gateway.
5. Gửi tin nhắn riêng cho bot hoặc nhắc đến nó trong kênh nhóm.

Cấu hình tối thiểu (tài khoản đơn):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
    },
  },
}
```

Private/LAN ship URLs (advanced):

By default, OpenClaw blocks private/internal hostnames and IP ranges for this plugin (SSRF hardening).
If your ship URL is on a private network (for example `http://192.168.1.50:8080` or `http://localhost:8080`),
you must explicitly opt in:

```json5
{
  channels: {
    tlon: {
      allowPrivateNetwork: true,
    },
  },
}
```
## Kênh nhóm

Khám phá thiết bị tự động được bật theo mặc định. Bạn cũng có thể ghim kênh thủ công:

```json5
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

Disable auto-discovery:

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```
## Kiểm soát truy cập

Danh sách cho phép tin nhắn riêng (để trống = cho phép tất cả):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

Group authorization (restricted by default):

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```
## Mục tiêu gửi (CLI/cron)

Sử dụng với `openclaw message send` hoặc gửi cron:

- Tin nhắn riêng: `~sampel-palnet` hoặc `dm/~sampel-palnet`
- Nhóm: `chat/~host-ship/channel` hoặc `group:~host-ship/channel`
## Ghi chú

- Trả lời nhóm yêu cầu phải có mention (ví dụ: `~your-bot-ship`) để phản hồi.
- Trả lời trong thread: nếu tin nhắn đến nằm trong thread, OpenClaw sẽ trả lời trong thread đó.
- Media: `sendMedia` sẽ chuyển về dạng văn bản + URL (không có tính năng tải lên gốc).