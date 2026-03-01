---
summary: Sử dụng Z.AI (các mô hình GLM) với OpenClaw
read_when:
  - Bạn muốn các mô hình Z.AI / GLM trong OpenClaw
  - Bạn cần một thiết lập ZAI_API_KEY đơn giản
title: Z.AI
x-i18n:
  source_path: providers\zai.md
  source_hash: e3db40cb27b48179b7eaf5fb8fadaec6340f99616f199df44ffbb79372c53659
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:17:49.899Z'
---

# Z.AI

Z.AI là nền tảng API cho các mô hình **GLM**. Nó cung cấp các REST API cho GLM và sử dụng các khóa API
để xác thực. Tạo khóa API của bạn trong bảng điều khiển Z.AI. OpenClaw sử dụng nhà cung cấp `zai`
với khóa API Z.AI.

## Thiết lập CLI

```bash
openclaw onboard --auth-choice zai-api-key
# or non-interactive
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Config snippet

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

- GLM models are available as `zai/<model>` (example: `zai/glm-5`).
- `tool_stream` is enabled by default for Z.AI tool-call streaming. Set
  `agents.defaults.models["zai/<model>"].params.tool_stream` to `false` để vô hiệu hóa nó.
- Xem [/providers/glm](/providers/glm) để xem tổng quan về họ mô hình.
- Z.AI sử dụng xác thực Bearer với khóa API của bạn.