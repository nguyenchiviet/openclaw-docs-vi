---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình bot Telegram'
read_when:
  - Làm việc với các tính năng Telegram hoặc webhook
title: Telegram
x-i18n:
  source_path: channels\telegram.md
  source_hash: f70da83c546499866a28da6c54368c2e5a39ac759d8d2e506017f372b18bf017
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:28:00.401Z'
---

# Telegram (Bot API)

Trạng thái: sẵn sàng cho sản xuất với tin nhắn riêng bot + nhóm qua grammY. Long polling là chế độ mặc định; chế độ webhook là tùy chọn.

<CardGroup cols={3}>
  <Card title="Ghép nối" icon="link" href="/channels/pairing">
    Chính sách tin nhắn riêng mặc định cho Telegram là ghép nối.
  </Card>
  <Card title="Khắc phục sự cố kênh" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán và sách hướng dẫn sửa chữa liên kênh.
  </Card>
  <Card title="Cấu hình Gateway" icon="settings" href="/gateway/configuration">
    Các mẫu và ví dụ cấu hình kênh đầy đủ.
  </Card>
</CardGroup>
## Thiết lập nhanh

<Steps>
  <Step title="Tạo token bot trong BotFather">
    Mở Telegram và trò chuyện với **@BotFather** (xác nhận handle chính xác là `@BotFather`).

    Chạy `/newbot`, làm theo hướng dẫn và lưu token.

  </Step>

  <Step title="Cấu hình token và chính sách tin nhắn riêng">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    Env fallback: `TELEGRAM_BOT_TOKEN=...` (default account only).
    Telegram does **not** use `openclaw channels login telegram`; configure token in config/env, then start gateway.

  </Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    Pairing codes expire after 1 hour.

  </Step>

  <Step title="Add the bot to a group">
    Add the bot to your group, then set `channels.telegram.groups` and `groupPolicy` to match your access model.
  </Step>
</Steps>

<Note>
Token resolution order is account-aware. In practice, config values win over env fallback, and `TELEGRAM_BOT_TOKEN` chỉ áp dụng cho tài khoản mặc định.
</Note>
## Cài đặt phía Telegram

<AccordionGroup>
  <Accordion title="Chế độ riêng tư và khả năng hiển thị nhóm">
    Bot Telegram mặc định sử dụng **Chế độ riêng tư**, điều này giới hạn những tin nhắn nhóm mà chúng nhận được.

    Nếu bot phải xem tất cả tin nhắn nhóm, hãy:

    - tắt chế độ riêng tư qua `/setprivacy`, hoặc
    - đặt bot làm quản trị viên nhóm.

    Khi chuyển đổi chế độ riêng tư, hãy xóa + thêm lại bot trong mỗi nhóm để Telegram áp dụng thay đổi.

  </Accordion>

  <Accordion title="Quyền nhóm">
    Trạng thái quản trị viên được kiểm soát trong cài đặt nhóm Telegram.

    Bot quản trị viên nhận tất cả tin nhắn nhóm, điều này hữu ích cho hành vi nhóm luôn hoạt động.

  </Accordion>

  <Accordion title="Các tùy chọn hữu ích của BotFather">

    - `/setjoingroups` để cho phép/từ chối thêm vào nhóm
    - `/setprivacy` cho hành vi hiển thị nhóm

  </Accordion>
</AccordionGroup>
## Kiểm soát truy cập và kích hoạt

