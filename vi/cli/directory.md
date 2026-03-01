---
summary: 'Tài liệu tham khảo CLI cho `openclaw directory` (self, peers, groups)'
read_when:
  - Bạn muốn tra cứu ID của các liên hệ/nhóm/chính mình cho một kênh
  - Bạn đang phát triển một bộ điều hợp thư mục kênh
title: thư mục
x-i18n:
  source_path: cli\directory.md
  source_hash: 7c878d9013aeaa22c8a21563fac30b465a86be85d8c917c5d4591b5c3d4b2025
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:34.039Z'
---

# `openclaw directory`

Tra cứu thư mục cho các kênh hỗ trợ (liên hệ/ngang hàng, nhóm và "tôi").

## Các cờ chung

- `--channel <name>`: id/bí danh kênh (bắt buộc khi có nhiều kênh được cấu hình; tự động khi chỉ có một kênh được cấu hình)
- `--account <id>`: id tài khoản (mặc định: mặc định kênh)
- `--json`: xuất JSON

## Ghi chú

- `directory` được dùng để giúp bạn tìm các ID mà bạn có thể dán vào các lệnh khác (đặc biệt là `openclaw message send --target ...`).
- Đối với nhiều kênh, kết quả được hỗ trợ bởi cấu hình (danh sách cho phép / nhóm được cấu hình) thay vì thư mục nhà cung cấp trực tiếp.
- Đầu ra mặc định là `id` (và đôi khi `name`) được phân tách bằng tab; sử dụng `--json` để viết kịch bản.

## Sử dụng kết quả với `message send`

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## ID formats (by channel)

- WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (group)
- Telegram: `@username` or numeric chat id; groups are numeric ids
- Slack: `user:U…` and `channel:C…`
- Discord: `user:<id>` and `channel:<id>`
- Matrix (plugin): `user:@user:server`, `room:!roomId:server`, or `#alias:server`
- Microsoft Teams (plugin): `user:<id>` and `conversation:<id>`
- Zalo (plugin): user id (Bot API)
- Zalo Personal / `zalouser` (plugin): thread id (DM/group) from `zca` (`me`, `friend list`, `group list`)

## Self (“me”)

```bash
openclaw directory self --channel zalouser
```

## Peers (contacts/users)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## Groups

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```