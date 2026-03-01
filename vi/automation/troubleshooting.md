---
summary: Khắc phục sự cố lập lịch và gửi cron và heartbeat
read_when:
  - Cron không chạy
  - Cron đã chạy nhưng không có tin nhắn nào được gửi
  - Nhịp tim có vẻ im lặng hoặc bị bỏ qua
title: Khắc phục sự cố Tự động hóa
x-i18n:
  source_path: automation\troubleshooting.md
  source_hash: da9ccbd94651fedba573f4f344c1981d6e38d2023c19edd737dc71ce35a4bac2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:12:29.452Z'
---

# Khắc phục sự cố tự động hóa

Sử dụng trang này để khắc phục các vấn đề về bộ lập lịch và giao hàng (`cron` + `heartbeat`).
## Thang lệnh

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then run automation checks:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```
## Cron không kích hoạt

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Good output looks like:

- `cron status` reports enabled and a future `nextWakeAtMs`.
- Job is enabled and has a valid schedule/timezone.
- `cron runs` shows `ok` or explicit skip reason.

Common signatures:

- `cron: scheduler disabled; jobs will not run automatically` → cron disabled in config/env.
- `cron: timer tick failed` → scheduler tick crashed; inspect surrounding stack/log context.
- `reason: not-due` in run output → manual run called without `--force` và công việc chưa đến hạn.
## Cron đã kích hoạt nhưng không có giao hàng

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

Good output looks like:

- Run status is `ok`.
- Delivery mode/target are set for isolated jobs.
- Channel probe reports target channel connected.

Common signatures:

- Run succeeded but delivery mode is `none` → no external message is expected.
- Delivery target missing/invalid (`channel`/`to`) → run may succeed internally but skip outbound.
- Channel auth errors (`unauthorized`, `missing_scope`, `Forbidden`) → việc giao hàng bị chặn bởi thông tin xác thực/quyền hạn của kênh.
## Heartbeat bị chặn hoặc bỏ qua

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

Good output looks like:

- Heartbeat enabled with non-zero interval.
- Last heartbeat result is `ran` (or skip reason is understood).

Common signatures:

- `heartbeat skipped` with `reason=quiet-hours` → outside `activeHours`.
- `requests-in-flight` → main lane busy; heartbeat deferred.
- `empty-heartbeat-file` → interval heartbeat skipped because `HEARTBEAT.md` has no actionable content and no tagged cron event is queued.
- `alerts-disabled` → cài đặt hiển thị chặn các tin nhắn heartbeat gửi đi.
## Những lưu ý về múi giờ và activeHours

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

Quick rules:

- `Config path not found: agents.defaults.userTimezone` means the key is unset; heartbeat falls back to host timezone (or `activeHours.timezone` if set).
- Cron without `--tz` uses gateway host timezone.
- Heartbeat `activeHours` uses configured timezone resolution (`user`, `local`, or explicit IANA tz).
- ISO timestamps without timezone are treated as UTC for cron `at` schedules.

Common signatures:

- Jobs run at the wrong wall-clock time after host timezone changes.
- Heartbeat always skipped during your daytime because `activeHours.timezone` bị sai.

Liên quan:

- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)
- [/automation/cron-vs-heartbeat](/automation/cron-vs-heartbeat)
- [/concepts/timezone](/concepts/timezone)