<Tabs>
  <Tab title="Chính sách tin nhắn riêng">
    `channels.telegram.dmPolicy` kiểm soát quyền truy cập tin nhắn trực tiếp:

    - `pairing` (mặc định)
    - `allowlist` (yêu cầu ít nhất một ID người gửi trong `allowFrom`)
    - `open` (yêu cầu `allowFrom` bao gồm `"*"`)
    - `disabled`

    `channels.telegram.allowFrom` chấp nhận ID người dùng Telegram dạng số. Tiền tố `telegram:` / `tg:` được chấp nhận và chuẩn hóa.
    `dmPolicy: "allowlist"` với `allowFrom` trống sẽ chặn tất cả tin nhắn riêng và bị từ chối bởi xác thực cấu hình.
    Trình hướng dẫn thiết lập ban đầu chấp nhận đầu vào `@username` và phân giải thành ID dạng số.
    Nếu bạn đã nâng cấp và cấu hình của bạn chứa các mục danh sách cho phép `@username`, hãy chạy `openclaw doctor --fix` để phân giải chúng (cố gắng tối đa; yêu cầu token bot Telegram).
    Nếu trước đây bạn dựa vào tệp danh sách cho phép pairing-store, `openclaw doctor --fix` có thể khôi phục các mục vào `channels.telegram.allowFrom` trong luồng danh sách cho phép (ví dụ khi `dmPolicy: "allowlist"` chưa có ID rõ ràng).

    ### Tìm ID người dùng Telegram của bạn

    An toàn hơn (không có bot bên thứ ba):

    1. Gửi tin nhắn riêng cho bot của bạn.
    2. Chạy `openclaw logs --follow`.
    3. Đọc `from.id`.

    Phương pháp Bot API chính thức:

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    Third-party method (less private): `@userinfobot` or `@getidsbot`.

  </Tab>

  <Tab title="Group policy and allowlists">
    Two controls apply together:

    1. **Which groups are allowed** (`channels.telegram.groups`)
       - no `groups` config:
         - with `groupPolicy: "open"`: any group can pass group-ID checks
         - with `groupPolicy: "allowlist"` (default): groups are blocked until you add `groups` entries (or `"*"`)
       - `groups` configured: acts as allowlist (explicit IDs or `"*"`)

    2. **Which senders are allowed in groups** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (default)
       - `disabled`

    `groupAllowFrom` is used for group sender filtering. If not set, Telegram falls back to `allowFrom`.
    `groupAllowFrom` entries should be numeric Telegram user IDs (`telegram:` / `tg:` prefixes are normalized).
    Non-numeric entries are ignored for sender authorization.
    Security boundary (`2026.2.25+`): group sender auth does **not** inherit DM pairing-store approvals.
    Pairing stays DM-only. For groups, set `groupAllowFrom` or per-group/per-topic `allowFrom`.
    Runtime note: if `channels.telegram` is completely missing, runtime defaults to fail-closed `groupPolicy="allowlist"` unless `channels.defaults.groupPolicy` is explicitly set.

    Example: allow any member in one specific group:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```
</Tab>

  <Tab title="Hành vi mention">
    Phản hồi nhóm yêu cầu mention theo mặc định.

    Mention có thể đến từ:

    - mention `@botusername` gốc, hoặc
    - các mẫu mention trong:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`

    Lệnh chuyển đổi cấp phiên:

    - `/activation always`
    - `/activation mention`

    Những lệnh này chỉ cập nhật trạng thái phiên. Sử dụng cấu hình để lưu trữ lâu dài.

    Ví dụ cấu hình lâu dài:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    Getting the group chat ID:

    - forward a group message to `@userinfobot` / `@getidsbot`
    - or read `chat.id` from `openclaw logs --follow`
    - or inspect Bot API `getUpdates`

  </Tab>
</Tabs>
## Hành vi runtime

- Telegram được sở hữu bởi tiến trình gateway.
- Định tuyến là xác định: tin nhắn đến từ Telegram sẽ trả lời lại Telegram (mô hình không chọn kênh).
- Tin nhắn đến được chuẩn hóa thành envelope kênh chia sẻ với metadata trả lời và placeholder phương tiện.
- Phiên nhóm được cô lập theo ID nhóm. Chủ đề diễn đàn thêm `:topic:<threadId>` để giữ các chủ đề cô lập.
- Tin nhắn riêng có thể mang `message_thread_id`; OpenClaw định tuyến chúng với khóa phiên nhận biết thread và bảo toàn ID thread cho các phản hồi.
- Long polling sử dụng grammY runner với trình tự theo từng chat/từng thread. Đồng thời sink runner tổng thể sử dụng `agents.defaults.maxConcurrent`.
- Telegram Bot API không hỗ trợ xác nhận đã đọc (`sendReadReceipts` không áp dụng).
## Tham khảo tính năng

