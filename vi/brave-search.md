---
summary: Thiết lập Brave Search API cho web_search
read_when:
  - Bạn muốn sử dụng Brave Search cho web_search
  - Bạn cần một BRAVE_API_KEY hoặc thông tin chi tiết về gói dịch vụ
title: Tìm kiếm Brave
x-i18n:
  source_path: brave-search.md
  source_hash: 81cd0a13239c13f4cf41d3f7b72ea0810c9e3f9f5a19ffc8955aa1822f726261
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:13:04.005Z'
---

# Brave Search API

OpenClaw sử dụng Brave Search làm nhà cung cấp mặc định cho `web_search`.

## Lấy khóa API

1. Tạo tài khoản Brave Search API tại [https://brave.com/search/api/](https://brave.com/search/api/)
2. Trong bảng điều khiển, chọn gói **Data for Search** và tạo khóa API.
3. Lưu trữ khóa trong cấu hình (khuyến nghị) hoặc đặt `BRAVE_API_KEY` trong môi trường Gateway.

## Ví dụ cấu hình

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## Notes

- The Data for AI plan is **not** compatible with `web_search`.
- Brave cung cấp gói miễn phí cộng với các gói trả phí; kiểm tra cổng thông tin API Brave để biết giới hạn hiện tại.

Xem [Công cụ web](/tools/web) để biết cấu hình đầy đủ của web_search.