---
summary: >-
  Chạy OpenClaw trên các LLM cục bộ (LM Studio, vLLM, LiteLLM, custom OpenAI
  endpoints)
read_when:
  - Bạn muốn phục vụ các mô hình từ GPU box của riêng bạn
  - Bạn đang kết nối LM Studio hoặc một proxy tương thích OpenAI
  - Bạn cần hướng dẫn về mô hình cục bộ an toàn nhất
title: Các Mô Hình Cục Bộ
x-i18n:
  source_path: gateway\local-models.md
  source_hash: 82164e8c4f0c74797a6d3da784e5cc494b5bc419169a27fc21a588aa8c9e569a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:57:34.672Z'
---

# Các mô hình cục bộ

Chạy cục bộ là khả thi, nhưng OpenClaw yêu cầu ngữ cảnh lớn + các biện pháp phòng chống mạnh mẽ chống lại prompt injection. Các card nhỏ sẽ cắt ngắn ngữ cảnh và làm rò rỉ an toàn. Hướng tới mục tiêu cao: **≥2 Mac Studios được tối ưu hóa tối đa hoặc thiết bị GPU tương đương (~$30k+)**. Một GPU **24 GB** duy nhất chỉ hoạt động cho các prompt nhẹ hơn với độ trễ cao hơn. Sử dụng **biến thể mô hình lớn nhất / đầy đủ mà bạn có thể chạy**; các checkpoint được lượng tử hóa tích cực hoặc "nhỏ" sẽ tăng rủi ro prompt injection (xem [Security](/gateway/security)).
## Khuyến nghị: LM Studio + MiniMax M2.1 (Responses API, kích thước đầy đủ)

Ngăn xếp cục bộ tốt nhất hiện tại. Tải MiniMax M2.1 trong LM Studio, bật máy chủ cục bộ (mặc định `http://127.0.0.1:1234`), và sử dụng Responses API để giữ lý luận riêng biệt với văn bản cuối cùng.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**Setup checklist**

- Install LM Studio: [https://lmstudio.ai](https://lmstudio.ai)
- In LM Studio, download the **largest MiniMax M2.1 build available** (avoid “small”/heavily quantized variants), start the server, confirm `http://127.0.0.1:1234/v1/models` lists it.
- Keep the model loaded; cold-load adds startup latency.
- Adjust `contextWindow`/`maxTokens` if your LM Studio build differs.
- For WhatsApp, stick to Responses API so only final text is sent.

Keep hosted models configured even when running local; use `models.mode: "merge"` so fallbacks stay available.

### Hybrid config: hosted primary, local fallback

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
### Local-first với mạng lưới an toàn được lưu trữ

Hoán đổi thứ tự chính và dự phòng; giữ nguyên khối nhà cung cấp và `models.mode: "merge"` để bạn có thể quay lại Sonnet hoặc Opus khi hộp cục bộ bị ngừng hoạt động.

### Lưu trữ khu vực / định tuyến dữ liệu

- Các biến thể MiniMax/Kimi/GLM được lưu trữ cũng tồn tại trên OpenRouter với các điểm cuối được ghim theo khu vực (ví dụ: được lưu trữ ở Mỹ). Chọn biến thể khu vực đó để giữ lưu lượng truy cập trong khu vực pháp lý mà bạn chọn trong khi vẫn sử dụng `models.mode: "merge"` cho các dự phòng Anthropic/OpenAI.
- Chỉ cục bộ vẫn là con đường bảo mật mạnh nhất; định tuyến khu vực được lưu trữ là giải pháp trung gian khi bạn cần các tính năng của nhà cung cấp nhưng muốn kiểm soát luồng dữ liệu.
## Các proxy cục bộ tương thích khác với OpenAI

vLLM, LiteLLM, OAI-proxy, hoặc các gateway tùy chỉnh hoạt động nếu chúng hiển thị một điểm cuối kiểu OpenAI `/v1`. Thay thế khối provider ở trên bằng điểm cuối và ID mô hình của bạn:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Keep `models.mode: "merge"` để các mô hình được lưu trữ vẫn có sẵn làm dự phòng.
## Khắc phục sự cố

- Gateway có thể kết nối được với proxy? `curl http://127.0.0.1:1234/v1/models`.
- Mô hình LM Studio bị gỡ tải? Tải lại; khởi động lạnh là nguyên nhân phổ biến gây "treo".
- Lỗi ngữ cảnh? Giảm `contextWindow` hoặc tăng giới hạn máy chủ của bạn.
- Bảo mật: các mô hình cục bộ bỏ qua các bộ lọc phía nhà cung cấp; giữ các agent hẹp và bật nén để giới hạn bán kính tác động của prompt injection.