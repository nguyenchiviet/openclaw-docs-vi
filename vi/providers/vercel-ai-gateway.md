---
title: Vercel AI Gateway
summary: Thiết lập Vercel AI Gateway (xác thực + lựa chọn mô hình)
read_when:
  - Bạn muốn sử dụng Vercel AI Gateway với OpenClaw
  - Bạn cần biến môi trường API key hoặc lựa chọn xác thực CLI
x-i18n:
  source_path: providers\vercel-ai-gateway.md
  source_hash: fc2235ac9f59e56d2c628c8b61f94fad51d2cd764c01d6bf7b1da2ff0b26cc3d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:17:01.136Z'
---

# Vercel AI Gateway

[Vercel AI Gateway](https://vercel.com/ai-gateway) cung cấp một API thống nhất để truy cập hàng trăm mô hình thông qua một điểm cuối duy nhất.

- Nhà cung cấp: `vercel-ai-gateway`
- Xác thực: `AI_GATEWAY_API_KEY`
- API: Tương thích với Anthropic Messages

## Bắt đầu nhanh

1. Đặt khóa API (khuyến nghị: lưu trữ nó cho Gateway):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Set a default model:

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Non-interactive example

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## Environment note

If the Gateway runs as a daemon (launchd/systemd), make sure `AI_GATEWAY_API_KEY`
is available to that process (for example, in `~/.openclaw/.env` or via
`env.shellEnv`).

## Model ID shorthand

OpenClaw accepts Vercel Claude shorthand model refs and normalizes them at
runtime:

- `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
- `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`