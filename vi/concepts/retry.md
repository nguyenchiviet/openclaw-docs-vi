---
summary: Chính sách thử lại cho các cuộc gọi nhà cung cấp đi
read_when:
  - Cập nhật hành vi thử lại của nhà cung cấp hoặc các giá trị mặc định
  - Gỡ lỗi lỗi gửi nhà cung cấp hoặc giới hạn tốc độ
title: Chính sách Thử lại
x-i18n:
  source_path: concepts\retry.md
  source_hash: 55bb261ff567f46ce447be9c0ee0c5b5e6d2776287d7662762656c14108dd607
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:36.273Z'
---

# Chính sách thử lại

## Mục tiêu

- Thử lại cho mỗi yêu cầu HTTP, không phải cho toàn bộ luồng nhiều bước.
- Bảo toàn thứ tự bằng cách chỉ thử lại bước hiện tại.
- Tránh trùng lặp các hoạt động không idempotent.
## Mặc định

- Số lần thử: 3
- Giới hạn độ trễ tối đa: 30000 ms
- Jitter: 0.1 (10 phần trăm)
- Mặc định nhà cung cấp:
  - Độ trễ tối thiểu Telegram: 400 ms
  - Độ trễ tối thiểu Discord: 500 ms
## Hành vi

### Discord

- Chỉ thử lại khi có lỗi giới hạn tốc độ (HTTP 429).
- Sử dụng `retry_after` của Discord khi có sẵn, nếu không sử dụng backoff theo cấp số nhân.

### Telegram

- Thử lại khi có lỗi tạm thời (429, timeout, connect/reset/closed, temporarily unavailable).
- Sử dụng `retry_after` khi có sẵn, nếu không sử dụng backoff theo cấp số nhân.
- Lỗi phân tích cú pháp Markdown không được thử lại; chúng quay lại văn bản thuần túy.
## Cấu hình

Đặt chính sách thử lại cho mỗi nhà cung cấp trong `~/.openclaw/openclaw.json`:

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```
## Ghi chú

- Các lần thử lại áp dụng cho mỗi yêu cầu (gửi tin nhắn, tải lên phương tiện, phản ứng, bình chọn, nhãn dán).
- Các luồng tổng hợp không thử lại các bước đã hoàn thành.