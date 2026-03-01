---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw qr` (tạo QR code ghép nối iOS + mã thiết
  lập)
read_when:
  - Bạn muốn ghép nối ứng dụng iOS với Gateway một cách nhanh chóng
  - Bạn cần đầu ra mã thiết lập cho chia sẻ từ xa/thủ công
title: >-
  I need more context to provide an accurate translation. "QR" typically refers
  to "QR code" (mã QR in Vietnamese), but could you provide the full text or
  sentence you'd like me to translate?
x-i18n:
  source_path: cli\qr.md
  source_hash: 8d01185290f091a635a4f87f6886d4d8942e627cbbc5b81a383499bb55b05710
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:04.490Z'
---

# `openclaw qr`

Tạo mã QR ghép nối iOS và mã thiết lập từ cấu hình Gateway hiện tại của bạn.

## Cách sử dụng

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## Options

- `--remote`: use `gateway.remote.url` plus remote token/password from config
- `--url <url>`: override gateway URL used in payload
- `--public-url <url>`: override public URL used in payload
- `--token <token>`: override gateway token for payload
- `--password <password>`: override gateway password for payload
- `--setup-code-only`: print only setup code
- `--no-ascii`: skip ASCII QR rendering
- `--json`: emit JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## Notes

- `--token` and `--password` are mutually exclusive.
- After scanning, approve device pairing with:
  - `openclaw devices list`
  - `openclaw devices approve <requestId>`