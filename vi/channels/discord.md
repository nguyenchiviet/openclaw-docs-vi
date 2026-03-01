---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình bot Discord'
read_when:
  - Đang làm việc trên các tính năng kênh Discord
title: Discord
x-i18n:
  source_path: channels\discord.md
  source_hash: 6b7bea74be40ce6b9b2aed681f5bf6179ebc1fcbf0d395838d79d2fb6415e89b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:17:25.618Z'
---

# Discord (Bot API)

Trạng thái: sẵn sàng cho tin nhắn riêng và kênh guild thông qua gateway chính thức của Discord.

<CardGroup cols={3}>
  <Card title="Ghép nối" icon="link" href="/channels/pairing">
    Tin nhắn riêng Discord mặc định sử dụng chế độ ghép nối.
  </Card>
  <Card title="Lệnh slash" icon="terminal" href="/tools/slash-commands">
    Hành vi lệnh gốc và danh mục lệnh.
  </Card>
  <Card title="Khắc phục sự cố kênh" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán và quy trình sửa chữa liên kênh.
  </Card>
</CardGroup>
## Thiết lập nhanh

Bạn sẽ cần tạo một ứng dụng mới với bot, thêm bot vào máy chủ của bạn và ghép nối nó với OpenClaw. Chúng tôi khuyến nghị thêm bot của bạn vào máy chủ riêng của bạn. Nếu bạn chưa có, [tạo một máy chủ trước](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (chọn **Create My Own > For me and my friends**).

<Steps>
  <Step title="Tạo ứng dụng Discord và bot">
    Truy cập [Discord Developer Portal](https://discord.com/developers/applications) và nhấp **New Application**. Đặt tên nó như "OpenClaw".

    Nhấp **Bot** trên thanh bên. Đặt **Username** thành tên bạn gọi agent OpenClaw của mình.

  </Step>

  <Step title="Bật privileged intents">
    Vẫn trên trang **Bot**, cuộn xuống **Privileged Gateway Intents** và bật:

    - **Message Content Intent** (bắt buộc)
    - **Server Members Intent** (khuyến nghị; bắt buộc cho danh sách cho phép vai trò và khớp tên với ID)
    - **Presence Intent** (tùy chọn; chỉ cần cho cập nhật trạng thái)

  </Step>

  <Step title="Sao chép token bot của bạn">
    Cuộn lên trên trang **Bot** và nhấp **Reset Token**.

    <Note>
    Mặc dù có tên như vậy, điều này tạo ra token đầu tiên của bạn — không có gì đang bị "reset."
    </Note>

    Sao chép token và lưu nó ở đâu đó. Đây là **Bot Token** của bạn và bạn sẽ cần nó ngay sau đây.

  </Step>

  <Step title="Tạo URL mời và thêm bot vào máy chủ của bạn">
    Nhấp **OAuth2** trên thanh bên. Bạn sẽ tạo URL mời với các quyền phù hợp để thêm bot vào máy chủ của bạn.

    Cuộn xuống **OAuth2 URL Generator** và bật:

    - `bot`
    - `applications.commands`

    Một phần **Bot Permissions** sẽ xuất hiện bên dưới. Bật:

    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (tùy chọn)

    Sao chép URL được tạo ở cuối, dán nó vào trình duyệt của bạn, chọn máy chủ của bạn và nhấp **Continue** để kết nối. Bây giờ bạn sẽ thấy bot của mình trong máy chủ Discord.

  </Step>

  <Step title="Bật Developer Mode và thu thập ID của bạn">
    Quay lại ứng dụng Discord, bạn cần bật Developer Mode để có thể sao chép các ID nội bộ.

    1. Nhấp **User Settings** (biểu tượng bánh răng bên cạnh avatar của bạn) → **Advanced** → bật **Developer Mode**
    2. Nhấp chuột phải vào **biểu tượng máy chủ** của bạn trên thanh bên → **Copy Server ID**
    3. Nhấp chuột phải vào **avatar của chính bạn** → **Copy User ID**

  </Step>
Lưu **Server ID** và **User ID** cùng với Bot Token của bạn — bạn sẽ gửi cả ba thông tin này cho OpenClaw ở bước tiếp theo.

  </Step>

  <Step title="Cho phép tin nhắn riêng từ thành viên server">
    Để ghép nối hoạt động, Discord cần cho phép bot của bạn gửi tin nhắn riêng cho bạn. Nhấp chuột phải vào **biểu tượng server** → **Privacy Settings** → bật **Direct Messages**.

    Điều này cho phép các thành viên server (bao gồm bot) gửi tin nhắn riêng cho bạn. Giữ tính năng này được bật nếu bạn muốn sử dụng tin nhắn riêng Discord với OpenClaw. Nếu bạn chỉ định sử dụng các kênh guild, bạn có thể tắt tin nhắn riêng sau khi ghép nối.

  </Step>

  <Step title="Bước 0: Đặt token bot của bạn một cách an toàn (không gửi nó trong chat)">
    Token bot Discord của bạn là thông tin bí mật (như mật khẩu). Đặt nó trên máy chạy OpenClaw trước khi nhắn tin cho agent của bạn.

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

    If OpenClaw is already running as a background service, use `openclaw gateway restart` instead.

  </Step>

  <Step title="Configure OpenClaw and pair">

    <Tabs>
      <Tab title="Ask your agent">
        Chat with your OpenClaw agent on any existing channel (e.g. Telegram) and tell it. If Discord is your first channel, use the CLI / config tab instead.

        > "I already set my Discord bot token in config. Please finish Discord setup with User ID `<user_id>` and Server ID `<server_id>`."
      </Tab>
      <Tab title="CLI / config">
        If you prefer file-based config, set:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

        Env fallback for the default account:

```bash
DISCORD_BOT_TOKEN=...
```

      </Tab>
    </Tabs>

  </Step>

  <Step title="Phê duyệt ghép nối tin nhắn riêng đầu tiên">
    Đợi cho đến khi gateway đang chạy, sau đó gửi tin nhắn riêng cho bot của bạn trong Discord. Nó sẽ phản hồi với mã ghép nối.
<Tabs>
  <Tab title="Hỏi agent của bạn">
    Gửi mã ghép nối đến agent của bạn trên kênh hiện tại:

    > "Phê duyệt mã ghép nối Discord này: `<CODE>`"
  </Tab>
  <Tab title="CLI">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

      </Tab>
    </Tabs>

    Pairing codes expire after 1 hour.

    You should now be able to chat with your agent in Discord via DM.

  </Step>
</Steps>

<Note>
Token resolution is account-aware. Config token values win over env fallback. `DISCORD_BOT_TOKEN` chỉ được sử dụng cho tài khoản mặc định.
</Note>
## Khuyến nghị: Thiết lập không gian làm việc guild

Khi tin nhắn riêng đã hoạt động, bạn có thể thiết lập máy chủ Discord của mình như một không gian làm việc đầy đủ, nơi mỗi kênh có phiên agent riêng với ngữ cảnh riêng. Điều này được khuyến nghị cho các máy chủ riêng tư chỉ có bạn và bot của bạn.

<Steps>
  <Step title="Thêm máy chủ của bạn vào danh sách cho phép guild">
    Điều này cho phép agent của bạn phản hồi trong bất kỳ kênh nào trên máy chủ của bạn, không chỉ tin nhắn riêng.

    <Tabs>
      <Tab title="Hỏi agent của bạn">
        > "Thêm Discord Server ID `<server_id>` của tôi vào danh sách cho phép guild"
      </Tab>
      <Tab title="Cấu hình">

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

      </Tab>
    </Tabs>

  </Step>

  <Step title="Allow responses without @mention">
    By default, your agent only responds in guild channels when @mentioned. For a private server, you probably want it to respond to every message.

    <Tabs>
      <Tab title="Ask your agent">
        > "Allow my agent to respond on this server without having to be @mentioned"
      </Tab>
      <Tab title="Config">
        Set `requireMention: false` in your guild config:

```json5
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

      </Tab>
</Tabs>

  </Step>

  <Step title="Lập kế hoạch cho bộ nhớ trong các kênh guild">
    Theo mặc định, bộ nhớ dài hạn (MEMORY.md) chỉ tải trong các phiên tin nhắn riêng. Các kênh guild không tự động tải MEMORY.md.

    <Tabs>
      <Tab title="Hỏi agent của bạn">
        > "Khi tôi đặt câu hỏi trong các kênh Discord, hãy sử dụng memory_search hoặc memory_get nếu bạn cần ngữ cảnh dài hạn từ MEMORY.md."
      </Tab>
      <Tab title="Thủ công">
        Nếu bạn cần ngữ cảnh chia sẻ trong mọi kênh, hãy đặt các hướng dẫn ổn định trong `AGENTS.md` hoặc `USER.md` (chúng được chèn cho mọi phiên). Giữ các ghi chú dài hạn trong `MEMORY.md` và truy cập chúng theo yêu cầu với các công cụ bộ nhớ.
      </Tab>
    </Tabs>

  </Step>
</Steps>

Bây giờ hãy tạo một số kênh trên máy chủ Discord của bạn và bắt đầu trò chuyện. Agent của bạn có thể thấy tên kênh, và mỗi kênh có phiên riêng biệt — vì vậy bạn có thể thiết lập `#coding`, `#home`, `#research`, hoặc bất cứ thứ gì phù hợp với quy trình làm việc của bạn.
## Mô hình thời gian chạy

- Gateway sở hữu kết nối Discord.
- Định tuyến phản hồi là xác định: Discord đến sẽ phản hồi lại Discord.
- Theo mặc định (`session.dmScope=main`), các cuộc trò chuyện trực tiếp chia sẻ phiên chính của agent (`agent:main:main`).
- Các kênh guild là các khóa phiên cô lập (`agent:<agentId>:discord:channel:<channelId>`).
- Tin nhắn riêng nhóm được bỏ qua theo mặc định (`channels.discord.dm.groupEnabled=false`).
- Các lệnh slash gốc chạy trong các phiên lệnh cô lập (`agent:<agentId>:discord:slash:<userId>`), trong khi vẫn mang `CommandTargetSessionKey` đến phiên cuộc trò chuyện được định tuyến.
## Kênh diễn đàn

Các kênh diễn đàn và phương tiện Discord chỉ chấp nhận bài đăng dạng chủ đề. OpenClaw hỗ trợ hai cách để tạo chúng:

- Gửi tin nhắn đến kênh cha của diễn đàn (`channel:<forumId>`) để tự động tạo chủ đề. Tiêu đề chủ đề sử dụng dòng không trống đầu tiên trong tin nhắn của bạn.
- Sử dụng `openclaw message thread create` để tạo chủ đề trực tiếp. Không truyền `--message-id` cho các kênh diễn đàn.

Ví dụ: gửi đến kênh cha của diễn đàn để tạo chủ đề

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

Example: create a forum thread explicitly

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

Forum parents do not accept Discord components. If you need components, send to the thread itself (`channel:<threadId>`).
## Thành phần tương tác

OpenClaw hỗ trợ các container thành phần Discord v2 cho tin nhắn agent. Sử dụng công cụ tin nhắn với payload `components`. Kết quả tương tác được chuyển trở lại agent dưới dạng tin nhắn đến thông thường và tuân theo các cài đặt Discord `replyToMode` hiện có.

Các khối được hỗ trợ:

- `text`, `section`, `separator`, `actions`, `media-gallery`, `file`
- Hàng hành động cho phép tối đa 5 nút hoặc một menu chọn duy nhất
- Các loại chọn: `string`, `user`, `role`, `mentionable`, `channel`

Theo mặc định, các thành phần chỉ sử dụng một lần. Đặt `components.reusable=true` để cho phép các nút, menu chọn và biểu mẫu được sử dụng nhiều lần cho đến khi hết hạn.

Để hạn chế ai có thể nhấp vào nút, đặt `allowedUsers` trên nút đó (ID người dùng Discord, thẻ, hoặc `*`). Khi được cấu hình, người dùng không khớp sẽ nhận được thông báo từ chối tạm thời.

Các lệnh slash `/model` và `/models` mở một bộ chọn mô hình tương tác với menu thả xuống nhà cung cấp và mô hình cộng với bước Gửi. Phản hồi của bộ chọn là tạm thời và chỉ người dùng gọi lệnh mới có thể sử dụng nó.

Tệp đính kèm:

- Các khối `file` phải trỏ đến một tham chiếu tệp đính kèm (`attachment://<filename>`)
- Cung cấp tệp đính kèm qua `media`/`path`/`filePath` (tệp đơn); sử dụng `media-gallery` cho nhiều tệp
- Sử dụng `filename` để ghi đè tên tải lên khi nó cần khớp với tham chiếu tệp đính kèm

Biểu mẫu modal:

- Thêm `components.modal` với tối đa 5 trường
- Các loại trường: `text`, `checkbox`, `radio`, `select`, `role-select`, `user-select`
- OpenClaw tự động thêm nút kích hoạt

Ví dụ:

```json5
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```
## Kiểm soát truy cập và định tuyến

<Tabs>
  <Tab title="Chính sách tin nhắn riêng">
    `channels.discord.dmPolicy` kiểm soát quyền truy cập tin nhắn riêng (cũ: `channels.discord.dm.policy`):

    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `channels.discord.allowFrom` bao gồm `"*"`; cũ: `channels.discord.dm.allowFrom`)
    - `disabled`

    Nếu chính sách tin nhắn riêng không mở, người dùng không xác định sẽ bị chặn (hoặc được nhắc ghép nối trong chế độ `pairing`).

    Thứ tự ưu tiên đa tài khoản:

    - `channels.discord.accounts.default.allowFrom` chỉ áp dụng cho tài khoản `default`.
    - Các tài khoản có tên kế thừa `channels.discord.allowFrom` khi `allowFrom` của chúng không được đặt.
    - Các tài khoản có tên không kế thừa `channels.discord.accounts.default.allowFrom`.

    Định dạng đích tin nhắn riêng để gửi:

    - `user:<id>`
    - `<@id>` mention

    ID số thuần túy có tính mơ hồ và bị từ chối trừ khi có loại đích người dùng/kênh rõ ràng được cung cấp.

  </Tab>

  <Tab title="Chính sách guild">
    Xử lý guild được kiểm soát bởi `channels.discord.groupPolicy`:

    - `open`
    - `allowlist`
    - `disabled`

    Cơ sở bảo mật khi `channels.discord` tồn tại là `allowlist`.

    Hành vi `allowlist`:

    - guild phải khớp với `channels.discord.guilds` (`id` được ưu tiên, slug được chấp nhận)
    - danh sách cho phép người gửi tùy chọn: `users` (khuyến nghị ID ổn định) và `roles` (chỉ ID vai trò); nếu một trong hai được cấu hình, người gửi được cho phép khi họ khớp với `users` HOẶC `roles`
    - khớp tên/thẻ trực tiếp bị vô hiệu hóa theo mặc định; chỉ bật `channels.discord.dangerouslyAllowNameMatching: true` như chế độ tương thích khẩn cấp
    - tên/thẻ được hỗ trợ cho `users`, nhưng ID an toàn hơn; `openclaw security audit` cảnh báo khi các mục tên/thẻ được sử dụng
    - nếu một guild có `channels` được cấu hình, các kênh không được liệt kê sẽ bị từ chối
    - nếu một guild không có khối `channels`, tất cả các kênh trong guild được cho phép đó sẽ được phép

    Ví dụ:

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```
Nếu bạn chỉ đặt `DISCORD_BOT_TOKEN` và không tạo khối `channels.discord`, fallback thời gian chạy là `groupPolicy="allowlist"` (với cảnh báo trong logs), ngay cả khi `channels.defaults.groupPolicy` là `open`.

  </Tab>

  <Tab title="Mentions và tin nhắn riêng nhóm">
    Tin nhắn guild được kiểm soát bằng mention theo mặc định.

    Phát hiện mention bao gồm:

    - mention bot rõ ràng
    - các mẫu mention đã cấu hình (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - hành vi reply-to-bot ngầm định trong các trường hợp được hỗ trợ

    `requireMention` được cấu hình theo guild/kênh (`channels.discord.guilds...`).

    Tin nhắn riêng nhóm:

    - mặc định: bị bỏ qua (`dm.groupEnabled=false`)
    - danh sách cho phép tùy chọn qua `dm.groupChannels` (ID kênh hoặc slugs)

  </Tab>
</Tabs>

### Định tuyến agent dựa trên vai trò

Sử dụng `bindings[].match.roles` để định tuyến thành viên Discord guild đến các agent khác nhau theo ID vai trò. Các ràng buộc dựa trên vai trò chỉ chấp nhận ID vai trò và được đánh giá sau các ràng buộc peer hoặc parent-peer và trước các ràng buộc chỉ guild. Nếu một ràng buộc cũng đặt các trường khớp khác (ví dụ `peer` + `guildId` + `roles`), tất cả các trường đã cấu hình phải khớp.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```
## Thiết lập Developer Portal

<AccordionGroup>
  <Accordion title="Tạo ứng dụng và bot">

    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Sao chép token bot

  </Accordion>

  <Accordion title="Quyền đặc biệt">
    Trong **Bot -> Privileged Gateway Intents**, bật:

    - Message Content Intent
    - Server Members Intent (khuyến nghị)

    Presence intent là tùy chọn và chỉ cần thiết nếu bạn muốn nhận cập nhật trạng thái. Thiết lập trạng thái bot (`setPresence`) không yêu cầu bật cập nhật trạng thái cho thành viên.

  </Accordion>

  <Accordion title="Phạm vi OAuth và quyền cơ bản">
    Trình tạo URL OAuth:

    - phạm vi: `bot`, `applications.commands`

    Quyền cơ bản thông thường:

    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (tùy chọn)

    Tránh `Administrator` trừ khi thực sự cần thiết.

  </Accordion>

  <Accordion title="Sao chép ID">
    Bật Discord Developer Mode, sau đó sao chép:

    - ID máy chủ
    - ID kênh
    - ID người dùng

    Ưu tiên ID số trong cấu hình OpenClaw để kiểm tra và thăm dò đáng tin cậy.

  </Accordion>
</AccordionGroup>
## Lệnh gốc và xác thực lệnh

- `commands.native` mặc định là `"auto"` và được bật cho Discord.
- Ghi đè theo kênh: `channels.discord.commands.native`.
- `commands.native=false` xóa rõ ràng các lệnh gốc Discord đã đăng ký trước đó.
- Xác thực lệnh gốc sử dụng cùng danh sách cho phép/chính sách Discord như xử lý tin nhắn bình thường.
- Các lệnh vẫn có thể hiển thị trong giao diện Discord cho người dùng không được ủy quyền; việc thực thi vẫn thực thi xác thực OpenClaw và trả về "không được ủy quyền".

Xem [Lệnh slash](/tools/slash-commands) để biết danh mục lệnh và hành vi.

Cài đặt lệnh slash mặc định:

- `ephemeral: true`
## Chi tiết tính năng

<AccordionGroup>
  <Accordion title="Thẻ trả lời và trả lời gốc">
    Discord hỗ trợ thẻ trả lời trong đầu ra của agent:

    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`

    Được điều khiển bởi `channels.discord.replyToMode`:

    - `off` (mặc định)
    - `first`
    - `all`

    Lưu ý: `off` vô hiệu hóa luồng trả lời ngầm định. Các thẻ `[[reply_to_*]]` rõ ràng vẫn được tôn trọng.

    ID tin nhắn được hiển thị trong ngữ cảnh/lịch sử để agent có thể nhắm mục tiêu các tin nhắn cụ thể.

  </Accordion>

  <Accordion title="Xem trước luồng trực tiếp">
    OpenClaw có thể truyền phát bản nháp trả lời bằng cách gửi tin nhắn tạm thời và chỉnh sửa khi văn bản đến.

    - `channels.discord.streaming` điều khiển truyền phát xem trước (`off` | `partial` | `block` | `progress`, mặc định: `off`).
    - `progress` được chấp nhận để đảm bảo tính nhất quán giữa các kênh và ánh xạ tới `partial` trên Discord.
    - `channels.discord.streamMode` là bí danh cũ và được tự động di chuyển.
    - `partial` chỉnh sửa một tin nhắn xem trước duy nhất khi token đến.
    - `block` phát ra các khối có kích thước bản nháp (sử dụng `draftChunk` để điều chỉnh kích thước và điểm ngắt).

    Ví dụ:

```json5
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

    `block` mode chunking defaults (clamped to `channels.discord.textChunkLimit`):

```json5
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

    Truyền phát xem trước chỉ dành cho văn bản; trả lời phương tiện sẽ quay về phương thức giao hàng bình thường.
Lưu ý: truyền phát xem trước tách biệt với truyền phát theo khối. Khi truyền phát theo khối được bật rõ ràng cho Discord, OpenClaw bỏ qua luồng xem trước để tránh truyền phát kép.

  </Accordion>

  <Accordion title="Lịch sử, ngữ cảnh và hành vi thread">
    Ngữ cảnh lịch sử guild:

    - `channels.discord.historyLimit` mặc định `20`
    - dự phòng: `messages.groupChat.historyLimit`
    - `0` vô hiệu hóa

    Điều khiển lịch sử tin nhắn riêng:

    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`

    Hành vi thread:

    - Các thread Discord được định tuyến như phiên kênh
    - metadata thread cha có thể được sử dụng để liên kết phiên cha
    - cấu hình thread kế thừa cấu hình kênh cha trừ khi có mục cụ thể cho thread

    Chủ đề kênh được chèn như ngữ cảnh **không đáng tin cậy** (không phải như system prompt).

  </Accordion>

  <Accordion title="Phiên gắn thread cho subagent">
    Discord có thể gắn một thread với mục tiêu phiên để các tin nhắn tiếp theo trong thread đó tiếp tục định tuyến đến cùng một phiên (bao gồm phiên subagent).

    Lệnh:

    - `/focus <target>` gắn thread hiện tại/mới với mục tiêu subagent/phiên
    - `/unfocus` xóa ràng buộc thread hiện tại
    - `/agents` hiển thị các lần chạy đang hoạt động và trạng thái ràng buộc
    - `/session idle <duration|off>` kiểm tra/cập nhật tự động bỏ tập trung khi không hoạt động cho ràng buộc đã tập trung
    - `/session max-age <duration|off>` kiểm tra/cập nhật tuổi tối đa cứng cho ràng buộc đã tập trung

    Cấu hình:

```json5
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // opt-in
      },
    },
  },
}
```
Ghi chú:

- `session.threadBindings.*` thiết lập các giá trị mặc định toàn cục.
- `channels.discord.threadBindings.*` ghi đè hành vi Discord.
- `spawnSubagentSessions` phải là true để tự động tạo/liên kết thread cho `sessions_spawn({ thread: true })`.
- `spawnAcpSessions` phải là true để tự động tạo/liên kết thread cho ACP (`/acp spawn ... --thread ...` hoặc `sessions_spawn({ runtime: "acp", thread: true })`).
- Nếu liên kết thread bị vô hiệu hóa cho một tài khoản, `/focus` và các thao tác liên kết thread liên quan sẽ không khả dụng.

Xem [Sub-agents](/tools/subagents), [ACP Agents](/tools/acp-agents), và [Configuration Reference](/gateway/configuration-reference).

</Accordion>

<Accordion title="Thông báo phản ứng">
Chế độ thông báo phản ứng theo guild:

- `off`
- `own` (mặc định)
- `all`
- `allowlist` (sử dụng `guilds.<id>.users`)

Các sự kiện phản ứng được chuyển đổi thành sự kiện hệ thống và đính kèm vào phiên Discord được định tuyến.

</Accordion>

<Accordion title="Phản ứng xác nhận">
`ackReaction` gửi một emoji xác nhận trong khi OpenClaw đang xử lý tin nhắn đến.

Thứ tự giải quyết:

- `channels.discord.accounts.<accountId>.ackReaction`
- `channels.discord.ackReaction`
- `messages.ackReaction`
- dự phòng emoji danh tính agent (`agents.list[].identity.emoji`, nếu không có thì "👀")

Ghi chú:

- Discord chấp nhận emoji unicode hoặc tên emoji tùy chỉnh.
- Sử dụng `""` để vô hiệu hóa phản ứng cho một kênh hoặc tài khoản.

</Accordion>

<Accordion title="Ghi cấu hình">
Ghi cấu hình được khởi tạo từ kênh được bật theo mặc định.

Điều này ảnh hưởng đến luồng `/config set|unset` (khi các tính năng lệnh được bật).

Vô hiệu hóa:

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

</Accordion>
<Accordion title="Cấu hình proxy Gateway">
    Định tuyến lưu lượng WebSocket của Discord gateway và các tra cứu REST khởi động (ID ứng dụng + phân giải danh sách cho phép) thông qua proxy HTTP(S) với `channels.discord.proxy`.

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    Per-account override:

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  </Accordion>

  <Accordion title="PluralKit support">
    Enable PluralKit resolution to map proxied messages to system member identity:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; needed for private systems
      },
    },
  },
}
```

    Notes:

    - allowlists can use `pk:<memberId>`
    - member display names are matched by name/slug only when `channels.discord.dangerouslyAllowNameMatching: true`
    - lookups use original message ID and are time-window constrained
    - if lookup fails, proxied messages are treated as bot messages and dropped unless `allowBots=true`

  </Accordion>

  <Accordion title="Cấu hình trạng thái hiện diện">
    Cập nhật trạng thái hiện diện chỉ được áp dụng khi bạn đặt trường trạng thái hoặc hoạt động.
Ví dụ chỉ trạng thái:

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    Activity example (custom status is the default activity type):

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    Streaming example:

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    Activity type map:

    - 0: Playing
    - 1: Streaming (requires `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (uses the activity text as the status state; emoji is optional)
    - 5: Competing

  </Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord supports button-based exec approvals in DMs and can optionally post approval prompts in the originating channel.

    Config path:

    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, default: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
