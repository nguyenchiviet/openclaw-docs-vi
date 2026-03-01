---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw skills` (liệt kê/thông tin/kiểm tra) và
  điều kiện áp dụng Skills
read_when:
  - Bạn muốn xem những Skills nào khả dụng và sẵn sàng để chạy
  - Bạn muốn gỡ lỗi các binaries/env/config bị thiếu cho Skills
title: kỹ năng
x-i18n:
  source_path: cli\skills.md
  source_hash: 7878442c88a27ec8033f3125c319e9a6a85a1c497a404a06112ad45185c261b0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:07:17.447Z'
---

# `openclaw skills`

Kiểm tra Skills (được đóng gói + không gian làm việc + ghi đè được quản lý) và xem những gì đủ điều kiện so với những gì thiếu yêu cầu.

Liên quan:

- Hệ thống Skills: [Skills](/tools/skills)
- Cấu hình Skills: [Cấu hình Skills](/tools/skills-config)
- Cài đặt ClawHub: [ClawHub](/tools/clawhub)

## Lệnh

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```
