---
summary: >-
  iMessage thông qua máy chủ BlueBubbles macOS (gửi/nhận REST, nhập văn bản,
  phản ứng, ghép nối, các hành động nâng cao).
read_when:
  - Thiết lập kênh BlueBubbles
  - Khắc phục sự cố ghép nối webhook
  - Cấu hình iMessage trên macOS
title: BlueBubbles
x-i18n:
  source_path: channels\bluebubbles.md
  source_hash: ef020b53e3f832ba6fec1e02087c1ae61f053337cdeb2d82f8959efcde2441cf
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:14:16.034Z'
---

# BlueBubbles (macOS REST)

Trạng thái: plugin tích hợp sẵn giao tiếp với máy chủ BlueBubbles macOS qua HTTP. **Được khuyến nghị cho tích hợp iMessage** do API phong phú hơn và thiết lập dễ dàng hơn so với kênh imsg cũ.
## Tổng quan

- Chạy trên macOS thông qua ứng dụng hỗ trợ BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
- Khuyến nghị/đã kiểm tra: macOS Sequoia (15). macOS Tahoe (26) hoạt động; chức năng chỉnh sửa hiện đang bị lỗi trên Tahoe, và cập nhật biểu tượng nhóm có thể báo thành công nhưng không đồng bộ.
- OpenClaw giao tiếp với nó thông qua REST API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
- Tin nhắn đến được nhận qua webhook; phản hồi gửi đi, chỉ báo đang gõ, xác nhận đã đọc và tapback là các lệnh gọi REST.
- Tệp đính kèm và sticker được xử lý như phương tiện truyền thông đến (và hiển thị cho agent khi có thể).
- Ghép nối/danh sách cho phép hoạt động giống như các kênh khác (`/channels/pairing` v.v.) với `channels.bluebubbles.allowFrom` + mã ghép nối.
- Phản ứng được hiển thị như các sự kiện hệ thống giống như Slack/Telegram để agent có thể "đề cập" chúng trước khi phản hồi.
- Tính năng nâng cao: chỉnh sửa, thu hồi, trả lời theo chuỗi, hiệu ứng tin nhắn, quản lý nhóm.
## Bắt đầu nhanh

1. Cài đặt máy chủ BlueBubbles trên Mac của bạn (làm theo hướng dẫn tại [bluebubbles.app/install](https://bluebubbles.app/install)).
2. Trong cấu hình BlueBubbles, bật web API và đặt mật khẩu.
3. Chạy `openclaw onboard` và chọn BlueBubbles, hoặc cấu hình thủ công:

   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook",
       },
     },
   }
   ```

4. Point BlueBubbles webhooks to your gateway (example: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`).
5. Start the gateway; it will register the webhook handler and start pairing.

Security note:

- Always set a webhook password.
- Webhook authentication is always required. OpenClaw rejects BlueBubbles webhook requests unless they include a password/guid that matches `channels.bluebubbles.password` (for example `?password=<password>` or `x-password`), bất kể cấu trúc loopback/proxy.
## Giữ Messages.app hoạt động (thiết lập VM / headless)

Một số thiết lập macOS VM / luôn bật có thể khiến Messages.app chuyển sang trạng thái "idle" (các sự kiện đến sẽ dừng cho đến khi ứng dụng được mở/đưa lên foreground). Một giải pháp đơn giản là **kích hoạt Messages mỗi 5 phút** bằng AppleScript + LaunchAgent.

### 1) Lưu AppleScript

Lưu thành:

- `~/Scripts/poke-messages.scpt`

Script ví dụ (không tương tác; không chiếm focus):

```applescript
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- Touch the scripting interface to keep the process responsive.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignore transient failures (first-run prompts, locked session, etc).
end try
```

### 2) Install a LaunchAgent

Save this as:

- `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```
Ghi chú:

- Điều này chạy **mỗi 300 giây** và **khi đăng nhập**.
- Lần chạy đầu tiên có thể kích hoạt lời nhắc **Automation** của macOS (`osascript` → Messages). Hãy phê duyệt chúng trong cùng phiên người dùng chạy LaunchAgent.

