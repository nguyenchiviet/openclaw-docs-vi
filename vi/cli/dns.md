---
summary: Tài liệu tham khảo CLI cho `openclaw dns` (trợ giúp khám phá vùng rộng)
read_when:
  - Bạn muốn khám phá trên diện rộng (DNS-SD) thông qua Tailscale + CoreDNS
  - You’re setting up split DNS for a custom discovery domain (example: openclaw.internal)
title: DNS
x-i18n:
  source_path: cli\dns.md
  source_hash: d2011e41982ffb4b71ab98211574529bc1c8b7769ab1838abddd593f42b12380
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:30.470Z'
---

# `openclaw dns`

Trợ giúp DNS cho khám phá thiết bị trên diện rộng (Tailscale + CoreDNS). Hiện tập trung vào macOS + Homebrew CoreDNS.

Liên quan:

- Khám phá thiết bị Gateway: [Khám phá](/gateway/discovery)
- Cấu hình khám phá trên diện rộng: [Cấu hình](/gateway/configuration)

## Thiết lập

```bash
openclaw dns setup
openclaw dns setup --apply
```