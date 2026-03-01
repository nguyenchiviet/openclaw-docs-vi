---
summary: Thiết lập Slack và hành vi runtime (Socket Mode + HTTP Events API)
read_when:
  - Thiết lập Slack hoặc gỡ lỗi chế độ socket/HTTP của Slack
title: Slack
x-i18n:
  source_path: channels\slack.md
  source_hash: b9d27313bdba921bee9bc9f3d7836416b4f556858c3fd88d64a791bedf417d3b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:27:22.870Z'
---

# Slack

Trạng thái: sẵn sàng cho sản xuất với tin nhắn riêng + kênh thông qua tích hợp ứng dụng Slack. Chế độ mặc định là Socket Mode; chế độ HTTP Events API cũng được hỗ trợ.

<CardGroup cols={3}>
  <Card title="Ghép nối" icon="link" href="/channels/pairing">
    Tin nhắn riêng Slack mặc định sử dụng chế độ ghép nối.
  </Card>
  <Card title="Lệnh slash" icon="terminal" href="/tools/slash-commands">
    Hành vi lệnh gốc và danh mục lệnh.
  </Card>
  <Card title="Khắc phục sự cố kênh" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán và sách hướng dẫn sửa chữa liên kênh.
  </Card>
</CardGroup>
## Thiết lập nhanh

<Tabs>
  <Tab title="Chế độ Socket (mặc định)">
    <Steps>
      <Step title="Tạo ứng dụng Slack và token">
        Trong cài đặt ứng dụng Slack:

        - bật **Chế độ Socket**
        - tạo **Token Ứng dụng** (`xapp-...`) với `connections:write`
        - cài đặt ứng dụng và sao chép **Token Bot** (`xoxb-...`)
      </Step>

      <Step title="Cấu hình OpenClaw">

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        Env fallback (default account only):

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

      </Step>

      <Step title="Subscribe app events">
        Subscribe bot events for:

        - `app_mention`
        - `message.channels`, `message.groups`, `message.im`, `message.mpim`
        - `reaction_added`, `reaction_removed`
        - `member_joined_channel`, `member_left_channel`
        - `channel_rename`
        - `pin_added`, `pin_removed`

        Also enable App Home **Messages Tab** for DMs.
      </Step>

      <Step title="Start gateway">

```bash
openclaw gateway
```

      </Step>
    </Steps>

  </Tab>
<Tab title="Chế độ HTTP Events API">
    <Steps>
      <Step title="Cấu hình ứng dụng Slack cho HTTP">

        - đặt chế độ thành HTTP (`channels.slack.mode="http"`)
        - sao chép **Signing Secret** của Slack
        - đặt Event Subscriptions + Interactivity + Slash command Request URL thành cùng một đường dẫn webhook (mặc định `/slack/events`)

      </Step>

      <Step title="Cấu hình chế độ HTTP của OpenClaw">

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      </Step>

      <Step title="Use unique webhook paths for multi-account HTTP">
        Per-account HTTP mode is supported.

        Give each account a distinct `webhookPath` để các đăng ký không bị xung đột.
      </Step>
    </Steps>

  </Tab>
</Tabs>
## Mô hình token

- `botToken` + `appToken` được yêu cầu cho Socket Mode.
- HTTP mode yêu cầu `botToken` + `signingSecret`.
- Token cấu hình ghi đè env fallback.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback chỉ áp dụng cho tài khoản mặc định.
- `userToken` (`xoxp-...`) chỉ dành cho cấu hình (không có env fallback) và mặc định là hành vi chỉ đọc (`userTokenReadOnly: true`).
- Tùy chọn: thêm `chat:write.customize` nếu bạn muốn tin nhắn gửi đi sử dụng danh tính agent đang hoạt động (`username` và biểu tượng tùy chỉnh). `icon_emoji` sử dụng cú pháp `:emoji_name:`.

<Tip>
Đối với các hành động/đọc thư mục, user token có thể được ưu tiên khi được cấu hình. Đối với ghi, bot token vẫn được ưu tiên; ghi bằng user-token chỉ được phép khi `userTokenReadOnly: false` và bot token không khả dụng.
</Tip>
## Kiểm soát truy cập và định tuyến

