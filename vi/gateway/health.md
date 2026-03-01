---
summary: Các bước kiểm tra sức khỏe cho kết nối kênh
read_when:
  - Chẩn đoán tình trạng kênh WhatsApp
title: Kiểm Tra Sức Khỏe
x-i18n:
  source_path: gateway\health.md
  source_hash: 74f242e98244c135e1322682ed6b67d70f3b404aca783b1bb5de96a27c2c1b01
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:57:26.080Z'
---

# Kiểm tra Sức khỏe (CLI)

Hướng dẫn ngắn để xác minh kết nối kênh mà không cần đoán.

## Kiểm tra nhanh

- `openclaw status` — tóm tắt cục bộ: khả năng tiếp cận gateway/chế độ, gợi ý cập nhật, tuổi xác thực kênh được liên kết, phiên + hoạt động gần đây.
- `openclaw status --all` — chẩn đoán cục bộ đầy đủ (chỉ đọc, màu sắc, an toàn để dán để gỡ lỗi).
- `openclaw status --deep` — cũng kiểm tra Gateway đang chạy (kiểm tra từng kênh khi được hỗ trợ).
- `openclaw health --json` — yêu cầu Gateway đang chạy cung cấp ảnh chụp sức khỏe đầy đủ (chỉ WS; không có socket Baileys trực tiếp).
- Gửi `/status` dưới dạng tin nhắn độc lập trong WhatsApp/WebChat để nhận phản hồi trạng thái mà không gọi agent.
- Nhật ký: theo dõi `/tmp/openclaw/openclaw-*.log` và lọc theo `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

## Chẩn đoán sâu

- Thông tin xác thực trên đĩa: `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (mtime nên gần đây).
- Kho phiên: `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (đường dẫn có thể được ghi đè trong cấu hình). Số lượng và người nhận gần đây được hiển thị qua `status`.
- Luồng liên kết lại: `openclaw channels logout && openclaw channels login --verbose` khi mã trạng thái 409–515 hoặc `loggedOut` xuất hiện trong nhật ký. (Lưu ý: luồng đăng nhập QR tự động khởi động lại một lần cho trạng thái 515 sau khi ghép nối.)

## Khi có sự cố

- `logged out` hoặc trạng thái 409–515 → liên kết lại với `openclaw channels logout` rồi `openclaw channels login`.
- Gateway không thể tiếp cận → khởi động nó: `openclaw gateway --port 18789` (sử dụng `--force` nếu cổng bận).
- Không có tin nhắn đến → xác nhận điện thoại được liên kết đang trực tuyến và người gửi được phép (`channels.whatsapp.allowFrom`); đối với trò chuyện nhóm, hãy đảm bảo danh sách cho phép + quy tắc đề cập phù hợp (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## Lệnh "health" chuyên dụng

`openclaw health --json` yêu cầu Gateway đang chạy cung cấp ảnh chụp sức khỏe của nó (không có socket kênh trực tiếp từ CLI). Nó báo cáo thông tin xác thực được liên kết/tuổi xác thực khi có sẵn, tóm tắt kiểm tra từng kênh, tóm tắt kho phiên và thời lượng kiểm tra. Nó thoát với mã khác không nếu Gateway không thể tiếp cận hoặc kiểm tra không thành công/hết thời gian chờ. Sử dụng `--timeout <ms>` để ghi đè mặc định 10 giây.