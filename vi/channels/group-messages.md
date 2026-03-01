---
summary: >-
  Hành vi và cấu hình cho việc xử lý tin nhắn nhóm WhatsApp (mentionPatterns
  được chia sẻ giữa các giao diện)
read_when:
  - Thay đổi quy tắc tin nhắn nhóm hoặc đề cập
title: Tin nhắn nhóm
x-i18n:
  source_path: channels\group-messages.md
  source_hash: 181a72f12f5021af77c2e4c913120f711e0c0bc271d218d75cb6fe80dab675bb
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:18:12.888Z'
---

# Tin nhắn nhóm (kênh WhatsApp web)

Mục tiêu: để Clawd tham gia các nhóm WhatsApp, chỉ thức dậy khi được gọi, và giữ cuộc trò chuyện đó tách biệt khỏi phiên tin nhắn riêng cá nhân.

Lưu ý: `agents.list[].groupChat.mentionPatterns` hiện cũng được sử dụng bởi Telegram/Discord/Slack/iMessage; tài liệu này tập trung vào hành vi cụ thể của WhatsApp. Đối với thiết lập đa agent, hãy đặt `agents.list[].groupChat.mentionPatterns` cho mỗi agent (hoặc sử dụng `messages.groupChat.mentionPatterns` làm giá trị dự phòng toàn cục).
## Những gì đã được triển khai (2025-12-03)

- Chế độ kích hoạt: `mention` (mặc định) hoặc `always`. `mention` yêu cầu một ping (thực sự @-mention WhatsApp qua `mentionedJids`, mẫu regex, hoặc E.164 của bot ở bất kỳ đâu trong văn bản). `always` đánh thức agent trên mọi tin nhắn nhưng nó chỉ nên trả lời khi có thể thêm giá trị có ý nghĩa; nếu không nó trả về token im lặng `NO_REPLY`. Mặc định có thể được đặt trong cấu hình (`channels.whatsapp.groups`) và ghi đè cho từng nhóm qua `/activation`. Khi `channels.whatsapp.groups` được đặt, nó cũng hoạt động như danh sách cho phép nhóm (bao gồm `"*"` để cho phép tất cả).
- Chính sách nhóm: `channels.whatsapp.groupPolicy` kiểm soát việc tin nhắn nhóm có được chấp nhận hay không (`open|disabled|allowlist`). `allowlist` sử dụng `channels.whatsapp.groupAllowFrom` (dự phòng: `channels.whatsapp.allowFrom` rõ ràng). Mặc định là `allowlist` (bị chặn cho đến khi bạn thêm người gửi).
- Phiên theo nhóm: khóa phiên trông như `agent:<agentId>:whatsapp:group:<jid>` vì vậy các lệnh như `/verbose on` hoặc `/think high` (được gửi dưới dạng tin nhắn độc lập) được giới hạn trong nhóm đó; trạng thái tin nhắn riêng cá nhân không bị ảnh hưởng. Heartbeat được bỏ qua cho các luồng nhóm.
- Tiêm ngữ cảnh: tin nhắn nhóm **chỉ đang chờ** (mặc định 50) mà _không_ kích hoạt một lần chạy được đặt tiền tố dưới `[Chat messages since your last reply - for context]`, với dòng kích hoạt dưới `[Current message - respond to this]`. Tin nhắn đã có trong phiên không được tiêm lại.
- Hiển thị người gửi: mỗi lô nhóm hiện kết thúc bằng `[from: Sender Name (+E164)]` để Pi biết ai đang nói.
- Tạm thời/xem một lần: chúng tôi mở gói những tin nhắn đó trước khi trích xuất văn bản/đề cập, vì vậy ping bên trong chúng vẫn kích hoạt.
- Lời nhắc hệ thống nhóm: ở lượt đầu tiên của phiên nhóm (và bất cứ khi nào `/activation` thay đổi chế độ) chúng tôi tiêm một đoạn ngắn vào lời nhắc hệ thống như `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` Nếu siêu dữ liệu không có sẵn, chúng tôi vẫn nói với agent rằng đó là cuộc trò chuyện nhóm.
## Ví dụ cấu hình (WhatsApp)

Thêm khối `groupChat` vào `~/.openclaw/openclaw.json` để ping theo tên hiển thị hoạt động ngay cả khi WhatsApp loại bỏ `@` hiển thị trong nội dung văn bản:

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

Notes:

- The regexes are case-insensitive; they cover a display-name ping like `@openclaw` and the raw number with or without `+`/spaces.
- WhatsApp still sends canonical mentions via `mentionedJids` when someone taps the contact, so the number fallback is rarely needed but is a useful safety net.

### Activation command (owner-only)

Use the group chat command:

- `/activation mention`
- `/activation always`

Only the owner number (from `channels.whatsapp.allowFrom`, or the bot’s own E.164 when unset) can change this. Send `/status` như một tin nhắn độc lập trong nhóm để xem chế độ kích hoạt hiện tại.
## Cách sử dụng

1. Thêm tài khoản WhatsApp của bạn (tài khoản đang chạy OpenClaw) vào nhóm.
2. Nói `@openclaw …` (hoặc bao gồm số). Chỉ những người gửi được cho phép mới có thể kích hoạt trừ khi bạn đặt `groupPolicy: "open"`.
3. Lời nhắc agent sẽ bao gồm ngữ cảnh nhóm gần đây cộng với dấu hiệu `[from: …]` ở cuối để nó có thể trả lời đúng người.
4. Các chỉ thị cấp phiên (`/verbose on`, `/think high`, `/new` hoặc `/reset`, `/compact`) chỉ áp dụng cho phiên của nhóm đó; gửi chúng dưới dạng tin nhắn độc lập để chúng được đăng ký. Phiên tin nhắn riêng cá nhân của bạn vẫn độc lập.
## Kiểm tra / xác minh

- Kiểm tra thủ công:
  - Gửi một `@openclaw` ping trong nhóm và xác nhận có phản hồi tham chiếu đến tên người gửi.
  - Gửi ping thứ hai và xác minh khối lịch sử được bao gồm sau đó được xóa ở lượt tiếp theo.
- Kiểm tra nhật ký gateway (chạy với `--verbose`) để xem các mục `inbound web message` hiển thị `from: <groupJid>` và hậu tố `[from: …]`.
## Những điều cần lưu ý

- Heartbeat được cố ý bỏ qua cho các nhóm để tránh phát sóng gây nhiễu.
- Tính năng ngăn chặn echo sử dụng chuỗi batch kết hợp; nếu bạn gửi văn bản giống hệt nhau hai lần mà không có mention, chỉ lần đầu tiên sẽ nhận được phản hồi.
- Các mục trong session store sẽ xuất hiện dưới dạng `agent:<agentId>:whatsapp:group:<jid>` trong session store (`~/.openclaw/agents/<agentId>/sessions/sessions.json` theo mặc định); một mục bị thiếu chỉ có nghĩa là nhóm chưa kích hoạt một lần chạy nào.
- Chỉ báo đang gõ trong các nhóm tuân theo `agents.defaults.typingMode` (mặc định: `message` khi không được mention).