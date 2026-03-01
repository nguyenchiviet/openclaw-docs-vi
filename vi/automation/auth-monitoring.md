---
summary: Theo dõi thời hạn OAuth cho các nhà cung cấp mô hình
read_when:
  - Thiết lập giám sát hoặc cảnh báo hết hạn xác thực
  - Tự động hóa kiểm tra làm mới OAuth cho Claude Code / Codex
title: Giám sát Xác thực
x-i18n:
  source_path: automation\auth-monitoring.md
  source_hash: eef179af9545ed7ab881f3ccbef998869437fb50cdb4088de8da7223b614fa2b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:04:40.123Z'
---

# Giám sát xác thực

OpenClaw hiển thị tình trạng hết hạn OAuth qua `openclaw models status`. Sử dụng điều này cho
tự động hóa và cảnh báo; các script là tùy chọn bổ sung cho quy trình làm việc trên điện thoại.

## Ưu tiên: Kiểm tra CLI (di động)

```bash
openclaw models status --check
```

Exit codes:

- `0`: OK
- `1`: expired or missing credentials
- `2`: expiring soon (within 24h)

This works in cron/systemd and requires no extra scripts.

## Optional scripts (ops / phone workflows)

These live under `scripts/` and are **optional**. They assume SSH access to the
gateway host and are tuned for systemd + Termux.

- `scripts/claude-auth-status.sh` now uses `openclaw models status --json` as the
  source of truth (falling back to direct file reads if the CLI is unavailable),
  so keep `openclaw` on `PATH` for timers.
- `scripts/auth-monitor.sh`: cron/systemd timer target; sends alerts (ntfy or phone).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}`: systemd user timer.
- `scripts/claude-auth-status.sh`: Claude Code + OpenClaw auth checker (full/json/simple).
- `scripts/mobile-reauth.sh`: guided re‑auth flow over SSH.
- `scripts/termux-quick-auth.sh`: one‑tap widget status + open auth URL.
- `scripts/termux-auth-widget.sh`: full guided widget flow.
- `scripts/termux-sync-widget.sh`: đồng bộ thông tin xác thực Claude Code → OpenClaw.

Nếu bạn không cần tự động hóa điện thoại hoặc bộ hẹn giờ systemd, hãy bỏ qua các script này.