---
summary: 'Hỗ trợ kênh WhatsApp, kiểm soát truy cập, hành vi gửi tin và các hoạt động'
read_when:
  - Đang làm việc trên hành vi kênh WhatsApp/web hoặc định tuyến hộp thư
title: WhatsApp
x-i18n:
  source_path: channels\whatsapp.md
  source_hash: 42d5364a0aa9041a1a584e814a4f578abbe292ec952ba2462d65b0fd3a943c35
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:30:37.507Z'
---

# WhatsApp (Kênh Web)

Trạng thái: sẵn sàng sản xuất qua WhatsApp Web (Baileys). Gateway sở hữu (các) phiên đã liên kết.

<CardGroup cols={3}>
  <Card title="Ghép nối" icon="link" href="/channels/pairing">
    Chính sách tin nhắn riêng mặc định là ghép nối cho người gửi không xác định.
  </Card>
  <Card title="Khắc phục sự cố kênh" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán liên kênh và sách hướng dẫn sửa chữa.
  </Card>
  <Card title="Cấu hình Gateway" icon="settings" href="/gateway/configuration">
    Các mẫu và ví dụ cấu hình kênh đầy đủ.
  </Card>
</CardGroup>
## Thiết lập nhanh

<Steps>
  <Step title="Cấu hình chính sách truy cập WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

  </Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    For a specific account:

```bash
openclaw channels login --channel whatsapp --account work
```

  </Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  </Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    Yêu cầu ghép nối sẽ hết hạn sau 1 giờ. Các yêu cầu đang chờ được giới hạn tối đa 3 yêu cầu mỗi kênh.

  </Step>
</Steps>

<Note>
OpenClaw khuyến nghị chạy WhatsApp trên một số điện thoại riêng biệt khi có thể. (Metadata kênh và quy trình thiết lập ban đầu được tối ưu hóa cho thiết lập đó, nhưng thiết lập số cá nhân cũng được hỗ trợ.)
</Note>
## Mô hình triển khai

