---
summary: Sử dụng Qwen OAuth (miễn phí) trong OpenClaw
read_when:
  - Bạn muốn sử dụng Qwen với OpenClaw
  - Bạn muốn truy cập OAuth miễn phí cho Qwen Coder
title: Qwen
x-i18n:
  source_path: providers\qwen.md
  source_hash: 88b88e224e2fecbb1ca26e24fbccdbe25609be40b38335d0451343a5da53fdd4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:46.114Z'
---

# Qwen

Qwen cung cấp luồng OAuth miễn phí cho các mô hình Qwen Coder và Qwen Vision
(2.000 yêu cầu/ngày, tuân theo giới hạn tốc độ của Qwen).

## Bật plugin

```bash
openclaw plugins enable qwen-portal-auth
```

Restart the Gateway after enabling.

## Authenticate

```bash
openclaw models auth login --provider qwen-portal --set-default
```

This runs the Qwen device-code OAuth flow and writes a provider entry to your
`models.json` (plus a `qwen` alias for quick switching).

## Model IDs

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

Switch models with:

```bash
openclaw models set qwen-portal/coder-model
```

## Reuse Qwen Code CLI login

If you already logged in with the Qwen Code CLI, OpenClaw will sync credentials
from `~/.qwen/oauth_creds.json` when it loads the auth store. You still need a
`models.providers.qwen-portal` entry (use the login command above to create one).

## Notes

- Tokens auto-refresh; re-run the login command if refresh fails or access is revoked.
- Default base URL: `https://portal.qwen.ai/v1` (override with
  `models.providers.qwen-portal.baseUrl` nếu Qwen cung cấp một endpoint khác).
- Xem [Nhà cung cấp mô hình](/concepts/model-providers) để biết các quy tắc toàn cầu nhà cung cấp.