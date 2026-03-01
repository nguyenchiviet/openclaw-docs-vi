---
summary: 'Xử lý múi giờ cho agents, envelopes và prompts'
read_when:
  - Bạn cần hiểu cách các dấu thời gian được chuẩn hóa cho mô hình
  - Cấu hình múi giờ người dùng cho các system prompts
title: Múi giờ
x-i18n:
  source_path: concepts\timezone.md
  source_hash: 9ee809c96897db1126c7efcaa5bf48a63cdcb2092abd4b3205af224ebd882766
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:51:49.334Z'
---

# Múi giờ

OpenClaw chuẩn hóa dấu thời gian để mô hình nhìn thấy **một thời gian tham chiếu duy nhất**.
## Bao thư tin nhắn (mặc định là cục bộ)

Các tin nhắn đến được bao bọc trong một bao thư như:

```
[Provider ... 2026-01-05 16:26 PST] message text
```

The timestamp in the envelope is **host-local by default**, with minutes precision.

You can override this with:

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
- `envelopeTimezone: "user"` uses `agents.defaults.userTimezone` (falls back to host timezone).
- Use an explicit IANA timezone (e.g., `"Europe/Vienna"`) for a fixed offset.
- `envelopeTimestamp: "off"` removes absolute timestamps from envelope headers.
- `envelopeElapsed: "off"` removes elapsed time suffixes (the `+2m` style).

### Examples

**Local (default):**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**Fixed timezone:**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**Elapsed time:**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```
## Tải trọng công cụ (dữ liệu nhà cung cấp thô + các trường được chuẩn hóa)

Các lệnh gọi công cụ (`channels.discord.readMessages`, `channels.slack.readMessages`, v.v.) trả về **dấu thời gian nhà cung cấp thô**.
Chúng tôi cũng đính kèm các trường được chuẩn hóa để đảm bảo tính nhất quán:

- `timestampMs` (mili giây epoch UTC)
- `timestampUtc` (chuỗi UTC ISO 8601)

Các trường nhà cung cấp thô được bảo tồn.
## Múi giờ người dùng cho lời nhắc hệ thống

Đặt `agents.defaults.userTimezone` để cho mô hình biết múi giờ địa phương của người dùng. Nếu không được đặt, OpenClaw sẽ phân giải **múi giờ máy chủ lúc chạy** (không ghi cấu hình).

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

The system prompt includes:

- `Ngày & Giờ hiện tại` section with local time and timezone
- `Định dạng giờ: 12 giờ` or `24 giờ`

You can control the prompt format with `agents.defaults.timeFormat` (`auto` | `12` | `24`).

Xem [Ngày & Giờ](/date-time) để biết hành vi đầy đủ và các ví dụ.