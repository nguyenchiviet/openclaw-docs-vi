---
title: Tham chiếu cấu hình
description: Complete field-by-field reference for ~/.openclaw/openclaw.json
summary: >-
  Tài liệu tham khảo đầy đủ cho mọi khóa cấu hình OpenClaw, giá trị mặc định và
  cài đặt kênh
read_when:
  - >-
    Bạn cần ngữ nghĩa chính xác của từng trường cấu hình hoặc các giá trị mặc
    định.
  - 'Bạn đang xác thực các khối cấu hình của kênh, mô hình, Gateway hoặc công cụ.'
x-i18n:
  source_path: gateway\configuration-reference.md
  source_hash: f9d1b64358a0f3990305de1a7fa6f1f666c82d37e883afecd86fd05acab65152
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:15:03.442Z'
---

# Tham chiếu cấu hình

Mọi trường có sẵn trong `~/.openclaw/openclaw.json`. Để có cái nhìn tổng quan theo tác vụ, hãy xem [Cấu hình](/gateway/configuration).

Định dạng cấu hình là **JSON5** (cho phép bình luận + dấu phẩy cuối dòng). Tất cả các trường đều là tùy chọn — OpenClaw sử dụng các giá trị mặc định an toàn khi bị bỏ qua.

---

## Kênh

Mỗi kênh tự động khởi động khi phần cấu hình của nó tồn tại (trừ khi `enabled: false`).

### Truy cập tin nhắn riêng và nhóm

Tất cả các kênh đều hỗ trợ chính sách tin nhắn riêng và chính sách nhóm:

| Chính sách tin nhắn riêng | Hành vi                                                        |
| ------------------- | --------------------------------------------------------------- |
| `pairing` (mặc định) | Người gửi không xác định nhận mã ghép nối một lần; chủ sở hữu phải phê duyệt |
| `allowlist`         | Chỉ người gửi trong `allowFrom` (hoặc kho cho phép đã ghép nối)             |
| `open`              | Cho phép tất cả tin nhắn riêng đến (yêu cầu `allowFrom: ["*"]`)             |
| `disabled`          | Bỏ qua tất cả tin nhắn riêng đến                                          |

| Chính sách nhóm          | Hành vi                                               |
| --------------------- | ------------------------------------------------------ |
| `allowlist` (mặc định) | Chỉ các nhóm khớp với danh sách cho phép đã cấu hình          |
| `open`                | Bỏ qua danh sách cho phép của nhóm (kiểm soát nhắc đến vẫn áp dụng) |
| `disabled`            | Chặn tất cả tin nhắn nhóm/phòng                          |

<Note>
`channels.defaults.groupPolicy` đặt giá trị mặc định khi `groupPolicy` của nhà cung cấp chưa được đặt.
Mã ghép nối hết hạn sau 1 giờ. Các yêu cầu ghép nối tin nhắn riêng đang chờ xử lý được giới hạn ở **3 yêu cầu mỗi kênh**.
Nếu một khối nhà cung cấp bị thiếu hoàn toàn (`channels.<provider>` không có), chính sách nhóm thời gian chạy sẽ quay về `allowlist` (fail-closed) với cảnh báo khởi động.
</Note>

### Ghi đè mô hình kênh

Sử dụng `channels.modelByChannel` để ghim các ID kênh cụ thể vào một mô hình. Các giá trị chấp nhận `provider/model` hoặc các bí danh mô hình đã cấu hình. Ánh xạ kênh được áp dụng khi một phiên chưa có ghi đè mô hình (ví dụ, được đặt qua `/model`).
``json5
{
  channels: {
    modelByChannel: {
      discord: {
        "123456789012345678": "anthropic/claude-opus-4-6",
      },
      slack: {
        C1234567890: "openai/gpt-4.1",
      },
      telegram: {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6",
      },
    },
  },
}
``__OC_I19N_0001__channels.defaults__OC_I19N_0002__`__OC_I19N_0003__`
- `channels.defaults.groupPolicy`: chính sách nhóm dự phòng khi `groupPolicy` cấp nhà cung cấp chưa được đặt.
- `channels.defaults.heartbeat.showOk`: bao gồm trạng thái kênh khỏe mạnh trong đầu ra heartbeat.
- __OC_I19N_0003__: bao gồm trạng thái suy giảm/lỗi trong đầu ra heartbeat.
- `channels.defaults.heartbeat.useIndicator`: hiển thị đầu ra heartbeat kiểu chỉ báo nhỏ gọn.

### WhatsApp

WhatsApp chạy qua kênh web của Gateway (Baileys Web). Nó tự động khởi động khi có một phiên được liên kết.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blue ticks (false in self-chat mode)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```
<Accordion title="WhatsApp đa tài khoản">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- Outbound commands default to account `default` if present; otherwise the first configured account id (sorted).
- Legacy single-account Baileys auth dir is migrated by `openclaw doctor` into `whatsapp/default`.
- Per-account overrides: `channels.whatsapp.accounts.<id>.sendReadReceipts`, `channels.whatsapp.accounts.<id>.dmPolicy`, `channels.whatsapp.accounts.<id>.allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (default: off)
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        autoSelectFamily: true,
        dnsResultOrder: "ipv4first",
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```</Accordion>
- Mã thông báo bot: `channels.telegram.botToken` hoặc `channels.telegram.tokenFile`, với `TELEGRAM_BOT_TOKEN` làm dự phòng cho tài khoản mặc định.
- `configWrites: false` chặn các ghi cấu hình do Telegram khởi tạo (di chuyển ID siêu nhóm, `/config set|unset`).
- Xem trước luồng Telegram sử dụng `sendMessage` + `editMessageText` (hoạt động trong tin nhắn trực tiếp và trò chuyện nhóm).
- Chính sách thử lại: xem [Chính sách thử lại](/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "123456789012345678"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress maps to partial on Discord)
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // opt-in for sessions_spawn({ thread: true })
      },
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```
- Mã thông báo: `channels.discord.token`, với `DISCORD_BOT_TOKEN` làm dự phòng cho tài khoản mặc định.
- Sử dụng `user:<id>` (Tin nhắn riêng) hoặc `channel:<id>` (kênh guild) cho các mục tiêu gửi; ID số trần sẽ bị từ chối.
- Slug của guild là chữ thường với khoảng trắng được thay thế bằng `-`; khóa kênh sử dụng tên đã được slug (không có `#`). Ưu tiên ID guild.
- Tin nhắn do bot tạo bị bỏ qua theo mặc định. `allowBots: true` cho phép chúng (tin nhắn của chính bot vẫn bị lọc).
- `maxLinesPerMessage` (mặc định 17) chia các tin nhắn dài ngay cả khi dưới 2000 ký tự.
- `channels.discord.threadBindings` kiểm soát định tuyến ràng buộc luồng Discord:
  - `enabled`: Ghi đè Discord cho các tính năng phiên ràng buộc luồng (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`, và gửi/định tuyến ràng buộc)
  - `idleHours`: Ghi đè Discord cho tính năng tự động hủy tập trung do không hoạt động sau số giờ nhất định (`0` tắt)
  - `maxAgeHours`: Ghi đè Discord cho tuổi tối đa cứng tính bằng giờ (`0` tắt)
  - `spawnSubagentSessions`: công tắc chọn tham gia để tạo/ràng buộc luồng tự động `sessions_spawn({ thread: true })`
- `channels.discord.ui.components.accentColor` đặt màu nhấn cho các vùng chứa thành phần Discord v2.
- `channels.discord.voice` cho phép các cuộc trò chuyện kênh thoại Discord và tùy chọn tự động tham gia + ghi đè TTS.
- `channels.discord.voice.daveEncryption` và `channels.discord.voice.decryptionFailureTolerance` chuyển tiếp đến các tùy chọn DAVE của `@discordjs/voice` (`true` và `24` theo mặc định).
- OpenClaw còn cố gắng khôi phục nhận thoại bằng cách rời/tham gia lại một phiên thoại sau nhiều lần giải mã thất bại.
- `channels.discord.streaming` là khóa chế độ truyền phát chính tắc. Các giá trị `streamMode` cũ và giá trị boolean `streaming` được tự động di chuyển.
- `channels.discord.dangerouslyAllowNameMatching` bật lại tính năng khớp tên/thẻ có thể thay đổi (chế độ tương thích khẩn cấp).

**Các chế độ thông báo phản ứng:** `off` (không có), `own` (tin nhắn của bot, mặc định), `all` (tất cả tin nhắn), `allowlist` (từ `guilds.<id>.users` trên tất cả tin nhắn).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```
- JSON tài khoản dịch vụ: nội tuyến (`serviceAccount`) hoặc dựa trên tệp (`serviceAccountFile`).
- SecretRef tài khoản dịch vụ cũng được hỗ trợ (`serviceAccountRef`).
- Dự phòng biến môi trường: `GOOGLE_CHAT_SERVICE_ACCOUNT` hoặc `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Sử dụng `spaces/<spaceId>` hoặc `users/<userId>` cho các mục tiêu phân phối.
- `channels.googlechat.dangerouslyAllowNameMatching` kích hoạt lại tính năng khớp chính email có thể thay đổi (chế độ tương thích khẩn cấp).

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      streaming: "partial", // off | partial | block | progress (preview mode)
      nativeStreaming: true, // use Slack native streaming API when streaming=partial
      mediaMaxMb: 20,
    },
  },
}
```
- **Chế độ Socket** yêu cầu cả `botToken` và `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` để dự phòng biến môi trường tài khoản mặc định).
- **Chế độ HTTP** yêu cầu `botToken` cộng với `signingSecret` (ở cấp gốc hoặc cho mỗi tài khoản).
- `configWrites: false` chặn ghi cấu hình do Slack khởi tạo.
- `channels.slack.streaming` là khóa chế độ truyền phát chính tắc. Các giá trị cũ `streamMode` và giá trị boolean `streaming` được tự động di chuyển.
- Sử dụng `user:<id>` (Tin nhắn riêng) hoặc `channel:<id>` cho các mục tiêu phân phối.

