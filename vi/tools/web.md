---
summary: >-
  Công cụ tìm kiếm web + tìm nạp (Brave Search API, Perplexity
  direct/OpenRouter, Gemini Google Search grounding)
read_when:
  - Bạn muốn bật web_search hoặc web_fetch
  - Bạn cần thiết lập khóa API Brave Search
  - Bạn muốn sử dụng Perplexity Sonar để tìm kiếm trên web
  - Bạn muốn sử dụng Gemini với tính năng grounding của Google Search
title: Công cụ Web
x-i18n:
  source_path: tools\web.md
  source_hash: 0dbdb3857890a2471f89f8224b34e719d93966f73b6fce62cd93e47bfa7c8f14
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:23.587Z'
---

# Công cụ web

OpenClaw đi kèm với hai công cụ web nhẹ:

- `web_search` — Tìm kiếm trên web thông qua Brave Search API (mặc định), Perplexity Sonar, hoặc Gemini với tính năng grounding của Google Search.
- `web_fetch` — Tìm nạp HTTP + trích xuất nội dung có thể đọc được (HTML → markdown/text).

Đây **không phải** là tự động hóa trình duyệt. Đối với các trang web nặng JS hoặc đăng nhập, hãy sử dụng
[Công cụ Browser](/tools/browser).
## Cách hoạt động

- `web_search` gọi nhà cung cấp được cấu hình của bạn và trả về kết quả.
  - **Brave** (mặc định): trả về kết quả có cấu trúc (tiêu đề, URL, đoạn trích).
  - **Perplexity**: trả về câu trả lời được tổng hợp bằng AI với trích dẫn từ tìm kiếm web thời gian thực.
  - **Gemini**: trả về câu trả lời được tổng hợp bằng AI dựa trên Google Search với trích dẫn.
- Kết quả được lưu vào bộ nhớ cache theo truy vấn trong 15 phút (có thể cấu hình).
- `web_fetch` thực hiện HTTP GET đơn giản và trích xuất nội dung có thể đọc được
  (HTML → markdown/text). Nó **không** thực thi JavaScript.
- `web_fetch` được bật theo mặc định (trừ khi bị tắt rõ ràng).
## Chọn nhà cung cấp tìm kiếm

| Nhà cung cấp         | Ưu điểm                                      | Nhược điểm                               | Khóa API                                     |
| ------------------- | -------------------------------------------- | ---------------------------------------- | -------------------------------------------- |
| **Brave** (mặc định) | Nhanh, kết quả có cấu trúc, miễn phí         | Kết quả tìm kiếm truyền thống             | `BRAVE_API_KEY`                              |
| **Perplexity**      | Câu trả lời tổng hợp bằng AI, trích dẫn, thời gian thực | Yêu cầu quyền truy cập Perplexity hoặc OpenRouter | `OPENROUTER_API_KEY` hoặc `PERPLEXITY_API_KEY` |
| **Gemini**          | Tìm kiếm Google, tổng hợp bằng AI            | Yêu cầu khóa API Gemini                  | `GEMINI_API_KEY`                             |

Xem [Thiết lập Brave Search](/brave-search) và [Perplexity Sonar](/perplexity) để biết chi tiết cụ thể của từng nhà cung cấp.

### Tự động phát hiện

Nếu không có `provider` được đặt rõ ràng, OpenClaw sẽ tự động phát hiện nhà cung cấp nào sẽ sử dụng dựa trên các khóa API có sẵn, kiểm tra theo thứ tự này:

1. **Brave** — `BRAVE_API_KEY` biến môi trường hoặc `search.apiKey` cấu hình
2. **Gemini** — `GEMINI_API_KEY` biến môi trường hoặc `search.gemini.apiKey` cấu hình
3. **Perplexity** — `PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY` biến môi trường hoặc `search.perplexity.apiKey` cấu hình
4. **Grok** — `XAI_API_KEY` biến môi trường hoặc `search.grok.apiKey` cấu hình

Nếu không tìm thấy khóa nào, nó sẽ quay lại Brave (bạn sẽ nhận được lỗi khóa bị thiếu nhắc bạn cấu hình một khóa).

