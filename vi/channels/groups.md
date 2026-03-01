---
summary: >-
  Hành vi chat nhóm trên các nền tảng
  (WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Microsoft Teams/Zalo)
read_when:
  - Thay đổi hành vi chat nhóm hoặc cổng kiểm soát mention
title: Nhóm
x-i18n:
  source_path: channels\groups.md
  source_hash: 3464d9d4f283ea437a6e4ac3984df163f44715c51469e449241bf7a4d9ba6332
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:18:42.272Z'
---

# Nhóm

OpenClaw xử lý các cuộc trò chuyện nhóm một cách nhất quán trên tất cả các nền tảng: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams, Zalo.
## Giới thiệu cho người mới bắt đầu (2 phút)

OpenClaw "hoạt động" trên các tài khoản nhắn tin của chính bạn. Không có bot WhatsApp riêng biệt.
Nếu **bạn** có trong một nhóm, OpenClaw có thể thấy nhóm đó và phản hồi ở đó.

Hành vi mặc định:

- Các nhóm bị hạn chế (`groupPolicy: "allowlist"`).
- Phản hồi yêu cầu được nhắc đến trừ khi bạn tắt rõ ràng tính năng kiểm soát nhắc đến.

Dịch: những người gửi được cho phép có thể kích hoạt OpenClaw bằng cách nhắc đến nó.

> TL;DR
>
> - **Quyền truy cập tin nhắn riêng** được kiểm soát bởi `*.allowFrom`.
> - **Quyền truy cập nhóm** được kiểm soát bởi `*.groupPolicy` + danh sách cho phép (`*.groups`, `*.groupAllowFrom`).
> - **Kích hoạt phản hồi** được kiểm soát bởi kiểm soát nhắc đến (`requireMention`, `/activation`).

Luồng nhanh (điều gì xảy ra với tin nhắn nhóm):

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Group message flow](/images/groups-flow.svg)

If you want...

| Goal                                         | What to set                                                |
| -------------------------------------------- | ---------------------------------------------------------- |
| Allow all groups but only reply on @mentions | `groups: { "*": { requireMention: true } }`                |
| Disable all group replies                    | `groupPolicy: "disabled"`                                  |
| Only specific groups                         | `groups: { "<group-id>": { ... } }` (no `"*"` key)         |
| Only you can trigger in groups               | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |
## Khóa phiên

- Phiên nhóm sử dụng khóa phiên `agent:<agentId>:<channel>:group:<id>` (phòng/kênh sử dụng `agent:<agentId>:<channel>:channel:<id>`).
- Chủ đề diễn đàn Telegram thêm `:topic:<threadId>` vào id nhóm để mỗi chủ đề có phiên riêng.
- Trò chuyện trực tiếp sử dụng phiên chính (hoặc theo người gửi nếu được cấu hình).
- Heartbeat được bỏ qua cho phiên nhóm.
## Mẫu: tin nhắn riêng cá nhân + nhóm công khai (agent đơn)

Có — điều này hoạt động tốt nếu lưu lượng "cá nhân" của bạn là **tin nhắn riêng** và lưu lượng "công khai" của bạn là **nhóm**.

Tại sao: ở chế độ agent đơn, tin nhắn riêng thường được xử lý trong khóa phiên **chính** (`agent:main:main`), trong khi nhóm luôn sử dụng khóa phiên **không phải chính** (`agent:main:<channel>:group:<id>`). Nếu bạn bật sandboxing với `mode: "non-main"`, các phiên nhóm đó sẽ chạy trong Docker trong khi phiên tin nhắn riêng chính của bạn vẫn ở trên máy chủ.

Điều này cung cấp cho bạn một "bộ não" agent (không gian làm việc + bộ nhớ được chia sẻ), nhưng hai tư thế thực thi:

- **Tin nhắn riêng**: đầy đủ công cụ (máy chủ)
- **Nhóm**: sandbox + công cụ bị hạn chế (Docker)

> Nếu bạn cần không gian làm việc/nhân cách thực sự riêng biệt ("cá nhân" và "công khai" không bao giờ được trộn lẫn), hãy sử dụng agent thứ hai + bindings. Xem [Định tuyến đa Agent](/concepts/multi-agent).