**Các chế độ thông báo phản ứng:** `off`, `own` (mặc định), `all`, `allowlist` (từ `reactionAllowlist`).

**Cách ly phiên luồng:** `thread.historyScope` là theo luồng (mặc định) hoặc chia sẻ trên toàn kênh. `thread.inheritParent` sao chép bản ghi kênh cha sang các luồng mới.

| Nhóm hành động      | Mặc định | Ghi chú                      |
| ----------------- | -------- | --------------------------- |
| phản ứng          | đã bật   | Phản ứng + liệt kê phản ứng |
| tin nhắn          | đã bật   | Đọc/gửi/chỉnh sửa/xóa       |
| ghim              | đã bật   | Ghim/bỏ ghim/liệt kê        |
| thông tin thành viên | đã bật   | Thông tin thành viên        |
| danh sách biểu tượng cảm xúc | đã bật   | Danh sách biểu tượng cảm xúc tùy chỉnh |

### Mattermost

Mattermost được phân phối dưới dạng một plugin: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```
Chế độ trò chuyện: `oncall` (phản hồi khi được @-đề cập, mặc định), `onmessage` (mọi tin nhắn), `onchar` (tin nhắn bắt đầu bằng tiền tố kích hoạt).

- `channels.mattermost.configWrites`: cho phép hoặc từ chối ghi cấu hình do Mattermost khởi tạo.
- `channels.mattermost.requireMention`: yêu cầu `@mention` trước khi trả lời trong các kênh.

### Signal

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // optional account binding
      dmPolicy: "pairing",
      allowFrom: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      configWrites: true,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Reaction notification modes:** `off`, `own` (default), `all`, `allowlist__OC_I19N_0011__reactionAllowlist`).

- `channels.signal.account`: pin channel startup to a specific Signal account identity.
- `channels.signal.configWrites`: cho phép hoặc từ chối ghi cấu hình do Signal khởi tạo.
### BlueBubbles

BlueBubbles là đường dẫn iMessage được khuyến nghị (được hỗ trợ bởi plugin, cấu hình dưới `channels.bluebubbles`).

```json5
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, group controls, and advanced actions:
      // see /channels/bluebubbles
    },
  },
}
```

- Core key paths covered here: `channels.bluebubbles`, `channels.bluebubbles.dmPolicy`.
- Full BlueBubbles channel configuration is documented in [BlueBubbles](/channels/bluebubbles).

### iMessage

OpenClaw spawns `imsg rpc` (JSON-RPC over stdio). No daemon or port required.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```
- Yêu cầu Quyền truy cập toàn bộ đĩa vào Cơ sở dữ liệu Tin nhắn.
- Ưu tiên các mục tiêu `chat_id:<id>`. Sử dụng `imsg chats --limit 20` để liệt kê các cuộc trò chuyện.
- `cliPath` có thể trỏ đến một trình bao bọc SSH; đặt `remoteHost` (`host` hoặc `user@host`) để tìm nạp tệp đính kèm SCP.
- `attachmentRoots` và `remoteAttachmentRoots` hạn chế các đường dẫn tệp đính kèm đến (mặc định: `/Users/*/Library/Messages/Attachments`).
- SCP sử dụng kiểm tra khóa máy chủ nghiêm ngặt, vì vậy hãy đảm bảo khóa máy chủ chuyển tiếp đã tồn tại trong `~/.ssh/known_hosts`.
- `channels.imessage.configWrites`: cho phép hoặc từ chối ghi cấu hình do iMessage khởi tạo.

<Accordion title="Ví dụ về trình bao bọc SSH iMessage">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Microsoft Teams

Microsoft Teams is extension-backed and configured under `channels.msteams`.

```json5
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, team/channel policies:
      // see /channels/msteams
    },
  },
}
```
- Các đường dẫn khóa chính được đề cập ở đây: `channels.msteams`, `channels.msteams.configWrites`.
- Cấu hình Teams đầy đủ (thông tin xác thực, webhook, chính sách tin nhắn riêng/nhóm, ghi đè theo nhóm/kênh) được ghi lại trong [Microsoft Teams](/channels/msteams).

### IRC

IRC được hỗ trợ bởi tiện ích mở rộng và được cấu hình dưới `channels.irc`.

```json5
{
  channels: {
    irc: {
      enabled: true,
      dmPolicy: "pairing",
      configWrites: true,
      nickserv: {
        enabled: true,
        service: "NickServ",
        password: "${IRC_NICKSERV_PASSWORD}",
        register: false,
        registerEmail: "bot@example.com",
      },
    },
  },
}
```

- Core key paths covered here: `channels.irc`, `channels.irc.dmPolicy`, `channels.irc.configWrites`, `channels.irc.nickserv.*`.
- Cấu hình kênh IRC đầy đủ (máy chủ/cổng/TLS/kênh/danh sách cho phép/kiểm soát nhắc đến) được ghi lại trong [IRC](/channels/irc).

### Đa tài khoản (tất cả các kênh)

Chạy nhiều tài khoản trên mỗi kênh (mỗi tài khoản có `accountId` riêng):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- `default` is used when `accountId` is omitted (CLI + routing).
- Env tokens only apply to the **default** account.
- Base channel settings apply to all accounts unless overridden per account.
- Use `bindings[].match.accountId` to route each account to a different agent.
- If you add a non-default account via `openclaw channels add` (or channel onboarding) while still on a single-account top-level channel config, OpenClaw moves account-scoped top-level single-account values into `channels.<channel>.accounts.default` first so the original account keeps working.
- Existing channel-only bindings (no `accountId`) keep matching the default account; account-scoped bindings remain optional.
- `openclaw doctor --fix` also repairs mixed shapes by moving account-scoped top-level single-account values into `accounts.default` when named accounts exist but `default` bị thiếu.
### Các kênh mở rộng khác

Nhiều kênh mở rộng được cấu hình là `channels.<id>` và được ghi lại trong các trang kênh chuyên dụng của chúng (ví dụ Feishu, Matrix, LINE, Nostr, Zalo, Nextcloud Talk, Synology Chat và Twitch).
Xem chỉ mục kênh đầy đủ: [Kênh](/channels).

### Kiểm soát nhắc đến trong trò chuyện nhóm

Tin nhắn nhóm mặc định **yêu cầu nhắc đến** (nhắc đến siêu dữ liệu hoặc mẫu regex). Áp dụng cho các cuộc trò chuyện nhóm trên WhatsApp, Telegram, Discord, Google Chat và iMessage.

**Các loại nhắc đến:**

- **Nhắc đến siêu dữ liệu**: Nhắc đến @-trên nền tảng gốc. Bị bỏ qua trong chế độ tự trò chuyện của WhatsApp.
- **Mẫu văn bản**: Mẫu Regex trong `agents.list[].groupChat.mentionPatterns`. Luôn được kiểm tra.
- Kiểm soát nhắc đến chỉ được thực thi khi có thể phát hiện (nhắc đến gốc hoặc ít nhất một mẫu).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` sets the global default. Channels can override with `channels.<channel>.historyLimit` (or per-account). Set `0` để tắt.

#### Giới hạn lịch sử tin nhắn riêng
`json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
``

Resolution: per-DM override → provider default → no limit (all retained).

Supported: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Self-chat mode

Include your own number in `allowFrom` to enable self-chat mode (ignores native @-mentions, only responds to text patterns):

