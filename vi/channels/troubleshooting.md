---
summary: Khắc phục sự cố nhanh ở cấp độ kênh với chữ ký lỗi và cách sửa cho từng kênh
read_when:
  - Kênh truyền tải hiển thị đã kết nối nhưng phản hồi thất bại
  - >-
    Bạn cần kiểm tra cụ thể theo kênh trước khi xem tài liệu chi tiết của nhà
    cung cấp
title: Khắc phục sự cố Kênh
x-i18n:
  source_path: channels\troubleshooting.md
  source_hash: c210b2aefa1d76d42ffed61e7fec02a1f8f6c521ba83e5080a98eb2ce8245696
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:28:54.104Z'
---

# Khắc phục sự cố kênh

Sử dụng trang này khi kênh kết nối nhưng hoạt động không đúng.
## Thang lệnh

Chạy các lệnh này theo thứ tự trước:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Healthy baseline:

- `Runtime: running`
- `RPC probe: ok`
- Kiểm tra kênh hiển thị đã kết nối/sẵn sàng
## WhatsApp

### Dấu hiệu lỗi WhatsApp

| Triệu chứng                     | Kiểm tra nhanh nhất                                 | Khắc phục                                               |
| ------------------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| Đã kết nối nhưng không trả lời tin nhắn riêng | `openclaw pairing list whatsapp`                    | Phê duyệt người gửi hoặc chuyển đổi chính sách/danh sách cho phép tin nhắn riêng. |
| Tin nhắn nhóm bị bỏ qua        | Kiểm tra `requireMention` + mẫu mention trong cấu hình | Mention bot hoặc nới lỏng chính sách mention cho nhóm đó. |
| Ngắt kết nối/đăng nhập lại ngẫu nhiên | `openclaw channels status --probe` + logs           | Đăng nhập lại và xác minh thư mục thông tin xác thực hoạt động tốt. |

