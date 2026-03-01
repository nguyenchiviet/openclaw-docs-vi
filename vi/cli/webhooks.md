---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw webhooks` (trợ giúp webhook + Gmail
  Pub/Sub)
read_when:
  - Bạn muốn kết nối các sự kiện Gmail Pub/Sub vào OpenClaw
  - Bạn muốn các lệnh trợ giúp webhook
title: webhooks
x-i18n:
  source_path: cli\webhooks.md
  source_hash: 785ec62afe6631b340ce4a4541ceb34cd6b97704cf7a9889762cb4c1f29a5ca0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:47.941Z'
---

# `openclaw webhooks`

Trợ giúp webhook và tích hợp (Gmail Pub/Sub, trợ giúp webhook).

Liên quan:

- Webhooks: [Webhook](/automation/webhook)
- Gmail Pub/Sub: [Gmail Pub/Sub](/automation/gmail-pubsub)

## Gmail

```bash
openclaw webhooks gmail setup --account you@example.com
openclaw webhooks gmail run
```

Xem [tài liệu Gmail Pub/Sub](/automation/gmail-pubsub) để biết chi tiết.