### Nhà cung cấp rõ ràng

Đặt nhà cung cấp trong cấu hình:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave", // or "perplexity" or "gemini"
      },
    },
  },
}
```

Example: switch to Perplexity Sonar (direct API):

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
## Lấy khóa API Brave

1. Tạo tài khoản Brave Search API tại [https://brave.com/search/api/](https://brave.com/search/api/)
2. Trong bảng điều khiển, chọn kế hoạch **Data for Search** (không phải "Data for AI") và tạo khóa API.
3. Chạy `openclaw configure --section web` để lưu trữ khóa trong cấu hình (được khuyến nghị), hoặc đặt `BRAVE_API_KEY` trong môi trường của bạn.

Brave cung cấp một tầng miễn phí cộng với các kế hoạch trả phí; kiểm tra cổng thông tin API Brave để xem
các giới hạn và giá hiện tại.

### Nơi đặt khóa (được khuyến nghị)

**Được khuyến nghị:** chạy `openclaw configure --section web`. Nó lưu trữ khóa trong
`~/.openclaw/openclaw.json` dưới `tools.web.search.apiKey`.

**Lựa chọn thay thế môi trường:** đặt `BRAVE_API_KEY` trong môi trường quy trình Gateway. Đối với cài đặt gateway, đặt nó trong `~/.openclaw/.env` (hoặc môi trường dịch vụ của bạn). Xem [Biến môi trường](/help/faq#how-does-openclaw-load-environment-variables).
## Sử dụng Perplexity (trực tiếp hoặc qua OpenRouter)

Các mô hình Perplexity Sonar có khả năng tìm kiếm web tích hợp sẵn và trả về các câu trả lời được tổng hợp bởi AI kèm theo trích dẫn. Bạn có thể sử dụng chúng qua OpenRouter (không cần thẻ tín dụng - hỗ trợ tiền điện tử/thanh toán trước).

### Lấy khóa API OpenRouter

1. Tạo tài khoản tại [https://openrouter.ai/](https://openrouter.ai/)
2. Thêm tín dụng (hỗ trợ tiền điện tử, thanh toán trước hoặc thẻ tín dụng)
3. Tạo khóa API trong cài đặt tài khoản của bạn

### Thiết lập tìm kiếm Perplexity

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // API key (optional if OPENROUTER_API_KEY or PERPLEXITY_API_KEY is set)
          apiKey: "sk-or-v1-...",
          // Base URL (key-aware default if omitted)
          baseUrl: "https://openrouter.ai/api/v1",
          // Model (defaults to perplexity/sonar-pro)
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

**Environment alternative:** set `OPENROUTER_API_KEY` or `PERPLEXITY_API_KEY` in the Gateway
environment. For a gateway install, put it in `~/.openclaw/.env`.

If no base URL is set, OpenClaw chooses a default based on the API key source:

- `PERPLEXITY_API_KEY` or `pplx-...` → `https://api.perplexity.ai`
- `OPENROUTER_API_KEY` or `sk-or-...` → `https://openrouter.ai/api/v1`
- Unknown key formats → OpenRouter (safe fallback)

### Available Perplexity models

| Model                            | Description                          | Best for          |
| -------------------------------- | ------------------------------------ | ----------------- |
| `perplexity/sonar`               | Fast Q&A with web search             | Quick lookups     |
| `perplexity/sonar-pro` (default) | Multi-step reasoning with web search | Complex questions |
| `perplexity/sonar-reasoning-pro` | Phân tích chuỗi suy luận            | Nghiên cứu sâu     |
## Sử dụng Gemini (Tìm kiếm có căn cứ của Google)

