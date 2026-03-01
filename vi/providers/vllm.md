---
summary: Chạy OpenClaw với vLLM (máy chủ cục bộ tương thích OpenAI)
read_when:
  - Bạn muốn chạy OpenClaw với một máy chủ vLLM cục bộ
  - >-
    Bạn muốn các endpoint /v1 tương thích với OpenAI với các mô hình của riêng
    bạn
title: vLLM
x-i18n:
  source_path: providers\vllm.md
  source_hash: 47a7b4a302fa829dd9de2da048d6ecd0cea843b84acf92455653a900976216c3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:17:06.864Z'
---

# vLLM

vLLM có thể phục vụ các mô hình mã nguồn mở (và một số mô hình tùy chỉnh) thông qua API HTTP **tương thích với OpenAI**. OpenClaw có thể kết nối với vLLM bằng cách sử dụng API `openai-completions`.

OpenClaw cũng có thể **tự động khám phá** các mô hình có sẵn từ vLLM khi bạn chọn tham gia với `VLLM_API_KEY` (bất kỳ giá trị nào cũng hoạt động nếu máy chủ của bạn không thực thi xác thực) và bạn không xác định một mục `models.providers.vllm` rõ ràng.
## Bắt đầu nhanh

1. Khởi động vLLM với máy chủ tương thích OpenAI.

URL cơ sở của bạn phải hiển thị các điểm cuối `/v1` (ví dụ: `/v1/models`, `/v1/chat/completions`). vLLM thường chạy trên:

- `http://127.0.0.1:8000/v1`

2. Chọn tham gia (bất kỳ giá trị nào cũng hoạt động nếu không có xác thực được cấu hình):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Select a model (replace with one of your vLLM model IDs):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```
## Khám phá mô hình (nhà cung cấp ngầm định)

Khi `VLLM_API_KEY` được đặt (hoặc một hồ sơ xác thực tồn tại) và bạn **không** định nghĩa `models.providers.vllm`, OpenClaw sẽ truy vấn:

- `GET http://127.0.0.1:8000/v1/models`

…và chuyển đổi các ID được trả về thành các mục mô hình.

Nếu bạn đặt `models.providers.vllm` một cách rõ ràng, khám phá tự động sẽ bị bỏ qua và bạn phải định nghĩa các mô hình theo cách thủ công.
## Cấu hình rõ ràng (mô hình thủ công)

Sử dụng cấu hình rõ ràng khi:

- vLLM chạy trên một máy chủ/cổng khác.
- Bạn muốn ghim các giá trị `contextWindow`/`maxTokens`.
- Máy chủ của bạn yêu cầu một khóa API thực (hoặc bạn muốn kiểm soát các tiêu đề).

```json5
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local vLLM Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
## Khắc phục sự cố

- Kiểm tra máy chủ có thể truy cập được:

```bash
curl http://127.0.0.1:8000/v1/models
```

- If requests fail with auth errors, set a real `VLLM_API_KEY` that matches your server configuration, or configure the provider explicitly under `models.providers.vllm`.