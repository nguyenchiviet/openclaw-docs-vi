---
summary: 'Định tuyến đa tác nhân: các tác nhân cô lập, tài khoản kênh và ràng buộc'
title: Định Tuyến Đa Tác Nhân
read_when: You want multiple isolated agents (workspaces + auth) in one gateway process.
status: active
x-i18n:
  source_path: concepts\multi-agent.md
  source_hash: 4f16c0414312d931df2423c3f759f32ebc04ef68216d7aa7eb1c282ff2d0c3b4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:17.544Z'
---

# Định tuyến Đa Agent

Mục tiêu: nhiều agent _cô lập_ (workspace riêng + `agentDir` + phiên), cộng với nhiều tài khoản kênh (ví dụ: hai WhatsApps) trong một Gateway đang chạy. Lưu lượng đến được định tuyến đến một agent thông qua các ràng buộc.
## "One agent" là gì?

Một **agent** là một bộ não có phạm vi đầy đủ với:

- **Workspace** (tệp, AGENTS.md/SOUL.md/USER.md, ghi chú cục bộ, quy tắc nhân cách).
- **Thư mục trạng thái** (`agentDir`) cho hồ sơ xác thực, sổ đăng ký mô hình và cấu hình cho từng agent.
- **Kho lưu trữ phiên** (lịch sử trò chuyện + trạng thái định tuyến) dưới `~/.openclaw/agents/<agentId>/sessions`.

Hồ sơ xác thực là **cho từng agent**. Mỗi agent đọc từ:

```text
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Main agent credentials are **not** shared automatically. Never reuse `agentDir`
across agents (it causes auth/session collisions). If you want to share creds,
copy `auth-profiles.json` into the other agent's `agentDir`.

Skills are per-agent via each workspace’s `skills/` folder, with shared skills
available from `~/.openclaw/skills`. Xem [Skills: cho từng agent vs chia sẻ](/tools/skills#per-agent-vs-shared-skills).

Gateway có thể lưu trữ **một agent** (mặc định) hoặc **nhiều agent** cạnh nhau.

**Ghi chú Workspace:** workspace của mỗi agent là **cwd mặc định**, không phải một sandbox cứng. Các đường dẫn tương đối được phân giải bên trong workspace, nhưng các đường dẫn tuyệt đối có thể truy cập các vị trí máy chủ khác trừ khi sandboxing được bật. Xem [Sandboxing](/gateway/sandboxing).
## Đường dẫn (bản đồ nhanh)

- Cấu hình: `~/.openclaw/openclaw.json` (hoặc `OPENCLAW_CONFIG_PATH`)
- Thư mục trạng thái: `~/.openclaw` (hoặc `OPENCLAW_STATE_DIR`)
- Workspace: `~/.openclaw/workspace` (hoặc `~/.openclaw/workspace-<agentId>`)
- Thư mục agent: `~/.openclaw/agents/<agentId>/agent` (hoặc `agents.list[].agentDir`)
- Phiên: `~/.openclaw/agents/<agentId>/sessions`

### Chế độ agent đơn (mặc định)

Nếu bạn không làm gì, OpenClaw chạy một agent duy nhất:

- `agentId` mặc định là **`main`**.
- Các phiên được khóa dưới dạng `agent:main:<mainKey>`.
- Workspace mặc định là `~/.openclaw/workspace` (hoặc `~/.openclaw/workspace-<profile>` khi `OPENCLAW_PROFILE` được đặt).
- Trạng thái mặc định là `~/.openclaw/agents/main/agent`.
## Trợ giúp agent

Sử dụng trình hướng dẫn agent để thêm một agent cách ly mới:

```bash
openclaw agents add work
```

Then add `bindings` (or let the wizard do it) to route inbound messages.

Verify with:

```bash
openclaw agents list --bindings
```
## Bắt đầu nhanh

<Steps>
  <Step title="Tạo không gian làm việc cho từng agent">

Sử dụng trình hướng dẫn hoặc tạo không gian làm việc theo cách thủ công:

```bash
openclaw agents add coding
openclaw agents add social
```

Each agent gets its own workspace with `SOUL.md`, `AGENTS.md`, and optional `USER.md`, plus a dedicated `agentDir` and session store under `~/.openclaw/agents/<agentId>`.

  </Step>

  <Step title="Create channel accounts">

Create one account per agent on your preferred channels:

- Discord: one bot per agent, enable Message Content Intent, copy each token.
- Telegram: one bot per agent via BotFather, copy each token.
- WhatsApp: link each phone number per account.

```bash
openclaw channels login --channel whatsapp --account work
```

See channel guides: [Discord](/channels/discord), [Telegram](/channels/telegram), [WhatsApp](/channels/whatsapp).

  </Step>

  <Step title="Add agents, accounts, and bindings">

Add agents under `agents.list`, channel accounts under `channels.<channel>.accounts`, and connect them with `bindings` (examples below).

  </Step>

  <Step title="Restart and verify">

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

  </Step>
</Steps>
## Nhiều agent = nhiều người, nhiều tính cách

Với **nhiều agent**, mỗi `agentId` trở thành một **persona hoàn toàn cô lập**:

- **Các số điện thoại/tài khoản khác nhau** (trên mỗi kênh `accountId`).
- **Các tính cách khác nhau** (các tệp không gian làm việc cho mỗi agent như `AGENTS.md` và `SOUL.md`).
- **Xác thực + phiên riêng biệt** (không có giao tiếp chéo trừ khi được bật rõ ràng).

Điều này cho phép **nhiều người** chia sẻ một máy chủ Gateway duy nhất trong khi giữ "bộ não" AI và dữ liệu của họ cô lập.
## Một số WhatsApp, nhiều người (chia tách tin nhắn riêng)

Bạn có thể định tuyến **các tin nhắn riêng WhatsApp khác nhau** đến các agent khác nhau trong khi vẫn ở trên **một tài khoản WhatsApp**. Khớp với E.164 của người gửi (như `+15551234567`) với `peer.kind: "direct"`. Các câu trả lời vẫn đến từ cùng một số WhatsApp (không có danh tính người gửi riêng cho từng agent).

Chi tiết quan trọng: các cuộc trò chuyện trực tiếp sụp đổ thành **khóa phiên chính** của agent, vì vậy cách ly thực sự yêu cầu **một agent cho mỗi người**.

Ví dụ:

```json5
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

