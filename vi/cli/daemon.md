---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw daemon` (bí danh cũ để quản lý dịch vụ
  gateway)
read_when:
  - Bạn vẫn sử dụng `openclaw daemon ...` trong các script
  - Bạn cần các lệnh vòng đời dịch vụ (install/start/stop/restart/status)
title: daemon (tiến trình nền)
x-i18n:
  source_path: cli\daemon.md
  source_hash: 290792d3d94a18dadcc62c0482685db3afa4d0ba11536622e894ac056c6f7843
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:23.650Z'
---

# `openclaw daemon`

Bí danh kế thừa cho các lệnh quản lý dịch vụ Gateway.

`openclaw daemon ...` ánh xạ tới cùng một bề mặt điều khiển dịch vụ như các lệnh dịch vụ `openclaw gateway ...`.

## Cách sử dụng

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## Subcommands

- `status`: show service install state and probe Gateway health
- `install`: install service (`launchd`/`systemd`/`schtasks`)
- `uninstall`: remove service
- `start`: start service
- `stop`: stop service
- `restart`: restart service

## Common options

- `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
- `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
- lifecycle (`uninstall|start|stop|restart`): `--json`

## Prefer

Use [`openclaw gateway`](/cli/gateway) để xem tài liệu và ví dụ hiện tại.