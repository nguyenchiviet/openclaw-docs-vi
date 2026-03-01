---
summary: Tài liệu tham khảo CLI cho `openclaw logs` (theo dõi nhật ký gateway qua RPC)
read_when:
  - Bạn cần theo dõi nhật ký Gateway từ xa (không cần SSH)
  - Bạn muốn các dòng nhật ký JSON cho công cụ
title: nhật ký
x-i18n:
  source_path: cli\logs.md
  source_hash: 81be02b6f8acad32ccf2d280827c7188a3c2f6bba0de5cbfa39fcc0bee3129cd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:46.302Z'
---

# `openclaw logs`

Theo dõi nhật ký tệp Gateway qua RPC (hoạt động ở chế độ từ xa).

Liên quan:

- Tổng quan về ghi nhật ký: [Logging](/logging)

## Ví dụ

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

Use `--local-time` để hiển thị dấu thời gian theo múi giờ địa phương của bạn.