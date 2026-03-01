---
title: Cloudflare AI Gateway
summary: Thiết lập Cloudflare AI Gateway (xác thực + lựa chọn mô hình)
read_when:
  - Bạn muốn sử dụng Cloudflare AI Gateway với OpenClaw
  - 'Bạn cần ID tài khoản, ID gateway hoặc biến môi trường API key'
x-i18n:
  source_path: providers\cloudflare-ai-gateway.md
  source_hash: db77652c37652ca20f7c50f32382dbaeaeb50ea5bdeaf1d4fd17dc394e58950c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:39.876Z'
---

# Cloudflare AI Gateway

Cloudflare AI Gateway nằm phía trước các API của nhà cung cấp và cho phép bạn thêm phân tích, bộ nhớ đệm và các điều khiển. Đối với Anthropic, OpenClaw sử dụng Anthropic Messages API thông qua điểm cuối Gateway của bạn.

- Nhà cung cấp: `cloudflare-ai-gateway`
- URL cơ sở: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
- Mô hình mặc định: `cloudflare-ai-gateway/claude-sonnet-4-5`
- Khóa API: `CLOUDFLARE_AI_GATEWAY_API_KEY` (khóa API nhà cung cấp của bạn cho các yêu cầu thông qua Gateway)

Đối với các mô hình Anthropic, hãy sử dụng khóa API Anthropic của bạn.
## Bắt đầu nhanh

1. Đặt khóa API của nhà cung cấp và chi tiết Gateway:

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2. Set a default model:

```json5
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```
## Ví dụ không tương tác

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```
## Gateway được xác thực

Nếu bạn đã bật xác thực Gateway trong Cloudflare, hãy thêm header `cf-aig-authorization` (điều này ngoài khóa API của nhà cung cấp của bạn).

```json5
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```
## Ghi chú về môi trường

Nếu Gateway chạy như một daemon (launchd/systemd), hãy đảm bảo `CLOUDFLARE_AI_GATEWAY_API_KEY` có sẵn cho quá trình đó (ví dụ: trong `~/.openclaw/.env` hoặc thông qua `env.shellEnv`).