<Tabs>
  <Tab title="Chính sách tin nhắn riêng">
    `channels.slack.dmPolicy` kiểm soát quyền truy cập tin nhắn riêng (cũ: `channels.slack.dm.policy`):

    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `channels.slack.allowFrom` bao gồm `"*"`; cũ: `channels.slack.dm.allowFrom`)
    - `disabled`

    Cờ tin nhắn riêng:

    - `dm.enabled` (mặc định true)
    - `channels.slack.allowFrom` (ưu tiên)
    - `dm.allowFrom` (cũ)
    - `dm.groupEnabled` (tin nhắn riêng nhóm mặc định false)
    - `dm.groupChannels` (danh sách cho phép MPIM tùy chọn)

    Thứ tự ưu tiên đa tài khoản:

    - `channels.slack.accounts.default.allowFrom` chỉ áp dụng cho tài khoản `default`.
    - Các tài khoản có tên kế thừa `channels.slack.allowFrom` khi `allowFrom` của chúng không được đặt.
    - Các tài khoản có tên không kế thừa `channels.slack.accounts.default.allowFrom`.

    Ghép nối trong tin nhắn riêng sử dụng `openclaw pairing approve slack <code>`.

  </Tab>

  <Tab title="Chính sách kênh">
    `channels.slack.groupPolicy` kiểm soát xử lý kênh:

    - `open`
    - `allowlist`
    - `disabled`

    Danh sách cho phép kênh nằm dưới `channels.slack.channels`.

    Lưu ý runtime: nếu `channels.slack` hoàn toàn bị thiếu (thiết lập chỉ env), runtime sẽ quay về `groupPolicy="allowlist"` và ghi log cảnh báo (ngay cả khi `channels.defaults.groupPolicy` được đặt).

    Phân giải tên/ID:

    - các mục trong danh sách cho phép kênh và danh sách cho phép tin nhắn riêng được phân giải khi khởi động khi quyền truy cập token cho phép
    - các mục không phân giải được giữ nguyên như đã cấu hình
    - khớp ủy quyền đến mặc định là ID-first; khớp trực tiếp username/slug yêu cầu `channels.slack.dangerouslyAllowNameMatching: true`

  </Tab>

  <Tab title="Đề cập và người dùng kênh">
    Tin nhắn kênh được kiểm soát bằng đề cập theo mặc định.

    Nguồn đề cập:

    - đề cập ứng dụng rõ ràng (`<@botId>`)
    - mẫu regex đề cập (`agents.list[].groupChat.mentionPatterns`, dự phòng `messages.groupChat.mentionPatterns`)
    - hành vi thread trả lời bot ngầm định

    Kiểm soát theo kênh (`channels.slack.channels.<id|name>`):

    - `requireMention`

  </Tab>
</Tabs>
- `users` (danh sách cho phép)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    - `toolsBySender` định dạng khóa: `id:`, `e164:`, `username:`, `name:`, hoặc `"*"` ký tự đại diện
      (các khóa cũ không có tiền tố vẫn ánh xạ chỉ đến `id:`)

  </Tab>
</Tabs>
## Lệnh và hành vi slash

- Chế độ tự động lệnh gốc được **tắt** cho Slack (`commands.native: "auto"` không kích hoạt lệnh gốc của Slack).
- Kích hoạt trình xử lý lệnh gốc của Slack với `channels.slack.commands.native: true` (hoặc toàn cục `commands.native: true`).
- Khi lệnh gốc được kích hoạt, đăng ký các lệnh slash tương ứng trong Slack (tên `/<command>`).
- Nếu lệnh gốc không được kích hoạt, bạn có thể chạy một lệnh slash đã cấu hình thông qua `channels.slack.slashCommand`.
- Menu tham số gốc hiện điều chỉnh chiến lược hiển thị của chúng:
  - tối đa 5 tùy chọn: khối nút
  - 6-100 tùy chọn: menu chọn tĩnh
  - hơn 100 tùy chọn: chọn bên ngoài với lọc tùy chọn không đồng bộ khi có sẵn trình xử lý tùy chọn tương tác
  - nếu giá trị tùy chọn được mã hóa vượt quá giới hạn của Slack, luồng sẽ quay lại sử dụng nút
- Đối với tải trọng tùy chọn dài, menu tham số lệnh Slash sử dụng hộp thoại xác nhận trước khi gửi giá trị đã chọn.

Cài đặt lệnh slash mặc định:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Phiên slash sử dụng khóa cô lập:

- `agent:<agentId>:slack:slash:<userId>`

và vẫn định tuyến việc thực thi lệnh đối với phiên cuộc trò chuyện đích (`CommandTargetSessionKey`).
## Threading, phiên và thẻ trả lời

- Tin nhắn riêng định tuyến như `direct`; kênh như `channel`; MPIM như `group`.
- Với `session.dmScope=main` mặc định, tin nhắn riêng Slack thu gọn thành phiên chính của agent.
- Phiên kênh: `agent:<agentId>:slack:channel:<channelId>`.
- Trả lời thread có thể tạo hậu tố phiên thread (`:thread:<threadTs>`) khi có thể áp dụng.
- `channels.slack.thread.historyScope` mặc định là `thread`; `thread.inheritParent` mặc định là `false`.
- `channels.slack.thread.initialHistoryLimit` kiểm soát số lượng tin nhắn thread hiện có được tải khi phiên thread mới bắt đầu (mặc định `20`; đặt `0` để vô hiệu hóa).

