---
summary: Phân tích vị trí kênh đến (Telegram + WhatsApp) và các trường ngữ cảnh
read_when:
  - Thêm hoặc chỉnh sửa phân tích vị trí kênh
  - Sử dụng các trường ngữ cảnh vị trí trong prompt agent hoặc công cụ
title: Phân Tích Vị Trí Kênh
x-i18n:
  source_path: channels\location.md
  source_hash: 5602ef105c3da7e47497bfed8fc343dd8d7f3c019ff7e423a08b25092c5a1837
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:20:01.629Z'
---

# Phân tích vị trí kênh

OpenClaw chuẩn hóa các vị trí được chia sẻ từ các kênh trò chuyện thành:

- văn bản dễ đọc được thêm vào nội dung tin nhắn đến, và
- các trường có cấu trúc trong payload ngữ cảnh tự động trả lời.

Hiện tại được hỗ trợ:

- **Telegram** (ghim vị trí + địa điểm + vị trí trực tiếp)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` với `geo_uri`)

## Định dạng văn bản

Các vị trí được hiển thị dưới dạng các dòng thân thiện không có dấu ngoặc:

- Ghim:
  - `📍 48.858844, 2.294351 ±12m`
- Địa điểm có tên:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- Chia sẻ trực tiếp:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

Nếu kênh bao gồm chú thích/bình luận, nó sẽ được thêm vào dòng tiếp theo:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## Context fields

When a location is present, these fields are added to `ctx`:

- `LocationLat` (number)
- `LocationLon` (number)
- `LocationAccuracy` (number, meters; optional)
- `LocationName` (string; optional)
- `LocationAddress` (string; optional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)

## Channel notes

- **Telegram**: venues map to `LocationName/LocationAddress`; live locations use `live_period`.
- **WhatsApp**: `locationMessage.comment` and `liveLocationMessage.caption` are appended as the caption line.
- **Matrix**: `geo_uri` is parsed as a pin location; altitude is ignored and `LocationIsLive` luôn là false.