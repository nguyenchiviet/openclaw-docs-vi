---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw doctor` (kiểm tra sức khỏe + sửa chữa
  hướng dẫn)
read_when:
  - Bạn gặp vấn đề về kết nối/xác thực và muốn được hướng dẫn khắc phục
  - Bạn đã cập nhật và muốn kiểm tra lại xem có vấn đề gì không.
title: bác sĩ
x-i18n:
  source_path: cli\doctor.md
  source_hash: 40f852b3b5ecc5894816ddd6f82756531c5644216e19048373de19934eaba3f5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:31.559Z'
---

# `openclaw doctor`

Kiểm tra sức khỏe + sửa chữa nhanh cho Gateway và các kênh.

Liên quan:

- Khắc phục sự cố: [Khắc phục sự cố](/gateway/troubleshooting)
- Kiểm tra bảo mật: [Bảo mật](/gateway/security)

## Ví dụ

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notes:

- Interactive prompts (like keychain/OAuth fixes) only run when stdin is a TTY and `--non-interactive` is **not** set. Headless runs (cron, Telegram, no terminal) will skip prompts.
- `--fix` (alias for `--repair`) writes a backup to `~/.openclaw/openclaw.json.bak` and drops unknown config keys, listing each removal.
- State integrity checks now detect orphan transcript files in the sessions directory and can archive them as `.deleted.<timestamp>` to reclaim space safely.
- Doctor includes a memory-search readiness check and can recommend `openclaw configure --section model` when embedding credentials are missing.
- If sandbox mode is enabled but Docker is unavailable, doctor reports a high-signal warning with remediation (`install Docker` or `openclaw config set agents.defaults.sandbox.mode off`).

## macOS: `launchctl` env overrides

If you previously ran `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (or `...PASSWORD`), that value overrides your config file and can cause persistent “unauthorized” errors.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```