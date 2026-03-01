---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw voicecall` (bề mặt lệnh plugin
  voice-call)
read_when:
  - Bạn sử dụng plugin voice-call và muốn các điểm vào CLI
  - Bạn muốn các ví dụ nhanh cho `voicecall call|continue|status|tail|expose`
title: cuộc gọi thoại
x-i18n:
  source_path: cli\voicecall.md
  source_hash: 2c99e7a3d256e1c74a0f07faba9675cc5a88b1eb2fc6e22993caf3874d4f340a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:48.358Z'
---

# `openclaw voicecall`

`voicecall` là một lệnh do plugin cung cấp. Nó chỉ xuất hiện nếu plugin voice-call được cài đặt và bật.

Tài liệu chính:

- Plugin Voice-call: [Voice Call](/plugins/voice-call)

## Các lệnh thường dùng

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Exposing webhooks (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

Lưu ý bảo mật: chỉ công khai webhook endpoint cho các mạng mà bạn tin tưởng. Ưu tiên Tailscale Serve hơn Funnel khi có thể.