Ví dụ (tin nhắn riêng trên máy chủ, nhóm được sandbox + chỉ có công cụ nhắn tin):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // groups/channels are non-main -> sandboxed
        scope: "session", // strongest isolation (one container per group/channel)
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // If allow is non-empty, everything else is blocked (deny still wins).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

Want “groups can only see folder X” instead of “no host access”? Keep `workspaceAccess: "none"` and mount only allowlisted paths into the sandbox:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```
Liên quan:

- Các khóa cấu hình và giá trị mặc định: [Cấu hình Gateway](/gateway/configuration#agentsdefaultssandbox)
- Gỡ lỗi tại sao một công cụ bị chặn: [Sandbox vs Chính sách Công cụ vs Nâng cao](/gateway/sandbox-vs-tool-policy-vs-elevated)
- Chi tiết bind mounts: [Sandboxing](/gateway/sandboxing#custom-bind-mounts)
## Nhãn hiển thị

- Nhãn giao diện sử dụng `displayName` khi có sẵn, được định dạng là `<channel>:<token>`.
- `#room` được dành riêng cho phòng/kênh; trò chuyện nhóm sử dụng `g-<slug>` (chữ thường, dấu cách -> `-`, giữ `#@+._-`).
## Chính sách nhóm

Kiểm soát cách xử lý tin nhắn nhóm/phòng cho mỗi kênh:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // numeric Telegram user id (wizard can resolve @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| Policy        | Behavior                                                     |
| ------------- | ------------------------------------------------------------ |
| `"open"`      | Groups bypass allowlists; mention-gating still applies.      |
| `"disabled"`  | Block all group messages entirely.                           |
| `"allowlist"` | Only allow groups/rooms that match the configured allowlist. |

Notes:

