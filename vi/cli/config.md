---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw config` (lấy/đặt/bỏ đặt các giá trị cấu
  hình)
read_when:
  - Bạn muốn đọc hoặc chỉnh sửa cấu hình một cách không tương tác
title: cấu hình
x-i18n:
  source_path: cli\config.md
  source_hash: 2b03372059b4ffe6b8ab90ae05edec08003129927e03cf6e4e58f0139a1031a6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:21.235Z'
---

# `openclaw config`

Trợ giúp cấu hình: lấy/đặt/bỏ đặt giá trị theo đường dẫn. Chạy mà không có lệnh con để mở trình hướng dẫn cấu hình (giống như `openclaw configure`).

## Ví dụ

```bash
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
```

## Paths

Paths use dot or bracket notation:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Use the agent list index to target a specific agent:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## Values

Values are parsed as JSON5 when possible; otherwise they are treated as strings.
Use `--strict-json` to require JSON5 parsing. `--json` remains supported as a legacy alias.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

Khởi động lại Gateway sau khi chỉnh sửa.