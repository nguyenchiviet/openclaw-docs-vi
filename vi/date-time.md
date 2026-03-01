---
summary: 'Xử lý ngày tháng và thời gian trên các envelope, prompt, tool và connector'
read_when:
  - Bạn đang thay đổi cách hiển thị dấu thời gian cho mô hình hoặc người dùng
  - >-
    Bạn đang gỡ lỗi định dạng thời gian trong các tin nhắn hoặc đầu ra system
    prompt
title: Ngày và Giờ
x-i18n:
  source_path: date-time.md
  source_hash: 753af5946a006215d6af2467fa478f3abb42b1dff027cf85d5dc4c7ba4b58d39
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:34.289Z'
---

# Ngày & Giờ

OpenClaw mặc định sử dụng **giờ địa phương của máy chủ cho dấu thời gian giao thức truyền tải** và **múi giờ của người dùng chỉ trong lời nhắc hệ thống**.
Dấu thời gian của nhà cung cấp được bảo toàn để các công cụ giữ ngữ nghĩa gốc của chúng (thời gian hiện tại có sẵn qua `session_status`).
## Bao thư tin nhắn (mặc định là cục bộ)

Các tin nhắn đến được bao bọc với một dấu thời gian (độ chính xác theo phút):

```
[Provider ... 2026-01-05 16:26 PST] message text
```

This envelope timestamp is **host-local by default**, regardless of the provider timezone.

You can override this behavior:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA timezone
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

- `envelopeTimezone: "utc"` uses UTC.
- `envelopeTimezone: "local"` uses the host timezone.
- `envelopeTimezone: "user"` uses `agents.defaults.userTimezone` (falls back to host timezone).
- Use an explicit IANA timezone (e.g., `"America/Chicago"`) for a fixed zone.
- `envelopeTimestamp: "off"` removes absolute timestamps from envelope headers.
- `envelopeElapsed: "off"` removes elapsed time suffixes (the `+2m` style).

### Examples

**Local (default):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**User timezone:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Elapsed time enabled:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```
## System prompt: Current Date & Time

Nếu múi giờ của người dùng được biết, system prompt bao gồm một phần **Current Date & Time** chuyên dụng với **múi giờ duy nhất** (không có định dạng đồng hồ/thời gian) để giữ cho prompt caching ổn định:

```
Time zone: America/Chicago
```

When the agent needs the current time, use the `session_status` tool; thẻ trạng thái bao gồm một dòng dấu thời gian.
## Dòng sự kiện hệ thống (mặc định là cục bộ)

Các sự kiện hệ thống được xếp hàng được chèn vào ngữ cảnh agent được đặt tiền tố bằng dấu thời gian sử dụng
cùng lựa chọn múi giờ như các phong bì tin nhắn (mặc định: cục bộ máy chủ).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

### Configure user timezone + format

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

- `userTimezone` sets the **user-local timezone** for prompt context.
- `timeFormat` controls **12h/24h display** in the prompt. `auto` tuân theo tùy chọn của hệ điều hành.
## Phát hiện định dạng thời gian (tự động)

When `timeFormat: "auto"`, OpenClaw kiểm tra tùy chọn hệ điều hành (macOS/Windows)
và quay lại định dạng theo ngôn ngữ. Giá trị được phát hiện được **lưu vào bộ nhớ đệm cho mỗi quy trình**
để tránh các lệnh gọi hệ thống lặp lại.
## Tải trọng công cụ + kết nối (thời gian nhà cung cấp thô + các trường được chuẩn hóa)

Các công cụ kênh trả về **dấu thời gian gốc của nhà cung cấp** và thêm các trường được chuẩn hóa để đảm bảo tính nhất quán:

- `timestampMs`: epoch milliseconds (UTC)
- `timestampUtc`: ISO 8601 UTC string

Các trường nhà cung cấp thô được bảo toàn để không mất dữ liệu.

- Slack: chuỗi kiểu epoch từ API
- Discord: dấu thời gian ISO UTC
- Telegram/WhatsApp: dấu thời gian số/ISO dành riêng cho nhà cung cấp

Nếu bạn cần thời gian địa phương, hãy chuyển đổi nó ở phía hạ lưu bằng múi giờ đã biết.
## Tài liệu liên quan

- [System Prompt](/concepts/system-prompt)
- [Múi giờ](/concepts/timezone)
- [Tin nhắn](/concepts/messages)