---
summary: >-
  Hỗ trợ Signal thông qua signal-cli (JSON-RPC + SSE), đường dẫn thiết lập và mô
  hình số
read_when:
  - Thiết lập hỗ trợ Signal
  - Gỡ lỗi gửi/nhận Signal
title: Signal
x-i18n:
  source_path: channels\signal.md
  source_hash: 524d9868f138d46495bb9518b0cbb9c6ef6174248552377a8a659d616501a394
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:25:29.253Z'
---

# Signal (signal-cli)

Trạng thái: tích hợp CLI bên ngoài. Gateway giao tiếp với `signal-cli` qua HTTP JSON-RPC + SSE.
## Điều kiện tiên quyết

- OpenClaw đã được cài đặt trên máy chủ của bạn (quy trình Linux dưới đây đã được kiểm tra trên Ubuntu 24).
- `signal-cli` có sẵn trên máy chủ nơi gateway chạy.
- Một số điện thoại có thể nhận tin nhắn SMS xác minh (cho đường dẫn đăng ký SMS).
- Truy cập trình duyệt cho captcha Signal (`signalcaptchas.org`) trong quá trình đăng ký.
## Thiết lập nhanh (người mới bắt đầu)

1. Sử dụng **số Signal riêng biệt** cho bot (khuyến nghị).
2. Cài đặt `signal-cli` (cần Java nếu bạn sử dụng bản JVM).
3. Chọn một đường dẫn thiết lập:
   - **Đường dẫn A (liên kết QR):** `signal-cli link -n "OpenClaw"` và quét bằng Signal.
   - **Đường dẫn B (đăng ký SMS):** đăng ký số chuyên dụng với xác minh captcha + SMS.
4. Cấu hình OpenClaw và khởi động lại gateway.
5. Gửi tin nhắn riêng đầu tiên và phê duyệt ghép nối (`openclaw pairing approve signal <CODE>`).

Cấu hình tối thiểu:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Field reference:

| Field       | Description                                       |
| ----------- | ------------------------------------------------- |
| `account`   | Bot phone number in E.164 format (`+15551234567`) |
| `cliPath`   | Path to `signal-cli` (`signal-cli` if on `PATH`)  |
| `dmPolicy`  | DM access policy (`pairing` recommended)          |
| `allowFrom` | Phone numbers or `uuid:<id>` các giá trị được phép gửi tin nhắn riêng |
## Nó là gì

- Kênh Signal qua `signal-cli` (không phải libsignal nhúng).
- Định tuyến xác định: phản hồi luôn quay lại Signal.
- Tin nhắn riêng chia sẻ phiên chính của agent; nhóm được cô lập (`agent:<agentId>:signal:group:<groupId>`).
## Ghi cấu hình

Theo mặc định, Signal được phép ghi các cập nhật cấu hình được kích hoạt bởi `/config set|unset` (yêu cầu `commands.config: true`).

Vô hiệu hóa bằng:

```json5
{
  channels: { signal: { configWrites: false } },
}
```
## Mô hình số điện thoại (quan trọng)

- Gateway kết nối với một **thiết bị Signal** (tài khoản `signal-cli`).
- Nếu bạn chạy bot trên **tài khoản Signal cá nhân của mình**, nó sẽ bỏ qua tin nhắn của chính bạn (bảo vệ chống lặp).
- Để "Tôi nhắn tin cho bot và nó trả lời," hãy sử dụng **số bot riêng biệt**.
## Đường dẫn thiết lập A: liên kết tài khoản Signal hiện có (QR)

1. Cài đặt `signal-cli` (bản JVM hoặc native).
2. Liên kết tài khoản bot:
   - `signal-cli link -n "OpenClaw"` sau đó quét mã QR trong Signal.
3. Cấu hình Signal và khởi động gateway.

Ví dụ:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Multi-account support: use `channels.signal.accounts` with per-account config and optional `name`. See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) để xem mẫu chung.
## Thiết lập đường dẫn B: đăng ký số bot chuyên dụng (SMS, Linux)

Sử dụng cách này khi bạn muốn có một số bot chuyên dụng thay vì liên kết tài khoản ứng dụng Signal hiện có.

1. Lấy một số có thể nhận SMS (hoặc xác minh bằng giọng nói cho điện thoại cố định).
   - Sử dụng số bot chuyên dụng để tránh xung đột tài khoản/phiên.
2. Cài đặt `signal-cli` trên máy chủ gateway:

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

