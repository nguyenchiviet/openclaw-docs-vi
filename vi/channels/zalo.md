---
summary: 'Trạng thái hỗ trợ bot Zalo, khả năng và cấu hình'
read_when:
  - Làm việc với các tính năng Zalo hoặc webhook
title: Zalo
x-i18n:
  source_path: channels\zalo.md
  source_hash: 5c787e2a8c1c335df02a6676482a1fedfb79728af4cc54e4c8097221a6abca6e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:30:13.927Z'
---

# Zalo (Bot API)

Trạng thái: thử nghiệm. Tin nhắn riêng được hỗ trợ; xử lý nhóm có sẵn với các điều khiển chính sách nhóm rõ ràng.
## Plugin cần thiết

Zalo được cung cấp dưới dạng plugin và không được tích hợp sẵn trong bản cài đặt cốt lõi.

- Cài đặt qua CLI: `openclaw plugins install @openclaw/zalo`
- Hoặc chọn **Zalo** trong quá trình thiết lập ban đầu và xác nhận lời nhắc cài đặt
- Chi tiết: [Plugins](/tools/plugin)
## Thiết lập nhanh (người mới bắt đầu)

1. Cài đặt plugin Zalo:
   - Từ mã nguồn: `openclaw plugins install ./extensions/zalo`
   - Từ npm (nếu đã xuất bản): `openclaw plugins install @openclaw/zalo`
   - Hoặc chọn **Zalo** trong thiết lập ban đầu và xác nhận lời nhắc cài đặt
2. Đặt token:
   - Biến môi trường: `ZALO_BOT_TOKEN=...`
   - Hoặc cấu hình: `channels.zalo.botToken: "..."`.
3. Khởi động lại Gateway (hoặc hoàn thành thiết lập ban đầu).
4. Truy cập tin nhắn riêng mặc định là ghép nối; phê duyệt mã ghép nối khi liên hệ lần đầu.