<AccordionGroup>
  <Accordion title="Số điện thoại riêng biệt (khuyến nghị)">
    Đây là chế độ hoạt động sạch sẽ nhất:

    - danh tính WhatsApp riêng biệt cho OpenClaw
    - danh sách cho phép tin nhắn riêng và ranh giới định tuyến rõ ràng hơn
    - khả năng nhầm lẫn tự trò chuyện thấp hơn

    Mô hình chính sách tối thiểu:

    ```json5
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Personal-number fallback">
    Onboarding supports personal-number mode and writes a self-chat-friendly baseline:

    - `dmPolicy: "allowlist"`
    - `allowFrom` includes your personal number
    - `selfChatMode: true`

    In runtime, self-chat protections key off the linked self number and `allowFrom`.

  </Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    The messaging platform channel is WhatsApp Web-based (`Baileys`) trong kiến trúc kênh OpenClaw hiện tại.

    Không có kênh nhắn tin WhatsApp Twilio riêng biệt trong sổ đăng ký kênh trò chuyện tích hợp sẵn.

  </Accordion>
</AccordionGroup>
## Mô hình thời gian chạy

- Gateway sở hữu socket WhatsApp và vòng lặp kết nối lại.
- Gửi tin nhắn ra ngoài yêu cầu một listener WhatsApp hoạt động cho tài khoản đích.
- Các cuộc trò chuyện trạng thái và phát sóng bị bỏ qua (`@status`, `@broadcast`).
- Các cuộc trò chuyện trực tiếp sử dụng quy tắc phiên tin nhắn riêng (`session.dmScope`; mặc định `main` thu gọn tin nhắn riêng vào phiên chính của agent).
- Các phiên nhóm được cô lập (`agent:<agentId>:whatsapp:group:<jid>`).
## Kiểm soát truy cập và kích hoạt

<Tabs>
  <Tab title="Chính sách tin nhắn riêng">
    `channels.whatsapp.dmPolicy` kiểm soát quyền truy cập trò chuyện trực tiếp:

    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `allowFrom` phải bao gồm `"*"`)
    - `disabled`

    `allowFrom` chấp nhận số định dạng E.164 (được chuẩn hóa nội bộ).

    Ghi đè đa tài khoản: `channels.whatsapp.accounts.<id>.dmPolicy` (và `allowFrom`) có ưu tiên cao hơn các mặc định cấp kênh cho tài khoản đó.

    Chi tiết hành vi runtime:

    - các ghép nối được lưu trữ trong kho cho phép kênh và được hợp nhất với `allowFrom` đã cấu hình
    - nếu không có danh sách cho phép nào được cấu hình, số tự liên kết được cho phép theo mặc định
    - tin nhắn riêng `fromMe` gửi đi không bao giờ được tự động ghép nối

  </Tab>

  <Tab title="Chính sách nhóm + danh sách cho phép">
    Quyền truy cập nhóm có hai lớp:

    1. **Danh sách cho phép thành viên nhóm** (`channels.whatsapp.groups`)
       - nếu `groups` bị bỏ qua, tất cả nhóm đều đủ điều kiện
       - nếu `groups` có mặt, nó hoạt động như danh sách cho phép nhóm (`"*"` được cho phép)

    2. **Chính sách người gửi nhóm** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: bỏ qua danh sách cho phép người gửi
       - `allowlist`: người gửi phải khớp với `groupAllowFrom` (hoặc `*`)
       - `disabled`: chặn tất cả tin nhắn đến từ nhóm

    Dự phòng danh sách cho phép người gửi:

    - nếu `groupAllowFrom` không được đặt, runtime sẽ dự phòng về `allowFrom` khi có sẵn
    - danh sách cho phép người gửi được đánh giá trước khi kích hoạt mention/reply

    Lưu ý: nếu không có khối `channels.whatsapp` nào cả, dự phòng chính sách nhóm runtime là `allowlist` (với cảnh báo log), ngay cả khi `channels.defaults.groupPolicy` được đặt.

  </Tab>

  <Tab title="Mentions + /kích hoạt">
    Phản hồi nhóm yêu cầu mention theo mặc định.

    Phát hiện mention bao gồm:

    - mention WhatsApp rõ ràng về danh tính bot
    - các mẫu regex mention đã cấu hình (`agents.list[].groupChat.mentionPatterns`, dự phòng `messages.groupChat.mentionPatterns`)
    - phát hiện reply-to-bot ngầm định (người gửi reply khớp với danh tính bot)

    Lưu ý bảo mật:

    - quote/reply chỉ thỏa mãn cổng mention; nó **không** cấp quyền ủy quyền người gửi
    - với `groupPolicy: "allowlist"`, người gửi không có trong danh sách cho phép vẫn bị chặn ngay cả khi họ phản hồi tin nhắn của người dùng có trong danh sách cho phép

    Lệnh kích hoạt cấp phiên:
- `/activation mention`
    - `/activation always`

    `activation` cập nhật trạng thái phiên (không phải cấu hình toàn cục). Nó được kiểm soát bởi chủ sở hữu.

  </Tab>
</Tabs>
## Hành vi số cá nhân và tự trò chuyện

Khi số tự liên kết cũng có mặt trong `allowFrom`, các biện pháp bảo vệ tự trò chuyện WhatsApp sẽ kích hoạt:

- bỏ qua xác nhận đã đọc cho các lượt tự trò chuyện
- bỏ qua hành vi tự động kích hoạt mention-JID mà nếu không sẽ ping chính bạn
- nếu `messages.responsePrefix` không được đặt, các phản hồi tự trò chuyện mặc định là `[{identity.name}]` hoặc `[openclaw]`
## Chuẩn hóa tin nhắn và ngữ cảnh

<AccordionGroup>
  <Accordion title="Phong bì đến + ngữ cảnh trả lời">
    Tin nhắn WhatsApp đến được bao bọc trong phong bì đến chung.

    Nếu có trả lời trích dẫn, ngữ cảnh sẽ được thêm vào theo dạng này:

    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```

    Reply metadata fields are also populated when available (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).

  </Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    Media-only inbound messages are normalized with placeholders such as:

    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`

    Location and contact payloads are normalized into textual context before routing.

  </Accordion>

  <Accordion title="Pending group history injection">
    For groups, unprocessed messages can be buffered and injected as context when the bot is finally triggered.

    - default limit: `50`
    - config: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` disables

    Injection markers:

    - `[Tin nhắn trò chuyện kể từ lần trả lời cuối - để tham khảo]`
    - `[Tin nhắn hiện tại - hãy trả lời tin nhắn này]`

  </Accordion>

  <Accordion title="Read receipts">
    Read receipts are enabled by default for accepted inbound WhatsApp messages.

    Disable globally:

    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
Ghi đè theo tài khoản:

```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```

Cuộc trò chuyện với chính mình sẽ bỏ qua xác nhận đã đọc ngay cả khi được bật toàn cục.

  </Accordion>
</AccordionGroup>
## Giao hàng, phân đoạn và phương tiện