``json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
`
### Lệnh (xử lý lệnh trò chuyện)

`json5
{
  commands: {
    native: "auto", // register native commands when supported
    text: true, // parse /commands in chat messages
    bash: false, // allow ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // allow /config
    debug: false, // allow /debug
    restart: false, // allow /restart + gateway restart tool
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
``

<Accordion title="Command details">

- Text commands must be **standalone** messages with leading `/`.
- `native: "auto"` turns on native commands for Discord/Telegram, leaves Slack off.
- Override per channel: `channels.discord.commands.native` (bool or `"auto"`). `false` clears previously registered commands.
- `channels.telegram.customCommands` adds extra Telegram bot menu entries.
- `bash: true` enables `! <cmd>` for host shell. Requires `tools.elevated.enabled` and sender in `tools.elevated.allowFrom.<channel>`.
- `config: true` enables `/config` (reads/writes `openclaw.json`).
- `channels.<provider>.configWrites` kiểm soát các thay đổi cấu hình trên mỗi kênh (mặc định: true).
- `allowFrom` là trên mỗi nhà cung cấp. Khi được đặt, đây là nguồn ủy quyền **duy nhất** (danh sách cho phép/ghép nối kênh và `useAccessGroups` bị bỏ qua).
- `useAccessGroups: false` cho phép các lệnh bỏ qua chính sách nhóm truy cập khi `allowFrom` không được đặt.

</Accordion>

---
## Mặc định của agent

### `agents.defaults.workspace`

Mặc định: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Optional repository root shown in the system prompt's Runtime line. If unset, OpenClaw auto-detects by walking upward from the workspace.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Disables automatic creation of workspace bootstrap files (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md__OC_I19N_0012__HEARTBEAT.md__OC_I19N_0013__BOOTSTRAP.md__OC_I19N_0014__`__OC_I19N_0015__``
### __OC_I19N_0000__

Số ký tự tối đa cho mỗi tệp bootstrap của không gian làm việc trước khi bị cắt bớt. Mặc định: __OC_I19N_0001__.

``__OC_I19N_0002__`__OC_I19N_0003__agents.defaults.bootstrapTotalMaxChars__OC_I19N_0004__150000__OC_I19N_0005__`__OC_I19N_0006__`__OC_I19N_0007__agents.defaults.imageMaxDimensionPx__OC_I19N_0008__1200__OC_I19N_0009__`__OC_I19N_0010__``
### `agents.defaults.userTimezone`

Múi giờ cho ngữ cảnh lời nhắc hệ thống (không phải dấu thời gian tin nhắn). Mặc định là múi giờ của máy chủ.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Time format in system prompt. Default: `auto__OC_I19N_0004__``json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```
- `model`: chấp nhận một chuỗi (`"provider/model"`) hoặc một đối tượng (`{ primary, fallbacks }`).
  - Dạng chuỗi chỉ đặt mô hình chính.
  - Dạng đối tượng đặt mô hình chính cộng với các mô hình dự phòng theo thứ tự.
- `imageModel`: chấp nhận một chuỗi (`"provider/model"`) hoặc một đối tượng (`{ primary, fallbacks }`).
  - Được sử dụng bởi đường dẫn công cụ `image` làm cấu hình mô hình thị giác của nó.
  - Cũng được sử dụng làm định tuyến dự phòng khi mô hình được chọn/mặc định không thể chấp nhận đầu vào hình ảnh.
- `model.primary`: định dạng `provider/model` (ví dụ: `anthropic/claude-opus-4-6`). Nếu bạn bỏ qua nhà cung cấp, OpenClaw sẽ giả định `anthropic` (không dùng nữa).
- `models`: danh mục mô hình đã cấu hình và danh sách cho phép cho `/model`. Mỗi mục có thể bao gồm `alias` (phím tắt) và `params` (đặc trưng của nhà cung cấp, ví dụ: `temperature`, `maxTokens`, `cacheRetention`, `context1m`).
- Ưu tiên hợp nhất `params` (cấu hình): `agents.defaults.models["provider/model"].params` là cơ sở, sau đó `agents.list[].params` (ID agent khớp) ghi đè theo khóa.
- Các trình ghi cấu hình sửa đổi các trường này (ví dụ: `/models set`, `/models set-image`, và các lệnh thêm/xóa dự phòng) lưu dạng đối tượng chuẩn và bảo toàn các danh sách dự phòng hiện có khi có thể.
- `maxConcurrent`: số lần chạy agent song song tối đa trên các phiên (mỗi phiên vẫn được tuần tự hóa). Mặc định: 1.

**Các bí danh viết tắt tích hợp sẵn** (chỉ áp dụng khi mô hình nằm trong `agents.defaults.models`):

| Bí danh          | Mô hình                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Các bí danh đã cấu hình của bạn luôn ưu tiên hơn các giá trị mặc định.

Các mô hình Z.AI GLM-4.x tự động bật chế độ suy nghĩ trừ khi bạn đặt `--thinking off` hoặc tự định nghĩa `agents.defaults.models["zai/<model>"].params.thinking`.
Các mô hình Z.AI bật `tool_stream` theo mặc định để truyền phát cuộc gọi công cụ. Đặt `agents.defaults.models["zai/<model>"].params.tool_stream` thành `false` để tắt nó.

### `agents.defaults.cliBackends`
Các backend CLI tùy chọn cho các lần chạy dự phòng chỉ văn bản (không gọi công cụ). Hữu ích như một bản sao lưu khi các nhà cung cấp API gặp sự cố.

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- Các backend CLI ưu tiên văn bản; các công cụ luôn bị vô hiệu hóa.
- Phiên được hỗ trợ khi `sessionArg` được đặt.
- Truyền ảnh qua được hỗ trợ khi `imageArg` chấp nhận đường dẫn tệp.

### `agents.defaults.heartbeat`

Các lần chạy nhịp tim định kỳ.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        directPolicy: "allow", // allow (default) | block
        target: "none", // default: none | options: last | whatsapp | telegram | discord | ...
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
        suppressToolErrorWarnings: false,
      },
    },
  },
}
```

- `every`: duration string (ms/s/m/h). Default: `30m`.
- `suppressToolErrorWarnings`: khi đúng, ngăn chặn các tải trọng cảnh báo lỗi công cụ trong các lần chạy nhịp tim.
- `directPolicy`: chính sách gửi trực tiếp/tin nhắn riêng. `allow` (mặc định) cho phép gửi trực tiếp đến mục tiêu. `block` ngăn chặn việc gửi trực tiếp đến mục tiêu và phát ra `reason=dm-blocked`.
- Theo từng agent: đặt `agents.list[].heartbeat`. Khi bất kỳ agent nào định nghĩa `heartbeat`, **chỉ những agent đó** mới chạy nhịp tim.
- Nhịp tim chạy hết lượt của agent — khoảng thời gian ngắn hơn sẽ tiêu tốn nhiều token hơn.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        identifierPolicy: "strict", // strict | off | custom
        identifierInstructions: "Preserve deployment IDs, ticket IDs, and host:port pairs exactly.", // used when identifierPolicy=custom
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` or `safeguard` (chunked summarization for long histories). See [Compaction](/concepts/compaction).
- `identifierPolicy`: `strict` (default), `off`, or `custom`. `strict` prepends built-in opaque identifier retention guidance during compaction summarization.
- `identifierInstructions`: optional custom identifier-preservation text used when `identifierPolicy=custom`.
- `memoryFlush`: lượt tác nhân im lặng trước khi tự động nén để lưu trữ các ký ức bền vững. Bỏ qua khi không gian làm việc chỉ đọc.

### `agents.defaults.contextPruning`

Cắt tỉa **kết quả công cụ cũ** từ ngữ cảnh trong bộ nhớ trước khi gửi đến LLM. **Không** sửa đổi lịch sử phiên trên đĩa.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` enables pruning passes.
- `ttl` kiểm soát tần suất việc cắt tỉa có thể chạy lại (sau lần chạm bộ nhớ đệm cuối cùng).
- Cắt tỉa sẽ cắt mềm các kết quả công cụ quá lớn trước, sau đó xóa cứng các kết quả công cụ cũ hơn nếu cần.

**Cắt mềm** giữ lại phần đầu + cuối và chèn `...` vào giữa.

**Xóa cứng** thay thế toàn bộ kết quả công cụ bằng trình giữ chỗ.

Lưu ý:

- Các khối hình ảnh không bao giờ bị cắt tỉa/xóa.
- Tỷ lệ dựa trên ký tự (xấp xỉ), không phải số lượng token chính xác.
- Nếu có ít hơn `keepLastAssistants` tin nhắn trợ lý, việc cắt tỉa sẽ bị bỏ qua.

</Accordion>

Xem [Cắt tỉa phiên](/concepts/session-pruning) để biết chi tiết về hành vi.

### Truyền phát theo khối

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```
- Các kênh không phải Telegram yêu cầu `*.blockStreaming: true` rõ ràng để bật trả lời theo khối.
- Ghi đè kênh: `channels.<channel>.blockStreamingCoalesce` (và các biến thể theo từng tài khoản). Signal/Slack/Discord/Google Chat mặc định `minChars: 1500`.
- `humanDelay`: tạm dừng ngẫu nhiên giữa các trả lời theo khối. `natural` = 800–2500ms. Ghi đè theo từng agent: `agents.list[].humanDelay`.

