---
summary: Gửi bình chọn qua gateway + CLI
read_when:
  - Thêm hoặc chỉnh sửa hỗ trợ bình chọn
  - Gỡ lỗi việc gửi poll từ CLI hoặc gateway
title: Khảo sát
x-i18n:
  source_path: automation\poll.md
  source_hash: 760339865d27ec40def7996cac1d294d58ab580748ad6b32cc34d285d0314eaf
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:12:25.187Z'
---

# Bình chọn

## Các kênh được hỗ trợ

- WhatsApp (kênh web)
- Discord
- MS Teams (Adaptive Cards)
## CLI

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Options:

- `--channel`: `whatsapp` (default), `discord`, or `msteams`
- `--poll-multi`: allow selecting multiple options
- `--poll-duration-hours`: Chỉ dành cho Discord (mặc định là 24 khi bỏ qua)
## Gateway RPC

Phương thức: `poll`

Tham số:

- `to` (string, bắt buộc)
- `question` (string, bắt buộc)
- `options` (string[], bắt buộc)
- `maxSelections` (number, tùy chọn)
- `durationHours` (number, tùy chọn)
- `channel` (string, tùy chọn, mặc định: `whatsapp`)
- `idempotencyKey` (string, bắt buộc)
## Sự khác biệt giữa các kênh

- WhatsApp: 2-12 tùy chọn, `maxSelections` phải nằm trong số lượng tùy chọn, bỏ qua `durationHours`.
- Discord: 2-10 tùy chọn, `durationHours` được giới hạn từ 1-768 giờ (mặc định 24). `maxSelections > 1` cho phép chọn nhiều; Discord không hỗ trợ số lượng lựa chọn nghiêm ngặt.
- MS Teams: Cuộc thăm dò Adaptive Card (do OpenClaw quản lý). Không có API thăm dò gốc; `durationHours` bị bỏ qua.
## Công cụ agent (Tin nhắn)

Sử dụng công cụ `message` với hành động `poll` (`to`, `pollQuestion`, `pollOption`, tùy chọn `pollMulti`, `pollDurationHours`, `channel`).

Lưu ý: Discord không có chế độ "chọn chính xác N"; `pollMulti` được ánh xạ thành chọn nhiều.
Các cuộc thăm dò ý kiến Teams được hiển thị dưới dạng Adaptive Cards và yêu cầu Gateway phải trực tuyến
để ghi lại phiếu bầu trong `~/.openclaw/msteams-polls.json`.