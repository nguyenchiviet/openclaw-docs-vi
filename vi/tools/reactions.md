---
summary: Ngữ nghĩa phản ứng được chia sẻ trên các kênh
read_when:
  - Làm việc với các phản ứng trong bất kỳ kênh nào
title: Phản ứng
x-i18n:
  source_path: tools\reactions.md
  source_hash: 0f11bff9adb4bd02604f96ebe2573a623702796732b6e17dfeda399cb7be0fa6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:31:48.810Z'
---

# Công cụ phản ứng

Ngữ nghĩa phản ứng được chia sẻ trên các kênh:

- `emoji` là bắt buộc khi thêm phản ứng.
- `emoji=""` xóa phản ứng của bot khi được hỗ trợ.
- `remove: true` xóa emoji được chỉ định khi được hỗ trợ (yêu cầu `emoji`).

Ghi chú kênh:

- **Discord/Slack**: `emoji` trống xóa tất cả phản ứng của bot trên tin nhắn; `remove: true` chỉ xóa emoji đó.
- **Google Chat**: `emoji` trống xóa phản ứng của ứng dụng trên tin nhắn; `remove: true` chỉ xóa emoji đó.
- **Telegram**: `emoji` trống xóa phản ứng của bot; `remove: true` cũng xóa phản ứng nhưng vẫn yêu cầu `emoji` không trống để xác thực công cụ.
- **WhatsApp**: `emoji` trống xóa phản ứng của bot; `remove: true` ánh xạ tới emoji trống (vẫn yêu cầu `emoji`).
- **Signal**: thông báo phản ứng đến phát ra các sự kiện hệ thống khi `channels.signal.reactionNotifications` được bật.