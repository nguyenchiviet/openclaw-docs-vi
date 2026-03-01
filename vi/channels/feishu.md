---
summary: 'Tổng quan, tính năng và cấu hình bot Feishu'
read_when:
  - Bạn muốn kết nối một bot Feishu/Lark
  - Bạn đang cấu hình kênh Feishu
title: Feishu
x-i18n:
  source_path: channels\feishu.md
  source_hash: b7b5e52893d3b92a67292a828d547fe2d90ee39c17c112b3df9b474f6d7b7545
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:15:41.302Z'
---

# Bot Feishu

Feishu (Lark) là một nền tảng trò chuyện nhóm được các công ty sử dụng để nhắn tin và cộng tác. Plugin này kết nối OpenClaw với bot Feishu/Lark bằng cách sử dụng đăng ký sự kiện WebSocket của nền tảng để có thể nhận tin nhắn mà không cần phải công khai URL webhook.

---
## Plugin cần thiết

Cài đặt plugin Feishu:

```bash
openclaw plugins install @openclaw/feishu
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/feishu
```

---
## Bắt đầu nhanh

Có hai cách để thêm kênh Feishu:

### Phương pháp 1: trình hướng dẫn thiết lập ban đầu (khuyến nghị)

Nếu bạn vừa cài đặt OpenClaw, hãy chạy trình hướng dẫn:

```bash
openclaw onboard
```

The wizard guides you through:

1. Creating a Feishu app and collecting credentials
2. Configuring app credentials in OpenClaw
3. Starting the gateway

✅ **After configuration**, check gateway status:

- `openclaw gateway status`
- `openclaw logs --follow`

### Method 2: CLI setup

If you already completed initial install, add the channel via CLI:

```bash
openclaw channels add
```

Choose **Feishu**, then enter the App ID and App Secret.

✅ **After configuration**, manage the gateway:

- `openclaw gateway status`
- `openclaw gateway restart`
- `openclaw logs --follow`

---
## Bước 1: Tạo ứng dụng Feishu

### 1. Mở Feishu Open Platform

Truy cập [Feishu Open Platform](https://open.feishu.cn/app) và đăng nhập.

Các tenant Lark (toàn cầu) nên sử dụng [https://open.larksuite.com/app](https://open.larksuite.com/app) và đặt `domain: "lark"` trong cấu hình Feishu.

### 2. Tạo ứng dụng

1. Nhấp **Create enterprise app**
2. Điền tên ứng dụng + mô tả
3. Chọn biểu tượng ứng dụng

![Create enterprise app](../images/feishu-step2-create-app.png)

### 3. Sao chép thông tin xác thực

Từ **Credentials & Basic Info**, sao chép:

- **App ID** (định dạng: `cli_xxx`)
- **App Secret**

❗ **Quan trọng:** giữ bí mật App Secret.

![Get credentials](../images/feishu-step3-credentials.png)

### 4. Cấu hình quyền

Trên **Permissions**, nhấp **Batch import** và dán:

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![Configure permissions](../images/feishu-step4-permissions.png)

### 5. Kích hoạt khả năng bot
Trong **App Capability** > **Bot**:

1. Bật khả năng bot
2. Đặt tên bot

![Bật khả năng bot](../images/feishu-step5-bot-capability.png)

### 6. Cấu hình đăng ký sự kiện

⚠️ **Quan trọng:** trước khi thiết lập đăng ký sự kiện, hãy đảm bảo:

1. Bạn đã chạy `openclaw channels add` cho Feishu
2. Gateway đang chạy (`openclaw gateway status`)

Trong **Event Subscription**:

1. Chọn **Use long connection to receive events** (WebSocket)
2. Thêm sự kiện: `im.message.receive_v1`

⚠️ Nếu Gateway không chạy, việc thiết lập kết nối dài có thể không lưu được.

![Cấu hình đăng ký sự kiện](../images/feishu-step6-event-subscription.png)

### 7. Xuất bản ứng dụng

1. Tạo phiên bản trong **Version Management & Release**
2. Gửi để xem xét và xuất bản
3. Chờ phê duyệt từ quản trị viên (ứng dụng doanh nghiệp thường tự động phê duyệt)

---
## Bước 2: Cấu hình OpenClaw

### Cấu hình với trình hướng dẫn (khuyến nghị)

```bash
openclaw channels add
```

Choose **Feishu** and paste your App ID + App Secret.

### Configure via config file

Edit `~/.openclaw/openclaw.json`:

```json5
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

If you use `connectionMode: "webhook"`, set `verificationToken`. The Feishu webhook server binds to `127.0.0.1` by default; set `webhookHost` only if you intentionally need a different bind address.

### Configure via environment variables

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Lark (global) domain

If your tenant is on Lark (international), set the domain to `lark` (or a full domain string). You can set it at `channels.feishu.domain` or per account (`channels.feishu.accounts.<id>.domain`).

```json5
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```
---

## Bước 3: Khởi động + kiểm tra

### 1. Khởi động gateway

```bash
openclaw gateway
```

### 2. Send a test message

In Feishu, find your bot and send a message.

### 3. Approve pairing

By default, the bot replies with a pairing code. Approve it:

```bash
openclaw pairing approve feishu <CODE>
```

Sau khi được phê duyệt, bạn có thể trò chuyện bình thường.

---
## Tổng quan

- **Kênh bot Feishu**: Bot Feishu được quản lý bởi gateway
- **Định tuyến xác định**: phản hồi luôn trở về Feishu
- **Cách ly phiên**: Tin nhắn riêng chia sẻ phiên chính; nhóm được cách ly
- **Kết nối WebSocket**: kết nối dài thông qua Feishu SDK, không cần URL công khai

---
## Kiểm soát truy cập

### Tin nhắn riêng

- **Mặc định**: `dmPolicy: "pairing"` (người dùng không xác định sẽ nhận mã ghép nối)
- **Phê duyệt ghép nối**:

  ```bash
  openclaw pairing list feishu
  openclaw pairing approve feishu <CODE>
  ```

- **Allowlist mode**: set `channels.feishu.allowFrom` with allowed Open IDs

### Group chats

**1. Group policy** (`channels.feishu.groupPolicy`):

- `"open"` = allow everyone in groups (default)
- `"allowlist"` = only allow `groupAllowFrom`
- `"disabled"` = disable group messages

**2. Mention requirement** (`channels.feishu.groups.<chat_id>.requireMention`):

- `true` = require @mention (default)
- `false` = phản hồi mà không cần nhắc đến

---
## Ví dụ cấu hình nhóm

### Cho phép tất cả nhóm, yêu cầu @mention (mặc định)

```json5
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // Default requireMention: true
    },
  },
}
```

### Allow all groups, no @mention required

```json5
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### Allow specific users in groups only