<AccordionGroup>
  <Accordion title="Phân đoạn văn bản">
    - giới hạn đoạn mặc định: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - chế độ `newline` ưu tiên ranh giới đoạn văn (dòng trống), sau đó quay lại phân đoạn an toàn theo độ dài
  </Accordion>

  <Accordion title="Hành vi phương tiện gửi đi">
    - hỗ trợ tải trọng hình ảnh, video, âm thanh (ghi chú thoại PTT) và tài liệu
    - `audio/ogg` được viết lại thành `audio/ogg; codecs=opus` để tương thích với ghi chú thoại
    - phát lại GIF động được hỗ trợ qua `gifPlayback: true` khi gửi video
    - chú thích được áp dụng cho mục phương tiện đầu tiên khi gửi tải trọng phản hồi đa phương tiện
    - nguồn phương tiện có thể là HTTP(S), `file://`, hoặc đường dẫn cục bộ
  </Accordion>

  <Accordion title="Giới hạn kích thước phương tiện và hành vi dự phòng">
    - giới hạn lưu phương tiện đến: `channels.whatsapp.mediaMaxMb` (mặc định `50`)
    - giới hạn phương tiện gửi đi cho phản hồi tự động: `agents.defaults.mediaMaxMb` (mặc định `5MB`)
    - hình ảnh được tối ưu hóa tự động (thay đổi kích thước/quét chất lượng) để phù hợp với giới hạn
    - khi gửi phương tiện thất bại, dự phòng mục đầu tiên sẽ gửi cảnh báo văn bản thay vì bỏ qua phản hồi một cách im lặng
  </Accordion>
</AccordionGroup>
## Phản ứng xác nhận

WhatsApp hỗ trợ phản ứng xác nhận ngay lập tức khi nhận tin nhắn đến thông qua `channels.whatsapp.ackReaction`.

```json5
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Behavior notes:

- sent immediately after inbound is accepted (pre-reply)
- failures are logged but do not block normal reply delivery
- group mode `mentions` reacts on mention-triggered turns; group activation `always` acts as bypass for this check
- WhatsApp uses `channels.whatsapp.ackReaction` (legacy `messages.ackReaction` không được sử dụng ở đây)
## Đa tài khoản và thông tin xác thực

<AccordionGroup>
  <Accordion title="Lựa chọn tài khoản và mặc định">
    - id tài khoản đến từ `channels.whatsapp.accounts`
    - lựa chọn tài khoản mặc định: `default` nếu có, nếu không thì id tài khoản được cấu hình đầu tiên (đã sắp xếp)
    - id tài khoản được chuẩn hóa nội bộ để tra cứu
  </Accordion>

  <Accordion title="Đường dẫn thông tin xác thực và tương thích cũ">
    - đường dẫn xác thực hiện tại: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - tệp sao lưu: `creds.json.bak`
    - xác thực mặc định cũ trong `~/.openclaw/credentials/` vẫn được nhận dạng/di chuyển cho luồng tài khoản mặc định
  </Accordion>

  <Accordion title="Hành vi đăng xuất">
    `openclaw channels logout --channel whatsapp [--account <id>]` xóa trạng thái xác thực WhatsApp cho tài khoản đó.

    Trong các thư mục xác thực cũ, `oauth.json` được bảo tồn trong khi các tệp xác thực Baileys bị xóa.

  </Accordion>
</AccordionGroup>
## Công cụ, hành động và ghi cấu hình

- Hỗ trợ công cụ agent bao gồm hành động phản ứng WhatsApp (`react`).
- Cổng hành động:
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Ghi cấu hình được khởi tạo từ kênh được bật theo mặc định (tắt thông qua `channels.whatsapp.configWrites=false`).
## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="Chưa liên kết (cần QR)">
    Triệu chứng: trạng thái kênh báo cáo chưa liên kết.

    Khắc phục:

    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```

  </Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Symptom: linked account with repeated disconnects or reconnect attempts.

    Fix:

    ```bash
    openclaw doctor
    openclaw logs --follow
    ```

    If needed, re-link with `channels login`.

  </Accordion>

  <Accordion title="No active listener when sending">
    Outbound sends fail fast when no active gateway listener exists for the target account.

    Make sure gateway is running and the account is linked.

  </Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Check in this order:

    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist entries
    - mention gating (`requireMention` + mention patterns)
    - duplicate keys in `openclaw.json` (JSON5): later entries override earlier ones, so keep a single `groupPolicy` cho mỗi phạm vi

  </Accordion>

  <Accordion title="Cảnh báo runtime Bun">
    Gateway WhatsApp nên sử dụng Node. Bun được đánh dấu là không tương thích cho hoạt động ổn định của gateway WhatsApp/Telegram.
  </Accordion>
</AccordionGroup>
## Tham chiếu cấu hình

Tham chiếu chính:

- [Tham chiếu cấu hình - WhatsApp](/gateway/configuration-reference#whatsapp)

Các trường WhatsApp quan trọng:

- truy cập: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- gửi tin: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- đa tài khoản: `accounts.<id>.enabled`, `accounts.<id>.authDir`, ghi đè cấp tài khoản
- hoạt động: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- hành vi phiên: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`
## Liên quan

- [Ghép nối](/channels/pairing)
- [Định tuyến kênh](/channels/channel-routing)
- [Định tuyến đa agent](/concepts/multi-agent)
- [Khắc phục sự cố](/channels/troubleshooting)