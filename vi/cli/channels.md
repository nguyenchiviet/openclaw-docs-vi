---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw channels` (tài khoản, trạng thái, đăng
  nhập/đăng xuất, nhật ký)
read_when:
  - >-
    Bạn muốn thêm/xóa tài khoản kênh (WhatsApp/Telegram/Discord/Google
    Chat/Slack/Mattermost (plugin)/Signal/iMessage)
  - Bạn muốn kiểm tra trạng thái kênh hoặc xem nhật ký kênh theo thời gian thực
title: kênh
x-i18n:
  source_path: cli\channels.md
  source_hash: f80eb8e73e1b7c7131cef706ce3eb2b5f95e7aa601a27562d2ccf0793ff6d647
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:05.687Z'
---

# `openclaw channels`

Quản lý các tài khoản kênh trò chuyện và trạng thái thời gian chạy của chúng trên Gateway.

Tài liệu liên quan:

- Hướng dẫn kênh: [Channels](/channels/index)
- Cấu hình Gateway: [Configuration](/gateway/configuration)
## Các lệnh phổ biến

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```
## Thêm / xóa tài khoản

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Tip: `openclaw channels add --help` shows per-channel flags (token, app token, signal-cli paths, etc).

When you run `openclaw channels add` without flags, the interactive wizard can prompt:

- account ids per selected channel
- optional display names for those accounts
- `Bind configured channel accounts to agents now?`

If you confirm bind now, the wizard asks which agent should own each configured channel account and writes account-scoped routing bindings.

You can also manage the same routing rules later with `openclaw agents bindings`, `openclaw agents bind`, and `openclaw agents unbind` (see [agents](/cli/agents)).

When you add a non-default account to a channel that is still using single-account top-level settings (no `channels.<channel>.accounts` entries yet), OpenClaw moves account-scoped single-account top-level values into `channels.<channel>.accounts.default`, then writes the new account. This preserves the original account behavior while moving to the multi-account shape.

Routing behavior stays consistent:

- Existing channel-only bindings (no `accountId`) continue to match the default account.
- `channels add` does not auto-create or rewrite bindings in non-interactive mode.
- Interactive setup can optionally add account-scoped bindings.

If your config was already in a mixed state (named accounts present, missing `default`, and top-level single-account values still set), run `openclaw doctor --fix` to move account-scoped values into `accounts.default`.
## Đăng nhập / Đăng xuất (tương tác)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```
## Khắc phục sự cố

- Chạy `openclaw status --deep` để kiểm tra toàn diện.
- Sử dụng `openclaw doctor` để nhận hướng dẫn sửa chữa.
- `openclaw channels list` in ra `Claude: HTTP 403 ... user:profile` → ảnh chụp nhanh cách sử dụng cần phạm vi `user:profile`. Sử dụng `--no-usage`, hoặc cung cấp khóa phiên claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), hoặc xác thực lại qua Claude Code CLI.
## Kiểm tra khả năng

Tìm nạp các gợi ý khả năng của nhà cung cấp (intents/scopes nếu có) cộng với hỗ trợ tính năng tĩnh:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notes:

- `--channel` is optional; omit it to list every channel (including extensions).
- `--target` accepts `channel:<id>` or a raw numeric channel id and only applies to Discord.
- Probes are provider-specific: Discord intents + optional channel permissions; Slack bot + user scopes; Telegram bot flags + webhook; Signal daemon version; MS Teams app token + Graph roles/scopes (annotated where known). Channels without probes report `Probe: unavailable`.
## Phân giải tên thành ID

Phân giải tên kênh/người dùng thành ID bằng cách sử dụng thư mục nhà cung cấp:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Notes:

- Use `--kind user|group|auto` để buộc loại mục tiêu.
- Phân giải ưu tiên các kết quả khớp hoạt động khi nhiều mục nhập chia sẻ cùng một tên.