<AccordionGroup>
  <Accordion title="Xem trước luồng trực tiếp (chỉnh sửa tin nhắn)">
    OpenClaw có thể truyền phát các phản hồi từng phần bằng cách gửi một tin nhắn Telegram tạm thời và chỉnh sửa nó khi văn bản đến.

    Yêu cầu:

    - `channels.telegram.streaming` là `off | partial | block | progress` (mặc định: `off`)
    - `progress` ánh xạ tới `partial` trên Telegram (tương thích với đặt tên chéo kênh)
    - các giá trị `channels.telegram.streamMode` cũ và boolean `streaming` được tự động ánh xạ

    Tính năng này hoạt động trong cuộc trò chuyện trực tiếp và nhóm/chủ đề.

    Đối với phản hồi chỉ có văn bản, OpenClaw giữ nguyên tin nhắn xem trước và thực hiện chỉnh sửa cuối cùng tại chỗ (không có tin nhắn thứ hai).

    Đối với phản hồi phức tạp (ví dụ như tải trọng phương tiện), OpenClaw quay lại giao hàng cuối cùng bình thường và sau đó dọn dẹp tin nhắn xem trước.

    Truyền phát xem trước tách biệt với truyền phát theo khối. Khi truyền phát theo khối được bật rõ ràng cho Telegram, OpenClaw bỏ qua luồng xem trước để tránh truyền phát kép.

    Luồng lý luận chỉ dành cho Telegram:

    - `/reasoning stream` gửi lý luận đến xem trước trực tiếp trong khi tạo
    - câu trả lời cuối cùng được gửi mà không có văn bản lý luận

  </Accordion>

  <Accordion title="Định dạng và dự phòng HTML">
    Văn bản gửi đi sử dụng `parse_mode: "HTML"` của Telegram.

    - Văn bản kiểu Markdown được hiển thị thành HTML an toàn cho Telegram.
    - HTML thô của mô hình được thoát để giảm lỗi phân tích cú pháp của Telegram.
    - Nếu Telegram từ chối HTML đã phân tích, OpenClaw thử lại dưới dạng văn bản thuần túy.

    Xem trước liên kết được bật theo mặc định và có thể tắt bằng `channels.telegram.linkPreview: false`.

  </Accordion>

  <Accordion title="Lệnh gốc và lệnh tùy chỉnh">
    Đăng ký menu lệnh Telegram được xử lý khi khởi động với `setMyCommands`.

    Mặc định lệnh gốc:

    - `commands.native: "auto"` bật lệnh gốc cho Telegram

    Thêm mục menu lệnh tùy chỉnh:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

  </Accordion>
</AccordionGroup>
Quy tắc:

- tên được chuẩn hóa (loại bỏ `/` ở đầu, chuyển thành chữ thường)
- mẫu hợp lệ: `a-z`, `0-9`, `_`, độ dài `1..32`
- lệnh tùy chỉnh không thể ghi đè lệnh gốc
- xung đột/trùng lặp sẽ bị bỏ qua và ghi log

Ghi chú:

- lệnh tùy chỉnh chỉ là mục menu; chúng không tự động triển khai hành vi
- lệnh plugin/skill vẫn có thể hoạt động khi gõ ngay cả khi không hiển thị trong menu Telegram

Nếu lệnh gốc bị vô hiệu hóa, các lệnh tích hợp sẽ bị xóa. Lệnh tùy chỉnh/plugin vẫn có thể đăng ký nếu được cấu hình.

Lỗi thiết lập phổ biến:

- `setMyCommands failed` thường có nghĩa là DNS/HTTPS đi ra tới `api.telegram.org` bị chặn.

### Lệnh ghép nối thiết bị (plugin `device-pair`)

Khi plugin `device-pair` được cài đặt:

1. `/pair` tạo mã thiết lập
2. dán mã vào ứng dụng iOS
3. `/pair approve` phê duyệt yêu cầu đang chờ mới nhất