Cấu hình tối thiểu:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```
## Zalo là gì

Zalo là ứng dụng nhắn tin tập trung vào Việt Nam; Bot API của nó cho phép Gateway chạy bot cho các cuộc trò chuyện 1:1.
Nó phù hợp cho hỗ trợ hoặc thông báo khi bạn muốn định tuyến xác định trở lại Zalo.

- Kênh Zalo Bot API được sở hữu bởi Gateway.
- Định tuyến xác định: phản hồi quay trở lại Zalo; mô hình không bao giờ chọn kênh.
- Tin nhắn riêng chia sẻ phiên chính của agent.
- Nhóm được hỗ trợ với kiểm soát chính sách (`groupPolicy` + `groupAllowFrom`) và mặc định là hành vi danh sách cho phép fail-closed.
## Thiết lập (đường dẫn nhanh)

### 1) Tạo token bot (Zalo Bot Platform)

1. Truy cập [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) và đăng nhập.
2. Tạo bot mới và cấu hình các thiết lập của nó.
3. Sao chép token bot (định dạng: `12345689:abc-xyz`).

### 2) Cấu hình token (env hoặc config)

Ví dụ:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

Env option: `ZALO_BOT_TOKEN=...` (works for the default account only).

Multi-account support: use `channels.zalo.accounts` with per-account tokens and optional `name`.

3. Khởi động lại Gateway. Zalo bắt đầu khi token được phân giải (env hoặc config).
4. Truy cập tin nhắn riêng mặc định là ghép nối. Phê duyệt mã khi bot được liên hệ lần đầu.
## Cách hoạt động (hành vi)

- Tin nhắn đến được chuẩn hóa thành envelope kênh chia sẻ với các placeholder phương tiện.
- Phản hồi luôn được định tuyến trở lại cùng một cuộc trò chuyện Zalo.
- Long-polling theo mặc định; chế độ webhook có sẵn với `channels.zalo.webhookUrl`.
## Giới hạn

- Văn bản gửi đi được chia thành các khối 2000 ký tự (giới hạn API Zalo).
- Tải xuống/tải lên phương tiện được giới hạn bởi `channels.zalo.mediaMaxMb` (mặc định 5).
- Truyền phát theo khối bị chặn theo mặc định do giới hạn 2000 ký tự khiến streaming ít hữu ích hơn.
## Kiểm soát truy cập (Tin nhắn riêng)

### Truy cập tin nhắn riêng

- Mặc định: `channels.zalo.dmPolicy = "pairing"`. Người gửi không xác định sẽ nhận được mã ghép nối; tin nhắn sẽ bị bỏ qua cho đến khi được phê duyệt (mã hết hạn sau 1 giờ).
- Phê duyệt thông qua:
  - `openclaw pairing list zalo`
  - `openclaw pairing approve zalo <CODE>`
- Ghép nối là phương thức trao đổi token mặc định. Chi tiết: [Ghép nối](/channels/pairing)
- `channels.zalo.allowFrom` chấp nhận ID người dùng dạng số (không có tính năng tra cứu tên người dùng).
## Kiểm soát truy cập (Nhóm)

- `channels.zalo.groupPolicy` kiểm soát xử lý tin nhắn đến trong nhóm: `open | allowlist | disabled`.
- Hành vi mặc định là từ chối tất cả: `allowlist`.
- `channels.zalo.groupAllowFrom` hạn chế ID người gửi nào có thể kích hoạt bot trong nhóm.
- Nếu `groupAllowFrom` không được đặt, Zalo sẽ quay về `allowFrom` để kiểm tra người gửi.
- `groupPolicy: "disabled"` chặn tất cả tin nhắn nhóm.
- `groupPolicy: "open"` cho phép bất kỳ thành viên nhóm nào (có cổng mention).
- Lưu ý runtime: nếu `channels.zalo` bị thiếu hoàn toàn, runtime vẫn sẽ quay về `groupPolicy="allowlist"` để đảm bảo an toàn.
## Long-polling so với webhook

- Mặc định: long-polling (không cần URL công khai).
- Chế độ webhook: đặt `channels.zalo.webhookUrl` và `channels.zalo.webhookSecret`.
  - Khóa bí mật webhook phải có từ 8-256 ký tự.
  - URL webhook phải sử dụng HTTPS.
  - Zalo gửi sự kiện với header `X-Bot-Api-Secret-Token` để xác minh.
  - Gateway HTTP xử lý các yêu cầu webhook tại `channels.zalo.webhookPath` (mặc định là đường dẫn URL webhook).
  - Các yêu cầu phải sử dụng `Content-Type: application/json` (hoặc các loại media `+json`).
  - Các sự kiện trùng lặp (`event_name + message_id`) sẽ bị bỏ qua trong một khoảng thời gian phát lại ngắn.
  - Lưu lượng đột biến bị giới hạn tốc độ theo đường dẫn/nguồn và có thể trả về HTTP 429.

**Lưu ý:** getUpdates (polling) và webhook loại trừ lẫn nhau theo tài liệu API Zalo.
## Các loại tin nhắn được hỗ trợ

- **Tin nhắn văn bản**: Hỗ trợ đầy đủ với việc chia nhỏ theo 2000 ký tự.
- **Tin nhắn hình ảnh**: Tải xuống và xử lý hình ảnh đến; gửi hình ảnh qua `sendPhoto`.
- **Sticker**: Được ghi lại nhưng không được xử lý đầy đủ (không có phản hồi từ agent).
- **Các loại không được hỗ trợ**: Được ghi lại (ví dụ: tin nhắn từ người dùng được bảo vệ).
## Khả năng

| Tính năng       | Trạng thái                                               |
| --------------- | -------------------------------------------------------- |
| Tin nhắn riêng  | ✅ Được hỗ trợ                                           |
| Nhóm            | ⚠️ Được hỗ trợ với kiểm soát chính sách (danh sách cho phép theo mặc định) |
| Phương tiện (hình ảnh) | ✅ Được hỗ trợ                                    |
| Phản ứng        | ❌ Không được hỗ trợ                                     |
| Chủ đề          | ❌ Không được hỗ trợ                                     |
| Bình chọn       | ❌ Không được hỗ trợ                                     |
| Lệnh gốc        | ❌ Không được hỗ trợ                                     |
| Streaming       | ⚠️ Bị chặn (giới hạn 2000 ký tự)                        |
## Mục tiêu gửi (CLI/cron)

- Sử dụng id chat làm mục tiêu.
- Ví dụ: `openclaw message send --channel zalo --target 123456789 --message "hi"`.
## Khắc phục sự cố

**Bot không phản hồi:**

- Kiểm tra token có hợp lệ: `openclaw channels status --probe`
- Xác minh người gửi đã được phê duyệt (ghép nối hoặc allowFrom)
- Kiểm tra nhật ký Gateway: `openclaw logs --follow`

**Webhook không nhận được sự kiện:**

- Đảm bảo URL webhook sử dụng HTTPS
- Xác minh token bí mật có độ dài 8-256 ký tự
- Xác nhận endpoint HTTP của Gateway có thể truy cập được trên đường dẫn đã cấu hình
- Kiểm tra rằng polling getUpdates không đang chạy (chúng loại trừ lẫn nhau)
## Tham chiếu cấu hình (Zalo)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.zalo.enabled`: bật/tắt khởi động kênh.
- `channels.zalo.botToken`: token bot từ Zalo Bot Platform.
- `channels.zalo.tokenFile`: đọc token từ đường dẫn tệp.
- `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: pairing).
- `channels.zalo.allowFrom`: danh sách cho phép tin nhắn riêng (ID người dùng). `open` yêu cầu `"*"`. Trình hướng dẫn sẽ yêu cầu ID số.
- `channels.zalo.groupPolicy`: `open | allowlist | disabled` (mặc định: allowlist).
- `channels.zalo.groupAllowFrom`: danh sách cho phép người gửi nhóm (ID người dùng). Quay về `allowFrom` khi không được đặt.
- `channels.zalo.mediaMaxMb`: giới hạn phương tiện đến/đi (MB, mặc định 5).
- `channels.zalo.webhookUrl`: bật chế độ webhook (yêu cầu HTTPS).
- `channels.zalo.webhookSecret`: bí mật webhook (8-256 ký tự).
- `channels.zalo.webhookPath`: đường dẫn webhook trên máy chủ HTTP Gateway.
- `channels.zalo.proxy`: URL proxy cho các yêu cầu API.

Tùy chọn đa tài khoản:

- `channels.zalo.accounts.<id>.botToken`: token theo tài khoản.
- `channels.zalo.accounts.<id>.tokenFile`: tệp token theo tài khoản.
- `channels.zalo.accounts.<id>.name`: tên hiển thị.
- `channels.zalo.accounts.<id>.enabled`: bật/tắt tài khoản.
- `channels.zalo.accounts.<id>.dmPolicy`: chính sách tin nhắn riêng theo tài khoản.
- `channels.zalo.accounts.<id>.allowFrom`: danh sách cho phép theo tài khoản.
- `channels.zalo.accounts.<id>.groupPolicy`: chính sách nhóm theo tài khoản.
- `channels.zalo.accounts.<id>.groupAllowFrom`: danh sách cho phép người gửi nhóm theo tài khoản.
- `channels.zalo.accounts.<id>.webhookUrl`: URL webhook theo tài khoản.
- `channels.zalo.accounts.<id>.webhookSecret`: bí mật webhook theo tài khoản.
- `channels.zalo.accounts.<id>.webhookPath`: đường dẫn webhook theo tài khoản.
- `channels.zalo.accounts.<id>.proxy`: URL proxy theo tài khoản.