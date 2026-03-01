---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw system` (sự kiện hệ thống, heartbeat,
  presence)
read_when:
  - Bạn muốn đưa một sự kiện hệ thống vào hàng đợi mà không cần tạo một cron job
  - Bạn cần bật hoặc tắt heartbeats
  - Bạn muốn kiểm tra các mục presence của hệ thống
title: hệ thống
x-i18n:
  source_path: cli\system.md
  source_hash: 36ae5dbdec327f5a32f7ef44bdc1f161bad69868de62f5071bb4d25a71bfdfe9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:41.235Z'
---

# `openclaw system`

Các công cụ hỗ trợ cấp hệ thống cho Gateway: xếp hàng các sự kiện hệ thống, kiểm soát nhịp tim,
và xem trạng thái hiện diện.

## Các lệnh phổ biến

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

Enqueue a system event on the **main** session. The next heartbeat will inject
it as a `Hệ thống:` line in the prompt. Use `--mode now` to trigger the heartbeat
immediately; `next-heartbeat` waits for the next scheduled tick.

Flags:

- `--text <text>`: required system event text.
- `--mode <mode>`: `now` or `next-heartbeat` (default).
- `--json`: machine-readable output.

## `system heartbeat last|enable|disable`

Heartbeat controls:

- `last`: show the last heartbeat event.
- `enable`: turn heartbeats back on (use this if they were disabled).
- `disable`: pause heartbeats.

Flags:

- `--json`: machine-readable output.

## `system presence`

List the current system presence entries the Gateway knows about (nodes,
instances, and similar status lines).

Flags:

- `--json`: đầu ra có thể đọc bằng máy.

## Ghi chú

- Yêu cầu một Gateway đang chạy có thể truy cập được bằng cấu hình hiện tại của bạn (cục bộ hoặc từ xa).
- Các sự kiện hệ thống là tạm thời và không được lưu trữ lại sau khi khởi động lại.