---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình Matrix'
read_when:
  - Đang làm việc trên các tính năng kênh Matrix
title: Matrix
x-i18n:
  source_path: channels\matrix.md
  source_hash: b1190c8cd14158c5c37fe047b790d0ef5dba8ce073fa36c91e079c9a721b7fe4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:20:57.929Z'
---

# Matrix (plugin)

Matrix là một giao thức nhắn tin mở, phi tập trung. OpenClaw kết nối như một **người dùng** Matrix
trên bất kỳ homeserver nào, vì vậy bạn cần một tài khoản Matrix cho bot. Khi đã đăng nhập, bạn có thể gửi tin nhắn riêng
cho bot trực tiếp hoặc mời nó vào các phòng (các "nhóm" Matrix). Beeper cũng là một tùy chọn client hợp lệ,
nhưng nó yêu cầu E2EE phải được bật.

Trạng thái: được hỗ trợ qua plugin (@vector-im/matrix-bot-sdk). Tin nhắn riêng, phòng, chuỗi tin nhắn, phương tiện, phản ứng,
bình chọn (gửi + poll-start dưới dạng văn bản), vị trí, và E2EE (với hỗ trợ mã hóa).
## Plugin bắt buộc

Matrix được cung cấp dưới dạng plugin và không được tích hợp sẵn trong bản cài đặt cốt lõi.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/matrix
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/matrix
```

Nếu bạn chọn Matrix trong quá trình cấu hình/thiết lập ban đầu và phát hiện git checkout,
OpenClaw sẽ tự động đề xuất đường dẫn cài đặt cục bộ.

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập

1. Cài đặt plugin Matrix:
   - Từ npm: `openclaw plugins install @openclaw/matrix`
   - Từ bản checkout cục bộ: `openclaw plugins install ./extensions/matrix`
2. Tạo tài khoản Matrix trên một homeserver:
   - Duyệt các tùy chọn hosting tại [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   - Hoặc tự host.
3. Lấy access token cho tài khoản bot:
   - Sử dụng Matrix login API với `curl` tại home server của bạn:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   - Replace `matrix.example.org` with your homeserver URL.
   - Or set `channels.matrix.userId` + `channels.matrix.password`: OpenClaw calls the same
     login endpoint, stores the access token in `~/.openclaw/credentials/matrix/credentials.json`,
     and reuses it on next start.

4. Configure credentials:
   - Env: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (or `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   - Or config: `channels.matrix.*`
   - If both are set, config takes precedence.
   - With access token: user ID is fetched automatically via `/whoami`.
   - When set, `channels.matrix.userId` should be the full Matrix ID (example: `@bot:example.org`).
5. Restart the gateway (or finish onboarding).
6. Start a DM with the bot or invite it to a room from any Matrix client
   (Element, Beeper, etc.; see [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)). Beeper requires E2EE,
   so set `channels.matrix.encryption: true` and verify the device.

Minimal config (access token, user ID auto-fetched):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

E2EE config (end to end encryption enabled):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```
## Mã hóa (E2EE)

Mã hóa đầu cuối được **hỗ trợ** thông qua Rust crypto SDK.

Kích hoạt với `channels.matrix.encryption: true`:

- Nếu mô-đun crypto được tải, các phòng được mã hóa sẽ tự động được giải mã.
- Media gửi đi được mã hóa khi gửi đến các phòng được mã hóa.
- Khi kết nối lần đầu, OpenClaw yêu cầu xác minh thiết bị từ các phiên khác của bạn.
- Xác minh thiết bị trong một client Matrix khác (Element, v.v.) để kích hoạt chia sẻ khóa.
- Nếu mô-đun crypto không thể được tải, E2EE bị vô hiệu hóa và các phòng được mã hóa sẽ không giải mã được;
  OpenClaw ghi lại một cảnh báo.
- Nếu bạn thấy lỗi thiếu mô-đun crypto (ví dụ, `@matrix-org/matrix-sdk-crypto-nodejs-*`),
  cho phép build scripts cho `@matrix-org/matrix-sdk-crypto-nodejs` và chạy
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` hoặc tải binary với
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