Khi `target` là `channel` hoặc `both`, lời nhắc phê duyệt sẽ hiển thị trong kênh. Chỉ những người phê duyệt được cấu hình mới có thể sử dụng các nút; người dùng khác sẽ nhận được thông báo từ chối tạm thời. Lời nhắc phê duyệt bao gồm văn bản lệnh, vì vậy chỉ nên bật giao hàng kênh trong các kênh đáng tin cậy. Nếu không thể xác định ID kênh từ khóa phiên, OpenClaw sẽ chuyển về giao hàng tin nhắn riêng.

Nếu phê duyệt thất bại với ID phê duyệt không xác định, hãy xác minh danh sách người phê duyệt và việc kích hoạt tính năng.

Tài liệu liên quan: [Phê duyệt thực thi](/tools/exec-approvals)

  </Accordion>
</AccordionGroup>
## Công cụ và cổng hành động

Các hành động tin nhắn Discord bao gồm nhắn tin, quản trị kênh, kiểm duyệt, trạng thái hiện diện và các hành động siêu dữ liệu.

Ví dụ cốt lõi:

- nhắn tin: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- phản ứng: `react`, `reactions`, `emojiList`
- kiểm duyệt: `timeout`, `kick`, `ban`
- trạng thái hiện diện: `setPresence`

