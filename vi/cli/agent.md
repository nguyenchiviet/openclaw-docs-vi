---
summary: Tài liệu tham khảo CLI cho `openclaw agent` (gửi một lượt agent qua Gateway)
read_when:
  - Bạn muốn chạy một lượt agent từ scripts (tùy chọn gửi phản hồi)
title: tác nhân
x-i18n:
  source_path: cli\agent.md
  source_hash: dcf12fb94e207c68645f58235792596d65afecf8216b8f9ab3acb01e03b50a33
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:30:47.830Z'
---

# `openclaw agent`

Chạy một lượt agent thông qua Gateway (sử dụng `--local` cho embedded).
Sử dụng `--agent <id>` để nhắm mục tiêu trực tiếp đến một agent đã cấu hình.

Liên quan:

- Công cụ gửi agent: [Agent send](/tools/agent-send)

## Ví dụ

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```