---
summary: Cấu hình Moonshot K2 vs Kimi Coding (các nhà cung cấp riêng biệt + khóa)
read_when:
  - Bạn muốn thiết lập Moonshot K2 (Nền tảng Mở Moonshot) so với Kimi Coding
  - 'Bạn cần hiểu các endpoint riêng biệt, khóa và tham chiếu mô hình.'
  - Bạn muốn sao chép/dán cấu hình cho một trong hai nhà cung cấp
title: Moonshot AI
x-i18n:
  source_path: providers\moonshot.md
  source_hash: 9e4a6192faa21b881820d145e2415843b89e39a3be43451174b0ba9241aa873f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:30.752Z'
---

# Moonshot AI (Kimi)

Moonshot cung cấp API Kimi với các endpoint tương thích OpenAI. Cấu hình nhà cung cấp và đặt mô hình mặc định thành `moonshot/kimi-k2.5`, hoặc sử dụng Kimi Coding với `kimi-coding/k2p5`.

ID mô hình Kimi K2 hiện tại:

{/_moonshot-kimi-k2-ids:start_/ && null}

- `kimi-k2.5`
- `kimi-k2-0905-preview`
- `kimi-k2-turbo-preview`
- `kimi-k2-thinking`
- `kimi-k2-thinking-turbo`
  {/_moonshot-kimi-k2-ids:end_/ && null}

```bash
openclaw onboard --auth-choice moonshot-api-key
```

Kimi Coding:

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

Note: Moonshot and Kimi Coding are separate providers. Keys are not interchangeable, endpoints differ, and model refs differ (Moonshot uses `moonshot/...`, Kimi Coding uses `kimi-coding/...`).
## Đoạn cấu hình (Moonshot API)

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: {
        // moonshot-kimi-k2-aliases:start
        "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
        "moonshot/kimi-k2-0905-preview": { alias: "Kimi K2" },
        "moonshot/kimi-k2-turbo-preview": { alias: "Kimi K2 Turbo" },
        "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
        "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" },
        // moonshot-kimi-k2-aliases:end
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          // moonshot-kimi-k2-models:start
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-0905-preview",
            name: "Kimi K2 0905 Preview",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-turbo-preview",
            name: "Kimi K2 Turbo",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking",
            name: "Kimi K2 Thinking",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking-turbo",
            name: "Kimi K2 Thinking Turbo",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          // moonshot-kimi-k2-models:end
        ],
      },
    },
  },
}
```
## Kimi Coding

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: {
        "kimi-coding/k2p5": { alias: "Kimi K2.5" },
      },
    },
  },
}
```
## Ghi chú

- Tham chiếu mô hình Moonshot sử dụng `moonshot/<modelId>`. Tham chiếu mô hình Kimi Coding sử dụng `kimi-coding/<modelId>`.
- Ghi đè giá cả và siêu dữ liệu ngữ cảnh trong `models.providers` nếu cần.
- Nếu Moonshot công bố các giới hạn ngữ cảnh khác nhau cho một mô hình, hãy điều chỉnh `contextWindow` tương ứng.
- Sử dụng `https://api.moonshot.ai/v1` cho điểm cuối quốc tế và `https://api.moonshot.cn/v1` cho điểm cuối Trung Quốc.