---
summary: 'Quy tắc xử lý hình ảnh và phương tiện cho các phản hồi send, gateway, và agent'
read_when:
  - Sửa đổi đường ống xử lý phương tiện hoặc tệp đính kèm
title: Hỗ trợ Hình ảnh và Phương tiện
x-i18n:
  source_path: nodes\images.md
  source_hash: 971aed398ea01078efbad7a8a4bca17f2a975222a2c4db557565e4334c9450e0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:15.166Z'
---

# Hỗ trợ Hình ảnh & Phương tiện — 2025-12-05

Kênh WhatsApp chạy qua **Baileys Web**. Tài liệu này ghi lại các quy tắc xử lý phương tiện hiện tại cho các phản hồi gửi, gateway và agent.
## Mục tiêu

- Gửi phương tiện với chú thích tùy chọn qua `openclaw message send --media`.
- Cho phép trả lời tự động từ hộp thư web bao gồm phương tiện cùng với văn bản.
- Giữ các giới hạn theo loại hợp lý và có thể dự đoán được.
## Bề mặt CLI

- `openclaw message send --media <path-or-url> [--message <caption>]`
  - `--media` tùy chọn; chú thích có thể trống cho các lần gửi chỉ có phương tiện.
  - `--dry-run` in ra tải trọng đã giải quyết; `--json` phát ra `{ channel, to, messageId, mediaUrl, caption }`.
## Hành vi kênh WhatsApp Web

- Đầu vào: đường dẫn tệp cục bộ **hoặc** URL HTTP(S).
- Luồng: tải vào Buffer, phát hiện loại phương tiện và xây dựng tải trọng chính xác:
  - **Hình ảnh:** thay đổi kích thước và nén lại thành JPEG (cạnh tối đa 2048px) nhắm tới `agents.defaults.mediaMaxMb` (mặc định 5 MB), giới hạn ở 6 MB.
  - **Audio/Voice/Video:** chuyển qua lên tới 16 MB; âm thanh được gửi dưới dạng ghi chú thoại (`ptt: true`).
  - **Tài liệu:** bất cứ thứ gì khác, lên tới 100 MB, với tên tệp được bảo toàn khi có sẵn.
- Phát lại kiểu GIF của WhatsApp: gửi MP4 với `gifPlayback: true` (CLI: `--gif-playback`) để các ứng dụng di động lặp lại nội tuyến.
- Phát hiện MIME ưu tiên byte ma thuật, sau đó là tiêu đề, sau đó là phần mở rộng tệp.
- Chú thích đến từ `--message` hoặc `reply.text`; chú thích trống được phép.
- Ghi nhật ký: không chi tiết hiển thị `↩️`/`✅`; chi tiết bao gồm kích thước và đường dẫn/URL nguồn.
## Đường ống Trả lời Tự động

- `getReplyFromConfig` trả về `{ text?, mediaUrl?, mediaUrls? }`.
- Khi có media, người gửi web giải quyết các đường dẫn cục bộ hoặc URL bằng cách sử dụng cùng một đường ống như `openclaw message send`.
- Nhiều mục media được gửi tuần tự nếu được cung cấp.
## Phương tiện đến Lệnh (Pi)

- Khi các tin nhắn web đến bao gồm phương tiện, OpenClaw tải xuống tệp tạm thời và hiển thị các biến mẫu:
  - `{{MediaUrl}}` pseudo-URL cho phương tiện đến.
  - `{{MediaPath}}` đường dẫn tạm thời cục bộ được ghi trước khi chạy lệnh.
- Khi sandbox Docker cho mỗi phiên được bật, phương tiện đến được sao chép vào không gian làm việc sandbox và `MediaPath`/`MediaUrl` được viết lại thành đường dẫn tương đối như `media/inbound/<filename>`.
- Hiểu biết phương tiện (nếu được cấu hình qua `tools.media.*` hoặc `tools.media.models` được chia sẻ) chạy trước mẫu và có thể chèn các khối `[Image]`, `[Audio]` và `[Video]` vào `Body`.
  - Audio đặt `{{Transcript}}` và sử dụng bản ghi âm để phân tích lệnh để các lệnh gạch chéo vẫn hoạt động.
  - Mô tả video và hình ảnh bảo toàn bất kỳ văn bản chú thích nào để phân tích lệnh.
- Theo mặc định, chỉ tệp đính kèm hình ảnh/âm thanh/video đầu tiên phù hợp được xử lý; đặt `tools.media.<cap>.attachments` để xử lý nhiều tệp đính kèm.
## Giới hạn & Lỗi

**Giới hạn gửi đi (gửi WhatsApp web)**

- Hình ảnh: giới hạn ~6 MB sau khi nén lại.
- Audio/voice/video: giới hạn 16 MB; tài liệu: giới hạn 100 MB.
- Media quá lớn hoặc không đọc được → lỗi rõ ràng trong nhật ký và phản hồi bị bỏ qua.

**Giới hạn hiểu media (chuyển đổi âm thanh/mô tả)**

- Hình ảnh mặc định: 10 MB (`tools.media.image.maxBytes`).
- Audio mặc định: 20 MB (`tools.media.audio.maxBytes`).
- Video mặc định: 50 MB (`tools.media.video.maxBytes`).
- Media quá lớn bỏ qua hiểu, nhưng phản hồi vẫn được gửi với nội dung gốc.
## Ghi chú cho Bài kiểm tra

- Bao gồm các luồng gửi + trả lời cho các trường hợp hình ảnh/âm thanh/tài liệu.
- Xác thực nén lại cho hình ảnh (giới hạn kích thước) và cờ ghi chú thoại cho âm thanh.
- Đảm bảo các trả lời đa phương tiện được phân tán dưới dạng các lần gửi tuần tự.