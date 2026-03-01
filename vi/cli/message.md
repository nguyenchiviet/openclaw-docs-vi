---
summary: Tài liệu tham khảo CLI cho `openclaw message` (gửi + hành động kênh)
read_when:
  - Thêm hoặc sửa đổi các hành động CLI của tin nhắn
  - Thay đổi hành vi kênh gửi đi
title: tin nhắn
x-i18n:
  source_path: cli\message.md
  source_hash: 41b8483f92051e5ac96eff1887fa03a1c1158876a54a2ae2fdbfb2322bdc98d8
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:36:56.244Z'
---

# `openclaw message`

Lệnh gửi đi duy nhất để gửi tin nhắn và thực hiện các hành động kênh
(Discord/Google Chat/Slack/Mattermost (plugin)/Telegram/WhatsApp/Signal/iMessage/MS Teams).
## Cách sử dụng

```
openclaw message <subcommand> [flags]
```

Channel selection:

- `--channel` required if more than one channel is configured.
- If exactly one channel is configured, it becomes the default.
- Values: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost requires plugin)

Target formats (`--target`):

- WhatsApp: E.164 or group JID
- Telegram: chat id or `@username`
- Discord: `channel:<id>` or `user:<id>` (or `<@id>` mention; raw numeric ids are treated as channels)
- Google Chat: `spaces/<spaceId>` or `users/<userId>`
- Slack: `channel:<id>` or `user:<id>` (raw channel id is accepted)
- Mattermost (plugin): `channel:<id>`, `user:<id>`, or `@username` (bare ids are treated as channels)
- Signal: `+E.164`, `group:<id>`, `signal:+E.164`, `signal:group:<id>`, or `username:<name>`/`u:<name>`
- iMessage: handle, `chat_id:<id>`, `chat_guid:<guid>`, or `chat_identifier:<id>`
- MS Teams: conversation id (`19:...@thread.tacv2`) or `conversation:<id>` or `user:<aad-object-id>`

Name lookup:

- For supported providers (Discord/Slack/etc), channel names like `Help` or `#help` được phân giải thông qua bộ nhớ cache thư mục.
- Khi cache miss, OpenClaw sẽ cố gắng thực hiện tra cứu thư mục trực tiếp khi nhà cung cấp hỗ trợ.
## Các cờ phổ biến

- `--channel <name>`
- `--account <id>`
- `--target <dest>` (kênh hoặc người dùng đích cho gửi/thăm dò/đọc/v.v.)
- `--targets <name>` (lặp lại; chỉ phát sóng)
- `--json`
- `--dry-run`
- `--verbose`
## Hành động

### Cốt lõi

- `send`
  - Kênh: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams
  - Bắt buộc: `--target`, cộng với `--message` hoặc `--media`
  - Tùy chọn: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
  - Chỉ Telegram: `--buttons` (yêu cầu `channels.telegram.capabilities.inlineButtons` để cho phép)
  - Chỉ Telegram: `--thread-id` (id chủ đề diễn đàn)
  - Chỉ Slack: `--thread-id` (dấu thời gian luồng; `--reply-to` sử dụng cùng một trường)
  - Chỉ WhatsApp: `--gif-playback`

