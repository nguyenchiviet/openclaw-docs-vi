---
summary: Sử dụng API thống nhất của OpenRouter để truy cập nhiều mô hình trong OpenClaw
read_when:
  - Bạn muốn một khóa API duy nhất cho nhiều LLM
  - Bạn muốn chạy các mô hình thông qua OpenRouter trong OpenClaw
title: OpenRouter
x-i18n:
  source_path: providers\openrouter.md
  source_hash: b7e29fc9c456c64d567dd909a85166e6dea8388ebd22155a31e69c970e081586
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:45.612Z'
---

# OpenRouter

OpenRouter cung cấp một **API thống nhất** định tuyến các yêu cầu đến nhiều mô hình phía sau một điểm cuối duy nhất và khóa API. Nó tương thích với OpenAI, vì vậy hầu hết các SDK OpenAI hoạt động bằng cách chuyển đổi URL cơ sở.

## Thiết lập CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Config snippet

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Notes

- Model refs are `openrouter/<provider>/<model>`.
- Để xem thêm các tùy chọn mô hình/nhà cung cấp, hãy xem [/concepts/model-providers](/concepts/model-providers).
- OpenRouter sử dụng mã thông báo Bearer với khóa API của bạn phía sau.