Cổng hành động nằm dưới `channels.discord.actions.*`.

Hành vi cổng mặc định:

| Nhóm hành động                                                                                                                                                             | Mặc định |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | bật      |
| roles                                                                                                                                                                    | tắt      |
| moderation                                                                                                                                                               | tắt      |
| presence                                                                                                                                                                 | tắt      |
## Components v2 UI

OpenClaw sử dụng Discord components v2 cho việc phê duyệt thực thi và các dấu hiệu ngữ cảnh chéo. Các hành động tin nhắn Discord cũng có thể chấp nhận `components` cho giao diện người dùng tùy chỉnh (nâng cao; yêu cầu các phiên bản thành phần Carbon), trong khi `embeds` cũ vẫn có sẵn nhưng không được khuyến nghị.

- `channels.discord.ui.components.accentColor` đặt màu nhấn được sử dụng bởi các container thành phần Discord (hex).
- Đặt cho mỗi tài khoản với `channels.discord.accounts.<id>.ui.components.accentColor`.
- `embeds` bị bỏ qua khi có components v2.

Ví dụ:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```
## Kênh thoại

OpenClaw có thể tham gia các kênh thoại Discord để có cuộc trò chuyện liên tục theo thời gian thực. Điều này tách biệt với các tệp đính kèm tin nhắn thoại.

Yêu cầu:

- Bật lệnh gốc (`commands.native` hoặc `channels.discord.commands.native`).
- Cấu hình `channels.discord.voice`.
- Bot cần quyền Connect + Speak trong kênh thoại đích.

Sử dụng lệnh gốc chỉ dành cho Discord `/vc join|leave|status` để điều khiển phiên. Lệnh này sử dụng agent mặc định của tài khoản và tuân theo các quy tắc danh sách cho phép và chính sách nhóm giống như các lệnh Discord khác.

Ví dụ tự động tham gia:

```json5
{
  channels: {
    discord: {
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
    },
  },
}
```

Notes:

- `voice.tts` overrides `messages.tts` for voice playback only.
- Voice is enabled by default; set `channels.discord.voice.enabled=false` to disable it.
- `voice.daveEncryption` and `voice.decryptionFailureTolerance` pass through to `@discordjs/voice` join options.
- `@discordjs/voice` defaults are `daveEncryption=true` and `decryptionFailureTolerance=24` if unset.
- OpenClaw also watches receive decrypt failures and auto-recovers by leaving/rejoining the voice channel after repeated failures in a short window.
- If receive logs repeatedly show `DecryptionFailed(UnencryptedWhenPassthroughDisabled)`, this may be the upstream `@discordjs/voice` lỗi nhận được theo dõi trong [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419).
## Tin nhắn thoại

Tin nhắn thoại Discord hiển thị bản xem trước dạng sóng và yêu cầu âm thanh OGG/Opus cùng metadata. OpenClaw tự động tạo dạng sóng, nhưng cần có `ffmpeg` và `ffprobe` trên máy chủ gateway để kiểm tra và chuyển đổi tệp âm thanh.

Yêu cầu và ràng buộc:

- Cung cấp **đường dẫn tệp cục bộ** (URL sẽ bị từ chối).
- Bỏ qua nội dung văn bản (Discord không cho phép văn bản + tin nhắn thoại trong cùng một payload).
- Chấp nhận mọi định dạng âm thanh; OpenClaw chuyển đổi sang OGG/Opus khi cần thiết.

Ví dụ:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```
## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="Sử dụng intents không được phép hoặc bot không thấy tin nhắn guild">

    - bật Message Content Intent
    - bật Server Members Intent khi bạn phụ thuộc vào việc phân giải user/member
    - khởi động lại gateway sau khi thay đổi intents

  </Accordion>

  <Accordion title="Tin nhắn guild bị chặn bất ngờ">

    - xác minh `groupPolicy`
    - xác minh danh sách cho phép guild trong `channels.discord.guilds`
    - nếu bản đồ `channels` guild tồn tại, chỉ các kênh được liệt kê mới được phép
    - xác minh hành vi `requireMention` và các mẫu mention

    Kiểm tra hữu ích:

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  </Accordion>

  <Accordion title="Require mention false but still blocked">
    Common causes:

    - `groupPolicy="allowlist"` without matching guild/channel allowlist
    - `requireMention` configured in the wrong place (must be under `channels.discord.guilds` or channel entry)
    - sender blocked by guild/channel `users` allowlist

  </Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` permission checks only work for numeric channel IDs.

    If you use slug keys, runtime matching can still work, but probe cannot fully verify permissions.

  </Accordion>

  <Accordion title="DM and pairing issues">

    - DM disabled: `channels.discord.dm.enabled=false`
    - DM policy disabled: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    - awaiting pairing approval in `pairing` mode

  </Accordion>

  <Accordion title="Bot to bot loops">
    By default bot-authored messages are ignored.

    If you set `channels.discord.allowBots=true`, sử dụng quy tắc mention nghiêm ngặt và danh sách cho phép để tránh hành vi lặp.

  </Accordion>

  <Accordion title="Voice STT bị ngắt với DecryptionFailed(...)">
- giữ OpenClaw ở phiên bản hiện tại (`openclaw update`) để có logic khôi phục nhận giọng nói Discord
    - xác nhận `channels.discord.voice.daveEncryption=true` (mặc định)
    - bắt đầu từ `channels.discord.voice.decryptionFailureTolerance=24` (mặc định upstream) và chỉ điều chỉnh khi cần thiết
    - theo dõi logs để tìm:
      - `discord voice: DAVE decrypt failures detected`
      - `discord voice: repeated decrypt failures; attempting rejoin`
    - nếu lỗi vẫn tiếp tục sau khi tự động tham gia lại, thu thập logs và so sánh với [discord.js #11419](https://github.com/discordjs/discord.js/issues/11419)

  </Accordion>
</AccordionGroup>
## Tham chiếu cấu hình

Tham chiếu chính:

- [Tham chiếu cấu hình - Discord](/gateway/configuration-reference#discord)

Các trường Discord quan trọng:

- khởi động/xác thực: `enabled`, `token`, `accounts.*`, `allowBots`
- chính sách: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- lệnh: `commands.native`, `commands.useAccessGroups`, `configWrites`, `slashCommand.*`
- phản hồi/lịch sử: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- gửi tin: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- streaming: `streaming` (bí danh cũ: `streamMode`), `draftChunk`, `blockStreaming`, `blockStreamingCoalesce`
- media/thử lại: `mediaMaxMb`, `retry`
- hành động: `actions.*`
- trạng thái: `activity`, `status`, `activityType`, `activityUrl`
- giao diện: `ui.components.accentColor`
- tính năng: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`
## An toàn và vận hành

- Xử lý token bot như bí mật (`DISCORD_BOT_TOKEN` được ưu tiên trong môi trường có giám sát).
- Cấp quyền Discord tối thiểu cần thiết.
- Nếu trạng thái triển khai/lệnh bị cũ, khởi động lại gateway và kiểm tra lại với `openclaw channels status --probe`.
## Liên quan

- [Ghép nối](/channels/pairing)
- [Định tuyến kênh](/channels/channel-routing)
- [Định tuyến đa agent](/concepts/multi-agent)
- [Khắc phục sự cố](/channels/troubleshooting)
- [Lệnh slash](/tools/slash-commands)