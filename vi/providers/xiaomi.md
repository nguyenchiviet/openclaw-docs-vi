---
summary: Sử dụng Xiaomi MiMo (mimo-v2-flash) với OpenClaw
read_when:
  - Bạn muốn các mô hình Xiaomi MiMo trong OpenClaw
  - Bạn cần thiết lập XIAOMI_API_KEY
title: Xiaomi MiMo
x-i18n:
  source_path: providers\xiaomi.md
  source_hash: 366fd2297b2caf8c5ad944d7f1b6d233b248fe43aedd22a28352ae7f370d2435
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:17:02.254Z'
---

# Xiaomi MiMo

Xiaomi MiMo là nền tảng API cho các mô hình **MiMo**. Nó cung cấp các REST API tương thích với
các định dạng của OpenAI và Anthropic và sử dụng các khóa API để xác thực. Tạo khóa API của bạn trong
[bảng điều khiển Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw sử dụng
nhà cung cấp `xiaomi` với khóa API Xiaomi MiMo.

## Tổng quan mô hình

- **mimo-v2-flash**: cửa sổ ngữ cảnh 262144 token, tương thích với Anthropic Messages API.
- URL cơ sở: `https://api.xiaomimimo.com/anthropic`
- Xác thực: `Bearer $XIAOMI_API_KEY`

## Thiết lập CLI

```bash
openclaw onboard --auth-choice xiaomi-api-key
# or non-interactive
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## Config snippet

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Notes

- Model ref: `xiaomi/mimo-v2-flash`.
- The provider is injected automatically when `XIAOMI_API_KEY` được đặt (hoặc tồn tại một hồ sơ xác thực).
- Xem [/concepts/model-providers](/concepts/model-providers) để biết các quy tắc nhà cung cấp.