Chi tiết thêm: [Ghép nối](/channels/pairing#pair-via-telegram-recommended-for-ios).

  </Accordion>

  <Accordion title="Nút nội tuyến">
    Cấu hình phạm vi bàn phím nội tuyến:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    Per-account override:

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```
Phạm vi:

- `off`
- `dm`
- `group`
- `all`
- `allowlist` (mặc định)

`capabilities: ["inlineButtons"]` cũ ánh xạ tới `inlineButtons: "all"`.

Ví dụ hành động tin nhắn:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    Callback clicks are passed to the agent as text:
    `callback_data: <value>`

  </Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram tool actions include:

    - `sendMessage` (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    - `createForumTopic` (`chatId`, `name`, optional `iconColor`, `iconCustomEmojiId`)

    Channel message actions expose ergonomic aliases (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`).

    Gating controls:

    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (default: disabled)

    Note: `edit` and `topic-create` are currently enabled by default and do not have separate `channels.telegram.actions.*` bật/tắt.

Ngữ nghĩa xóa phản ứng: [/tools/reactions](/tools/reactions)

  </Accordion>

  <Accordion title="Thẻ luồng trả lời">
    Telegram hỗ trợ thẻ luồng trả lời rõ ràng trong đầu ra được tạo:
- `[[reply_to_current]]` trả lời tin nhắn kích hoạt
    - `[[reply_to:<id>]]` trả lời một ID tin nhắn Telegram cụ thể

    `channels.telegram.replyToMode` điều khiển xử lý:

    - `off` (mặc định)
    - `first`
    - `all`

    Lưu ý: `off` vô hiệu hóa luồng trả lời ngầm định. Các thẻ `[[reply_to_*]]` rõ ràng vẫn được tôn trọng.

  </Accordion>

  <Accordion title="Chủ đề diễn đàn và hành vi luồng">
    Siêu nhóm diễn đàn:

    - khóa phiên chủ đề thêm `:topic:<threadId>`
    - trả lời và gõ nhắm vào luồng chủ đề
    - đường dẫn cấu hình chủ đề:
      `channels.telegram.groups.<chatId>.topics.<threadId>`

    Trường hợp đặc biệt chủ đề chung (`threadId=1`):

    - gửi tin nhắn bỏ qua `message_thread_id` (Telegram từ chối `sendMessage(...thread_id=1)`)
    - hành động gõ vẫn bao gồm `message_thread_id`

    Kế thừa chủ đề: các mục chủ đề kế thừa cài đặt nhóm trừ khi bị ghi đè (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).

    Ngữ cảnh mẫu bao gồm:

    - `MessageThreadId`
    - `IsForum`

    Hành vi luồng tin nhắn riêng:

    - cuộc trò chuyện riêng với `message_thread_id` giữ định tuyến tin nhắn riêng nhưng sử dụng khóa phiên/mục tiêu trả lời nhận biết luồng.

  </Accordion>

  <Accordion title="Âm thanh, video và sticker">
    ### Tin nhắn âm thanh

    Telegram phân biệt ghi chú thoại và tệp âm thanh.

    - mặc định: hành vi tệp âm thanh
    - thẻ `[[audio_as_voice]]` trong phản hồi agent để buộc gửi ghi chú thoại

    Ví dụ hành động tin nhắn:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```
### Tin nhắn video

Telegram phân biệt tệp video và ghi chú video.

Ví dụ hành động tin nhắn:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    Video notes do not support captions; provided message text is sent separately.

    ### Stickers

    Inbound sticker handling:

    - static WEBP: downloaded and processed (placeholder `<media:sticker>`)
    - animated TGS: skipped
    - video WEBM: skipped

    Sticker context fields:

    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`

    Sticker cache file:

    - `~/.openclaw/telegram/sticker-cache.json`

    Stickers are described once (when possible) and cached to reduce repeated vision calls.

    Enable sticker actions:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    Send sticker action:

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```
Tìm kiếm sticker đã lưu trong bộ nhớ đệm:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  </Accordion>

  <Accordion title="Reaction notifications">
    Telegram reactions arrive as `message_reaction` updates (separate from message payloads).

    When enabled, OpenClaw enqueues system events like:

    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`

    Config:

    - `channels.telegram.reactionNotifications`: `off | own | all` (default: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (default: `minimal`)

    Notes:

    - `own` means user reactions to bot-sent messages only (best-effort via sent-message cache).
    - Reaction events still respect Telegram access controls (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`); unauthorized senders are dropped.
    - Telegram does not provide thread IDs in reaction updates.
      - non-forum groups route to group chat session
      - forum groups route to the group general-topic session (`:topic:1`), not the exact originating topic

    `allowed_updates` for polling/webhook include `message_reaction` automatically.

  </Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` sends an acknowledgement emoji while OpenClaw is processing an inbound message.

    Resolution order:

    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, else "👀")

    Notes:

    - Telegram expects unicode emoji (for example "👀").
    - Use `""` to disable the reaction for a channel or account.

  </Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Channel config writes are enabled by default (`configWrites !== false`).

Các thao tác ghi được kích hoạt bởi Telegram bao gồm:
- sự kiện di chuyển nhóm (`migrate_to_chat_id`) để cập nhật `channels.telegram.groups`
    - `/config set` và `/config unset` (yêu cầu kích hoạt lệnh)

    Vô hiệu hóa:

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  </Accordion>

  <Accordion title="Long polling vs webhook">
    Default: long polling.

    Webhook mode:

    - set `channels.telegram.webhookUrl`
    - set `channels.telegram.webhookSecret` (required when webhook URL is set)
    - optional `channels.telegram.webhookPath` (default `/telegram-webhook`)
    - optional `channels.telegram.webhookHost` (default `127.0.0.1`)
    - optional `channels.telegram.webhookPort` (default `8787`)

    Default local listener for webhook mode binds to `127.0.0.1:8787`.

    If your public endpoint differs, place a reverse proxy in front and point `webhookUrl` at the public URL.
    Set `webhookHost` (for example `0.0.0.0`) when you intentionally need external ingress.

  </Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` default is 4000.
    - `channels.telegram.chunkMode="newline"` prefers paragraph boundaries (blank lines) before length splitting.
    - `channels.telegram.mediaMaxMb` (default 5) caps inbound Telegram media download/processing size.
    - `channels.telegram.timeoutSeconds` overrides Telegram API client timeout (if unset, grammY default applies).
    - group context history uses `channels.telegram.historyLimit` or `messages.groupChat.historyLimit` (default 50); `0` disables.
    - DM history controls:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - `channels.telegram.retry` config applies to Telegram send helpers (CLI/tools/actions) for recoverable outbound API errors.

    CLI send target can be numeric chat ID or username:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  </Accordion>
</AccordionGroup>
## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="Bot không phản hồi tin nhắn nhóm không được nhắc đến">

    - Nếu `requireMention=false`, chế độ riêng tư Telegram phải cho phép hiển thị đầy đủ.
      - BotFather: `/setprivacy` -> Tắt
      - sau đó xóa + thêm lại bot vào nhóm
    - `openclaw channels status` cảnh báo khi cấu hình mong đợi tin nhắn nhóm không được nhắc đến.
    - `openclaw channels status --probe` có thể kiểm tra ID nhóm số rõ ràng; ký tự đại diện `"*"` không thể được kiểm tra thành viên.
    - kiểm tra phiên nhanh: `/activation always`.

  </Accordion>

  <Accordion title="Bot không thấy tin nhắn nhóm nào cả">

    - khi `channels.telegram.groups` tồn tại, nhóm phải được liệt kê (hoặc bao gồm `"*"`)
    - xác minh thành viên bot trong nhóm
    - xem lại nhật ký: `openclaw logs --follow` để biết lý do bỏ qua

  </Accordion>

  <Accordion title="Lệnh hoạt động một phần hoặc không hoạt động">

    - ủy quyền danh tính người gửi của bạn (ghép nối và/hoặc số `allowFrom`)
    - ủy quyền lệnh vẫn áp dụng ngay cả khi chính sách nhóm là `open`
    - `setMyCommands failed` thường chỉ ra vấn đề khả năng truy cập DNS/HTTPS đến `api.telegram.org`

  </Accordion>

  <Accordion title="Polling hoặc mạng không ổn định">

    - Node 22+ + fetch/proxy tùy chỉnh có thể kích hoạt hành vi hủy bỏ ngay lập tức nếu các loại AbortSignal không khớp.
    - Một số máy chủ phân giải `api.telegram.org` thành IPv6 trước; IPv6 egress bị hỏng có thể gây ra lỗi API Telegram không liên tục.
    - Nếu nhật ký bao gồm `TypeError: fetch failed` hoặc `Network request for 'getUpdates' failed!`, OpenClaw hiện thử lại những lỗi này như lỗi mạng có thể khôi phục.
    - Trên các máy chủ VPS với egress/TLS trực tiếp không ổn định, định tuyến các cuộc gọi API Telegram qua `channels.telegram.proxy`:

```yaml
channels:
  telegram:
    proxy: socks5://user:pass@proxy-host:1080
```

    - Node 22+ defaults to `autoSelectFamily=true` (except WSL2) and `dnsResultOrder=ipv4first`.
    - If your host is WSL2 or explicitly works better with IPv4-only behavior, force family selection:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

    - Environment overrides (temporary):
      - `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
      - `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
      - `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
    - Validate DNS answers:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```
</Accordion>
</AccordionGroup>

Thêm trợ giúp: [Khắc phục sự cố kênh](/channels/troubleshooting).
## Tham chiếu cấu hình Telegram

Tham chiếu chính:

- `channels.telegram.enabled`: bật/tắt khởi động kênh.
- `channels.telegram.botToken`: token bot (BotFather).
- `channels.telegram.tokenFile`: đọc token từ đường dẫn tệp.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: pairing).
- `channels.telegram.allowFrom`: danh sách cho phép tin nhắn riêng (ID người dùng Telegram dạng số). `allowlist` yêu cầu ít nhất một ID người gửi. `open` yêu cầu `"*"`. `openclaw doctor --fix` có thể phân giải các mục `@username` cũ thành ID và có thể khôi phục các mục danh sách cho phép từ tệp pairing-store trong luồng di chuyển danh sách cho phép.
- `channels.telegram.defaultTo`: đích Telegram mặc định được sử dụng bởi CLI `--deliver` khi không có `--reply-to` rõ ràng được cung cấp.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (mặc định: allowlist).
- `channels.telegram.groupAllowFrom`: danh sách cho phép người gửi nhóm (ID người dùng Telegram dạng số). `openclaw doctor --fix` có thể phân giải các mục `@username` cũ thành ID. Các mục không phải số sẽ bị bỏ qua tại thời điểm xác thực. Xác thực nhóm không sử dụng dự phòng pairing-store tin nhắn riêng (`2026.2.25+`).
- Thứ tự ưu tiên đa tài khoản:
  - `channels.telegram.accounts.default.allowFrom` và `channels.telegram.accounts.default.groupAllowFrom` chỉ áp dụng cho tài khoản `default`.
  - Các tài khoản có tên kế thừa `channels.telegram.allowFrom` và `channels.telegram.groupAllowFrom` khi các giá trị cấp tài khoản không được đặt.
  - Các tài khoản có tên không kế thừa `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`.
- `channels.telegram.groups`: mặc định theo nhóm + danh sách cho phép (sử dụng `"*"` cho mặc định toàn cục).
  - `channels.telegram.groups.<id>.groupPolicy`: ghi đè theo nhóm cho groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: mặc định cổng mention.
  - `channels.telegram.groups.<id>.skills`: bộ lọc skill (bỏ qua = tất cả skills, rỗng = không có).
  - `channels.telegram.groups.<id>.allowFrom`: ghi đè danh sách cho phép người gửi theo nhóm.
  - `channels.telegram.groups.<id>.systemPrompt`: lời nhắc hệ thống bổ sung cho nhóm.
  - `channels.telegram.groups.<id>.enabled`: vô hiệu hóa nhóm khi `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: ghi đè theo chủ đề (các trường giống như nhóm).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: ghi đè theo chủ đề cho groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: ghi đè cổng mention theo chủ đề.
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (mặc định: allowlist).
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: ghi đè theo tài khoản.
- `channels.telegram.commands.nativeSkills`: bật/tắt lệnh skills gốc của Telegram.
- `channels.telegram.replyToMode`: `off | first | all` (mặc định: `off`).
- `channels.telegram.textChunkLimit`: kích thước khối gửi đi (ký tự).
- `channels.telegram.chunkMode`: `length` (mặc định) hoặc `newline` để tách trên dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- `channels.telegram.linkPreview`: bật/tắt xem trước liên kết cho tin nhắn gửi đi (mặc định: true).
- `channels.telegram.streaming`: `off | partial | block | progress` (xem trước luồng trực tiếp; mặc định: `off`; `progress` ánh xạ tới `partial`; `block` là chế độ tương thích xem trước cũ).
- `channels.telegram.mediaMaxMb`: giới hạn tải xuống/xử lý phương tiện Telegram đến (MB).
- `channels.telegram.retry`: chính sách thử lại cho trình trợ giúp gửi Telegram (CLI/tools/actions) trên lỗi API gửi đi có thể khôi phục (số lần thử, minDelayMs, maxDelayMs, jitter).
- `channels.telegram.network.autoSelectFamily`: ghi đè Node autoSelectFamily (true=bật, false=tắt). Mặc định được bật trên Node 22+, với WSL2 mặc định tắt.
- `channels.telegram.network.dnsResultOrder`: ghi đè thứ tự kết quả DNS (`ipv4first` hoặc `verbatim`). Mặc định là `ipv4first` trên Node 22+.
- `channels.telegram.proxy`: URL proxy cho các cuộc gọi Bot API (SOCKS/HTTP).
- `channels.telegram.webhookUrl`: bật chế độ webhook (yêu cầu `channels.telegram.webhookSecret`).
- `channels.telegram.webhookSecret`: bí mật webhook (bắt buộc khi webhookUrl được đặt).
- `channels.telegram.webhookPath`: đường dẫn webhook cục bộ (mặc định `/telegram-webhook`).
- `channels.telegram.webhookHost`: host bind webhook cục bộ (mặc định `127.0.0.1`).
- `channels.telegram.webhookPort`: cổng bind webhook cục bộ (mặc định `8787`).
- `channels.telegram.actions.reactions`: kiểm soát phản ứng công cụ Telegram.
- `channels.telegram.actions.sendMessage`: kiểm soát gửi tin nhắn công cụ Telegram.
- `channels.telegram.actions.deleteMessage`: kiểm soát xóa tin nhắn công cụ Telegram.
- `channels.telegram.actions.sticker`: kiểm soát hành động sticker Telegram — gửi và tìm kiếm (mặc định: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — kiểm soát phản ứng nào kích hoạt sự kiện hệ thống (mặc định: `own` khi không được đặt).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — kiểm soát khả năng phản ứng của agent (mặc định: `minimal` khi không được đặt).

- [Tham chiếu cấu hình - Telegram](/gateway/configuration-reference#telegram)

Các trường tín hiệu cao đặc thù của Telegram:

- khởi động/xác thực: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- kiểm soát truy cập: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- lệnh/menu: `commands.native`, `commands.nativeSkills`, `customCommands`
- luồng/trả lời: `replyToMode`
- truyền phát: `streaming` (xem trước), `blockStreaming`
- định dạng/gửi: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- phương tiện/mạng: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- hành động/khả năng: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- phản ứng: `reactionNotifications`, `reactionLevel`
- ghi/lịch sử: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
## Liên quan

- [Ghép nối](/channels/pairing)
- [Định tuyến kênh](/channels/channel-routing)
- [Định tuyến đa agent](/concepts/multi-agent)
- [Khắc phục sự cố](/channels/troubleshooting)