Các mô hình Gemini hỗ trợ [tìm kiếm có căn cứ của Google](https://ai.google.dev/gemini-api/docs/grounding) tích hợp sẵn,
trả về các câu trả lời được tổng hợp bởi AI được hỗ trợ bởi kết quả Tìm kiếm Google trực tiếp với trích dẫn.

### Lấy khóa API Gemini

1. Truy cập [Google AI Studio](https://aistudio.google.com/apikey)
2. Tạo một khóa API
3. Đặt `GEMINI_API_KEY` trong môi trường Gateway, hoặc cấu hình `tools.web.search.gemini.apiKey`

### Thiết lập tìm kiếm Gemini

```json5
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // API key (optional if GEMINI_API_KEY is set)
          apiKey: "AIza...",
          // Model (defaults to "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**Environment alternative:** set `GEMINI_API_KEY` in the Gateway environment.
For a gateway install, put it in `~/.openclaw/.env`.

### Notes

- Citation URLs from Gemini grounding are automatically resolved from Google's
  redirect URLs to direct URLs.
- Redirect resolution uses the SSRF guard path (HEAD + redirect checks + http/https validation) before returning the final citation URL.
- This redirect resolver follows the trusted-network model (private/internal networks allowed by default) to match Gateway operator trust assumptions.
- The default model (`gemini-2.5-flash`) nhanh chóng và hiệu quả về chi phí.
  Bất kỳ mô hình Gemini nào hỗ trợ tìm kiếm có căn cứ đều có thể được sử dụng.
## web_search

Tìm kiếm trên web bằng nhà cung cấp được cấu hình của bạn.

### Yêu cầu

- `tools.web.search.enabled` không được là `false` (mặc định: bật)
- Khóa API cho nhà cung cấp được chọn của bạn:
  - **Brave**: `BRAVE_API_KEY` hoặc `tools.web.search.apiKey`
  - **Perplexity**: `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY`, hoặc `tools.web.search.perplexity.apiKey`

### Cấu hình

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // optional if BRAVE_API_KEY is set
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### Tool parameters

- `query` (required)
- `count` (1–10; default from config)
- `country` (optional): 2-letter country code for region-specific results (e.g., "DE", "US", "ALL"). If omitted, Brave chooses its default region.
- `search_lang` (optional): ISO language code for search results (e.g., "de", "en", "fr")
- `ui_lang` (optional): ISO language code for UI elements
- `freshness` (optional): filter by discovery time
  - Brave: `pd`, `pw`, `pm`, `py`, or `YYYY-MM-DDtoYYYY-MM-DD`
  - Perplexity: `pd`, `pw`, `pm`, `py`

**Examples:**

```javascript
// German-specific search
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de",
});

// French search with French UI
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr",
});

// Recent results (past week)
await web_search({
  query: "TMBG interview",
  freshness: "pw",
});
```
## web_fetch

Tìm nạp một URL và trích xuất nội dung có thể đọc được.

### Yêu cầu web_fetch

- `tools.web.fetch.enabled` không được là `false` (mặc định: được bật)
- Fallback Firecrawl tùy chọn: đặt `tools.web.fetch.firecrawl.apiKey` hoặc `FIRECRAWL_API_KEY`.

### Cấu hình web_fetch

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // optional if FIRECRAWL_API_KEY is set
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 day)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### web_fetch tool parameters

- `url` (required, http/https only)
- `extractMode` (`markdown` | `text`)
- `maxChars` (truncate long pages)

Notes:

- `web_fetch` uses Readability (main-content extraction) first, then Firecrawl (if configured). If both fail, the tool returns an error.
- Firecrawl requests use bot-circumvention mode and cache results by default.
- `web_fetch` sends a Chrome-like User-Agent and `Accept-Language` by default; override `userAgent` if needed.
- `web_fetch` blocks private/internal hostnames and re-checks redirects (limit with `maxRedirects`).
- `maxChars` is clamped to `tools.web.fetch.maxCharsCap`.
- `web_fetch` caps the downloaded response body size to `tools.web.fetch.maxResponseBytes` before parsing; oversized responses are truncated and include a warning.
- `web_fetch` is best-effort extraction; some sites will need the browser tool.
- See [Firecrawl](/tools/firecrawl) for key setup and service details.
- Responses are cached (default 15 minutes) to reduce repeated fetches.
- If you use tool profiles/allowlists, add `web_search`/`web_fetch` or `group:web`.
- If the Brave key is missing, `web_search` trả về một gợi ý thiết lập ngắn với liên kết tài liệu.