If you use the JVM build (`signal-cli-${VERSION}.tar.gz`), install JRE 25+ first.
Keep `signal-cli` updated; upstream notes that old releases can break as Signal server APIs change.

3. Register and verify the number:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

If captcha is required:

1. Open `https://signalcaptchas.org/registration/generate.html`.
2. Complete captcha, copy the `signalcaptcha://...` link target from "Open Signal".
3. Run from the same external IP as the browser session when possible.
4. Run registration again immediately (captcha tokens expire quickly):

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. Configure OpenClaw, restart gateway, verify channel:

```bash
# If you run the gateway as a user systemd service:
systemctl --user restart openclaw-gateway

# Then verify:
openclaw doctor
openclaw channels status --probe
```

5. Pair your DM sender:
   - Send any message to the bot number.
   - Approve code on the server: `openclaw pairing approve signal <PAIRING_CODE>`.
   - Save the bot number as a contact on your phone to avoid "Unknown contact".

Important: registering a phone number account with `signal-cli` can de-authenticate the main Signal app session for that number. Prefer a dedicated bot number, or use QR link mode if you need to keep your existing phone app setup.

Upstream references:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Captcha flow: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Linking flow: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`
## Chế độ daemon bên ngoài (httpUrl)

Nếu bạn muốn tự quản lý `signal-cli` (khởi động chậm JVM, khởi tạo container, hoặc CPU chia sẻ), hãy chạy daemon riêng biệt và trỏ OpenClaw đến nó:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

This skips auto-spawn and the startup wait inside OpenClaw. For slow starts when auto-spawning, set `channels.signal.startupTimeoutMs`.
## Kiểm soát truy cập (Tin nhắn riêng + nhóm)

Tin nhắn riêng:

- Mặc định: `channels.signal.dmPolicy = "pairing"`.
- Người gửi không xác định sẽ nhận mã ghép nối; tin nhắn bị bỏ qua cho đến khi được phê duyệt (mã hết hạn sau 1 giờ).
- Phê duyệt thông qua:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Ghép nối là trao đổi token mặc định cho tin nhắn riêng Signal. Chi tiết: [Ghép nối](/channels/pairing)
- Người gửi chỉ có UUID (từ `sourceUuid`) được lưu trữ dưới dạng `uuid:<id>` trong `channels.signal.allowFrom`.

Nhóm:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` kiểm soát ai có thể kích hoạt trong nhóm khi `allowlist` được đặt.
- Lưu ý runtime: nếu `channels.signal` hoàn toàn bị thiếu, runtime sẽ quay lại `groupPolicy="allowlist"` để kiểm tra nhóm (ngay cả khi `channels.defaults.groupPolicy` được đặt).
## Cách hoạt động (hành vi)

- `signal-cli` chạy như một daemon; Gateway đọc các sự kiện qua SSE.
- Tin nhắn đến được chuẩn hóa thành envelope kênh chung.
- Phản hồi luôn được định tuyến trở lại cùng số hoặc nhóm.
## Phương tiện + giới hạn

- Văn bản gửi đi được chia thành các khối `channels.signal.textChunkLimit` (mặc định 4000).
- Chia khối theo dòng mới tùy chọn: đặt `channels.signal.chunkMode="newline"` để tách theo dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- Hỗ trợ tệp đính kèm (base64 được lấy từ `signal-cli`).
- Giới hạn phương tiện mặc định: `channels.signal.mediaMaxMb` (mặc định 8).
- Sử dụng `channels.signal.ignoreAttachments` để bỏ qua tải xuống phương tiện.
- Ngữ cảnh lịch sử nhóm sử dụng `channels.signal.historyLimit` (hoặc `channels.signal.accounts.*.historyLimit`), dự phòng là `messages.groupChat.historyLimit`. Đặt `0` để vô hiệu hóa (mặc định 50).
## Gõ phím + xác nhận đã đọc

- **Chỉ báo đang gõ**: OpenClaw gửi tín hiệu đang gõ qua `signal-cli sendTyping` và làm mới chúng trong khi phản hồi đang chạy.
- **Xác nhận đã đọc**: khi `channels.signal.sendReadReceipts` là true, OpenClaw chuyển tiếp xác nhận đã đọc cho các tin nhắn riêng được phép.
- Signal-cli không hiển thị xác nhận đã đọc cho các nhóm.
## Phản ứng (công cụ tin nhắn)