Ghi chú:

- Kiểm soát truy cập tin nhắn riêng là **toàn cục cho mỗi tài khoản WhatsApp** (ghép nối/danh sách cho phép), không phải cho mỗi agent.
- Đối với các nhóm được chia sẻ, hãy liên kết nhóm với một agent hoặc sử dụng [Nhóm phát sóng](/channels/broadcast-groups).
## Quy tắc định tuyến (cách tin nhắn chọn agent)

Các ràng buộc là **xác định** và **quy tắc cụ thể nhất thắng**:

1. `peer` match (exact DM/group/channel id)
2. `parentPeer` match (thread inheritance)
3. `guildId + roles` (Discord role routing)
4. `guildId` (Discord)
5. `teamId` (Slack)
6. `accountId` match for a channel
7. channel-level match (`accountId: "*"`)
8. fallback to default agent (`agents.list[].default`, else first list entry, default: `main`)

Nếu nhiều ràng buộc khớp ở cùng một cấp, ràng buộc đầu tiên theo thứ tự cấu hình sẽ thắng.
Nếu một ràng buộc đặt nhiều trường khớp (ví dụ `peer` + `guildId`), tất cả các trường được chỉ định đều bắt buộc (ngữ nghĩa `AND`).

Chi tiết quan trọng về phạm vi tài khoản:

- Một ràng buộc bỏ qua `accountId` chỉ khớp với tài khoản mặc định.
- Sử dụng `accountId: "*"` cho fallback toàn kênh trên tất cả các tài khoản.
- Nếu bạn sau đó thêm cùng một ràng buộc cho cùng một agent với id tài khoản rõ ràng, OpenClaw nâng cấp ràng buộc chỉ kênh hiện có thành phạm vi tài khoản thay vì tạo bản sao.
## Nhiều tài khoản / số điện thoại

