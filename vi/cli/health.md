---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw health` (điểm cuối health của gateway
  thông qua RPC)
read_when:
  - Bạn muốn nhanh chóng kiểm tra tình trạng sức khỏe của Gateway đang chạy
title: sức khỏe
x-i18n:
  source_path: cli\health.md
  source_hash: 82a78a5a97123f7a5736699ae8d793592a736f336c5caced9eba06d14d973fd7
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:45.427Z'
---

# `openclaw health`

Lấy trạng thái sức khỏe từ Gateway đang chạy.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes:

- `--verbose` chạy các bài kiểm tra trực tiếp và in thời gian cho từng tài khoản khi có nhiều tài khoản được cấu hình.
- Đầu ra bao gồm các kho lưu trữ phiên cho từng agent khi có nhiều agent được cấu hình.