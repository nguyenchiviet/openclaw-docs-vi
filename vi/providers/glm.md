---
summary: Tổng quan về họ mô hình GLM + cách sử dụng trong OpenClaw
read_when:
  - Bạn muốn GLM models trong OpenClaw
  - Bạn cần quy ước đặt tên model và thiết lập.
title: Các mô hình GLM
x-i18n:
  source_path: providers\glm.md
  source_hash: e9c1671c4234c22edddbe23efc8f8cb44541718afb7666d5da49ac4313082c4b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:15:17.063Z'
---

# Các mô hình GLM

GLM là một **họ mô hình** (không phải là một công ty) có sẵn thông qua nền tảng Z.AI. Trong OpenClaw, các mô hình GLM được truy cập thông qua nhà cung cấp `zai` và các ID mô hình như `zai/glm-5`.

## Thiết lập CLI

```bash
openclaw onboard --auth-choice zai-api-key
```

## Config snippet

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

- GLM versions and availability can change; check Z.AI's docs for the latest.
- Example model IDs include `glm-5`, `glm-4.7`, and `glm-4.6`.
- Để biết chi tiết về nhà cung cấp, xem [/providers/zai](/providers/zai).
