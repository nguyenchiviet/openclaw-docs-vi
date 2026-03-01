---
summary: Chạy trực tiếp `openclaw agent` CLI (với tùy chọn gửi)
read_when:
  - Thêm hoặc sửa đổi điểm vào CLI của agent
title: Gửi Agent
x-i18n:
  source_path: tools\agent-send.md
  source_hash: a84d6a304333eebe155da2bf24cf5fc0482022a0a48ab34aa1465cd6e667022d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:27:49.247Z'
---

# `openclaw agent` (chạy agent trực tiếp)

`openclaw agent` chạy một lượt agent duy nhất mà không cần tin nhắn chat đến.
Theo mặc định, nó đi **qua Gateway**; thêm `--local` để buộc runtime nhúng
trên máy hiện tại.

## Hành vi

- Bắt buộc: `--message <text>`
- Lựa chọn phiên:
  - `--to <dest>` lấy khóa phiên (các mục tiêu nhóm/kênh bảo tồn cách ly; các cuộc trò chuyện trực tiếp sụp đổ thành `main`), **hoặc**
  - `--session-id <id>` tái sử dụng một phiên hiện có theo id, **hoặc**
  - `--agent <id>` nhắm mục tiêu một agent được cấu hình trực tiếp (sử dụng khóa phiên `main` của agent đó)
- Chạy cùng runtime agent nhúng như các trả lời đến bình thường.
- Các cờ thinking/verbose vẫn tồn tại trong kho phiên.
- Đầu ra:
  - mặc định: in văn bản trả lời (cộng với các dòng `MEDIA:<url>`)
  - `--json`: in payload có cấu trúc + siêu dữ liệu
- Giao hàng tùy chọn trở lại một kênh với `--deliver` + `--channel` (các định dạng mục tiêu khớp với `openclaw message --target`).
- Sử dụng `--reply-channel`/`--reply-to`/`--reply-account` để ghi đè giao hàng mà không thay đổi phiên.

Nếu Gateway không thể truy cập được, CLI **quay lại** chạy cục bộ nhúng.

## Ví dụ

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Flags

- `--local`: run locally (requires model provider API keys in your shell)
- `--deliver`: send the reply to the chosen channel
- `--channel`: delivery channel (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, default: `whatsapp`)
- `--reply-to`: delivery target override
- `--reply-channel`: delivery channel override
- `--reply-account`: delivery account id override
- `--thinking <off|minimal|low|medium|high|xhigh>`: persist thinking level (GPT-5.2 + Codex models only)
- `--verbose <on|full|off>`: persist verbose level
- `--timeout <seconds>`: override agent timeout
- `--json`: xuất JSON có cấu trúc