- Sử dụng `message action=react` với `channel=signal`.
- Mục tiêu: E.164 hoặc UUID của người gửi (sử dụng `uuid:<id>` từ kết quả ghép nối; UUID đơn thuần cũng hoạt động).
- `messageId` là dấu thời gian Signal cho tin nhắn mà bạn đang phản ứng.
- Phản ứng nhóm yêu cầu `targetAuthor` hoặc `targetAuthorUuid`.

Ví dụ:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Config:

- `channels.signal.actions.reactions`: enable/disable reaction actions (default true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` disables agent reactions (message tool `react` will error).
  - `minimal`/`extensive` enables agent reactions and sets the guidance level.
- Per-account overrides: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.
## Mục tiêu gửi (CLI/cron)

- Tin nhắn riêng: `signal:+15551234567` (hoặc E.164 thuần túy).
- Tin nhắn riêng UUID: `uuid:<id>` (hoặc UUID thuần túy).
- Nhóm: `signal:group:<groupId>`.
- Tên người dùng: `username:<name>` (nếu được hỗ trợ bởi tài khoản Signal của bạn).
## Khắc phục sự cố

Chạy thứ tự này trước:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then confirm DM pairing state if needed:

```bash
openclaw pairing list signal
```

Common failures:

- Daemon reachable but no replies: verify account/daemon settings (`httpUrl`, `account`) and receive mode.
- DMs ignored: sender is pending pairing approval.
- Group messages ignored: group sender/mention gating blocks delivery.
- Config validation errors after edits: run `openclaw doctor --fix`.
- Signal missing from diagnostics: confirm `channels.signal.enabled: true`.

Extra checks:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Để xem quy trình phân loại: [/channels/troubleshooting](/channels/troubleshooting).
## Ghi chú bảo mật

- `signal-cli` lưu trữ khóa tài khoản cục bộ (thường là `~/.local/share/signal-cli/data/`).
- Sao lưu trạng thái tài khoản Signal trước khi di chuyển hoặc xây dựng lại máy chủ.
- Giữ `channels.signal.dmPolicy: "pairing"` trừ khi bạn muốn mở rộng quyền truy cập tin nhắn riêng một cách rõ ràng.
- Xác minh SMS chỉ cần thiết cho quy trình đăng ký hoặc khôi phục, nhưng việc mất quyền kiểm soát số điện thoại/tài khoản có thể làm phức tạp việc đăng ký lại.
## Tham chiếu cấu hình (Signal)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.signal.enabled`: bật/tắt khởi động kênh.
- `channels.signal.account`: E.164 cho tài khoản bot.
- `channels.signal.cliPath`: đường dẫn đến `signal-cli`.
- `channels.signal.httpUrl`: URL daemon đầy đủ (ghi đè host/port).
- `channels.signal.httpHost`, `channels.signal.httpPort`: bind daemon (mặc định 127.0.0.1:8080).
- `channels.signal.autoStart`: tự động khởi chạy daemon (mặc định true nếu `httpUrl` không được đặt).
- `channels.signal.startupTimeoutMs`: thời gian chờ khởi động tính bằng ms (tối đa 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: bỏ qua tải xuống tệp đính kèm.
- `channels.signal.ignoreStories`: bỏ qua stories từ daemon.
- `channels.signal.sendReadReceipts`: chuyển tiếp xác nhận đã đọc.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: pairing).
- `channels.signal.allowFrom`: danh sách cho phép tin nhắn riêng (E.164 hoặc `uuid:<id>`). `open` yêu cầu `"*"`. Signal không có tên người dùng; sử dụng id điện thoại/UUID.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (mặc định: allowlist).
- `channels.signal.groupAllowFrom`: danh sách cho phép người gửi nhóm.
- `channels.signal.historyLimit`: số tin nhắn nhóm tối đa để bao gồm làm ngữ cảnh (0 để tắt).
- `channels.signal.dmHistoryLimit`: giới hạn lịch sử tin nhắn riêng theo lượt người dùng. Ghi đè theo người dùng: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: kích thước khối gửi đi (ký tự).
- `channels.signal.chunkMode`: `length` (mặc định) hoặc `newline` để tách theo dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- `channels.signal.mediaMaxMb`: giới hạn media gửi/nhận (MB).

Tùy chọn toàn cục liên quan:

- `agents.list[].groupChat.mentionPatterns` (Signal không hỗ trợ mention gốc).
- `messages.groupChat.mentionPatterns` (dự phòng toàn cục).
- `messages.responsePrefix`.