Các kênh hỗ trợ **nhiều tài khoản** (ví dụ: WhatsApp) sử dụng `accountId` để xác định
mỗi lần đăng nhập. Mỗi `accountId` có thể được định tuyến đến một agent khác, vì vậy một máy chủ có thể lưu trữ
nhiều số điện thoại mà không trộn lẫn các phiên.
## Khái niệm

- `agentId`: một "bộ não" (không gian làm việc, xác thực cho từng agent, kho phiên cho từng agent).
- `accountId`: một phiên bản tài khoản kênh (ví dụ: tài khoản WhatsApp `"personal"` so với `"biz"`).
- `binding`: định tuyến các tin nhắn đến cho một `agentId` theo `(channel, accountId, peer)` và tùy chọn các id guild/team.
- Các cuộc trò chuyện trực tiếp sụp đổ thành `agent:<agentId>:<mainKey>` (phần "chính" của từng agent; `session.mainKey`).
## Ví dụ về nền tảng

### Bot Discord cho mỗi agent

Mỗi tài khoản bot Discord ánh xạ tới một `accountId` duy nhất. Liên kết mỗi tài khoản với một agent và duy trì danh sách cho phép cho mỗi bot.

```json5
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

Notes:

- Invite each bot to the guild and enable Message Content Intent.
- Tokens live in `channels.discord.accounts.<id>.token` (default account can use `DISCORD_BOT_TOKEN`).

### Telegram bots per agent

```json5
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```
Ghi chú:

- Tạo một bot cho mỗi agent với BotFather và sao chép từng token.
- Các token nằm trong `channels.telegram.accounts.<id>.botToken` (tài khoản mặc định có thể sử dụng `TELEGRAM_BOT_TOKEN`).

### Số điện thoại WhatsApp cho mỗi agent

Liên kết từng tài khoản trước khi khởi động gateway:

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5):

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // Off by default: agent-to-agent messaging must be explicitly enabled + allowlisted.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```
## Ví dụ: WhatsApp trò chuyện hàng ngày + Telegram công việc sâu

Chia theo kênh: định tuyến WhatsApp đến một agent hàng ngày nhanh chóng và Telegram đến một agent Opus.

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

Notes:

- If you have multiple accounts for a channel, add `accountId` to the binding (for example `{ channel: "whatsapp", accountId: "personal" }`).
- To route a single DM/group to Opus while keeping the rest on chat, add a `match.peer` binding cho peer đó; các kết quả khớp peer luôn thắng các quy tắc toàn kênh.
## Ví dụ: cùng kênh, một peer tới Opus

Giữ WhatsApp trên agent nhanh, nhưng định tuyến một tin nhắn riêng tới Opus:

```json5
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

Liên kết peer luôn có ưu tiên, vì vậy hãy giữ chúng phía trên quy tắc toàn kênh.
## Agent gia đình được liên kết với một nhóm WhatsApp

Liên kết một agent gia đình chuyên dụng với một nhóm WhatsApp duy nhất, với gating theo mention
và chính sách công cụ chặt chẽ hơn:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

Notes:

- Tool allow/deny lists are **tools**, not skills. If a skill needs to run a
  binary, ensure `exec` is allowed and the binary exists in the sandbox.
- For stricter gating, set `agents.list[].groupChat.mentionPatterns` và giữ
  danh sách cho phép nhóm được bật cho kênh.
## Cấu hình Sandbox và Công cụ Theo Agent

Bắt đầu từ v2026.1.6, mỗi agent có thể có các hạn chế sandbox và công cụ riêng của nó:

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // Deny others
        },
      },
    ],
  },
}
```

Note: `setupCommand` lives under `sandbox.docker` and runs once on container creation.
Per-agent `sandbox.docker.*` overrides are ignored when the resolved scope is `"shared"`.

**Benefits:**

- **Security isolation**: Restrict tools for untrusted agents
- **Resource control**: Sandbox specific agents while keeping others on host
- **Flexible policies**: Different permissions per agent

Note: `tools.elevated` is **global** and sender-based; it is not configurable per agent.
If you need per-agent boundaries, use `agents.list[].tools` to deny `exec`.
For group targeting, use `agents.list[].groupChat.mentionPatterns` để @mentions ánh xạ rõ ràng đến agent dự định.

Xem [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) để xem các ví dụ chi tiết.