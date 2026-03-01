---
summary: 'Thiết lập, cấu hình và sử dụng plugin LINE Messaging API'
read_when:
  - Bạn muốn kết nối OpenClaw với LINE
  - Bạn cần thiết lập LINE webhook + thông tin xác thực
  - Bạn muốn các tùy chọn tin nhắn dành riêng cho LINE
title: LINE
x-i18n:
  source_path: channels\line.md
  source_hash: da3a10a79e4c93ab909250e85112308ef0bd5ec7edbb0e934173391a6bc9a8d2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:20:21.700Z'
---

# LINE (plugin)

LINE kết nối với OpenClaw thông qua LINE Messaging API. Plugin chạy như một webhook
receiver trên gateway và sử dụng channel access token + channel secret của bạn để
xác thực.

Trạng thái: được hỗ trợ qua plugin. Tin nhắn riêng, trò chuyện nhóm, media, vị trí, tin nhắn Flex,
tin nhắn template và quick replies được hỗ trợ. Reactions và threads
không được hỗ trợ.
## Plugin cần thiết

Cài đặt plugin LINE:

```bash
openclaw plugins install @openclaw/line
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/line
```
## Thiết lập

1. Tạo tài khoản LINE Developers và mở Console:
   [https://developers.line.biz/console/](https://developers.line.biz/console/)
2. Tạo (hoặc chọn) một Provider và thêm kênh **Messaging API**.
3. Sao chép **Channel access token** và **Channel secret** từ cài đặt kênh.
4. Bật **Use webhook** trong cài đặt Messaging API.
5. Đặt URL webhook thành endpoint gateway của bạn (yêu cầu HTTPS):

```
https://gateway-host/line/webhook
```

The gateway responds to LINE’s webhook verification (GET) and inbound events (POST).
If you need a custom path, set `channels.line.webhookPath` or
`channels.line.accounts.<id>.webhookPath` và cập nhật URL tương ứng.
## Cấu hình

Cấu hình tối thiểu:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Env vars (default account only):

- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_CHANNEL_SECRET`

Token/secret files:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

Multiple accounts:

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```
## Kiểm soát truy cập

Tin nhắn riêng mặc định sử dụng ghép nối. Người gửi không xác định sẽ nhận được mã ghép nối và tin nhắn của họ sẽ bị bỏ qua cho đến khi được phê duyệt.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Allowlists and policies:

- `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
- `channels.line.allowFrom`: allowlisted LINE user IDs for DMs
- `channels.line.groupPolicy`: `allowlist | open | disabled`
- `channels.line.groupAllowFrom`: allowlisted LINE user IDs for groups
- Per-group overrides: `channels.line.groups.<groupId>.allowFrom`
- Runtime note: if `channels.line` is completely missing, runtime falls back to `groupPolicy="allowlist"` for group checks (even if `channels.defaults.groupPolicy` is set).

LINE IDs are case-sensitive. Valid IDs look like:

- User: `U` + 32 hex chars
- Group: `C` + 32 hex chars
- Room: `R` + 32 ký tự hex
## Hành vi tin nhắn

- Văn bản được chia thành các đoạn 5000 ký tự.
- Định dạng Markdown bị loại bỏ; các khối mã và bảng được chuyển đổi thành
  thẻ Flex khi có thể.
- Phản hồi truyền phát được đệm; LINE nhận các đoạn đầy đủ với hoạt ảnh
  tải trong khi agent hoạt động.
- Tải xuống phương tiện được giới hạn bởi `channels.line.mediaMaxMb` (mặc định 10).
## Dữ liệu kênh (tin nhắn phong phú)

Sử dụng `channelData.line` để gửi phản hồi nhanh, vị trí, thẻ Flex, hoặc tin nhắn
mẫu.

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex payload */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

The LINE plugin also ships a `/card` command for Flex message presets:

```
/card info "Welcome" "Thanks for joining!"
```
## Khắc phục sự cố

- **Xác minh webhook thất bại:** đảm bảo URL webhook là HTTPS và
  `channelSecret` khớp với bảng điều khiển LINE.
- **Không có sự kiện đến:** xác nhận đường dẫn webhook khớp với `channels.line.webhookPath`
  và gateway có thể truy cập được từ LINE.
- **Lỗi tải xuống media:** tăng `channels.line.mediaMaxMb` nếu media vượt quá
  giới hạn mặc định.