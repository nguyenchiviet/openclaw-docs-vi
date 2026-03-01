---
summary: Cấu hình và thiết lập bot chat Twitch
read_when:
  - Thiết lập tích hợp chat Twitch cho OpenClaw
title: Twitch
x-i18n:
  source_path: channels\twitch.md
  source_hash: 4fa7daa11d1e5ed43c9a8f9f7092809bf2c643838fc5b0c8df27449e430796dc
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:29:00.438Z'
---

# Twitch (plugin)

Hỗ trợ chat Twitch thông qua kết nối IRC. OpenClaw kết nối như một người dùng Twitch (tài khoản bot) để nhận và gửi tin nhắn trong các kênh.
## Plugin cần thiết

Twitch được cung cấp dưới dạng plugin và không được tích hợp sẵn với bản cài đặt cốt lõi.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/twitch
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/twitch
```

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập nhanh (người mới bắt đầu)

1. Tạo một tài khoản Twitch riêng cho bot (hoặc sử dụng tài khoản hiện có).
2. Tạo thông tin xác thực: [Trình tạo Token Twitch](https://twitchtokengenerator.com/)
   - Chọn **Bot Token**
   - Xác minh các phạm vi `chat:read` và `chat:write` đã được chọn
   - Sao chép **Client ID** và **Access Token**
3. Tìm ID người dùng Twitch của bạn: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4. Cấu hình token:
   - Biến môi trường: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (chỉ tài khoản mặc định)
   - Hoặc cấu hình: `channels.twitch.accessToken`
   - Nếu cả hai đều được thiết lập, cấu hình sẽ được ưu tiên (biến môi trường dự phòng chỉ dành cho tài khoản mặc định).
5. Khởi động Gateway.

**⚠️ Quan trọng:** Thêm kiểm soát truy cập (`allowFrom` hoặc `allowedRoles`) để ngăn người dùng không được phép kích hoạt bot. `requireMention` mặc định là `true`.

Cấu hình tối thiểu:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // Bot's Twitch account
      accessToken: "oauth:abc123...", // OAuth Access Token (or use OPENCLAW_TWITCH_ACCESS_TOKEN env var)
      clientId: "xyz789...", // Client ID from Token Generator
      channel: "vevisk", // Which Twitch channel's chat to join (required)
      allowFrom: ["123456789"], // (recommended) Your Twitch user ID only - get it from https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```
## Nó là gì

- Một kênh Twitch thuộc sở hữu của Gateway.
- Định tuyến xác định: phản hồi luôn quay trở lại Twitch.
- Mỗi tài khoản ánh xạ tới một khóa phiên cô lập `agent:<agentId>:twitch:<accountName>`.
- `username` là tài khoản của bot (người xác thực), `channel` là phòng chat nào để tham gia.
## Thiết lập (chi tiết)

### Tạo thông tin xác thực