Xem [Truyền phát](/concepts/streaming) để biết chi tiết về hành vi + phân đoạn.

### Chỉ báo đang gõ

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- Defaults: `instant` for direct chats/mentions, `message` for unmentioned group chats.
- Per-session overrides: `session.typingMode`, `session.typingIntervalSeconds`.

See [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Optional **Docker sandboxing** for the embedded agent. See [Sandboxing](/gateway/sandboxing) for the full guide.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          network: "openclaw-sandbox-browser",
          cdpPort: 9222,
          cdpSourceRange: "172.21.0.1/32",
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```
<Accordion title="Chi tiết Sandbox">

**Truy cập không gian làm việc:**

- `none`: không gian làm việc sandbox theo phạm vi dưới `~/.openclaw/sandboxes`
- `ro`: không gian làm việc sandbox tại `/workspace`, không gian làm việc agent được gắn chỉ đọc tại `/agent`
- `rw`: không gian làm việc agent được gắn đọc/ghi tại `/workspace`

**Phạm vi:**

- `session`: container + không gian làm việc theo phiên
- `agent`: một container + không gian làm việc cho mỗi agent (mặc định)
- `shared`: container và không gian làm việc được chia sẻ (không có cách ly giữa các phiên)

**`setupCommand`** chạy một lần sau khi tạo container (qua `sh -lc`). Cần thoát mạng, quyền ghi root, người dùng root.

**Container mặc định là `network: "none"`** — đặt thành `"bridge"` (hoặc một mạng cầu nối tùy chỉnh) nếu agent cần truy cập ra ngoài.
`"host"` bị chặn. `"container:<id>"` bị chặn theo mặc định trừ khi bạn đặt rõ ràng
`sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true` (phá vỡ bảo mật).

**Tệp đính kèm đến** được đưa vào `media/inbound/*` trong không gian làm việc đang hoạt động.

**`docker.binds`** gắn thêm các thư mục máy chủ; các liên kết toàn cục và theo agent được hợp nhất.

**Trình duyệt sandbox** (`sandbox.browser.enabled`): Chromium + CDP trong một container. URL noVNC được đưa vào lời nhắc hệ thống. Không yêu cầu `browser.enabled` trong cấu hình chính.
Truy cập quan sát viên noVNC sử dụng xác thực VNC theo mặc định và OpenClaw phát ra một URL mã thông báo ngắn hạn (thay vì hiển thị mật khẩu trong URL được chia sẻ).

- `allowHostControl: false` (mặc định) chặn các phiên sandbox nhắm mục tiêu trình duyệt máy chủ.
- `network` mặc định là `openclaw-sandbox-browser` (mạng cầu nối chuyên dụng). Chỉ đặt thành `bridge` khi bạn muốn kết nối cầu nối toàn cục một cách rõ ràng.

</Accordion>
- __OC_I19N_0000__ tùy chọn hạn chế lưu lượng CDP vào tại rìa container theo một dải CIDR (ví dụ __OC_I19N_0001__).
- __OC_I19N_0002__ gắn thêm các thư mục máy chủ vào chỉ container trình duyệt sandbox. Khi được đặt (bao gồm __OC_I19N_0003__), nó sẽ thay thế __OC_I19N_0004__ cho container trình duyệt.

</Accordion>

Xây dựng ảnh:

``__OC_I19N_0005__`__OC_I19N_0006__agents.list__OC_I19N_0007__`__OC_I19N_0008__``
- `id`: ID agent ổn định (bắt buộc).
- `default`: khi nhiều giá trị được đặt, giá trị đầu tiên sẽ thắng (cảnh báo được ghi lại). Nếu không có giá trị nào được đặt, mục đầu tiên trong danh sách là mặc định.
- `model`: dạng chuỗi chỉ ghi đè `primary`; dạng đối tượng `{ primary, fallbacks }` ghi đè cả hai (`[]` vô hiệu hóa các dự phòng toàn cục). Các cron job chỉ ghi đè `primary` vẫn kế thừa các dự phòng mặc định trừ khi bạn đặt `fallbacks: []`.
- `params`: các tham số luồng trên mỗi agent được hợp nhất với mục mô hình đã chọn trong `agents.defaults.models`. Sử dụng điều này cho các ghi đè dành riêng cho agent như `cacheRetention`, `temperature`, hoặc `maxTokens` mà không cần sao chép toàn bộ danh mục mô hình.
- `identity.avatar`: đường dẫn tương đối trong không gian làm việc, URL `http(s)`, hoặc URI `data:`.
- `identity` lấy các giá trị mặc định: `ackReaction` từ `emoji`, `mentionPatterns` từ `name`/`emoji`.
- `subagents.allowAgents`: danh sách cho phép các ID agent cho `sessions_spawn` (`["*"]` = bất kỳ; mặc định: chỉ cùng agent).

---

## Định tuyến đa agent

Chạy nhiều agent riêng biệt bên trong một Gateway. Xem [Đa Agent](/concepts/multi-agent).

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

### Binding match fields

- `match.channel` (required)
- `match.accountId` (optional; `*` = any account; omitted = default account)
- `match.peer` (optional; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (optional; channel-specific)

**Deterministic match order:**

1. `match.peer`
2. `match.guildId`
3. __OC_I19N_0000__
4. __OC_I19N_0001__ (chính xác, không có ngang hàng/hội/nhóm)
5. __OC_I19N_0002__ (toàn kênh)
6. agent mặc định

Trong mỗi cấp, mục __OC_I19N_0003__ khớp đầu tiên sẽ được ưu tiên.

### Hồ sơ truy cập theo từng agent

<Accordion title="Truy cập đầy đủ (không có sandbox)">

``__OC_I19N_0004__`__OC_I19N_0005__`__OC_I19N_0006__``
</Accordion>

<Accordion title="Không có quyền truy cập hệ thống tệp (chỉ nhắn tin)">

``__OC_I19N_0000__``
</Accordion>

Xem [Sandbox & Công cụ Đa-Agent](__OC_I19N_0000__) để biết chi tiết về quyền ưu tiên.

---

## Phiên

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    parentForkMaxTokens: 100000, // skip parent-thread fork above this token count (0 disables)
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
      resetArchiveRetention: "30d", // duration or false
      maxDiskBytes: "500mb", // optional hard budget
      highWaterBytes: "400mb", // optional cleanup target
    },
    threadBindings: {
      enabled: true,
      idleHours: 24, // default inactivity auto-unfocus in hours (`0__OC_I19N_0001__0` disables)
    },
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```
<Accordion title="Chi tiết trường phiên">

- **`dmScope`**: cách các tin nhắn riêng được nhóm.
  - `main`: tất cả các tin nhắn riêng chia sẻ phiên chính.
  - `per-peer`: cách ly theo ID người gửi trên các kênh.
  - `per-channel-peer`: cách ly theo từng kênh + người gửi (khuyên dùng cho hộp thư đến nhiều người dùng).
  - __OC_I19N_0004__: cách ly theo từng tài khoản + kênh + người gửi (khuyên dùng cho nhiều tài khoản).
- **`identityLinks`**: ánh xạ các ID chuẩn tới các đối tác có tiền tố nhà cung cấp để chia sẻ phiên đa kênh.
- **`reset`**: chính sách đặt lại chính. `daily` đặt lại vào `atHour` giờ địa phương; `idle` đặt lại sau `idleMinutes`. Khi cả hai được cấu hình, cái nào hết hạn trước sẽ thắng.
- **`resetByType`**: ghi đè theo loại (`direct`, `group`, `thread`). `dm` cũ được chấp nhận làm bí danh cho `direct`.
- **`parentForkMaxTokens`**: số lượng `totalTokens` phiên cha tối đa được phép khi tạo phiên luồng phân nhánh (mặc định `100000`).
  - Nếu `totalTokens` của phiên cha vượt quá giá trị này, OpenClaw sẽ bắt đầu một phiên luồng mới thay vì kế thừa lịch sử bản ghi của phiên cha.
  - Đặt `0` để tắt bảo vệ này và luôn cho phép phân nhánh từ phiên cha.
- **`mainKey`**: trường cũ. Thời gian chạy hiện luôn sử dụng `"main"` cho nhóm trò chuyện trực tiếp chính.
- **`sendPolicy`**: khớp theo `channel`, `chatType` (`direct|group|channel`, với bí danh `dm` cũ), `keyPrefix`, hoặc `rawKeyPrefix`. Từ chối đầu tiên sẽ thắng.
- **`maintenance`**: kiểm soát dọn dẹp + lưu giữ kho phiên.
  - `mode`: `warn` chỉ phát ra cảnh báo; `enforce` áp dụng dọn dẹp.
  - `pruneAfter`: giới hạn tuổi cho các mục cũ (mặc định `30d`).
  - `maxEntries`: số lượng mục tối đa trong `sessions.json` (mặc định `500`).
  - `rotateBytes`: xoay vòng `sessions.json` khi nó vượt quá kích thước này (mặc định `10mb`).
  - `resetArchiveRetention`: lưu giữ cho các kho lưu trữ bản ghi `*.reset.<timestamp>`. Mặc định là `pruneAfter`; đặt `false` để tắt.
  - `maxDiskBytes`: ngân sách đĩa thư mục phiên tùy chọn. Ở chế độ `warn`, nó ghi lại cảnh báo; ở chế độ `enforce`, nó xóa các tạo phẩm/phiên cũ nhất trước.
  - `highWaterBytes`: mục tiêu tùy chọn sau khi dọn dẹp ngân sách. Mặc định là `80%` của `maxDiskBytes`.
- **`threadBindings`**: các giá trị mặc định toàn cầu cho các tính năng phiên ràng buộc luồng.
  - `enabled`: công tắc mặc định chính (các nhà cung cấp có thể ghi đè; Discord sử dụng `channels.discord.threadBindings.enabled`)
  - `idleHours`: mặc định tự động bỏ tập trung khi không hoạt động tính bằng giờ (`0` tắt; các nhà cung cấp có thể ghi đè)
  - `maxAgeHours`: mặc định tuổi tối đa cứng tính bằng giờ (`0` tắt; các nhà cung cấp có thể ghi đè)

</Accordion>
## Tin nhắn

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```
### Tiền tố phản hồi

Ghi đè theo kênh/tài khoản: `channels.<channel>.responsePrefix`, `channels.<channel>.accounts.<id>.responsePrefix`.

Độ phân giải (cụ thể nhất thắng): tài khoản → kênh → toàn cầu. `""` vô hiệu hóa và dừng chuỗi. `"auto"` suy ra `[{identity.name}]`.

**Biến mẫu:**

| Biến          | Mô tả            | Ví dụ                     |
| ----------------- | ---------------------- | --------------------------- |
| `{model}`         | Tên mô hình ngắn       | `claude-opus-4-6`           |
| `{modelFull}`     | Mã định danh mô hình đầy đủ  | `anthropic/claude-opus-4-6` |
| `{provider}`      | Tên nhà cung cấp          | `anthropic`                 |
| `{thinkingLevel}` | Mức độ suy nghĩ hiện tại | `high`, `low`, `off`        |
| `{identity.name}` | Tên định danh agent    | (giống như `"auto"`)          |

Các biến không phân biệt chữ hoa chữ thường. `{think}` là một bí danh cho `{thinkingLevel}`.

### Phản ứng xác nhận

- Mặc định là `identity.emoji` của agent đang hoạt động, nếu không thì là `"👀"`. Đặt `""` để vô hiệu hóa.
- Ghi đè theo kênh: `channels.<channel>.ackReaction`, `channels.<channel>.accounts.<id>.ackReaction`.
- Thứ tự phân giải: tài khoản → kênh → `messages.ackReaction` → dự phòng định danh.
- Phạm vi: `group-mentions` (mặc định), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: xóa xác nhận sau khi trả lời (chỉ dành cho Slack/Discord/Telegram/Google Chat).

### Khử rung đầu vào

Gộp các tin nhắn văn bản nhanh từ cùng một người gửi thành một lượt agent duy nhất. Phương tiện/tệp đính kèm được gửi ngay lập tức. Các lệnh điều khiển bỏ qua quá trình khử rung.
### TTS (chuyển văn bản thành giọng nói)

``__OC_I19N_0000__``
- `auto` kiểm soát TTS tự động. `/tts off|always|inbound|tagged` ghi đè cho mỗi phiên.
- `summaryModel` ghi đè `agents.defaults.model.primary` cho tóm tắt tự động.
- `modelOverrides` được bật theo mặc định; `modelOverrides.allowProvider` mặc định là `false` (chọn tham gia).
- Khóa API dự phòng về `ELEVENLABS_API_KEY`/`XI_API_KEY` và `OPENAI_API_KEY`.

---
## Đàm thoại

Cài đặt mặc định cho chế độ Đàm thoại (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice IDs fall back to `ELEVENLABS_VOICE_ID` or `SAG_VOICE_ID`.
- `apiKey` falls back to `ELEVENLABS_API_KEY`.
- `voiceAliases` cho phép các chỉ thị Đàm thoại sử dụng tên thân thiện.

---

## Công cụ

### Hồ sơ công cụ

`tools.profile` đặt danh sách cho phép cơ bản trước `tools.allow`/`tools.deny`:

| Hồ sơ     | Bao gồm                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | Chỉ `session_status`                                                                      |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Không giới hạn (giống như chưa đặt)                                                            |

### Nhóm công cụ

| Nhóm              | Công cụ                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` được chấp nhận làm bí danh cho `exec`)                            |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Tất cả các công cụ tích hợp sẵn (không bao gồm các plugin của nhà cung cấp)                                           |

### `tools.allow` / `tools.deny`

Chính sách cho phép/từ chối công cụ toàn cầu (từ chối được ưu tiên). Không phân biệt chữ hoa chữ thường, hỗ trợ các ký tự đại diện `*`. Được áp dụng ngay cả khi Docker sandbox tắt.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Further restrict tools for specific providers or models. Order: base profile → provider profile → allow/deny.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

Controls elevated (host) exec access:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["1234567890123", "987654321098765432"],
      },
    },
  },
}
```
- Ghi đè theo agent (`agents.list[].tools.elevated`) chỉ có thể hạn chế thêm.
- `/elevated on|off|ask|full` lưu trữ trạng thái theo phiên; chỉ thị nội tuyến áp dụng cho một tin nhắn duy nhất.
- `exec` nâng cao chạy trên máy chủ, bỏ qua sandboxing.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.loopDetection`

Tool-loop safety checks are **disabled by default**. Set `enabled: true` to activate detection.
Settings can be defined globally in `tools.loopDetection` and overridden per-agent at `agents.list[].tools.loopDetection`.

`json5
{
  tools: {
    loopDetection: {
      enabled: true,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
``

- `historySize`: max tool-call history retained for loop analysis.
- `warningThreshold`: repeating no-progress pattern threshold for warnings.
- `criticalThreshold`: higher repeating threshold for blocking critical loops.
- `globalCircuitBreakerThreshold`: hard stop threshold for any no-progress run.
- `detectors.genericRepeat`: warn on repeated same-tool/same-args calls.
- `detectors.knownPollNoProgress`: warn/block on known poll tools (`process.poll`, `command_status`, etc.).
- `detectors.pingPong`: warn/block on alternating no-progress pair patterns.
- If `warningThreshold >= criticalThreshold` or `criticalThreshold >= globalCircuitBreakerThreshold`, validation fails.

### `tools.web`
``json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // or BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

Configures inbound media understanding (image/audio/video):

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
``
<Accordion title="Các trường nhập mô hình phương tiện">

**Mục nhà cung cấp** (`type: "provider"` hoặc bỏ qua):

- `provider`: ID nhà cung cấp API (`openai`, `anthropic`, `google`/`gemini`, `groq`, v.v.)
- `model`: ghi đè ID mô hình
- `profile` / `preferredProfile`: lựa chọn hồ sơ xác thực

**Mục CLI** (`type: "cli"`):

- `command`: tệp thực thi để chạy
- `args`: đối số mẫu (hỗ trợ `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, v.v.)

**Các trường chung:**

- `capabilities`: danh sách tùy chọn (`image`, `audio`, `video`). Mặc định: `openai`/`anthropic`/`minimax` → hình ảnh, `google` → hình ảnh+âm thanh+video, `groq` → âm thanh.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: ghi đè cho từng mục.
- Các lỗi sẽ chuyển sang mục tiếp theo.

Xác thực nhà cung cấp tuân theo thứ tự tiêu chuẩn: hồ sơ xác thực → biến môi trường → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```
### `tools.sessions`

Kiểm soát các phiên có thể được nhắm mục tiêu bởi các công cụ phiên (`sessions_list`, `sessions_history`, `sessions_send`).

Mặc định: `tree` (phiên hiện tại + các phiên được tạo ra bởi nó, chẳng hạn như subagents).

```json5
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      visibility: "tree",
    },
  },
}
```

Notes:

- `self`: only the current session key.
- `tree`: current session + sessions spawned by the current session (subagents).
- `agent`: any session belonging to the current agent id (can include other users if you run per-sender sessions under the same agent id).
- `all`: any session. Cross-agent targeting still requires `tools.agentToAgent`.
- Sandbox clamp: when the current session is sandboxed and `agents.defaults.sandbox.sessionToolsVisibility="spawned"`, visibility is forced to `tree` even if `tools.sessions.visibility="all"`.

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        runTimeoutSeconds: 900,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```
- `model`: mô hình mặc định cho các sub-agent được tạo ra. Nếu bỏ qua, các sub-agent sẽ kế thừa mô hình của người gọi.
- `runTimeoutSeconds`: thời gian chờ mặc định (giây) cho `sessions_spawn` khi lệnh gọi công cụ bỏ qua `runTimeoutSeconds`. `0` nghĩa là không có thời gian chờ.
- Chính sách công cụ cho từng sub-agent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Nhà cung cấp tùy chỉnh và URL cơ sở

OpenClaw sử dụng danh mục mô hình pi-coding-agent. Thêm các nhà cung cấp tùy chỉnh thông qua `models.providers` trong cấu hình hoặc `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```
- Sử dụng `authHeader: true` + `headers` cho các nhu cầu xác thực tùy chỉnh.
- Ghi đè thư mục gốc cấu hình agent bằng `OPENCLAW_AGENT_DIR` (hoặc `PI_CODING_AGENT_DIR`).
- Ưu tiên hợp nhất cho các ID nhà cung cấp khớp:
  - `models.json` `apiKey`/`baseUrl` của agent không trống sẽ thắng.
  - `apiKey`/`baseUrl` của agent trống hoặc thiếu sẽ quay lại `models.providers` trong cấu hình.
  - Sử dụng `models.mode: "replace"` khi bạn muốn cấu hình ghi đè hoàn toàn `models.json`.

### Chi tiết trường nhà cung cấp

- `models.mode`: hành vi danh mục nhà cung cấp (`merge` hoặc `replace`).
- `models.providers`: bản đồ nhà cung cấp tùy chỉnh được lập khóa bằng ID nhà cung cấp.
- `models.providers.*.api`: bộ điều hợp yêu cầu (`openai-completions`, `openai-responses`, `anthropic-messages`, `google-generative-ai`, v.v.).
- `models.providers.*.apiKey`: thông tin xác thực nhà cung cấp (ưu tiên thay thế SecretRef/biến môi trường).
- `models.providers.*.auth`: chiến lược xác thực (`api-key`, `token`, `oauth`, `aws-sdk`).
- `models.providers.*.authHeader`: buộc truyền thông tin xác thực trong tiêu đề `Authorization` khi cần.
- `models.providers.*.baseUrl`: URL cơ sở API upstream.
- `models.providers.*.headers`: các tiêu đề tĩnh bổ sung cho định tuyến proxy/người thuê.
- `models.providers.*.models`: các mục danh mục mô hình nhà cung cấp rõ ràng.
- `models.bedrockDiscovery`: thư mục gốc cài đặt tự động khám phá Bedrock.
- `models.bedrockDiscovery.enabled`: bật/tắt thăm dò khám phá.
- `models.bedrockDiscovery.region`: khu vực AWS để khám phá.
- `models.bedrockDiscovery.providerFilter`: bộ lọc ID nhà cung cấp tùy chọn để khám phá có mục tiêu.
- `models.bedrockDiscovery.refreshInterval`: khoảng thời gian thăm dò để làm mới khám phá.
- `models.bedrockDiscovery.defaultContextWindow`: cửa sổ ngữ cảnh dự phòng cho các mô hình được khám phá.
- `models.bedrockDiscovery.defaultMaxTokens`: số token đầu ra tối đa dự phòng cho các mô hình được khám phá.

### Ví dụ về nhà cung cấp

<Accordion title="Cerebras (GLM 4.6 / 4.7)">
__OC_I19N_0000__
</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

Set `OPENCODE_API_KEY` (or `OPENCODE_ZEN_API_KEY`). Shortcut: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```
Đặt `ZAI_API_KEY`. `z.ai/*` và `z-ai/*` là các bí danh được chấp nhận. Phím tắt: `openclaw onboard --auth-choice zai-api-key`.

- Điểm cuối chung: `https://api.z.ai/api/paas/v4`
- Điểm cuối mã hóa (mặc định): `https://api.z.ai/api/coding/paas/v4`
- Đối với điểm cuối chung, hãy định nghĩa một nhà cung cấp tùy chỉnh với ghi đè URL cơ sở.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
Đối với điểm cuối tại Trung Quốc: __OC_I19N_0000__ hoặc __OC_I19N_0001__.

</Accordion>

<Accordion title="Kimi Lập trình">

``__OC_I19N_0002__`__OC_I19N_0003__openclaw onboard --auth-choice kimi-code-api-key__OC_I19N_0004__`__OC_I19N_0005__``
URL cơ sở nên bỏ qua `/v1` (ứng dụng khách Anthropic tự động thêm vào). Phím tắt: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (trực tiếp)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
Đặt `MINIMAX_API_KEY`. Phím tắt: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Mô hình cục bộ (LM Studio)">

Xem [Mô hình cục bộ](__OC_I118N_0002__). Tóm lại: chạy MiniMax M2.1 qua API Phản hồi của LM Studio trên phần cứng mạnh; giữ các mô hình được lưu trữ hợp nhất để dự phòng.

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
``__OC_I19N_0001__allowBundled__OC_I19N_0002__entries.<skillKey>.enabled: false__OC_I19N_0003__entries.<skillKey>.apiKey`: tiện lợi cho Skills khai báo một biến môi trường chính (chuỗi văn bản thuần túy hoặc đối tượng SecretRef).

---
## Plugin

``__OC_I19N_0000__`__OC_I19N_0001__~/.openclaw/extensions__OC_I19N_0002__<workspace>/.openclaw/extensions__OC_I19N_0003__plugins.load.paths__OC_I19N_0004__allow__OC_I19N_0005__deny__OC_I19N_0006__plugins.entries.<id>.apiKey__OC_I19N_0007__plugins.entries.<id>.env__OC_I19N_0008__plugins.entries.<id>.config__OC_I19N_0009__plugins.slots.memory__OC_I19N_0010__"none"__OC_I19N_0011__plugins.installs__OC_I19N_0012__openclaw plugins update`.
  - Bao gồm `source`, `spec`, `sourcePath`, `installPath`, __OC_I118N_0004__, `resolvedName`, `resolvedVersion`, `resolvedSpec`, `integrity`, `shasum`, `resolvedAt`, `installedAt`.
  - Coi `plugins.installs.*` là trạng thái được quản lý; ưu tiên các lệnh CLI hơn là chỉnh sửa thủ công.

Xem [Plugins](/tools/plugin).

---

## Trình duyệt

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // default trusted-network mode
      // allowPrivateNetwork: true, // legacy alias
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false__OC_I19N_0002__act:evaluate__OC_I19N_0003__wait --fn__OC_I19N_0004__ssrfPolicy.dangerouslyAllowPrivateNetwork__OC_I19N_0005__true` khi chưa được đặt (mô hình mạng đáng tin cậy).
- Đặt `ssrfPolicy.dangerouslyAllowPrivateNetwork: false` để điều hướng trình duyệt chỉ công khai nghiêm ngặt.
- `ssrfPolicy.allowPrivateNetwork` vẫn được hỗ trợ như một bí danh kế thừa.
- Ở chế độ nghiêm ngặt, sử dụng `ssrfPolicy.hostnameAllowlist` và `ssrfPolicy.allowedHostnames` cho các ngoại lệ rõ ràng.
- Hồ sơ từ xa chỉ có thể đính kèm (bắt đầu/dừng/đặt lại bị vô hiệu hóa).
- Thứ tự tự động phát hiện: trình duyệt mặc định nếu dựa trên Chromium → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Dịch vụ điều khiển: chỉ local loopback (cổng được lấy từ `gateway.port`, mặc định `18791`).

---
## Giao diện người dùng

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: accent color for native app UI chrome (Talk Mode bubble tint, etc.).
- `assistant`: Kiểm soát ghi đè nhận dạng giao diện người dùng. Sẽ quay về nhận dạng agent đang hoạt động.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // none | token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowedOrigins: ["https://control.example.com"], // required for non-loopback Control UI
      // dangerouslyAllowHostHeaderOriginFallback: false, // dangerous Host-header origin fallback mode
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    // Optional. Default false.
    allowRealIpFallback: false,
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```
<Accordion title="Chi tiết trường Gateway">

- `mode`: `local` (chạy gateway) hoặc `remote` (kết nối với gateway từ xa). Gateway từ chối khởi động trừ khi `local`.
- `port`: một cổng đa kênh duy nhất cho WS + HTTP. Thứ tự ưu tiên: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (mặc định), `lan` (`0.0.0.0`), `tailnet` (chỉ IP Tailscale), hoặc `custom`.
- **Xác thực**: bắt buộc theo mặc định. Các liên kết không phải local loopback yêu cầu một mã thông báo/mật khẩu chia sẻ. Trình hướng dẫn thiết lập ban đầu tạo một mã thông báo theo mặc định.
- `gateway.auth.mode: "none"`: chế độ không xác thực rõ ràng. Chỉ sử dụng cho các thiết lập local loopback đáng tin cậy; điều này cố ý không được cung cấp bởi các lời nhắc thiết lập ban đầu.
- `gateway.auth.mode: "trusted-proxy"`: ủy quyền xác thực cho một proxy ngược nhận biết danh tính và tin cậy các tiêu đề danh tính từ `gateway.trustedProxies` (xem [Xác thực Proxy đáng tin cậy](/gateway/trusted-proxy-auth)).
- `gateway.auth.allowTailscale`: khi `true`, các tiêu đề danh tính của Tailscale Serve có thể đáp ứng xác thực Giao diện người dùng điều khiển/WebSocket (được xác minh qua `tailscale whois`); các điểm cuối API HTTP vẫn yêu cầu xác thực bằng mã thông báo/mật khẩu. Luồng không mã thông báo này giả định máy chủ gateway được tin cậy. Mặc định là `true` khi `tailscale.mode = "serve"`.
- `gateway.auth.rateLimit`: bộ giới hạn xác thực thất bại tùy chọn. Áp dụng cho mỗi IP máy khách và mỗi phạm vi xác thực (mã bí mật chia sẻ và mã thông báo thiết bị được theo dõi độc lập). Các nỗ lực bị chặn trả về `429 Too Many Requests` + `Retry-After`.
  - `gateway.auth.rateLimit.exemptLoopback` mặc định là `true`; đặt `false` khi bạn cố ý muốn lưu lượng truy cập localhost cũng bị giới hạn tốc độ (cho các thiết lập thử nghiệm hoặc triển khai proxy nghiêm ngặt).
- Các nỗ lực xác thực WS từ trình duyệt luôn bị điều tiết với miễn trừ local loopback bị vô hiệu hóa (phòng thủ chuyên sâu chống lại tấn công vét cạn localhost dựa trên trình duyệt).
- `tailscale.mode`: `serve` (chỉ tailnet, liên kết local loopback) hoặc `funnel` (công khai, yêu cầu xác thực).
- `controlUi.allowedOrigins`: danh sách cho phép nguồn gốc trình duyệt rõ ràng cho các kết nối WebSocket của Gateway. Bắt buộc khi các máy khách trình duyệt được mong đợi từ các nguồn gốc không phải local loopback.
- `controlUi.dangerouslyAllowHostHeaderOriginFallback`: chế độ nguy hiểm cho phép dự phòng nguồn gốc tiêu đề Host cho các triển khai cố ý dựa vào chính sách nguồn gốc tiêu đề Host.
- `remote.transport`: `ssh` (mặc định) hoặc `direct` (ws/wss). Đối với `direct`, `remote.url` phải là `ws://` hoặc `wss://`.
- `gateway.remote.token` / `.password` là các trường thông tin xác thực của máy khách từ xa. Chúng không tự cấu hình xác thực gateway.
- Các đường dẫn gọi gateway cục bộ có thể sử dụng `gateway.remote.*` làm dự phòng khi `gateway.auth.*` chưa được đặt.
- `trustedProxies`: các IP proxy ngược chấm dứt TLS. Chỉ liệt kê các proxy mà bạn kiểm soát.
- `allowRealIpFallback`: khi `true`, gateway chấp nhận `X-Real-IP` nếu `X-Forwarded-For` bị thiếu. Mặc định `false` cho hành vi đóng khi lỗi.
- `gateway.tools.deny`: tên công cụ bổ sung bị chặn cho HTTP `POST /tools/invoke` (mở rộng danh sách từ chối mặc định).
- `gateway.tools.allow`: xóa tên công cụ khỏi danh sách từ chối HTTP mặc định.

</Accordion>

### Các điểm cuối tương thích OpenAI

- Hoàn thành trò chuyện: bị vô hiệu hóa theo mặc định. Bật bằng `gateway.http.endpoints.chatCompletions.enabled: true`.
- API phản hồi: `gateway.http.endpoints.responses.enabled`.
- Tăng cường bảo mật đầu vào URL phản hồi:
  - __OC_I19N_0000__
  - __OC_I19N_0001__
  - __OC_I19N_0002__
- Tiêu đề tăng cường bảo mật phản hồi tùy chọn:
  - __OC_I19N_0003__ (chỉ đặt cho các nguồn gốc HTTPS bạn kiểm soát; xem [Xác thực Proxy đáng tin cậy](__OC_I19N_0010__))

### Cô lập đa phiên bản

Chạy nhiều Gateway trên một máy chủ với các cổng và thư mục trạng thái duy nhất:

``__OC_I19N_0004__`__OC_I19N_0005__--dev__OC_I19N_0006__~/.openclaw-dev__OC_I19N_0007__19001__OC_I19N_0008__--profile <name>__OC_I19N_0009__~/.openclaw-<name>`).

Xem [Nhiều Gateway](__OC_I19N_0011__).

---

## Hooks

``__OC_I19N_0000__``
Xác thực: `Authorization: Bearer <token>` hoặc `x-openclaw-token: <token>`.

**Điểm cuối:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` từ tải trọng yêu cầu chỉ được chấp nhận khi `hooks.allowRequestSessionKey=true` (mặc định: `false`).
- `POST /hooks/<name>` → được giải quyết thông qua `hooks.mappings`

<Accordion title="Chi tiết ánh xạ">

- `match.path` khớp với đường dẫn con sau `/hooks` (ví dụ: `/hooks/gmail` → `gmail`).
- `match.source` khớp với một trường tải trọng cho các đường dẫn chung.
- Các mẫu như `{{messages[0].subject}}` đọc từ tải trọng.
- `transform` có thể trỏ đến một mô-đun JS/TS trả về một hành động hook.
  - `transform.module` phải là một đường dẫn tương đối và nằm trong `hooks.transformsDir` (các đường dẫn tuyệt đối và duyệt thư mục bị từ chối).
- `agentId` định tuyến đến một agent cụ thể; ID không xác định sẽ quay về mặc định.
- `allowedAgentIds`: hạn chế định tuyến rõ ràng (`*` hoặc bỏ qua = cho phép tất cả, `[]` = từ chối tất cả).
- `defaultSessionKey`: khóa phiên cố định tùy chọn cho các lần chạy agent hook mà không có `sessionKey` rõ ràng.
- `allowRequestSessionKey`: cho phép người gọi `/hooks/agent` đặt `sessionKey` (mặc định: `false`).
- `allowedSessionKeyPrefixes`: danh sách cho phép tiền tố tùy chọn cho các giá trị `sessionKey` rõ ràng (yêu cầu + ánh xạ), ví dụ: `["hook:"]`.
- `deliver: true` gửi phản hồi cuối cùng đến một kênh; `channel` mặc định là `last`.
- `model` ghi đè LLM cho lần chạy hook này (phải được cho phép nếu danh mục mô hình được đặt).

</Accordion>

### Tích hợp Gmail

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```
- Gateway tự động khởi động `gog gmail watch serve` khi khởi động nếu được cấu hình. Đặt `OPENCLAW_SKIP_GMAIL_WATCHER=1` để tắt.
- Không chạy một `gog gmail watch serve` riêng biệt cùng với Gateway.

---

## Máy chủ Canvas

``__OC_I19N_0000__`__OC_I19N_0001__http://<gateway-host>:<gateway.port>/__openclaw__/canvas/__OC_I19N_0002__http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/__OC_I19N_0003__gateway.bind: "loopback"__OC_I19N_0004__index.html__OC_I19N_0005__/__openclaw__/a2ui/__OC_I19N_0006__EMFILE` lỗi.

---

## Khám phá thiết bị

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (default): omit `cliPath` + `sshPort` from TXT records.
- `full`: include `cliPath` + `sshPort`.
- Hostname defaults to `openclaw`. Override with `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Writes a unicast DNS-SD zone under `~/.openclaw/dns/`. Để khám phá xuyên mạng, hãy ghép nối với một máy chủ DNS (khuyến nghị CoreDNS) + Tailscale split DNS.
Thiết lập: `openclaw dns setup --apply`.

---

## Môi trường

### `env` (biến môi trường nội tuyến)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Inline env vars are only applied if the process env is missing the key.
- `.env` files: CWD `.env` + `~/.openclaw/.env` (neither overrides existing vars).
- `shellEnv`: imports missing expected keys from your login shell profile.
- See [Environment](/help/environment) for full precedence.

### Env var substitution

Reference env vars in any config string with `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```
- Chỉ các tên viết hoa được khớp: __OC_I19N_0000__.
- Các biến bị thiếu/rỗng sẽ gây ra lỗi khi tải cấu hình.
- Thoát bằng __OC_I19N_0001__ để có một __OC_I19N_0002__ theo nghĩa đen.
- Hoạt động với __OC_I19N_0003__.

---
## Bí mật

Tham chiếu bí mật có tính chất bổ sung: các giá trị văn bản thuần vẫn hoạt động.

### `SecretRef`

Sử dụng một cấu trúc đối tượng:

```json5
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

Validation:

- `provider` pattern: `^[a-z][a-z0-9_-]{0,63}$`
- `source: "env"` id pattern: `^[A-Z][A-Z0-9_]{0,127}$`
- `source: "file"` id: absolute JSON pointer (for example `"/providers/openai/apiKey"`)
- `source: "exec"` id pattern: `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

### Supported fields in config

- `models.providers.<provider>.apiKey`
- `skills.entries.<skillKey>.apiKey`
- `channels.googlechat.serviceAccount`
- `channels.googlechat.serviceAccountRef`
- `channels.googlechat.accounts.<accountId>.serviceAccount`
- `channels.googlechat.accounts.<accountId>.serviceAccountRef`

### Cấu hình nhà cung cấp bí mật

``json5
{
  secrets: {
    providers: {
      default: { source: "env" }, // optional explicit env provider
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json",
        timeoutMs: 5000,
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        passEnv: ["PATH", "VAULT_ADDR"],
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
  },
}
```

Notes:

- `file` provider supports `mode: "json"` and `mode: "singleValue"` (`id` must be `"value"` in singleValue mode).
- `exec` provider requires an absolute `command` đường dẫn và sử dụng các tải trọng giao thức trên stdin/stdout.
- Theo mặc định, các đường dẫn lệnh symlink bị từ chối. Đặt `allowSymlinkCommand: true` để cho phép các đường dẫn symlink trong khi xác thực đường dẫn đích đã được giải quyết.
- Nếu `trustedDirs` được cấu hình, kiểm tra thư mục đáng tin cậy sẽ áp dụng cho đường dẫn đích đã được giải quyết.
- Môi trường con của `exec` là tối thiểu theo mặc định; truyền các biến bắt buộc một cách rõ ràng với `passEnv`.
- Các tham chiếu bí mật được giải quyết tại thời điểm kích hoạt thành một ảnh chụp nhanh trong bộ nhớ, sau đó các đường dẫn yêu cầu chỉ đọc ảnh chụp nhanh đó.

---

## Lưu trữ xác thực

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- Per-agent auth profiles stored at `<agentDir>/auth-profiles.json`.
- Auth profiles support value-level refs (`keyRef` for `api_key`, `tokenRef` for `token`).
- Static runtime credentials come from in-memory resolved snapshots; legacy static `auth.json` entries are scrubbed when discovered.
- Legacy OAuth imports from `~/.openclaw/credentials/oauth.json`.
- See [OAuth](/concepts/oauth).
- Secrets runtime behavior and `audit/configure/apply` công cụ: [Quản lý bí mật](/gateway/secrets).

---

## Ghi nhật ký

Theo mặc định, OpenClaw ghi nhật ký vào ``json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
``/tmp/openclaw/openclaw-YYYY-MM-DD.log``

- Default log file: ` và vào bảng điều khiển. Bạn có thể thay đổi vị trí tệp nhật ký bằng cách sử dụng khóa cấu hình ``.
- Set `logging.file` for a stable path.
- ``, và mức độ ghi nhật ký bảng điều khiển bằng `` bumps to `consoleLevel` when `` hoặc cờ CLI `--verbose`.

---

## Trình hướng dẫn

Siêu dữ liệu được ghi bởi các trình hướng dẫn CLI (`onboard`, `configure`, `doctor`):

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## Danh tính

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

Written by the macOS onboarding assistant. Derives defaults:

- `messages.ackReaction` from `identity.emoji` (falls back to 👀)
- `mentionPatterns` from `identity.name`/`identity.emoji`
- `avatar` accepts: workspace-relative path, `http(s)__OC_I19N_0008__data:` URI

---

## Bridge (cũ, đã loại bỏ)

Các bản dựng hiện tại không còn bao gồm cầu nối TCP. Các node kết nối qua WebSocket của Gateway. Các khóa ``bridge.*`` không còn là một phần của lược đồ cấu hình (xác thực sẽ thất bại cho đến khi bị loại bỏ; ``openclaw doctor --fix`` có thể loại bỏ các khóa không xác định).

<Accordion title="Cấu hình cầu nối cũ (tham khảo lịch sử)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    webhook: "https://example.invalid/legacy", // deprecated fallback for stored notify:true jobs
    webhookToken: "replace-with-dedicated-token", // optional bearer token for outbound webhook auth
    sessionRetention: "24h", // duration string or false
    runLog: {
      maxBytes: "2mb", // default 2_000_000 bytes
      keepLines: 2000, // default 2000
    },
  },
}
```

- `sessionRetention`: how long to keep completed isolated cron run sessions before pruning from `sessions.json`. Also controls cleanup of archived deleted cron transcripts. Default: `24h`; set `false` to disable.
- `runLog.maxBytes`: max size per run log file (`cron/runs/<jobId>.jsonl`) before pruning. Default: `2_000_000` bytes.
- `runLog.keepLines`: newest lines retained when run-log pruning is triggered. Default: `2000`.
- `webhookToken`: bearer token used for cron webhook POST delivery (`delivery.mode = "webhook"`), if omitted no auth header is sent.
- `webhook`: deprecated legacy fallback webhook URL (http/https) used only for stored jobs that still have `notify: true`.

Xem [Cron Jobs](/automation/cron-jobs).

---

## Các biến mẫu mô hình phương tiện

Các phần giữ chỗ mẫu được mở rộng trong `tools.media.models[].args`:

| Biến           | Mô tả                                       |
| ------------------ | ------------------------------------------------- |
| `{{Body}}`         | Toàn bộ nội dung tin nhắn đến                         |
| `{{RawBody}}`      | Nội dung thô (không có lịch sử/trình bao người gửi)             |
| `{{BodyStripped}}` | Nội dung đã loại bỏ các đề cập nhóm                 |
| `{{From}}`         | Định danh người gửi                                 |
| `{{To}}`           | Định danh đích                            |
| `{{MessageSid}}`   | ID tin nhắn kênh                                |
| `{{SessionId}}`    | UUID phiên hiện tại                              |
| `{{IsNewSession}}` | `"true"` khi phiên mới được tạo                 |
| `{{MediaUrl}}`     | URL giả của phương tiện đến                          |
| `{{MediaPath}}`    | Đường dẫn phương tiện cục bộ                                  |
| `{{MediaType}}`    | Loại phương tiện (hình ảnh/âm thanh/tài liệu/…)               |
| `{{Transcript}}`   | Bản ghi âm                                  |
| `{{Prompt}}`       | Lời nhắc phương tiện đã giải quyết cho các mục nhập CLI             |
| `{{MaxChars}}`     | Số ký tự đầu ra tối đa đã giải quyết cho các mục nhập CLI         |
| `{{ChatType}}`     | `"direct"` hoặc `"group"`                           |
| `{{GroupSubject}}` | Chủ đề nhóm (nỗ lực tốt nhất)                       |
| `{{GroupMembers}}` | Xem trước thành viên nhóm (nỗ lực tốt nhất)               |
| `{{SenderName}}`   | Tên hiển thị người gửi (nỗ lực tốt nhất)                 |
| `{{SenderE164}}`   | Số điện thoại người gửi (nỗ lực tốt nhất)                 |
| `{{Provider}}`     | Gợi ý nhà cung cấp (whatsapp, telegram, discord, v.v.) |

---

## Cấu hình bao gồm (__OC_I19N_0000__)

Chia cấu hình thành nhiều tệp:

``__OC_I19N_0001__`__OC_I19N_0002__dirname__OC_I19N_0003__../` chỉ được phép khi chúng vẫn phân giải bên trong ranh giới đó.
- Lỗi: thông báo rõ ràng cho các tệp bị thiếu, lỗi phân tích cú pháp và các bao gồm tuần hoàn.

---

_Liên quan: [Cấu hình](__OC_I19N_0004__) · [Ví dụ cấu hình](__OC_I19N_0005__) · [Doctor](__OC_I19N_0006__)_