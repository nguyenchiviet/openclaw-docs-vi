---
summary: >-
  Sử dụng API thống nhất của Kilo Gateway để truy cập nhiều mô hình trong
  OpenClaw
read_when:
  - Bạn muốn một khóa API duy nhất cho nhiều LLM
  - Bạn muốn chạy các mô hình thông qua Kilo Gateway trong OpenClaw
x-i18n:
  source_path: providers\kilocode.md
  source_hash: b7d054a277f236352baa35e7e0b8c3e2303e2ae92a8e75266e5ad4906727c4e7
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:59.884Z'
---

# Kilo Gateway

Kilo Gateway cung cấp một **API thống nhất** định tuyến các yêu cầu đến nhiều mô hình phía sau một điểm cuối duy nhất và khóa API. Nó tương thích với OpenAI, vì vậy hầu hết các SDK OpenAI hoạt động bằng cách chuyển đổi URL cơ sở.

## Lấy khóa API

1. Truy cập [app.kilo.ai](https://app.kilo.ai)
2. Đăng nhập hoặc tạo tài khoản
3. Điều hướng đến API Keys và tạo khóa mới

## Thiết lập CLI

```bash
openclaw onboard --kilocode-api-key <key>
```

Or set the environment variable:

```bash
export KILOCODE_API_KEY="your-api-key"
```

## Config snippet

```json5
{
  env: { KILOCODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kilocode/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Surfaced model refs

The built-in Kilo Gateway catalog currently surfaces these model refs:

- `kilocode/anthropic/claude-opus-4.6` (default)
- `kilocode/z-ai/glm-5:free`
- `kilocode/minimax/minimax-m2.5:free`
- `kilocode/anthropic/claude-sonnet-4.5`
- `kilocode/openai/gpt-5.2`
- `kilocode/google/gemini-3-pro-preview`
- `kilocode/google/gemini-3-flash-preview`
- `kilocode/x-ai/grok-code-fast-1`
- `kilocode/moonshotai/kimi-k2.5`

## Notes

- Model refs are `kilocode/<provider>/<model>` (e.g., `kilocode/anthropic/claude-opus-4.6`).
- Default model: `kilocode/anthropic/claude-opus-4.6`
- Base URL: `https://api.kilo.ai/api/gateway/`
- Để xem thêm các tùy chọn mô hình/nhà cung cấp, hãy xem [/concepts/model-providers](/concepts/model-providers).
- Kilo Gateway sử dụng mã thông báo Bearer với khóa API của bạn phía sau.