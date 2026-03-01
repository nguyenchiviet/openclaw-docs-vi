---
summary: >-
  Hỗ trợ iMessage cũ thông qua imsg (JSON-RPC qua stdio). Các thiết lập mới nên
  sử dụng BlueBubbles.
read_when:
  - Thiết lập hỗ trợ iMessage
  - Gỡ lỗi gửi/nhận iMessage
title: iMessage
x-i18n:
  source_path: channels\imessage.md
  source_hash: 9c8c5818b23fd3f2fad100fcd3cb4506f0ba7654d3d7c48835e0772e86f72cfd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:19:45.192Z'
---

# iMessage (legacy: imsg)

<Warning>
Đối với các triển khai iMessage mới, hãy sử dụng <a href="/channels/bluebubbles">BlueBubbles</a>.

Tích hợp `imsg` là phiên bản cũ và có thể bị loại bỏ trong bản phát hành tương lai.
</Warning>

Trạng thái: tích hợp CLI bên ngoài phiên bản cũ. Gateway khởi tạo `imsg rpc` và giao tiếp qua JSON-RPC trên stdio (không có daemon/cổng riêng biệt).

<CardGroup cols={3}>
  <Card title="BlueBubbles (khuyến nghị)" icon="message-circle" href="/channels/bluebubbles">
    Đường dẫn iMessage ưu tiên cho các thiết lập mới.
  </Card>
  <Card title="Ghép nối" icon="link" href="/channels/pairing">
    Tin nhắn riêng iMessage mặc định ở chế độ ghép nối.
  </Card>
  <Card title="Tham chiếu cấu hình" icon="settings" href="/gateway/configuration-reference#imessage">
    Tham chiếu đầy đủ các trường iMessage.
  </Card>
</CardGroup>
## Thiết lập nhanh

<Tabs>
  <Tab title="Mac cục bộ (đường dẫn nhanh)">
    <Steps>
      <Step title="Cài đặt và xác minh imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

      </Step>

      <Step title="Configure OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

      </Step>

      <Step title="Start gateway">

```bash
openclaw gateway
```

      </Step>

      <Step title="Approve first DM pairing (default dmPolicy)">

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        Pairing requests expire after 1 hour.
      </Step>
    </Steps>

  </Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw only requires a stdio-compatible `cliPath`, so you can point `cliPath` at a wrapper script that SSHes to a remote Mac and runs `imsg`.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    Cấu hình được khuyến nghị khi tính năng đính kèm được bật:
```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // used for SCP attachment fetches
      includeAttachments: true,
      // Optional: override allowed attachment roots.
      // Defaults include /Users/*/Library/Messages/Attachments
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
    },
  },
}
```

    If `remoteHost` is not set, OpenClaw attempts to auto-detect it by parsing the SSH wrapper script.
    `remoteHost` must be `host` or `user@host` (no spaces or SSH options).
    OpenClaw uses strict host-key checking for SCP, so the relay host key must already exist in `~/.ssh/known_hosts`.
    Attachment paths are validated against allowed roots (`attachmentRoots` / `remoteAttachmentRoots`).

  </Tab>
</Tabs>
## Yêu cầu và quyền (macOS)

- Messages phải được đăng nhập trên Mac đang chạy `imsg`.
- Quyền Full Disk Access là bắt buộc cho ngữ cảnh tiến trình đang chạy OpenClaw/`imsg` (truy cập Messages DB).
- Quyền Automation là bắt buộc để gửi tin nhắn thông qua Messages.app.

<Tip>
Quyền được cấp theo từng ngữ cảnh tiến trình. Nếu gateway chạy headless (LaunchAgent/SSH), hãy chạy một lệnh tương tác một lần trong cùng ngữ cảnh đó để kích hoạt các lời nhắc:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>
## Kiểm soát truy cập và định tuyến

