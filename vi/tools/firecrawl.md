---
summary: Firecrawl fallback cho web_fetch (chống bot + trích xuất được lưu cache)
read_when:
  - Bạn muốn trích xuất web được hỗ trợ bởi Firecrawl
  - Bạn cần một khóa API Firecrawl
  - Bạn muốn trích xuất chống bot cho web_fetch
title: Firecrawl
x-i18n:
  source_path: tools\firecrawl.md
  source_hash: 08a7ad45b41af41204e44d2b0be0f980b7184d80d2fa3977339e42a47beb2851
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:30:25.559Z'
---

# Firecrawl

OpenClaw có thể sử dụng **Firecrawl** làm trích xuất dự phòng cho `web_fetch`. Đây là một dịch vụ trích xuất nội dung được lưu trữ hỗ trợ vượt qua bot và bộ nhớ đệm, giúp xử lý các trang web nặng JS hoặc các trang chặn các yêu cầu HTTP thông thường.

## Lấy khóa API

1. Tạo tài khoản Firecrawl và tạo khóa API.
2. Lưu trữ nó trong cấu hình hoặc đặt `FIRECRAWL_API_KEY` trong môi trường gateway.

## Cấu hình Firecrawl

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Notes:

- `firecrawl.enabled` defaults to true when an API key is present.
- `maxAgeMs` controls how old cached results can be (ms). Default is 2 days.

## Stealth / bot circumvention

Firecrawl exposes a **proxy mode** parameter for bot circumvention (`basic`, `stealth`, or `auto`).
OpenClaw always uses `proxy: "auto"` plus `storeInCache: true` for Firecrawl requests.
If proxy is omitted, Firecrawl defaults to `auto`. `auto` retries with stealth proxies if a basic attempt fails, which may use more credits
than basic-only scraping.

## How `web_fetch` uses Firecrawl

`web_fetch` thứ tự trích xuất:

1. Readability (local)
2. Firecrawl (nếu được cấu hình)
3. Làm sạch HTML cơ bản (dự phòng cuối cùng)

Xem [Web tools](/tools/web) để thiết lập công cụ web đầy đủ.