Khắc phục sự cố đầy đủ: [/channels/whatsapp#troubleshooting-quick](/channels/whatsapp#troubleshooting-quick)
## Telegram

### Dấu hiệu lỗi Telegram

| Triệu chứng                           | Kiểm tra nhanh nhất                                   | Khắc phục                                                                         |
| --------------------------------- | ----------------------------------------------- | --------------------------------------------------------------------------- |
| `/start` nhưng không có luồng phản hồi khả dụng | `openclaw pairing list telegram`                | Phê duyệt ghép nối hoặc thay đổi chính sách tin nhắn riêng.                                        |
| Bot trực tuyến nhưng nhóm vẫn im lặng | Xác minh yêu cầu nhắc đến và chế độ riêng tư của bot | Tắt chế độ riêng tư để hiển thị trong nhóm hoặc nhắc đến bot.                   |
| Lỗi gửi với lỗi mạng | Kiểm tra nhật ký để tìm lỗi gọi API Telegram     | Khắc phục định tuyến DNS/IPv6/proxy đến `api.telegram.org`.                           |
| Nâng cấp và danh sách cho phép chặn bạn | `openclaw security audit` và danh sách cho phép cấu hình | Chạy `openclaw doctor --fix` hoặc thay thế `@username` bằng ID người gửi số. |

Khắc phục sự cố đầy đủ: [/channels/telegram#troubleshooting](/channels/telegram#troubleshooting)
## Discord

### Dấu hiệu lỗi Discord

| Triệu chứng                         | Kiểm tra nhanh nhất                       | Khắc phục                                                       |
| ----------------------------------- | ----------------------------------------- | --------------------------------------------------------------- |
| Bot online nhưng không phản hồi guild | `openclaw channels status --probe`  | Cho phép guild/kênh và xác minh intent nội dung tin nhắn.    |
| Tin nhắn nhóm bị bỏ qua          | Kiểm tra logs để tìm mention gating drops | Mention bot hoặc đặt guild/kênh `requireMention: false`. |
| Thiếu phản hồi tin nhắn riêng              | `openclaw pairing list discord`     | Phê duyệt ghép nối tin nhắn riêng hoặc điều chỉnh chính sách tin nhắn riêng.                   |

Khắc phục sự cố đầy đủ: [/channels/discord#troubleshooting](/channels/discord#troubleshooting)
## Slack

### Dấu hiệu lỗi Slack

| Triệu chứng                                | Kiểm tra nhanh nhất                             | Khắc phục                                               |
| ------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------- |
| Chế độ socket đã kết nối nhưng không phản hồi | `openclaw channels status --probe`        | Xác minh app token + bot token và các phạm vi bắt buộc. |
| Tin nhắn riêng bị chặn                            | `openclaw pairing list slack`             | Phê duyệt ghép nối hoặc nới lỏng chính sách tin nhắn riêng.               |
| Tin nhắn kênh bị bỏ qua                | Kiểm tra `groupPolicy` và danh sách kênh cho phép | Cho phép kênh hoặc chuyển chính sách sang `open`.     |

Khắc phục sự cố đầy đủ: [/channels/slack#troubleshooting](/channels/slack#troubleshooting)
## iMessage và BlueBubbles

### Dấu hiệu lỗi iMessage và BlueBubbles

| Triệu chứng                      | Kiểm tra nhanh nhất                                                     | Khắc phục                                             |
| -------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------- |
| Không có sự kiện đến             | Xác minh khả năng truy cập webhook/server và quyền ứng dụng             | Sửa URL webhook hoặc trạng thái server BlueBubbles.   |
| Có thể gửi nhưng không nhận trên macOS | Kiểm tra quyền riêng tư macOS cho tự động hóa Messages                  | Cấp lại quyền TCC và khởi động lại tiến trình kênh.   |
| Người gửi tin nhắn riêng bị chặn | `openclaw pairing list imessage` hoặc `openclaw pairing list bluebubbles` | Phê duyệt ghép nối hoặc cập nhật danh sách cho phép.  |

Khắc phục sự cố đầy đủ:

- [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](/channels/imessage#troubleshooting-macos-privacy-and-security-tcc)
- [/channels/bluebubbles#troubleshooting](/channels/bluebubbles#troubleshooting)
## Signal

### Dấu hiệu lỗi Signal

| Triệu chứng                     | Kiểm tra nhanh nhất                        | Khắc phục                                                |
| ------------------------------- | ------------------------------------------ | -------------------------------------------------------- |
| Daemon có thể truy cập nhưng bot im lặng | `openclaw channels status --probe`         | Xác minh URL/tài khoản daemon `signal-cli` và chế độ nhận. |
| Tin nhắn riêng bị chặn          | `openclaw pairing list signal`             | Phê duyệt người gửi hoặc điều chỉnh chính sách tin nhắn riêng.                      |
| Phản hồi nhóm không kích hoạt   | Kiểm tra danh sách cho phép nhóm và mẫu mention | Thêm người gửi/nhóm hoặc nới lỏng kiểm soát.                       |

Khắc phục sự cố đầy đủ: [/channels/signal#troubleshooting](/channels/signal#troubleshooting)
## Matrix

### Dấu hiệu lỗi Matrix

| Triệu chứng                                    | Kiểm tra nhanh nhất                          | Khắc phục                                              |
| ---------------------------------------------- | -------------------------------------------- | ------------------------------------------------------ |
| Đã đăng nhập nhưng bỏ qua tin nhắn trong phòng | `openclaw channels status --probe`           | Kiểm tra `groupPolicy` và danh sách phòng được phép.         |
| Tin nhắn riêng không được xử lý                | `openclaw pairing list matrix`               | Phê duyệt người gửi hoặc điều chỉnh chính sách tin nhắn riêng. |
| Phòng mã hóa gặp lỗi                           | Xác minh mô-đun mã hóa và cài đặt mã hóa     | Bật hỗ trợ mã hóa và tham gia lại/đồng bộ phòng.      |

Khắc phục sự cố đầy đủ: [/channels/matrix#troubleshooting](/channels/matrix#troubleshooting)