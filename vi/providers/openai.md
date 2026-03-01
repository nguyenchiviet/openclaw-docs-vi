---
summary: Sử dụng OpenAI thông qua API keys hoặc Codex subscription trong OpenClaw
read_when:
  - Bạn muốn sử dụng các mô hình OpenAI trong OpenClaw
  - Bạn muốn xác thực qua Codex subscription thay vì API keys
title: OpenAI
x-i18n:
  source_path: providers\openai.md
  source_hash: 063c5107f97dd272c538bc829a394120a5dca4d8952b2fdca86826e0a2602aa7
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:34.697Z'
---

# OpenAI

OpenAI cung cấp các API dành cho nhà phát triển cho các mô hình GPT. Codex hỗ trợ **đăng nhập ChatGPT** để truy cập theo đăng ký hoặc **đăng nhập bằng khóa API** để truy cập dựa trên mức sử dụng. Codex cloud yêu cầu đăng nhập ChatGPT.
## Tùy chọn A: Khóa API OpenAI (Nền tảng OpenAI)

**Tốt nhất cho:** truy cập API trực tiếp và thanh toán theo mức sử dụng.
Lấy khóa API của bạn từ bảng điều khiển OpenAI.

### Thiết lập CLI

```bash
openclaw onboard --auth-choice openai-api-key
# or non-interactive
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### Config snippet

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```
## Tùy chọn B: Đăng ký OpenAI Code (Codex)

**Phù hợp nhất cho:** sử dụng quyền truy cập đăng ký ChatGPT/Codex thay vì khóa API.
Codex cloud yêu cầu đăng nhập ChatGPT, trong khi CLI Codex hỗ trợ đăng nhập ChatGPT hoặc khóa API.

### Thiết lập CLI (Codex OAuth)

```bash
# Run Codex OAuth in the wizard
openclaw onboard --auth-choice openai-codex

# Or run OAuth directly
openclaw models auth login --provider openai-codex
```

### Config snippet (Codex subscription)

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### Codex transport default

OpenClaw uses `pi-ai` for model streaming. For `openai-codex/*` models you can set
`agents.defaults.models.<provider/model>.params.transport` to select transport:

- Default is `"auto"` (WebSocket-first, then SSE fallback).
- `"sse"`: force SSE
- `"websocket"`: force WebSocket
- `"auto"`: try WebSocket, then fall back to SSE

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.3-codex" },
      models: {
        "openai-codex/gpt-5.3-codex": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### OpenAI Responses server-side compaction

For direct OpenAI Responses models (`openai/*` using `api: "openai-responses"` with
`baseUrl` on `api.openai.com`), OpenClaw now auto-enables OpenAI server-side
compaction payload hints:

- Forces `store: true` (unless model compat sets `supportsStore: false`)
- Injects `context_management: [{ type: "compaction", compact_threshold: ... }]`

By default, `compact_threshold` is `70%` of model `contextWindow` (or `80000`
khi không có sẵn).

### Bật nén phía máy chủ một cách rõ ràng

Sử dụng điều này khi bạn muốn buộc `context_management` injection trên các mô hình Responses tương thích (ví dụ Azure OpenAI Responses):

```json5
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-4o": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### Enable with a custom threshold

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### Disable server-side compaction

```json5
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```
`responsesServerCompaction` chỉ kiểm soát việc tiêm `context_management`.
Các mô hình Direct OpenAI Responses vẫn buộc `store: true` trừ khi compat đặt
`supportsStore: false`.
## Ghi chú

- Các tham chiếu mô hình luôn sử dụng `provider/model` (xem [/concepts/models](/concepts/models)).
- Chi tiết xác thực + quy tắc tái sử dụng nằm trong [/concepts/oauth](/concepts/oauth).