- `groupPolicy` is separate from mention-gating (which requires @mentions).
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo: use `groupAllowFrom` (fallback: explicit `allowFrom`).
- DM pairing approvals (`*-allowFrom` chỉ áp dụng cho quyền truy cập tin nhắn riêng; việc ủy quyền người gửi nhóm vẫn được áp dụng rõ ràng cho danh sách cho phép của nhóm.
- Discord: allowlist sử dụng `channels.discord.guilds.<id>.channels`.
- Slack: allowlist sử dụng `channels.slack.channels`.
- Matrix: allowlist sử dụng `channels.matrix.groups` (ID phòng, bí danh, hoặc tên). Sử dụng `channels.matrix.groupAllowFrom` để hạn chế người gửi; allowlist `users` theo từng phòng cũng được hỗ trợ.
- Tin nhắn riêng nhóm được kiểm soát riêng biệt (`channels.discord.dm.*`, `channels.slack.dm.*`).
- Allowlist Telegram có thể khớp với ID người dùng (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) hoặc tên người dùng (`"@alice"` hoặc `"alice"`); tiền tố không phân biệt chữ hoa chữ thường.
- Mặc định là `groupPolicy: "allowlist"`; nếu allowlist nhóm của bạn trống, tin nhắn nhóm sẽ bị chặn.
- Tính an toàn thời gian chạy: khi một khối nhà cung cấp hoàn toàn bị thiếu (`channels.<provider>` không có), chính sách nhóm sẽ quay về chế độ fail-closed (thường là `allowlist`) thay vì kế thừa `channels.defaults.groupPolicy`.

Mô hình tư duy nhanh (thứ tự đánh giá cho tin nhắn nhóm):

1. `groupPolicy` (mở/tắt/allowlist)
2. allowlist nhóm (`*.groups`, `*.groupAllowFrom`, allowlist theo kênh cụ thể)
3. kiểm soát mention (`requireMention`, `/activation`)
## Cổng đề cập (mặc định)

Tin nhắn nhóm yêu cầu đề cập trừ khi được ghi đè cho từng nhóm. Các giá trị mặc định nằm trong từng hệ thống con dưới `*.groups."*"`.

Trả lời tin nhắn bot được tính là đề cập ngầm định (khi kênh hỗ trợ metadata trả lời). Điều này áp dụng cho Telegram, WhatsApp, Slack, Discord và Microsoft Teams.

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

Notes:

- `mentionPatterns` are case-insensitive regexes.
- Surfaces that provide explicit mentions still pass; patterns are a fallback.
- Per-agent override: `agents.list[].groupChat.mentionPatterns` (useful when multiple agents share a group).
- Mention gating is only enforced when mention detection is possible (native mentions or `mentionPatterns` are configured).
- Discord defaults live in `channels.discord.guilds."*"` (overridable per guild/channel).
- Group history context is wrapped uniformly across channels and is **pending-only** (messages skipped due to mention gating); use `messages.groupChat.historyLimit` for the global default and `channels.<channel>.historyLimit` (or `channels.<channel>.accounts.*.historyLimit`) for overrides. Set `0` để vô hiệu hóa.
## Hạn chế công cụ theo nhóm/kênh (tùy chọn)

Một số cấu hình kênh hỗ trợ hạn chế các công cụ có sẵn **bên trong một nhóm/phòng/kênh cụ thể**.

- `tools`: cho phép/từ chối công cụ cho toàn bộ nhóm.
- `toolsBySender`: ghi đè theo người gửi trong nhóm.
  Sử dụng tiền tố khóa rõ ràng:
  `id:<senderId>`, `e164:<phone>`, `username:<handle>`, `name:<displayName>`, và ký tự đại diện `"*"`.
  Các khóa không có tiền tố cũ vẫn được chấp nhận và khớp chỉ với `id:`.

Thứ tự phân giải (cụ thể nhất thắng):

1. khớp `toolsBySender` nhóm/kênh
2. `tools` nhóm/kênh
3. khớp `toolsBySender` mặc định (`"*"`)
4. `tools` mặc định (`"*"`)

Ví dụ (Telegram):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

Notes:

- Group/channel tool restrictions are applied in addition to global/agent tool policy (deny still wins).
- Some channels use different nesting for rooms/channels (e.g., Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).
## Danh sách cho phép nhóm

Khi `channels.whatsapp.groups`, `channels.telegram.groups`, hoặc `channels.imessage.groups` được cấu hình, các khóa hoạt động như một danh sách cho phép nhóm. Sử dụng `"*"` để cho phép tất cả các nhóm trong khi vẫn thiết lập hành vi đề cập mặc định.

Các mục đích phổ biến (sao chép/dán):

1. Vô hiệu hóa tất cả phản hồi nhóm

```json5
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2. Allow only specific groups (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3. Allow all groups but require mention (explicit)

```json5
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4. Only the owner can trigger in groups (WhatsApp)

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```
## Kích hoạt (chỉ chủ sở hữu)

Chủ sở hữu nhóm có thể bật/tắt kích hoạt theo từng nhóm:

- `/activation mention`
- `/activation always`

Chủ sở hữu được xác định bởi `channels.whatsapp.allowFrom` (hoặc E.164 của chính bot khi không được đặt). Gửi lệnh dưới dạng tin nhắn độc lập. Các giao diện khác hiện tại bỏ qua `/activation`.
## Trường ngữ cảnh

Tải trọng đến từ nhóm thiết lập:

- `ChatType=group`
- `GroupSubject` (nếu biết)
- `GroupMembers` (nếu biết)
- `WasMentioned` (đề cập kết quả kiểm soát)
- Chủ đề diễn đàn Telegram cũng bao gồm `MessageThreadId` và `IsForum`.

Lời nhắc hệ thống agent bao gồm phần giới thiệu nhóm ở lượt đầu tiên của phiên nhóm mới. Nó nhắc nhở mô hình phản hồi như con người, tránh bảng Markdown và tránh gõ chuỗi `\n` theo nghĩa đen.
## Đặc điểm riêng của iMessage

- Ưu tiên `chat_id:<id>` khi định tuyến hoặc tạo danh sách cho phép.
- Liệt kê cuộc trò chuyện: `imsg chats --limit 20`.
- Phản hồi nhóm luôn quay lại cùng một `chat_id`.
## Đặc điểm riêng của WhatsApp

Xem [Tin nhắn nhóm](/channels/group-messages) để biết hành vi chỉ dành cho WhatsApp (tiêm lịch sử, chi tiết xử lý đề cập).