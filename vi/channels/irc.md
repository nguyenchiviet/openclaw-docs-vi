---
title: IRC
description: Connect OpenClaw to IRC channels and direct messages.
summary: 'Thiết lập plugin IRC, kiểm soát truy cập và khắc phục sự cố'
read_when:
  - Bạn muốn kết nối OpenClaw với các kênh IRC hoặc tin nhắn riêng
  - >-
    Bạn đang cấu hình danh sách cho phép IRC, chính sách nhóm, hoặc cổng kiểm
    soát đề cập
x-i18n:
  source_path: channels\irc.md
  source_hash: 82ec2803ee4d34f480f75bd1714239761c533a482a559e9a57049256ff0aabba
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:19:28.026Z'
---

Sử dụng IRC khi bạn muốn OpenClaw trong các kênh cổ điển (`#room`) và tin nhắn trực tiếp.
IRC được cung cấp dưới dạng plugin mở rộng, nhưng nó được cấu hình trong cấu hình chính dưới `channels.irc`.
## Bắt đầu nhanh

1. Bật cấu hình IRC trong `~/.openclaw/openclaw.json`.
2. Đặt ít nhất:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Start/restart gateway:

```bash
openclaw gateway run
```
## Mặc định bảo mật

- `channels.irc.dmPolicy` mặc định là `"pairing"`.
- `channels.irc.groupPolicy` mặc định là `"allowlist"`.
- Với `groupPolicy="allowlist"`, đặt `channels.irc.groups` để xác định các kênh được phép.
- Sử dụng TLS (`channels.irc.tls=true`) trừ khi bạn cố ý chấp nhận truyền tải văn bản thuần.
## Kiểm soát truy cập

Có hai "cổng" riêng biệt cho các kênh IRC:

1. **Truy cập kênh** (`groupPolicy` + `groups`): liệu bot có chấp nhận tin nhắn từ một kênh hay không.
2. **Truy cập người gửi** (`groupAllowFrom` / `groups["#channel"].allowFrom` theo kênh): ai được phép kích hoạt bot trong kênh đó.

Các khóa cấu hình:

- Danh sách cho phép tin nhắn riêng (truy cập người gửi tin nhắn riêng): `channels.irc.allowFrom`
- Danh sách cho phép người gửi nhóm (truy cập người gửi kênh): `channels.irc.groupAllowFrom`
- Điều khiển theo kênh (quy tắc kênh + người gửi + nhắc đến): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` cho phép các kênh chưa được cấu hình (**vẫn bị giới hạn bởi nhắc đến theo mặc định**)

Các mục trong danh sách cho phép nên sử dụng danh tính người gửi ổn định (`nick!user@host`).
Khớp nick trần là có thể thay đổi và chỉ được bật khi `channels.irc.dangerouslyAllowNameMatching: true`.

### Lỗi thường gặp: `allowFrom` dành cho tin nhắn riêng, không phải kênh

Nếu bạn thấy log như:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…có nghĩa là người gửi không được phép gửi tin nhắn **nhóm/kênh**. Khắc phục bằng cách:

- đặt `channels.irc.groupAllowFrom` (toàn cục cho tất cả kênh), hoặc
- đặt danh sách cho phép người gửi theo kênh: `channels.irc.groups["#channel"].allowFrom`

Ví dụ (cho phép bất kỳ ai trong `#tuirc-dev` nói chuyện với bot):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```
## Kích hoạt phản hồi (nhắc đến)

Ngay cả khi một kênh được cho phép (thông qua `groupPolicy` + `groups`) và người gửi được cho phép, OpenClaw mặc định sử dụng **cơ chế cổng nhắc đến** trong các ngữ cảnh nhóm.

Điều đó có nghĩa là bạn có thể thấy các log như `drop channel … (missing-mention)` trừ khi tin nhắn bao gồm một mẫu nhắc đến khớp với bot.

Để làm cho bot phản hồi trong kênh IRC **mà không cần nhắc đến**, hãy tắt cơ chế cổng nhắc đến cho kênh đó:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Or to allow **all** IRC channels (no per-channel allowlist) and still reply without mentions:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```
## Ghi chú bảo mật (khuyến nghị cho các kênh công khai)

Nếu bạn cho phép `allowFrom: ["*"]` trong một kênh công khai, bất kỳ ai cũng có thể nhắc bot.
Để giảm rủi ro, hãy hạn chế các công cụ cho kênh đó.

### Cùng các công cụ cho mọi người trong kênh

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Different tools per sender (owner gets more power)

Use `toolsBySender` to apply a stricter policy to `"*"` and a looser one to your nick:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Notes:

- `toolsBySender` keys should use `id:` for IRC sender identity values:
  `id:eigen` or `id:eigen!~eigen@174.127.248.171` for stronger matching.
- Legacy unprefixed keys are still accepted and matched as `id:` only.
- The first matching sender policy wins; `"*"` là ký tự đại diện dự phòng.

Để biết thêm về quyền truy cập nhóm so với cổng mention (và cách chúng tương tác), xem: [/channels/groups](/channels/groups).
## NickServ

Để xác thực với NickServ sau khi kết nối:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Optional one-time registration on connect:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Disable `register` sau khi nick đã được đăng ký để tránh các lần thử REGISTER lặp lại.
## Biến môi trường

Tài khoản mặc định hỗ trợ:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (phân tách bằng dấu phẩy)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`
## Khắc phục sự cố

- Nếu bot kết nối nhưng không bao giờ trả lời trong các kênh, hãy xác minh `channels.irc.groups` **và** liệu mention-gating có đang loại bỏ tin nhắn hay không (`missing-mention`). Nếu bạn muốn bot trả lời mà không cần ping, hãy đặt `requireMention:false` cho kênh đó.
- Nếu đăng nhập thất bại, hãy xác minh tính khả dụng của nick và mật khẩu máy chủ.
- Nếu TLS thất bại trên mạng tùy chỉnh, hãy xác minh thiết lập host/port và chứng chỉ.