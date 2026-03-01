---
summary: Cài đặt Perplexity Sonar cho web_search
read_when:
  - Bạn muốn sử dụng Perplexity Sonar để tìm kiếm trên web
  - Bạn cần thiết lập PERPLEXITY_API_KEY hoặc OpenRouter
title: Perplexity Sonar
x-i18n:
  source_path: perplexity.md
  source_hash: f6c9824ad9bebe389f029d74c2a9ae53ab69572bbe5cc6fbbc9c43741eb8e421
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:11:27.121Z'
---

# Perplexity Sonar

OpenClaw có thể sử dụng Perplexity Sonar cho công cụ `web_search`. Bạn có thể kết nối thông qua API trực tiếp của Perplexity hoặc qua OpenRouter.
## Tùy chọn API

### Perplexity (trực tiếp)

- Base URL: [https://api.perplexity.ai](https://api.perplexity.ai)
- Biến môi trường: `PERPLEXITY_API_KEY`

### OpenRouter (thay thế)

- Base URL: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
- Biến môi trường: `OPENROUTER_API_KEY`
- Hỗ trợ tín dụng thanh toán trước/tiền điện tử.
## Ví dụ cấu hình

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```
## Chuyển từ Brave

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

If both `PERPLEXITY_API_KEY` and `OPENROUTER_API_KEY` are set, set
`tools.web.search.perplexity.baseUrl` (or `tools.web.search.perplexity.apiKey`)
to disambiguate.

If no base URL is set, OpenClaw chooses a default based on the API key source:

- `PERPLEXITY_API_KEY` or `pplx-...` → direct Perplexity (`https://api.perplexity.ai`)
- `OPENROUTER_API_KEY` or `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
- Định dạng khóa không xác định → OpenRouter (dự phòng an toàn)
## Mô hình

- `perplexity/sonar` — Hỏi đáp nhanh với tìm kiếm web
- `perplexity/sonar-pro` (mặc định) — Lập luận đa bước + tìm kiếm web
- `perplexity/sonar-reasoning-pro` — Nghiên cứu sâu

Xem [Công cụ Web](/tools/web) để biết cấu hình web_search đầy đủ.