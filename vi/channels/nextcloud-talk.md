---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình của Nextcloud Talk'
read_when:
  - Đang làm việc trên các tính năng kênh Nextcloud Talk
title: Nextcloud Talk
x-i18n:
  source_path: channels\nextcloud-talk.md
  source_hash: 2769144221e41391fc903a8a9289165fb9ffcc795dd54615e5009f1d6f48df3f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:21:38.920Z'
---

# Nextcloud Talk (plugin)

Trạng thái: được hỗ trợ qua plugin (webhook bot). Tin nhắn riêng, phòng, phản ứng và tin nhắn markdown được hỗ trợ.
## Plugin bắt buộc

Nextcloud Talk được cung cấp dưới dạng plugin và không được tích hợp sẵn trong bản cài đặt cốt lõi.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Nếu bạn chọn Nextcloud Talk trong quá trình cấu hình/thiết lập ban đầu và phát hiện git checkout,
OpenClaw sẽ tự động đề xuất đường dẫn cài đặt cục bộ.

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập nhanh (người mới bắt đầu)

1. Cài đặt plugin Nextcloud Talk.
2. Trên máy chủ Nextcloud của bạn, tạo một bot:

   ```bash
   ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
   ```

3. Enable the bot in the target room settings.
4. Configure OpenClaw:
   - Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   - Or env: `NEXTCLOUD_TALK_BOT_SECRET` (default account only)
5. Restart the gateway (or finish onboarding).

Minimal config:

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```
## Ghi chú

- Bot không thể khởi tạo tin nhắn riêng. Người dùng phải nhắn tin cho bot trước.
- URL webhook phải có thể truy cập được bởi Gateway; đặt `webhookPublicUrl` nếu đằng sau proxy.
- Tải lên media không được hỗ trợ bởi API bot; media được gửi dưới dạng URL.
- Payload webhook không phân biệt tin nhắn riêng và phòng; đặt `apiUser` + `apiPassword` để bật tra cứu loại phòng (nếu không tin nhắn riêng sẽ được xử lý như phòng).
## Kiểm soát truy cập (Tin nhắn riêng)

- Mặc định: `channels.nextcloud-talk.dmPolicy = "pairing"`. Người gửi không xác định sẽ nhận được mã ghép nối.
- Phê duyệt thông qua:
  - `openclaw pairing list nextcloud-talk`
  - `openclaw pairing approve nextcloud-talk <CODE>`
- Tin nhắn riêng công khai: `channels.nextcloud-talk.dmPolicy="open"` cộng với `channels.nextcloud-talk.allowFrom=["*"]`.
- `allowFrom` chỉ khớp với ID người dùng Nextcloud; tên hiển thị sẽ bị bỏ qua.
## Phòng (nhóm)

- Mặc định: `channels.nextcloud-talk.groupPolicy = "allowlist"` (cổng đề cập).
- Danh sách cho phép các phòng với `channels.nextcloud-talk.rooms`:

```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

- To allow no rooms, keep the allowlist empty or set `channels.nextcloud-talk.groupPolicy="disabled"`.
## Khả năng

| Tính năng       | Trạng thái    |
| --------------- | ------------- |
| Tin nhắn riêng  | Được hỗ trợ   |
| Phòng           | Được hỗ trợ   |
| Chuỗi tin nhắn  | Không hỗ trợ  |
| Phương tiện     | Chỉ URL       |
| Phản ứng        | Được hỗ trợ   |
| Lệnh gốc        | Không hỗ trợ  |
## Tham chiếu cấu hình (Nextcloud Talk)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.nextcloud-talk.enabled`: bật/tắt khởi động kênh.
- `channels.nextcloud-talk.baseUrl`: URL phiên bản Nextcloud.
- `channels.nextcloud-talk.botSecret`: khóa bí mật chia sẻ của bot.
- `channels.nextcloud-talk.botSecretFile`: đường dẫn tệp khóa bí mật.
- `channels.nextcloud-talk.apiUser`: người dùng API để tra cứu phòng (phát hiện tin nhắn riêng).
- `channels.nextcloud-talk.apiPassword`: mật khẩu API/ứng dụng để tra cứu phòng.
- `channels.nextcloud-talk.apiPasswordFile`: đường dẫn tệp mật khẩu API.
- `channels.nextcloud-talk.webhookPort`: cổng lắng nghe webhook (mặc định: 8788).
- `channels.nextcloud-talk.webhookHost`: máy chủ webhook (mặc định: 0.0.0.0).
- `channels.nextcloud-talk.webhookPath`: đường dẫn webhook (mặc định: /nextcloud-talk-webhook).
- `channels.nextcloud-talk.webhookPublicUrl`: URL webhook có thể truy cập từ bên ngoài.
- `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
- `channels.nextcloud-talk.allowFrom`: danh sách cho phép tin nhắn riêng (ID người dùng). `open` yêu cầu `"*"`.
- `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
- `channels.nextcloud-talk.groupAllowFrom`: danh sách cho phép nhóm (ID người dùng).
- `channels.nextcloud-talk.rooms`: cài đặt và danh sách cho phép theo từng phòng.
- `channels.nextcloud-talk.historyLimit`: giới hạn lịch sử nhóm (0 để tắt).
- `channels.nextcloud-talk.dmHistoryLimit`: giới hạn lịch sử tin nhắn riêng (0 để tắt).
- `channels.nextcloud-talk.dms`: ghi đè theo từng tin nhắn riêng (historyLimit).
- `channels.nextcloud-talk.textChunkLimit`: kích thước khối văn bản gửi đi (ký tự).
- `channels.nextcloud-talk.chunkMode`: `length` (mặc định) hoặc `newline` để tách theo dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- `channels.nextcloud-talk.blockStreaming`: tắt truyền phát theo khối cho kênh này.
- `channels.nextcloud-talk.blockStreamingCoalesce`: điều chỉnh kết hợp truyền phát theo khối.
- `channels.nextcloud-talk.mediaMaxMb`: giới hạn phương tiện đầu vào (MB).