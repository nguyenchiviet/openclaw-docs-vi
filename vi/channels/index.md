---
summary: Các nền tảng nhắn tin mà OpenClaw có thể kết nối
read_when:
  - Bạn muốn chọn một kênh chat cho OpenClaw
  - Bạn cần một cái nhìn tổng quan nhanh về các nền tảng nhắn tin được hỗ trợ
title: Kênh Chat
x-i18n:
  source_path: channels\index.md
  source_hash: 9e9c45e08c8a9df0c6ca1aa39530fc3c7402013548d7a0ad0d5ca733d6094220
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:19:06.959Z'
---

# Kênh trò chuyện

OpenClaw có thể trò chuyện với bạn trên bất kỳ ứng dụng chat nào bạn đã sử dụng. Mỗi kênh kết nối thông qua Gateway.
Văn bản được hỗ trợ ở mọi nơi; phương tiện và phản ứng khác nhau tùy theo kênh.

## Các kênh được hỗ trợ

- [WhatsApp](/channels/whatsapp) — Phổ biến nhất; sử dụng Baileys và yêu cầu ghép nối QR.
- [Telegram](/channels/telegram) — Bot API qua grammY; hỗ trợ nhóm.
- [Discord](/channels/discord) — Discord Bot API + Gateway; hỗ trợ máy chủ, kênh và tin nhắn riêng.
- [IRC](/channels/irc) — Máy chủ IRC cổ điển; kênh + tin nhắn riêng với điều khiển ghép nối/danh sách cho phép.
- [Slack](/channels/slack) — Bolt SDK; ứng dụng không gian làm việc.
- [Feishu](/channels/feishu) — Bot Feishu/Lark qua WebSocket (plugin, cài đặt riêng).
- [Google Chat](/channels/googlechat) — Ứng dụng Google Chat API qua HTTP webhook.
- [Mattermost](/channels/mattermost) — Bot API + WebSocket; kênh, nhóm, tin nhắn riêng (plugin, cài đặt riêng).
- [Signal](/channels/signal) — signal-cli; tập trung vào quyền riêng tư.
- [BlueBubbles](/channels/bluebubbles) — **Được khuyến nghị cho iMessage**; sử dụng BlueBubbles macOS server REST API với hỗ trợ tính năng đầy đủ (chỉnh sửa, hủy gửi, hiệu ứng, phản ứng, quản lý nhóm — chỉnh sửa hiện đang bị lỗi trên macOS 26 Tahoe).
- [iMessage (/channels/imessage) — Tích hợp macOS cũ qua imsg CLI (không được khuyến khích, sử dụng BlueBubbles cho thiết lập mới).
- [Microsoft Teams](/channels/msteams) — Bot Framework; hỗ trợ doanh nghiệp (plugin, cài đặt riêng).
- [Synology Chat](/channels/synology-chat) — Synology NAS Chat qua outgoing+incoming webhooks (plugin, cài đặt riêng).
- [LINE](/channels/line) — Bot LINE Messaging API (plugin, cài đặt riêng).
- [Nextcloud Talk](/channels/nextcloud-talk) — Chat tự lưu trữ qua Nextcloud Talk (plugin, cài đặt riêng).
- [Matrix](/channels/matrix) — Giao thức Matrix (plugin, cài đặt riêng).
- [Nostr](/channels/nostr) — Tin nhắn riêng phi tập trung qua NIP-04 (plugin, cài đặt riêng).
- [Tlon](/channels/tlon) — Ứng dụng nhắn tin dựa trên Urbit (plugin, cài đặt riêng).
- [Twitch](/channels/twitch) — Chat Twitch qua kết nối IRC (plugin, cài đặt riêng).
- [Zalo](/channels/zalo) — Zalo Bot API; ứng dụng nhắn tin phổ biến của Việt Nam (plugin, cài đặt riêng).
- [Zalo Personal](/channels/zalouser) — Tài khoản cá nhân Zalo qua đăng nhập QR (plugin, cài đặt riêng).
- [WebChat](/web/webchat) — Gateway WebChat UI qua WebSocket.

## Ghi chú

- Các kênh có thể chạy đồng thời; cấu hình nhiều kênh và OpenClaw sẽ định tuyến theo từng cuộc trò chuyện.
- Thiết lập nhanh nhất thường là **Telegram** (token bot đơn giản). WhatsApp yêu cầu ghép nối QR và
  lưu trữ nhiều trạng thái hơn trên đĩa.
- Hành vi nhóm khác nhau tùy theo kênh; xem [Nhóm](/channels/groups).
- Ghép nối tin nhắn riêng và danh sách cho phép được thực thi để đảm bảo an toàn; xem [Bảo mật](/gateway/security).
- Khắc phục sự cố: [Khắc phục sự cố kênh](/channels/troubleshooting).
- Các nhà cung cấp mô hình được tài liệu hóa riêng; xem [Nhà cung cấp mô hình](/providers/models).