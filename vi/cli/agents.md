---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw agents`
  (list/add/delete/bindings/bind/unbind/set identity)
read_when:
  - Bạn muốn có nhiều agent độc lập (workspaces + routing + auth)
title: tác nhân
x-i18n:
  source_path: cli\agents.md
  source_hash: b6a6b7b9ac330a6eb35dbbb6c080fcca621b6310983534fe7ad10b90e7f0c38c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:31:38.755Z'
---

# `openclaw agents`

Quản lý các agent độc lập (không gian làm việc + xác thực + định tuyến).

Liên quan:

- Định tuyến đa agent: [Multi-Agent Routing](/concepts/multi-agent)
- Không gian làm việc agent: [Agent workspace](/concepts/agent-workspace)
## Ví dụ

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```
## Liên kết định tuyến

Sử dụng liên kết định tuyến để ghim lưu lượng kênh đến với một agent cụ thể.

Liệt kê các liên kết:

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

Add bindings:

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

If you omit `accountId` (`--bind <channel>`), OpenClaw resolves it from channel defaults and plugin setup hooks when available.

### Binding scope behavior

- A binding without `accountId` matches the channel default account only.
- `accountId: "*"` is the channel-wide fallback (all accounts) and is less specific than an explicit account binding.
- If the same agent already has a matching channel binding without `accountId`, and you later bind with an explicit or resolved `accountId`, OpenClaw upgrades that existing binding in place instead of adding a duplicate.

Example:

```bash
# initial channel-only binding
openclaw agents bind --agent work --bind telegram

# later upgrade to account-scoped binding
openclaw agents bind --agent work --bind telegram:ops
```

After the upgrade, routing for that binding is scoped to `telegram:ops`. If you also want default-account routing, add it explicitly (for example `--bind telegram:default`).

Remove bindings:

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```
## Tệp danh tính

Mỗi không gian làm việc của agent có thể bao gồm một `IDENTITY.md` tại thư mục gốc của không gian làm việc:

- Đường dẫn ví dụ: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` đọc từ thư mục gốc của không gian làm việc (hoặc một `--identity-file` được chỉ định rõ ràng)

Đường dẫn avatar được giải quyết tương đối với thư mục gốc của không gian làm việc.
## Đặt danh tính

`set-identity` ghi các trường vào `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (đường dẫn tương đối workspace, URL http(s), hoặc data URI)

Tải từ `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Override fields explicitly:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Config sample:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```