<Tabs>
  <Tab title="Chính sách tin nhắn riêng">
    `channels.imessage.dmPolicy` kiểm soát tin nhắn riêng:

    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `allowFrom` bao gồm `"*"`)
    - `disabled`

    Trường danh sách cho phép: `channels.imessage.allowFrom`.

    Các mục trong danh sách cho phép có thể là handle hoặc đích trò chuyện (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

  </Tab>

  <Tab title="Chính sách nhóm + nhắc đến">
    `channels.imessage.groupPolicy` kiểm soát xử lý nhóm:

    - `allowlist` (mặc định khi được cấu hình)
    - `open`
    - `disabled`

    Danh sách cho phép người gửi nhóm: `channels.imessage.groupAllowFrom`.

    Dự phòng thời gian chạy: nếu `groupAllowFrom` không được đặt, việc kiểm tra người gửi nhóm iMessage sẽ dự phòng về `allowFrom` khi có sẵn.
    Lưu ý thời gian chạy: nếu `channels.imessage` hoàn toàn bị thiếu, thời gian chạy sẽ dự phòng về `groupPolicy="allowlist"` và ghi log cảnh báo (ngay cả khi `channels.defaults.groupPolicy` được đặt).

    Kiểm soát nhắc đến cho nhóm:

    - iMessage không có metadata nhắc đến gốc
    - phát hiện nhắc đến sử dụng các mẫu regex (`agents.list[].groupChat.mentionPatterns`, dự phòng `messages.groupChat.mentionPatterns`)
    - không có các mẫu được cấu hình, không thể thực thi kiểm soát nhắc đến

    Lệnh điều khiển từ người gửi được ủy quyền có thể bỏ qua kiểm soát nhắc đến trong nhóm.

  </Tab>

  <Tab title="Phiên và phản hồi xác định">
    - Tin nhắn riêng sử dụng định tuyến trực tiếp; nhóm sử dụng định tuyến nhóm.
    - Với `session.dmScope=main` mặc định, tin nhắn riêng iMessage thu gọn vào phiên chính của agent.
    - Phiên nhóm được cô lập (`agent:<agentId>:imessage:group:<chat_id>`).
    - Phản hồi định tuyến trở lại iMessage sử dụng metadata kênh/đích gốc.

    Hành vi luồng giống nhóm:

    Một số luồng iMessage nhiều người tham gia có thể đến với `is_group=false`.
    Nếu `chat_id` đó được cấu hình rõ ràng dưới `channels.imessage.groups`, OpenClaw xử lý nó như lưu lượng nhóm (kiểm soát nhóm + cô lập phiên nhóm).

  </Tab>
</Tabs>
## Mô hình triển khai

<AccordionGroup>
  <Accordion title="Người dùng macOS bot chuyên dụng (danh tính iMessage riêng biệt)">
    Sử dụng Apple ID chuyên dụng và người dùng macOS để lưu lượng bot được tách biệt khỏi hồ sơ Messages cá nhân của bạn.

    Quy trình điển hình:

    1. Tạo/đăng nhập người dùng macOS chuyên dụng.
    2. Đăng nhập vào Messages với Apple ID bot trong người dùng đó.
    3. Cài đặt `imsg` trong người dùng đó.
    4. Tạo wrapper SSH để OpenClaw có thể chạy `imsg` trong ngữ cảnh người dùng đó.
    5. Trỏ `channels.imessage.accounts.<id>.cliPath` và `.dbPath` đến hồ sơ người dùng đó.

    Lần chạy đầu tiên có thể yêu cầu phê duyệt GUI (Automation + Full Disk Access) trong phiên người dùng bot đó.

  </Accordion>

  <Accordion title="Mac từ xa qua Tailscale (ví dụ)">
    Cấu trúc mạng phổ biến:

    - gateway chạy trên Linux/VM
    - iMessage + `imsg` chạy trên Mac trong tailnet của bạn
    - wrapper `cliPath` sử dụng SSH để chạy `imsg`
    - `remoteHost` cho phép tải tệp đính kèm SCP

    Ví dụ:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    Use SSH keys so both SSH and SCP are non-interactive.
    Ensure the host key is trusted first (for example `ssh bot@mac-mini.tailnet-1234.ts.net`) so `known_hosts` is populated.

  </Accordion>

  <Accordion title="Multi-account pattern">
    iMessage supports per-account config under `channels.imessage.accounts`.

    Each account can override fields such as `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, cài đặt lịch sử, và danh sách cho phép gốc tệp đính kèm.

  </Accordion>
</AccordionGroup>
## Phương tiện, phân đoạn và mục tiêu gửi

<AccordionGroup>
  <Accordion title="Tệp đính kèm và phương tiện">
    - việc nhận tệp đính kèm đầu vào là tùy chọn: `channels.imessage.includeAttachments`
    - đường dẫn tệp đính kèm từ xa có thể được tải về qua SCP khi `remoteHost` được thiết lập
    - đường dẫn tệp đính kèm phải khớp với các thư mục gốc được phép:
      - `channels.imessage.attachmentRoots` (cục bộ)
      - `channels.imessage.remoteAttachmentRoots` (chế độ SCP từ xa)
      - mẫu thư mục gốc mặc định: `/Users/*/Library/Messages/Attachments`
    - SCP sử dụng kiểm tra khóa máy chủ nghiêm ngặt (`StrictHostKeyChecking=yes`)
    - kích thước phương tiện đầu ra sử dụng `channels.imessage.mediaMaxMb` (mặc định 16 MB)
  </Accordion>

  <Accordion title="Phân đoạn đầu ra">
    - giới hạn đoạn văn bản: `channels.imessage.textChunkLimit` (mặc định 4000)
    - chế độ phân đoạn: `channels.imessage.chunkMode`
      - `length` (mặc định)
      - `newline` (phân tách ưu tiên đoạn văn)
  </Accordion>

  <Accordion title="Định dạng địa chỉ">
    Mục tiêu rõ ràng được khuyến nghị:

    - `chat_id:123` (khuyến nghị cho định tuyến ổn định)
    - `chat_guid:...`
    - `chat_identifier:...`

    Mục tiêu handle cũng được hỗ trợ:

    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`

```bash
imsg chats --limit 20
```

  </Accordion>
</AccordionGroup>
## Ghi cấu hình

iMessage cho phép ghi cấu hình được khởi tạo từ kênh theo mặc định (cho `/config set|unset` khi `commands.config: true`).

Vô hiệu hóa:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```
## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="không tìm thấy imsg hoặc RPC không được hỗ trợ">
    Xác thực tệp nhị phân và hỗ trợ RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

    If probe reports RPC unsupported, update `imsg`.

  </Accordion>

  <Accordion title="DMs are ignored">
    Check:

    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - pairing approvals (`openclaw pairing list imessage`)

  </Accordion>

  <Accordion title="Group messages are ignored">
    Check:

    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - `channels.imessage.groups` allowlist behavior
    - mention pattern configuration (`agents.list[].groupChat.mentionPatterns`)

  </Accordion>

  <Accordion title="Remote attachments fail">
    Check:

    - `channels.imessage.remoteHost`
    - `channels.imessage.remoteAttachmentRoots`
    - SSH/SCP key auth from the gateway host
    - host key exists in `~/.ssh/known_hosts` on the gateway host
    - remote path readability on the Mac running Messages

  </Accordion>

  <Accordion title="macOS permission prompts were missed">
    Re-run in an interactive GUI terminal in the same user/session context and approve prompts:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    Confirm Full Disk Access + Automation are granted for the process context that runs OpenClaw/`imsg`.

  </Accordion>
</AccordionGroup>
## Tham chiếu cấu hình

- [Tham chiếu cấu hình - iMessage](/gateway/configuration-reference#imessage)
- [Cấu hình Gateway](/gateway/configuration)
- [Ghép nối](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)