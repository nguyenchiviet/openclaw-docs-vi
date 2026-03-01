---
summary: Thiết lập bot Mattermost và cấu hình OpenClaw
read_when:
  - Thiết lập Mattermost
  - Gỡ lỗi định tuyến Mattermost
title: Mattermost
x-i18n:
  source_path: channels\mattermost.md
  source_hash: f737f567c083caa955075bd6040e4793b7af7087dbe5d591474904020b1f109d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:21:42.541Z'
---

# Mattermost (plugin)

Trạng thái: được hỗ trợ qua plugin (bot token + sự kiện WebSocket). Hỗ trợ kênh, nhóm và tin nhắn riêng.
Mattermost là một nền tảng nhắn tin nhóm có thể tự lưu trữ; xem trang web chính thức tại
[mattermost.com](https://mattermost.com) để biết chi tiết sản phẩm và tải xuống.
## Plugin bắt buộc

Mattermost được cung cấp dưới dạng plugin và không được tích hợp sẵn trong bản cài đặt cốt lõi.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/mattermost
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/mattermost
```

Nếu bạn chọn Mattermost trong quá trình cấu hình/thiết lập ban đầu và phát hiện git checkout,
OpenClaw sẽ tự động đề xuất đường dẫn cài đặt cục bộ.

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập nhanh

1. Cài đặt plugin Mattermost.
2. Tạo tài khoản bot Mattermost và sao chép **token bot**.
3. Sao chép **URL cơ sở** của Mattermost (ví dụ: `https://chat.example.com`).
4. Cấu hình OpenClaw và khởi động gateway.

Cấu hình tối thiểu:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```
## Biến môi trường (tài khoản mặc định)

Đặt các biến này trên máy chủ gateway nếu bạn muốn sử dụng biến môi trường:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

Biến môi trường chỉ áp dụng cho tài khoản **mặc định** (`default`). Các tài khoản khác phải sử dụng giá trị cấu hình.
## Chế độ trò chuyện

Mattermost phản hồi tin nhắn riêng tự động. Hành vi kênh được điều khiển bởi `chatmode`:

- `oncall` (mặc định): chỉ phản hồi khi được @mention trong kênh.
- `onmessage`: phản hồi mọi tin nhắn kênh.
- `onchar`: phản hồi khi tin nhắn bắt đầu bằng tiền tố kích hoạt.

Ví dụ cấu hình:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Notes:

- `onchar` still responds to explicit @mentions.
- `channels.mattermost.requireMention` is honored for legacy configs but `chatmode` được ưu tiên.
## Kiểm soát truy cập (Tin nhắn riêng)

- Mặc định: `channels.mattermost.dmPolicy = "pairing"` (người gửi không xác định sẽ nhận mã ghép nối).
- Phê duyệt qua:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- Tin nhắn riêng công khai: `channels.mattermost.dmPolicy="open"` cộng với `channels.mattermost.allowFrom=["*"]`.
## Kênh (nhóm)

- Mặc định: `channels.mattermost.groupPolicy = "allowlist"` (cần được nhắc đến).
- Danh sách cho phép người gửi với `channels.mattermost.groupAllowFrom` (khuyến nghị sử dụng ID người dùng).
- Khớp `@username` có thể thay đổi và chỉ được bật khi `channels.mattermost.dangerouslyAllowNameMatching: true`.
- Kênh mở: `channels.mattermost.groupPolicy="open"` (cần được nhắc đến).
- Lưu ý runtime: nếu `channels.mattermost` hoàn toàn bị thiếu, runtime sẽ quay về `groupPolicy="allowlist"` để kiểm tra nhóm (ngay cả khi `channels.defaults.groupPolicy` được đặt).
## Mục tiêu cho việc gửi tin nhắn ra ngoài

Sử dụng các định dạng mục tiêu này với `openclaw message send` hoặc cron/webhooks:

- `channel:<id>` cho một kênh
- `user:<id>` cho tin nhắn riêng
- `@username` cho tin nhắn riêng (được phân giải qua API Mattermost)

ID đơn thuần được coi là kênh.
## Phản ứng (công cụ tin nhắn)

- Sử dụng `message action=react` với `channel=mattermost`.
- `messageId` là id bài đăng Mattermost.
- `emoji` chấp nhận tên như `thumbsup` hoặc `:+1:` (dấu hai chấm là tùy chọn).
- Đặt `remove=true` (boolean) để xóa phản ứng.
- Các sự kiện thêm/xóa phản ứng được chuyển tiếp dưới dạng sự kiện hệ thống đến phiên agent được định tuyến.

Ví dụ:

```
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

Config:

- `channels.mattermost.actions.reactions`: enable/disable reaction actions (default true).
- Per-account override: `channels.mattermost.accounts.<id>.actions.reactions`.
## Nhiều tài khoản

Mattermost hỗ trợ nhiều tài khoản dưới `channels.mattermost.accounts`:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```
## Khắc phục sự cố

- Không có phản hồi trong kênh: đảm bảo bot đã ở trong kênh và nhắc đến nó (oncall), sử dụng tiền tố kích hoạt (onchar), hoặc đặt `chatmode: "onmessage"`.
- Lỗi xác thực: kiểm tra token bot, URL cơ sở, và xem tài khoản có được kích hoạt hay không.
- Vấn đề đa tài khoản: biến môi trường chỉ áp dụng cho tài khoản `default`.