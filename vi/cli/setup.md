---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw setup` (khởi tạo cấu hình + không gian
  làm việc)
read_when:
  - >-
    Bạn đang thực hiện thiết lập lần đầu tiên mà không có trình hướng dẫn
    onboarding đầy đủ
  - Bạn muốn đặt đường dẫn workspace mặc định
title: thiết lập
x-i18n:
  source_path: cli\setup.md
  source_hash: 7f3fc8b246924edf48501785be2c0d356bd31bfbb133e75a139a5ee41dbf57f4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:16.938Z'
---

# `openclaw setup`

Khởi tạo `~/.openclaw/openclaw.json` và không gian làm việc của agent.

Liên quan:

- Bắt đầu: [Bắt đầu](/start/getting-started)
- Trình hướng dẫn: [Thiết lập ban đầu](/start/onboarding)

## Ví dụ

```bash
openclaw setup
openclaw setup --workspace ~/.openclaw/workspace
```

To run the wizard via setup:

```bash
openclaw setup --wizard
```