Tải nó:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```
## Thiết lập ban đầu

BlueBubbles có sẵn trong trình hướng dẫn thiết lập tương tác:

```
openclaw onboard
```

The wizard prompts for:

- **Server URL** (required): BlueBubbles server address (e.g., `http://192.168.1.100:1234`)
- **Password** (required): API password from BlueBubbles Server settings
- **Webhook path** (optional): Defaults to `/bluebubbles-webhook`
- **DM policy**: pairing, allowlist, open, or disabled
- **Allow list**: Phone numbers, emails, or chat targets

You can also add BlueBubbles via CLI:

```
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```
## Kiểm soát truy cập (Tin nhắn riêng + nhóm)

Tin nhắn riêng:

- Mặc định: `channels.bluebubbles.dmPolicy = "pairing"`.
- Người gửi không xác định sẽ nhận mã ghép nối; tin nhắn bị bỏ qua cho đến khi được phê duyệt (mã hết hạn sau 1 giờ).
- Phê duyệt qua:
  - `openclaw pairing list bluebubbles`
  - `openclaw pairing approve bluebubbles <CODE>`
- Ghép nối là phương thức trao đổi token mặc định. Chi tiết: [Ghép nối](/channels/pairing)

Nhóm:

- `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (mặc định: `allowlist`).
- `channels.bluebubbles.groupAllowFrom` kiểm soát ai có thể kích hoạt trong nhóm khi `allowlist` được thiết lập.

### Kiểm soát đề cập (nhóm)

BlueBubbles hỗ trợ kiểm soát đề cập cho cuộc trò chuyện nhóm, tương tự như hành vi của iMessage/WhatsApp:

- Sử dụng `agents.list[].groupChat.mentionPatterns` (hoặc `messages.groupChat.mentionPatterns`) để phát hiện đề cập.
- Khi `requireMention` được bật cho một nhóm, agent chỉ phản hồi khi được đề cập.
- Lệnh điều khiển từ người gửi được ủy quyền bỏ qua kiểm soát đề cập.

Cấu hình theo nhóm:

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // default for all groups
        "iMessage;-;chat123": { requireMention: false }, // override for specific group
      },
    },
  },
}
```

### Command gating

- Control commands (e.g., `/config`, `/model`) require authorization.
- Uses `allowFrom` and `groupAllowFrom` để xác định ủy quyền lệnh.
- Người gửi được ủy quyền có thể chạy lệnh điều khiển ngay cả khi không đề cập trong nhóm.
## Chỉ báo đang gõ + xác nhận đã đọc

- **Chỉ báo đang gõ**: Được gửi tự động trước và trong quá trình tạo phản hồi.
- **Xác nhận đã đọc**: Được điều khiển bởi `channels.bluebubbles.sendReadReceipts` (mặc định: `true`).
- **Chỉ báo đang gõ**: OpenClaw gửi sự kiện bắt đầu gõ; BlueBubbles tự động xóa chỉ báo gõ khi gửi hoặc hết thời gian chờ (dừng thủ công qua DELETE không đáng tin cậy).

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // disable read receipts
    },
  },
}
```
## Hành động nâng cao

BlueBubbles hỗ trợ các hành động tin nhắn nâng cao khi được bật trong cấu hình:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // tapbacks (default: true)
        edit: true, // edit sent messages (macOS 13+, broken on macOS 26 Tahoe)
        unsend: true, // unsend messages (macOS 13+)
        reply: true, // reply threading by message GUID
        sendWithEffect: true, // message effects (slam, loud, etc.)
        renameGroup: true, // rename group chats
        setGroupIcon: true, // set group chat icon/photo (flaky on macOS 26 Tahoe)
        addParticipant: true, // add participants to groups
        removeParticipant: true, // remove participants from groups
        leaveGroup: true, // leave group chats
        sendAttachment: true, // send attachments/media
      },
    },
  },
}
```