```json5
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["ou_xxx", "ou_yyy"],
    },
  },
}
```

---
## Lấy ID nhóm/người dùng

### ID nhóm (chat_id)

ID nhóm trông như `oc_xxx`.

**Phương pháp 1 (khuyến nghị)**

1. Khởi động gateway và @mention bot trong nhóm
2. Chạy `openclaw logs --follow` và tìm `chat_id`

**Phương pháp 2**

Sử dụng trình gỡ lỗi API Feishu để liệt kê các cuộc trò chuyện nhóm.

### ID người dùng (open_id)

ID người dùng trông như `ou_xxx`.

**Phương pháp 1 (khuyến nghị)**

1. Khởi động gateway và gửi tin nhắn riêng cho bot
2. Chạy `openclaw logs --follow` và tìm `open_id`

**Phương pháp 2**

Kiểm tra các yêu cầu ghép nối để lấy Open ID của người dùng:

```bash
openclaw pairing list feishu
```

---
## Lệnh thường dùng

| Lệnh   | Mô tả       |
| --------- | ----------------- |
| `/status` | Hiển thị trạng thái bot   |
| `/reset`  | Đặt lại phiên |
| `/model`  | Hiển thị/chuyển đổi mô hình |

> Lưu ý: Feishu chưa hỗ trợ menu lệnh gốc, vì vậy các lệnh phải được gửi dưới dạng văn bản.
## Lệnh quản lý Gateway

| Lệnh                    | Mô tả                   |
| -------------------------- | ----------------------------- |
| `openclaw gateway status`  | Hiển thị trạng thái gateway           |
| `openclaw gateway install` | Cài đặt/khởi động dịch vụ gateway |
| `openclaw gateway stop`    | Dừng dịch vụ gateway          |
| `openclaw gateway restart` | Khởi động lại dịch vụ gateway       |
| `openclaw logs --follow`   | Theo dõi nhật ký gateway             |

---
## Khắc phục sự cố

### Bot không phản hồi trong cuộc trò chuyện nhóm

1. Đảm bảo bot đã được thêm vào nhóm
2. Đảm bảo bạn @mention bot (hành vi mặc định)
3. Kiểm tra `groupPolicy` không được đặt thành `"disabled"`
4. Kiểm tra logs: `openclaw logs --follow`

### Bot không nhận được tin nhắn

1. Đảm bảo ứng dụng đã được xuất bản và phê duyệt
2. Đảm bảo đăng ký sự kiện bao gồm `im.message.receive_v1`
3. Đảm bảo **kết nối dài** được bật
4. Đảm bảo quyền ứng dụng đã đầy đủ
5. Đảm bảo Gateway đang chạy: `openclaw gateway status`
6. Kiểm tra logs: `openclaw logs --follow`

### Rò rỉ App Secret

1. Đặt lại App Secret trong Feishu Open Platform
2. Cập nhật App Secret trong cấu hình của bạn
3. Khởi động lại Gateway