- `poll`
  - Kênh: WhatsApp/Telegram/Discord/Matrix/MS Teams
  - Bắt buộc: `--target`, `--poll-question`, `--poll-option` (lặp lại)
  - Tùy chọn: `--poll-multi`
  - Chỉ Discord: `--poll-duration-hours`, `--silent`, `--message`
  - Chỉ Telegram: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`

- `react`
  - Kênh: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
  - Bắt buộc: `--message-id`, `--target`
  - Tùy chọn: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
  - Lưu ý: `--remove` yêu cầu `--emoji` (bỏ qua `--emoji` để xóa phản ứng của riêng bạn nếu được hỗ trợ; xem /tools/reactions)
  - Chỉ WhatsApp: `--participant`, `--from-me`
  - Phản ứng nhóm Signal: `--target-author` hoặc `--target-author-uuid` bắt buộc

- `reactions`
  - Kênh: Discord/Google Chat/Slack
  - Bắt buộc: `--message-id`, `--target`
  - Tùy chọn: `--limit`

- `read`
  - Kênh: Discord/Slack
  - Bắt buộc: `--target`
  - Tùy chọn: `--limit`, `--before`, `--after`
  - Chỉ Discord: `--around`

- `edit`
  - Kênh: Discord/Slack
  - Bắt buộc: `--message-id`, `--message`, `--target`

- `delete`
  - Kênh: Discord/Slack/Telegram
  - Bắt buộc: `--message-id`, `--target`

- `pin` / `unpin`
  - Kênh: Discord/Slack
  - Bắt buộc: `--message-id`, `--target`

- `pins` (danh sách)
  - Kênh: Discord/Slack
  - Bắt buộc: `--target`

- `permissions`
  - Kênh: Discord
  - Bắt buộc: `--target`

- `search`
- Kênh: Discord
- Bắt buộc: `--guild-id`, `--query`
- Tùy chọn: `--channel-id`, `--channel-ids` (lặp lại), `--author-id`, `--author-ids` (lặp lại), `--limit`

### Luồng

- `thread create`
  - Kênh: Discord
  - Bắt buộc: `--thread-name`, `--target` (id kênh)
  - Tùy chọn: `--message-id`, `--message`, `--auto-archive-min`

- `thread list`
  - Kênh: Discord
  - Bắt buộc: `--guild-id`
  - Tùy chọn: `--channel-id`, `--include-archived`, `--before`, `--limit`

- `thread reply`
  - Kênh: Discord
  - Bắt buộc: `--target` (id luồng), `--message`
  - Tùy chọn: `--media`, `--reply-to`

### Biểu tượng cảm xúc

- `emoji list`
  - Discord: `--guild-id`
  - Slack: không có cờ bổ sung

- `emoji upload`
  - Kênh: Discord
  - Bắt buộc: `--guild-id`, `--emoji-name`, `--media`
  - Tùy chọn: `--role-ids` (lặp lại)

### Nhãn dán

- `sticker send`
  - Kênh: Discord
  - Bắt buộc: `--target`, `--sticker-id` (lặp lại)
  - Tùy chọn: `--message`

- `sticker upload`
  - Kênh: Discord
  - Bắt buộc: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### Vai trò / Kênh / Thành viên / Thoại

- `role info` (Discord): `--guild-id`
- `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
- `channel info` (Discord): `--target`
- `channel list` (Discord): `--guild-id`
- `member info` (Discord/Slack): `--user-id` (+ `--guild-id` cho Discord)
- `voice status` (Discord): `--guild-id`, `--user-id`

### Sự kiện

- `event list` (Discord): `--guild-id`
- `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
  - Tùy chọn: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### Kiểm duyệt (Discord)
- `timeout`: `--guild-id`, `--user-id` (tùy chọn `--duration-min` hoặc `--until`; bỏ qua cả hai để xóa timeout)
- `kick`: `--guild-id`, `--user-id` (+ `--reason`)
- `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
  - `timeout` cũng hỗ trợ `--reason`

### Phát sóng

- `broadcast`
  - Kênh: bất kỳ kênh nào được cấu hình; sử dụng `--channel all` để nhắm mục tiêu tất cả các nhà cung cấp
  - Bắt buộc: `--targets` (lặp lại)
  - Tùy chọn: `--message`, `--media`, `--dry-run`
## Ví dụ

Gửi trả lời Discord:

```
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

Send a Discord message with components:

```
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

See [Discord components](/channels/discord#interactive-components) for the full schema.

Create a Discord poll:

```
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Create a Telegram poll (auto-close in 2 minutes):

```
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

Send a Teams proactive message:

```
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Create a Teams poll:

```
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

React in Slack:

```
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```
React trong một nhóm Signal:

```
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Send Telegram inline buttons:

```
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```