Sử dụng [Twitch Token Generator](https://twitchtokengenerator.com/):

- Chọn **Bot Token**
- Xác minh các phạm vi `chat:read` và `chat:write` đã được chọn
- Sao chép **Client ID** và **Access Token**

Không cần đăng ký ứng dụng thủ công. Token sẽ hết hạn sau vài giờ.

### Cấu hình bot

**Biến môi trường (chỉ tài khoản mặc định):**

```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Or config:**

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

If both env and config are set, config takes precedence.

### Access control (recommended)

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (recommended) Your Twitch user ID only
    },
  },
}
```

Prefer `allowFrom` for a hard allowlist. Use `allowedRoles` instead if you want role-based access.

**Available roles:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`.

**Tại sao lại dùng ID người dùng?** Tên người dùng có thể thay đổi, cho phép mạo danh. ID người dùng là vĩnh viễn.

Tìm ID người dùng Twitch của bạn: [https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/) (Chuyển đổi tên người dùng Twitch thành ID)
## Làm mới token (tùy chọn)

Token từ [Twitch Token Generator](https://twitchtokengenerator.com/) không thể được làm mới tự động - hãy tạo lại khi hết hạn.

Để làm mới token tự động, tạo ứng dụng Twitch của riêng bạn tại [Twitch Developer Console](https://dev.twitch.tv/console) và thêm vào cấu hình:

```json5
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

Bot sẽ tự động làm mới token trước khi hết hạn và ghi lại các sự kiện làm mới.
## Hỗ trợ nhiều tài khoản

Sử dụng `channels.twitch.accounts` với token riêng cho từng tài khoản. Xem [`gateway/configuration`](/gateway/configuration) để biết mẫu chung.

Ví dụ (một tài khoản bot trong hai kênh):

```json5
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**Lưu ý:** Mỗi tài khoản cần token riêng (một token cho mỗi kênh).
## Kiểm soát truy cập

### Hạn chế dựa trên vai trò

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### Allowlist by User ID (most secure)

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### Role-based access (alternative)

`allowFrom` is a hard allowlist. When set, only those user IDs are allowed.
If you want role-based access, leave `allowFrom` unset and configure `allowedRoles` instead:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### Disable @mention requirement

By default, `requireMention` is `true`. To disable and respond to all messages:

```json5
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```
## Khắc phục sự cố

Đầu tiên, chạy các lệnh chẩn đoán:

```bash
openclaw doctor
openclaw channels status --probe
```

### Bot doesn't respond to messages

**Check access control:** Ensure your user ID is in `allowFrom`, or temporarily remove
`allowFrom` and set `allowedRoles: ["all"]` to test.

**Check the bot is in the channel:** The bot must join the channel specified in `channel`.

### Token issues

**"Failed to connect" or authentication errors:**

- Verify `accessToken` is the OAuth access token value (typically starts with `oauth:` prefix)
- Check token has `chat:read` and `chat:write` scopes
- If using token refresh, verify `clientSecret` and `refreshToken` are set

### Token refresh not working

**Check logs for refresh events:**

```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

If you see "token refresh disabled (no refresh token)":

- Ensure `clientSecret` is provided
- Ensure `refreshToken` được cung cấp
## Cấu hình

**Cấu hình tài khoản:**

- `username` - Tên người dùng bot
- `accessToken` - Token truy cập OAuth với `chat:read` và `chat:write`
- `clientId` - Twitch Client ID (từ Token Generator hoặc ứng dụng của bạn)
- `channel` - Kênh để tham gia (bắt buộc)
- `enabled` - Kích hoạt tài khoản này (mặc định: `true`)
- `clientSecret` - Tùy chọn: Để làm mới token tự động
- `refreshToken` - Tùy chọn: Để làm mới token tự động
- `expiresIn` - Thời gian hết hạn token tính bằng giây
- `obtainmentTimestamp` - Thời gian nhận token
- `allowFrom` - Danh sách cho phép ID người dùng
- `allowedRoles` - Kiểm soát truy cập dựa trên vai trò (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
- `requireMention` - Yêu cầu @mention (mặc định: `true`)

**Tùy chọn nhà cung cấp:**

- `channels.twitch.enabled` - Bật/tắt khởi động kênh
- `channels.twitch.username` - Tên người dùng bot (cấu hình đơn giản cho một tài khoản)
- `channels.twitch.accessToken` - Token truy cập OAuth (cấu hình đơn giản cho một tài khoản)
- `channels.twitch.clientId` - Twitch Client ID (cấu hình đơn giản cho một tài khoản)
- `channels.twitch.channel` - Kênh để tham gia (cấu hình đơn giản cho một tài khoản)
- `channels.twitch.accounts.<accountName>` - Cấu hình nhiều tài khoản (tất cả các trường tài khoản ở trên)

Ví dụ đầy đủ:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```
## Hành động công cụ

Agent có thể gọi `twitch` với hành động:

- `send` - Gửi tin nhắn đến một kênh

Ví dụ:

```json5
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```
## An toàn & vận hành

- **Xử lý token như mật khẩu** - Không bao giờ commit token vào git
- **Sử dụng làm mới token tự động** cho các bot chạy lâu dài
- **Sử dụng danh sách cho phép ID người dùng** thay vì tên người dùng để kiểm soát truy cập
- **Giám sát logs** để theo dõi các sự kiện làm mới token và trạng thái kết nối
- **Phạm vi token tối thiểu** - Chỉ yêu cầu `chat:read` và `chat:write`
- **Nếu gặp vấn đề**: Khởi động lại gateway sau khi xác nhận không có tiến trình nào khác sở hữu phiên
## Giới hạn

- **500 ký tự** mỗi tin nhắn (tự động chia nhỏ tại ranh giới từ)
- Markdown được loại bỏ trước khi chia nhỏ
- Không giới hạn tốc độ (sử dụng giới hạn tốc độ tích hợp của Twitch)