### Lỗi gửi tin nhắn

1. Đảm bảo ứng dụng có quyền `im:message:send_as_bot`
2. Đảm bảo ứng dụng đã được xuất bản
3. Kiểm tra logs để biết lỗi chi tiết

---
## Cấu hình nâng cao

### Nhiều tài khoản

```json5
{
  channels: {
    feishu: {
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

### Message limits

- `textChunkLimit`: outbound text chunk size (default: 2000 chars)
- `mediaMaxMb`: media upload/download limit (default: 30MB)

### Streaming

Feishu supports streaming replies via interactive cards. When enabled, the bot updates a card as it generates text.

```json5
{
  channels: {
    feishu: {
      streaming: true, // enable streaming card output (default true)
      blockStreaming: true, // enable block-level streaming (default true)
    },
  },
}
```

Set `streaming: false` to wait for the full reply before sending.

### Multi-agent routing

Use `bindings` to route Feishu DMs or groups to different agents.

```json5
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```
Các trường định tuyến:

- `match.channel`: `"feishu"`
- `match.peer.kind`: `"direct"` hoặc `"group"`
- `match.peer.id`: Open ID của người dùng (`ou_xxx`) hoặc ID nhóm (`oc_xxx`)

Xem [Lấy ID nhóm/người dùng](#get-groupuser-ids) để biết mẹo tra cứu.

---
## Tham khảo cấu hình

Cấu hình đầy đủ: [Cấu hình Gateway](/gateway/configuration)

Các tùy chọn chính:

| Cài đặt                                           | Mô tả                           | Mặc định         |
| ------------------------------------------------- | ------------------------------- | ---------------- |
| `channels.feishu.enabled`                         | Bật/tắt kênh                    | `true`           |
| `channels.feishu.domain`                          | Tên miền API (`feishu` hoặc `lark`) | `feishu`         |
| `channels.feishu.connectionMode`                  | Chế độ truyền tải sự kiện       | `websocket`      |
| `channels.feishu.verificationToken`               | Bắt buộc cho chế độ webhook     | -                |
| `channels.feishu.webhookPath`                     | Đường dẫn webhook               | `/feishu/events` |
| `channels.feishu.webhookHost`                     | Host liên kết webhook           | `127.0.0.1`      |
| `channels.feishu.webhookPort`                     | Cổng liên kết webhook           | `3000`           |
| `channels.feishu.accounts.<id>.appId`             | ID ứng dụng                     | -                |
| `channels.feishu.accounts.<id>.appSecret`         | Khóa bí mật ứng dụng            | -                |
| `channels.feishu.accounts.<id>.domain`            | Ghi đè tên miền API theo tài khoản | `feishu`         |
| `channels.feishu.dmPolicy`                        | Chính sách tin nhắn riêng       | `pairing`        |
| `channels.feishu.allowFrom`                       | Danh sách cho phép tin nhắn riêng (danh sách open_id) | -                |
| `channels.feishu.groupPolicy`                     | Chính sách nhóm                 | `open`           |
| `channels.feishu.groupAllowFrom`                  | Danh sách cho phép nhóm         | -                |
| `channels.feishu.groups.<chat_id>.requireMention` | Yêu cầu @mention                | `true`           |
| `channels.feishu.groups.<chat_id>.enabled`        | Bật nhóm                        | `true`           |
| `channels.feishu.textChunkLimit`                  | Kích thước khối tin nhắn        | `2000`           |
| `channels.feishu.mediaMaxMb`                      | Giới hạn kích thước media       | `30`             |
| `channels.feishu.streaming`                       | Bật đầu ra thẻ streaming        | `true`           |
| `channels.feishu.blockStreaming`                  | Bật truyền phát theo khối       | `true`           |

---
## Tham chiếu dmPolicy

| Giá trị       | Hành vi                                                         |
| ------------- | --------------------------------------------------------------- |
| `"pairing"`   | **Mặc định.** Người dùng không xác định nhận mã ghép nối; phải được phê duyệt |
| `"allowlist"` | Chỉ người dùng trong `allowFrom` mới có thể trò chuyện                              |
| `"open"`      | Cho phép tất cả người dùng (yêu cầu `"*"` trong allowFrom)                   |
| `"disabled"`  | Tắt tin nhắn riêng                                                     |

---
## Các loại tin nhắn được hỗ trợ

### Nhận

- ✅ Văn bản
- ✅ Văn bản phong phú (bài đăng)
- ✅ Hình ảnh
- ✅ Tệp tin
- ✅ Âm thanh
- ✅ Video
- ✅ Sticker

### Gửi

- ✅ Văn bản
- ✅ Hình ảnh
- ✅ Tệp tin
- ✅ Âm thanh
- ⚠️ Văn bản phong phú (hỗ trợ một phần)