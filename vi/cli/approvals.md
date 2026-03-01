---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw approvals` (thực thi phê duyệt cho
  gateway hoặc các máy chủ node)
read_when:
  - Bạn muốn chỉnh sửa phê duyệt exec từ CLI
  - Bạn cần quản lý danh sách cho phép trên các máy chủ gateway hoặc node
title: phê duyệt
x-i18n:
  source_path: cli\approvals.md
  source_hash: 4329cdaaec2c5f5d619415b6431196512d4834dc1ccd7363576f03dd9b845130
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:31:30.465Z'
---

# `openclaw approvals`

Quản lý phê duyệt exec cho **máy chủ cục bộ**, **máy chủ gateway**, hoặc **máy chủ node**.
Theo mặc định, các lệnh nhắm đến tệp phê duyệt cục bộ trên đĩa. Sử dụng `--gateway` để nhắm đến gateway, hoặc `--node` để nhắm đến một node cụ thể.

Liên quan:

- Phê duyệt Exec: [Phê duyệt Exec](/tools/exec-approvals)
- Nodes: [Nodes](/nodes)

## Lệnh thông dụng

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## Replace approvals from a file

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## Allowlist helpers

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## Notes

- `--node` uses the same resolver as `openclaw nodes` (id, name, ip, or id prefix).
- `--agent` defaults to `"*"`, which applies to all agents.
- The node host must advertise `system.execApprovals.get/set` (macOS app or headless node host).
- Approvals files are stored per host at `~/.openclaw/exec-approvals.json`.