Available actions:

- **react**: Add/remove tapback reactions (`messageId`, `emoji`, `remove`)
- **edit**: Edit a sent message (`messageId`, `text`)
- **unsend**: Unsend a message (`messageId`)
- **reply**: Reply to a specific message (`messageId`, `text`, `to`)
- **sendWithEffect**: Send with iMessage effect (`text`, `to`, `effectId`)
- **renameGroup**: Rename a group chat (`chatGuid`, `displayName`)
- **setGroupIcon**: Set a group chat's icon/photo (`chatGuid`, `media`) — flaky on macOS 26 Tahoe (API may return success but the icon does not sync).
- **addParticipant**: Add someone to a group (`chatGuid`, `address`)
- **removeParticipant**: Remove someone from a group (`chatGuid`, `address`)
- **leaveGroup**: Leave a group chat (`chatGuid`)
- **sendAttachment**: Send media/files (`to`, `buffer`, `filename`, `asVoice`)
  - Voice memos: set `asVoice: true` with **MP3** or **CAF** audio to send as an iMessage voice message. BlueBubbles converts MP3 → CAF when sending voice memos.

### Message IDs (short vs full)

OpenClaw may surface _short_ message IDs (e.g., `1`, `2`) to save tokens.

- `MessageSid` / `ReplyToId` can be short IDs.
- `MessageSidFull` / `ReplyToIdFull` contain the provider full IDs.
- Short IDs are in-memory; they can expire on restart or cache eviction.
- Actions accept short or full `messageId`, but short IDs will error if no longer available.

Use full IDs for durable automations and storage:

- Templates: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
- Context: `MessageSidFull` / `ReplyToIdFull` trong payload đến

Xem [Cấu hình](/gateway/configuration) để biết các biến template.
## Truyền phát theo khối

Kiểm soát việc phản hồi được gửi dưới dạng một tin nhắn duy nhất hay được truyền phát theo từng khối:

```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // enable block streaming (off by default)
    },
  },
}
```
## Media + giới hạn

- Tệp đính kèm đến được tải xuống và lưu trữ trong bộ nhớ đệm media.
- Giới hạn media thông qua `channels.bluebubbles.mediaMaxMb` (mặc định: 8 MB).
- Văn bản gửi đi được chia thành các khối `channels.bluebubbles.textChunkLimit` (mặc định: 4000 ký tự).
## Tham chiếu cấu hình

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.bluebubbles.enabled`: Bật/tắt kênh.
- `channels.bluebubbles.serverUrl`: URL cơ sở REST API BlueBubbles.
- `channels.bluebubbles.password`: Mật khẩu API.
- `channels.bluebubbles.webhookPath`: Đường dẫn endpoint webhook (mặc định: `/bluebubbles-webhook`).
- `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: `pairing`).
- `channels.bluebubbles.allowFrom`: Danh sách cho phép tin nhắn riêng (handles, emails, số E.164, `chat_id:*`, `chat_guid:*`).
- `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (mặc định: `allowlist`).
- `channels.bluebubbles.groupAllowFrom`: Danh sách cho phép người gửi nhóm.
- `channels.bluebubbles.groups`: Cấu hình theo nhóm (`requireMention`, v.v.).
- `channels.bluebubbles.sendReadReceipts`: Gửi xác nhận đã đọc (mặc định: `true`).
- `channels.bluebubbles.blockStreaming`: Bật truyền phát theo khối (mặc định: `false`; bắt buộc cho phản hồi streaming).
- `channels.bluebubbles.textChunkLimit`: Kích thước chunk gửi đi tính bằng ký tự (mặc định: 4000).
- `channels.bluebubbles.chunkMode`: `length` (mặc định) chỉ chia khi vượt quá `textChunkLimit`; `newline` chia theo dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- `channels.bluebubbles.mediaMaxMb`: Giới hạn media đến tính bằng MB (mặc định: 8).
- `channels.bluebubbles.mediaLocalRoots`: Danh sách cho phép rõ ràng các thư mục cục bộ tuyệt đối được phép cho đường dẫn media cục bộ gửi đi. Gửi đường dẫn cục bộ bị từ chối theo mặc định trừ khi được cấu hình. Ghi đè theo tài khoản: `channels.bluebubbles.accounts.<accountId>.mediaLocalRoots`.
- `channels.bluebubbles.historyLimit`: Số tin nhắn nhóm tối đa cho ngữ cảnh (0 để tắt).
- `channels.bluebubbles.dmHistoryLimit`: Giới hạn lịch sử tin nhắn riêng.
- `channels.bluebubbles.actions`: Bật/tắt các hành động cụ thể.
- `channels.bluebubbles.accounts`: Cấu hình đa tài khoản.

Tùy chọn toàn cục liên quan:

- `agents.list[].groupChat.mentionPatterns` (hoặc `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.
## Địa chỉ / mục tiêu gửi