Trạng thái crypto được lưu trữ theo từng tài khoản + access token trong
`~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
(cơ sở dữ liệu SQLite). Trạng thái đồng bộ nằm cùng với nó trong `bot-storage.json`.
Nếu access token (thiết bị) thay đổi, một store mới được tạo và bot phải được
xác minh lại cho các phòng được mã hóa.

**Xác minh thiết bị:**
Khi E2EE được kích hoạt, bot sẽ yêu cầu xác minh từ các phiên khác của bạn khi khởi động.
Mở Element (hoặc client khác) và chấp thuận yêu cầu xác minh để thiết lập tin cậy.
Sau khi được xác minh, bot có thể giải mã tin nhắn trong các phòng được mã hóa.
## Đa tài khoản

Hỗ trợ đa tài khoản: sử dụng `channels.matrix.accounts` với thông tin xác thực cho từng tài khoản và `name` tùy chọn. Xem [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) để biết mẫu chung.

Mỗi tài khoản chạy như một người dùng Matrix riêng biệt trên bất kỳ homeserver nào. Cấu hình cho từng tài khoản
kế thừa từ cài đặt `channels.matrix` cấp cao nhất và có thể ghi đè bất kỳ tùy chọn nào
(chính sách tin nhắn riêng, nhóm, mã hóa, v.v.).

```json5
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "Main assistant",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "Alerts bot",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

Notes:

- Account startup is serialized to avoid race conditions with concurrent module imports.
- Env variables (`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, etc.) only apply to the **default** account.
- Base channel settings (DM policy, group policy, mention gating, etc.) apply to all accounts unless overridden per account.
- Use `bindings[].match.accountId` để định tuyến mỗi tài khoản đến một agent khác nhau.
- Trạng thái mã hóa được lưu trữ cho từng tài khoản + access token (kho khóa riêng biệt cho mỗi tài khoản).
## Mô hình định tuyến

- Phản hồi luôn được gửi trở lại Matrix.
- Tin nhắn riêng chia sẻ phiên chính của agent; các phòng được ánh xạ tới phiên nhóm.
## Kiểm soát truy cập (Tin nhắn riêng)

- Mặc định: `channels.matrix.dm.policy = "pairing"`. Người gửi không xác định sẽ nhận được mã ghép nối.
- Phê duyệt thông qua:
  - `openclaw pairing list matrix`
  - `openclaw pairing approve matrix <CODE>`
- Tin nhắn riêng công khai: `channels.matrix.dm.policy="open"` cộng với `channels.matrix.dm.allowFrom=["*"]`.
- `channels.matrix.dm.allowFrom` chấp nhận ID người dùng Matrix đầy đủ (ví dụ: `@user:server`). Trình hướng dẫn sẽ phân giải tên hiển thị thành ID người dùng khi tìm kiếm thư mục tìm thấy một kết quả khớp chính xác duy nhất.
- Không sử dụng tên hiển thị hoặc localpart đơn thuần (ví dụ: `"Alice"` hoặc `"alice"`). Chúng không rõ ràng và bị bỏ qua khi khớp danh sách cho phép. Sử dụng ID `@user:server` đầy đủ.
## Phòng (nhóm)

- Mặc định: `channels.matrix.groupPolicy = "allowlist"` (cần được nhắc đến). Sử dụng `channels.defaults.groupPolicy` để ghi đè mặc định khi chưa được thiết lập.
- Lưu ý runtime: nếu `channels.matrix` bị thiếu hoàn toàn, runtime sẽ quay về `groupPolicy="allowlist"` để kiểm tra phòng (ngay cả khi `channels.defaults.groupPolicy` đã được thiết lập).
- Danh sách cho phép các phòng với `channels.matrix.groups` (ID phòng hoặc bí danh; tên được phân giải thành ID khi tìm kiếm thư mục tìm thấy một kết quả khớp chính xác duy nhất):

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

- `requireMention: false` enables auto-reply in that room.
- `groups."*"` can set defaults for mention gating across rooms.
- `groupAllowFrom` restricts which senders can trigger the bot in rooms (full Matrix user IDs).
- Per-room `users` allowlists can further restrict senders inside a specific room (use full Matrix user IDs).
- The configure wizard prompts for room allowlists (room IDs, aliases, or names) and resolves names only on an exact, unique match.
- On startup, OpenClaw resolves room/user names in allowlists to IDs and logs the mapping; unresolved entries are ignored for allowlist matching.
- Invites are auto-joined by default; control with `channels.matrix.autoJoin` and `channels.matrix.autoJoinAllowlist`.
- To allow **no rooms**, set `channels.matrix.groupPolicy: "disabled"` (or keep an empty allowlist).
- Legacy key: `channels.matrix.rooms` (same shape as `groups`).
## Threads

- Hỗ trợ phản hồi theo luồng.
- `channels.matrix.threadReplies` kiểm soát việc phản hồi có ở trong luồng hay không:
  - `off`, `inbound` (mặc định), `always`
- `channels.matrix.replyToMode` kiểm soát metadata reply-to khi không phản hồi trong luồng:
  - `off` (mặc định), `first`, `all`
## Khả năng

| Tính năng       | Trạng thái                                                                            |
| --------------- | ------------------------------------------------------------------------------------- |
| Tin nhắn riêng  | ✅ Được hỗ trợ                                                                        |
| Phòng           | ✅ Được hỗ trợ                                                                        |
| Chủ đề          | ✅ Được hỗ trợ                                                                        |
| Phương tiện     | ✅ Được hỗ trợ                                                                        |
| E2EE            | ✅ Được hỗ trợ (yêu cầu mô-đun mã hóa)                                                |
| Phản ứng        | ✅ Được hỗ trợ (gửi/đọc qua công cụ)                                                  |
| Bình chọn       | ✅ Hỗ trợ gửi; bình chọn đến được chuyển đổi thành văn bản (phản hồi/kết thúc bị bỏ qua) |
| Vị trí          | ✅ Được hỗ trợ (geo URI; độ cao bị bỏ qua)                                            |
| Lệnh gốc        | ✅ Được hỗ trợ                                                                        |
## Khắc phục sự cố

Chạy thứ tự kiểm tra này trước:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then confirm DM pairing state if needed:

```bash
openclaw pairing list matrix
```

Common failures:

- Logged in but room messages ignored: room blocked by `groupPolicy` or room allowlist.
- DMs ignored: sender pending approval when `channels.matrix.dm.policy="pairing"`.
- Phòng được mã hóa thất bại: hỗ trợ mã hóa hoặc cài đặt mã hóa không khớp.

Để biết quy trình phân loại: [/channels/troubleshooting](/channels/troubleshooting).
## Tham chiếu cấu hình (Matrix)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.matrix.enabled`: bật/tắt khởi động kênh.
- `channels.matrix.homeserver`: URL homeserver.
- `channels.matrix.userId`: ID người dùng Matrix (tùy chọn với access token).
- `channels.matrix.accessToken`: access token.
- `channels.matrix.password`: mật khẩu để đăng nhập (token được lưu trữ).
- `channels.matrix.deviceName`: tên hiển thị thiết bị.
- `channels.matrix.encryption`: bật E2EE (mặc định: false).
- `channels.matrix.initialSyncLimit`: giới hạn đồng bộ ban đầu.
- `channels.matrix.threadReplies`: `off | inbound | always` (mặc định: inbound).
- `channels.matrix.textChunkLimit`: kích thước khối văn bản outbound (ký tự).
- `channels.matrix.chunkMode`: `length` (mặc định) hoặc `newline` để tách theo dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (mặc định: pairing).
- `channels.matrix.dm.allowFrom`: danh sách cho phép tin nhắn riêng (ID người dùng Matrix đầy đủ). `open` yêu cầu `"*"`. Trình hướng dẫn sẽ chuyển đổi tên thành ID khi có thể.
- `channels.matrix.groupPolicy`: `allowlist | open | disabled` (mặc định: allowlist).
- `channels.matrix.groupAllowFrom`: người gửi được cho phép cho tin nhắn nhóm (ID người dùng Matrix đầy đủ).
- `channels.matrix.allowlistOnly`: buộc áp dụng quy tắc danh sách cho phép cho tin nhắn riêng + phòng.
- `channels.matrix.groups`: danh sách cho phép nhóm + bản đồ cài đặt từng phòng.
- `channels.matrix.rooms`: danh sách cho phép/cấu hình nhóm cũ.
- `channels.matrix.replyToMode`: chế độ reply-to cho threads/tags.
- `channels.matrix.mediaMaxMb`: giới hạn media inbound/outbound (MB).
- `channels.matrix.autoJoin`: xử lý lời mời (`always | allowlist | off`, mặc định: always).
- `channels.matrix.autoJoinAllowlist`: ID/alias phòng được phép để tự động tham gia.
- `channels.matrix.accounts`: cấu hình đa tài khoản được khóa theo ID tài khoản (mỗi tài khoản kế thừa cài đặt cấp cao nhất).
- `channels.matrix.actions`: kiểm soát công cụ theo từng hành động (reactions/messages/pins/memberInfo/channelInfo).