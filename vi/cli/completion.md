---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw completion` (tạo/cài đặt các tập lệnh
  hoàn thành shell)
read_when:
  - Bạn muốn hoàn thành shell cho zsh/bash/fish/PowerShell
  - Bạn cần lưu trữ các tập lệnh hoàn thành dưới trạng thái OpenClaw
title: hoàn thành
x-i18n:
  source_path: cli\completion.md
  source_hash: 7bbf140a880bafdb7140149f85465d66d0d46e5a3da6a1e41fb78be2fd2bd4d0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:33:59.309Z'
---

# `openclaw completion`

Tạo các tập lệnh hoàn thành shell và tùy chọn cài đặt chúng vào hồ sơ shell của bạn.

## Cách sử dụng

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## Options

- `-s, --shell <shell>`: shell target (`zsh`, `bash`, `powershell`, `fish`; default: `zsh`)
- `-i, --install`: install completion by adding a source line to your shell profile
- `--write-state`: write completion script(s) to `$OPENCLAW_STATE_DIR/completions` without printing to stdout
- `-y, --yes`: skip install confirmation prompts

## Notes

- `--install` writes a small "OpenClaw Completion" block into your shell profile and points it at the cached script.
- Without `--install` or `--write-state`, lệnh in tập lệnh ra stdout.
- Việc tạo hoàn thành tải sẵn các cây lệnh để các lệnh con lồng nhau được bao gồm.