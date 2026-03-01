---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw pairing` (phê duyệt/liệt kê các yêu cầu
  ghép nối)
read_when:
  - Bạn đang sử dụng DM ở chế độ ghép đôi và cần phê duyệt những người gửi
title: ghép nối
x-i18n:
  source_path: cli\pairing.md
  source_hash: 266732af69e57b8849ddc9963426902f60e81daed6e5a80ef4ed5b7923ffa9e2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:02.962Z'
---

# `openclaw pairing`

Phê duyệt hoặc kiểm tra các yêu cầu ghép nối tin nhắn riêng (cho các kênh hỗ trợ ghép nối).

Liên quan:

- Luồng ghép nối: [Pairing](/channels/pairing)

## Commands

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## Notes

- Channel input: pass it positionally (`pairing list telegram`) or with `--channel <channel>`.
- `pairing list` supports `--account <accountId>` for multi-account channels.
- `pairing approve` supports `--account <accountId>` and `--notify`.
- If only one pairing-capable channel is configured, `pairing approve <code>` được phép.