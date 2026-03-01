---
summary: >-
  Tổng quan về ghép nối: phê duyệt ai có thể nhắn tin trực tiếp với bạn + những
  node nào có thể tham gia
read_when:
  - Thiết lập kiểm soát truy cập DM
  - Ghép nối một node iOS/Android mới
  - Đánh giá tình trạng bảo mật của OpenClaw
title: Ghép nối
x-i18n:
  source_path: channels\pairing.md
  source_hash: 65894ceee288fcd03547a4570ee06b9bdc6721cf6e32745a5f9507c34935eba3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:24:40.834Z'
---

# Ghép nối

"Ghép nối" là bước **phê duyệt chủ sở hữu** rõ ràng của OpenClaw.
Nó được sử dụng ở hai nơi:

1. **Ghép nối tin nhắn riêng** (ai được phép nói chuyện với bot)
2. **Ghép nối node** (thiết bị/node nào được phép tham gia mạng gateway)

Bối cảnh bảo mật: [Bảo mật](/gateway/security)
## 1) Ghép nối tin nhắn riêng (truy cập chat đến)

Khi một kênh được cấu hình với chính sách tin nhắn riêng `pairing`, người gửi không xác định sẽ nhận được mã ngắn và tin nhắn của họ **không được xử lý** cho đến khi bạn phê duyệt.

Các chính sách tin nhắn riêng mặc định được ghi lại trong: [Bảo mật](/gateway/security)

Mã ghép nối:

- 8 ký tự, chữ hoa, không có ký tự gây nhầm lẫn (`0O1I`).
- **Hết hạn sau 1 giờ**. Bot chỉ gửi tin nhắn ghép nối khi có yêu cầu mới được tạo (khoảng một lần mỗi giờ cho mỗi người gửi).
- Các yêu cầu ghép nối tin nhắn riêng đang chờ được giới hạn ở **3 yêu cầu mỗi kênh** theo mặc định; các yêu cầu bổ sung sẽ bị bỏ qua cho đến khi một yêu cầu hết hạn hoặc được phê duyệt.

### Phê duyệt người gửi

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Supported channels: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`, `feishu`.

### Where the state lives

Stored under `~/.openclaw/credentials/`:

- Pending requests: `<channel>-pairing.json`
- Approved allowlist store:
  - Default account: `<channel>-allowFrom.json`
  - Non-default account: `<channel>-<accountId>-allowFrom.json`

Hành vi phạm vi tài khoản:

- Các tài khoản không mặc định chỉ đọc/ghi tệp danh sách cho phép có phạm vi của chúng.
- Tài khoản mặc định sử dụng tệp danh sách cho phép có phạm vi kênh không có phạm vi.

Hãy coi những tệp này là nhạy cảm (chúng kiểm soát quyền truy cập vào trợ lý của bạn).
## 2) Ghép nối thiết bị node (iOS/Android/macOS/node không giao diện)

Các node kết nối với Gateway như **thiết bị** với `role: node`. Gateway
tạo một yêu cầu ghép nối thiết bị cần được phê duyệt.

### Ghép nối qua Telegram (khuyến nghị cho iOS)

Nếu bạn sử dụng plugin `device-pair`, bạn có thể thực hiện ghép nối thiết bị lần đầu hoàn toàn từ Telegram:

1. Trong Telegram, nhắn tin cho bot của bạn: `/pair`
2. Bot trả lời với hai tin nhắn: một tin nhắn hướng dẫn và một tin nhắn **mã thiết lập** riêng biệt (dễ sao chép/dán trong Telegram).
3. Trên điện thoại của bạn, mở ứng dụng OpenClaw iOS → Cài đặt → Gateway.
4. Dán mã thiết lập và kết nối.
5. Quay lại Telegram: `/pair approve`

Mã thiết lập là một payload JSON được mã hóa base64 chứa:

- `url`: URL WebSocket của Gateway (`ws://...` hoặc `wss://...`)
- `token`: một token ghép nối có thời hạn ngắn

Hãy xử lý mã thiết lập như một mật khẩu trong khi nó còn hiệu lực.

### Phê duyệt thiết bị node

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Node pairing state storage

Stored under `~/.openclaw/devices/`:

- `pending.json` (short-lived; pending requests expire)
- `paired.json` (paired devices + tokens)

### Notes

- The legacy `node.pair.*` API (CLI: `openclaw nodes pending/approve`) là một
  kho lưu trữ ghép nối riêng biệt thuộc sở hữu của gateway. Các node WS vẫn yêu cầu ghép nối thiết bị.
## Tài liệu liên quan

- Mô hình bảo mật + prompt injection: [Bảo mật](/gateway/security)
- Cập nhật an toàn (chạy doctor): [Cập nhật](/install/updating)
- Cấu hình kênh:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (legacy): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)