Điều khiển threading trả lời:

- `channels.slack.replyToMode`: `off|first|all` (mặc định `off`)
- `channels.slack.replyToModeByChatType`: theo `direct|group|channel`
- dự phòng cũ cho trò chuyện trực tiếp: `channels.slack.dm.replyToMode`

Thẻ trả lời thủ công được hỗ trợ:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Lưu ý: `replyToMode="off"` vô hiệu hóa **tất cả** reply threading trong Slack, bao gồm cả thẻ `[[reply_to_*]]` rõ ràng. Điều này khác với Telegram, nơi thẻ rõ ràng vẫn được tôn trọng trong chế độ `"off"`. Sự khác biệt phản ánh mô hình threading của nền tảng: thread Slack ẩn tin nhắn khỏi kênh, trong khi trả lời Telegram vẫn hiển thị trong luồng trò chuyện chính.
## Phương tiện, phân đoạn và gửi

<AccordionGroup>
  <Accordion title="Tệp đính kèm đến">
    Các tệp đính kèm Slack được tải xuống từ các URL riêng tư do Slack lưu trữ (luồng yêu cầu xác thực token) và ghi vào kho phương tiện khi tải thành công và giới hạn kích thước cho phép.

    Giới hạn kích thước đến trong thời gian chạy mặc định là `20MB` trừ khi được ghi đè bởi `channels.slack.mediaMaxMb`.

  </Accordion>

  <Accordion title="Văn bản và tệp đi">
    - các đoạn văn bản sử dụng `channels.slack.textChunkLimit` (mặc định 4000)
    - `channels.slack.chunkMode="newline"` cho phép phân tách theo đoạn văn trước
    - gửi tệp sử dụng API tải lên Slack và có thể bao gồm phản hồi trong chuỗi (`thread_ts`)
    - giới hạn phương tiện đi tuân theo `channels.slack.mediaMaxMb` khi được cấu hình; nếu không thì gửi kênh sử dụng mặc định theo loại MIME từ pipeline phương tiện
  </Accordion>

  <Accordion title="Đích gửi">
    Các đích rõ ràng được ưu tiên:

    - `user:<id>` cho tin nhắn riêng
    - `channel:<id>` cho kênh

    Tin nhắn riêng Slack được mở thông qua API cuộc trò chuyện Slack khi gửi đến đích người dùng.

  </Accordion>
</AccordionGroup>
## Actions và gates

Các action của Slack được kiểm soát bởi `channels.slack.actions.*`.

Các nhóm action có sẵn trong công cụ Slack hiện tại:

| Nhóm       | Mặc định |
| ---------- | -------- |
| messages   | enabled  |
| reactions  | enabled  |
| pins       | enabled  |
| memberInfo | enabled  |
| emojiList  | enabled  |
## Sự kiện và hành vi hoạt động

- Chỉnh sửa/xóa tin nhắn/phát sóng thread được ánh xạ thành các sự kiện hệ thống.
- Sự kiện thêm/xóa reaction được ánh xạ thành các sự kiện hệ thống.
- Sự kiện thành viên tham gia/rời khỏi, kênh được tạo/đổi tên, và thêm/xóa ghim được ánh xạ thành các sự kiện hệ thống.
- Cập nhật trạng thái thread trợ lý (cho chỉ báo "đang gõ..." trong thread) sử dụng `assistant.threads.setStatus` và yêu cầu phạm vi bot `assistant:write`.
- `channel_id_changed` có thể di chuyển các khóa cấu hình kênh khi `configWrites` được bật.
- Metadata chủ đề/mục đích kênh được coi là ngữ cảnh không đáng tin cậy và có thể được chèn vào ngữ cảnh định tuyến.
- Hành động khối và tương tác modal phát ra các sự kiện hệ thống `Slack interaction: ...` có cấu trúc với các trường payload phong phú:
  - hành động khối: các giá trị đã chọn, nhãn, giá trị picker, và metadata `workflow_*`
  - sự kiện `view_submission` và `view_closed` modal với metadata kênh được định tuyến và đầu vào form
## Phản ứng xác nhận

`ackReaction` gửi một emoji xác nhận trong khi OpenClaw đang xử lý tin nhắn đến.

Thứ tự ưu tiên:

- `channels.slack.accounts.<accountId>.ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- emoji dự phòng của agent (`agents.list[].identity.emoji`, nếu không có thì "👀")

Ghi chú:

- Slack yêu cầu shortcode (ví dụ `"eyes"`).
- Sử dụng `""` để tắt phản ứng cho một kênh hoặc tài khoản.
## Danh sách kiểm tra manifest và phạm vi

<AccordionGroup>
  <Accordion title="Ví dụ manifest ứng dụng Slack">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```
</Accordion>

  <Accordion title="Phạm vi user-token tùy chọn (các thao tác đọc)">
    Nếu bạn cấu hình `channels.slack.userToken`, các phạm vi đọc thông thường là:

    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (nếu bạn phụ thuộc vào việc đọc tìm kiếm Slack)

  </Accordion>
</AccordionGroup>
## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="Không có phản hồi trong kênh">
    Kiểm tra theo thứ tự:

    - `groupPolicy`
    - danh sách cho phép kênh (`channels.slack.channels`)
    - `requireMention`
    - danh sách cho phép `users` theo kênh

    Lệnh hữu ích:

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  </Accordion>

  <Accordion title="DM messages ignored">
    Check:

    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (or legacy `channels.slack.dm.policy`)
    - pairing approvals / allowlist entries

```bash
openclaw pairing list slack
```

  </Accordion>

  <Accordion title="Socket mode not connecting">
    Validate bot + app tokens and Socket Mode enablement in Slack app settings.
  </Accordion>

  <Accordion title="HTTP mode not receiving events">
    Validate:

    - signing secret
    - webhook path
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - unique `webhookPath` per HTTP account

  </Accordion>

  <Accordion title="Native/slash commands not firing">
    Verify whether you intended:

    - native command mode (`channels.slack.commands.native: true`) with matching slash commands registered in Slack
    - or single slash command mode (`channels.slack.slashCommand.enabled: true`)

    Also check `commands.useAccessGroups` và danh sách cho phép kênh/người dùng.

  </Accordion>
</AccordionGroup>
## Truyền phát văn bản

OpenClaw hỗ trợ truyền phát văn bản gốc của Slack thông qua API Agents và AI Apps.

`channels.slack.streaming` điều khiển hành vi xem trước trực tiếp:

- `off`: tắt truyền phát xem trước trực tiếp.
- `partial` (mặc định): thay thế văn bản xem trước bằng đầu ra một phần mới nhất.
- `block`: thêm các cập nhật xem trước theo khối.
- `progress`: hiển thị văn bản trạng thái tiến trình trong khi tạo, sau đó gửi văn bản cuối cùng.

`channels.slack.nativeStreaming` điều khiển API truyền phát gốc của Slack (`chat.startStream` / `chat.appendStream` / `chat.stopStream`) khi `streaming` là `partial` (mặc định: `true`).

Tắt truyền phát gốc của Slack (giữ hành vi xem trước bản nháp):

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

Legacy keys:

- `channels.slack.streamMode` (`replace | status_final | append`) is auto-migrated to `channels.slack.streaming`.
- boolean `channels.slack.streaming` is auto-migrated to `channels.slack.nativeStreaming`.

### Requirements

1. Enable **Agents and AI Apps** in your Slack app settings.
2. Ensure the app has the `assistant:write` scope.
3. A reply thread must be available for that message. Thread selection still follows `replyToMode`.

### Behavior

- First text chunk starts a stream (`chat.startStream`).
- Later text chunks append to the same stream (`chat.appendStream`).
- End of reply finalizes stream (`chat.stopStream`).
- Phương tiện và tải trọng không phải văn bản sẽ quay về phương thức gửi bình thường.
- Nếu truyền phát thất bại giữa chừng phản hồi, OpenClaw sẽ quay về phương thức gửi bình thường cho các tải trọng còn lại.
## Tham chiếu cấu hình

Tham chiếu chính:

- [Tham chiếu cấu hình - Slack](/gateway/configuration-reference#slack)

  Các trường Slack quan trọng:
  - chế độ/xác thực: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - truy cập tin nhắn riêng: `dm.enabled`, `dmPolicy`, `allowFrom` (cũ: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - chuyển đổi tương thích: `dangerouslyAllowNameMatching` (khẩn cấp; tắt trừ khi cần thiết)
  - truy cập kênh: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - luồng/lịch sử: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - gửi tin: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
  - vận hành/tính năng: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`
## Liên quan

- [Ghép nối](/channels/pairing)
- [Định tuyến kênh](/channels/channel-routing)
- [Khắc phục sự cố](/channels/troubleshooting)
- [Cấu hình](/gateway/configuration)
- [Lệnh slash](/tools/slash-commands)