Ưu tiên `chat_guid` để định tuyến ổn định:

- `chat_guid:iMessage;-;+15555550123` (ưu tiên cho nhóm)
- `chat_id:123`
- `chat_identifier:...`
- Tên người dùng trực tiếp: `+15555550123`, `user@example.com`
  - Nếu tên người dùng trực tiếp không có cuộc trò chuyện tin nhắn riêng hiện có, OpenClaw sẽ tạo một cuộc trò chuyện thông qua `POST /api/v1/chat/new`. Điều này yêu cầu BlueBubbles Private API được bật.
## Bảo mật

- Các yêu cầu webhook được xác thực bằng cách so sánh các tham số truy vấn hoặc header `guid`/`password` với `channels.bluebubbles.password`. Các yêu cầu từ `localhost` cũng được chấp nhận.
- Giữ bí mật mật khẩu API và endpoint webhook (xử lý chúng như thông tin xác thực).
- Tin cậy localhost có nghĩa là một reverse proxy cùng host có thể vô tình bỏ qua mật khẩu. Nếu bạn proxy gateway, hãy yêu cầu xác thực tại proxy và cấu hình `gateway.trustedProxies`. Xem [Bảo mật Gateway](/gateway/security#reverse-proxy-configuration).
- Bật HTTPS + quy tắc tường lửa trên máy chủ BlueBubbles nếu expose nó ra ngoài LAN của bạn.
## Khắc phục sự cố

- Nếu các sự kiện gõ/đọc ngừng hoạt động, hãy kiểm tra nhật ký webhook BlueBubbles và xác minh đường dẫn gateway khớp với `channels.bluebubbles.webhookPath`.
- Mã ghép nối hết hạn sau một giờ; sử dụng `openclaw pairing list bluebubbles` và `openclaw pairing approve bluebubbles <code>`.
- Phản ứng yêu cầu API riêng tư BlueBubbles (`POST /api/v1/message/react`); đảm bảo phiên bản máy chủ hiển thị nó.
- Chỉnh sửa/thu hồi yêu cầu macOS 13+ và phiên bản máy chủ BlueBubbles tương thích. Trên macOS 26 (Tahoe), chỉnh sửa hiện đang bị lỗi do thay đổi API riêng tư.
- Cập nhật biểu tượng nhóm có thể không ổn định trên macOS 26 (Tahoe): API có thể trả về thành công nhưng biểu tượng mới không đồng bộ.
- OpenClaw tự động ẩn các hành động đã biết bị lỗi dựa trên phiên bản macOS của máy chủ BlueBubbles. Nếu chỉnh sửa vẫn xuất hiện trên macOS 26 (Tahoe), hãy tắt thủ công bằng `channels.bluebubbles.actions.edit=false`.
- Để biết thông tin trạng thái/sức khỏe: `openclaw status --all` hoặc `openclaw status --deep`.

Để tham khảo quy trình kênh chung, xem [Kênh](/channels) và hướng dẫn [Plugin](/tools/plugin).