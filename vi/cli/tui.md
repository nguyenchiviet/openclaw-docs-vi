---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw tui` (giao diện người dùng terminal kết
  nối với Gateway)
read_when:
  - >-
    Bạn muốn một giao diện người dùng terminal cho Gateway (thân thiện với truy
    cập từ xa)
  - Bạn muốn truyền url/token/session từ các script
title: >-
  I'm ready to translate English text to Vietnamese. Please provide the English
  text you'd like me to translate.
x-i18n:
  source_path: cli\tui.md
  source_hash: aa6f37b960926997a3c80f8d429e0b1c634dfa622da224b0a70ca79f15701c91
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:38.486Z'
---

# `openclaw tui`

Mở giao diện người dùng terminal được kết nối với Gateway.

Liên quan:

- Hướng dẫn TUI: [TUI](/web/tui)

## Ví dụ

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```