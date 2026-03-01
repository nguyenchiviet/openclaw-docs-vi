---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw reset` (đặt lại trạng thái/cấu hình cục
  bộ)
read_when:
  - Bạn muốn xóa trạng thái cục bộ trong khi giữ CLI được cài đặt
  - Bạn muốn thực hiện một lần chạy thử để xem những gì sẽ bị xóa
title: đặt lại
x-i18n:
  source_path: cli\reset.md
  source_hash: 08afed5830f892e07d6e2e167f09aaf2d79fd5b2ba2a26a65dca857ebdbf873c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:02.711Z'
---

# `openclaw reset`

Đặt lại cấu hình/trạng thái cục bộ (giữ CLI được cài đặt).

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```