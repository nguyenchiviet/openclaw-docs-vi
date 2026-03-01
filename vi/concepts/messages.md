---
summary: 'Luồng tin nhắn, phiên làm việc, xếp hàng, và khả năng hiển thị lý luận'
read_when:
  - Giải thích cách các tin nhắn đến trở thành phản hồi
  - 'Làm rõ các phiên làm việc, chế độ xếp hàng, hoặc hành vi truyền phát'
  - Tài liệu về khả năng hiển thị lý luận và hàm ý sử dụng
title: Tin nhắn
x-i18n:
  source_path: concepts\messages.md
  source_hash: 773301d5c0c1e3b87d1b7ba6d670400cb8ab65d35943f6d54647490e377c369a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:39:40.342Z'
---

# Tin nhắn

Trang này liên kết cách OpenClaw xử lý các tin nhắn đến, phiên, xếp hàng,
truyền phát và khả năng hiển thị suy luận.
## Luồng tin nhắn (cấp cao)

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

Key knobs live in configuration:

- `messages.*` for prefixes, queueing, and group behavior.
- `agents.defaults.*` for block streaming and chunking defaults.
- Channel overrides (`channels.whatsapp.*`, `channels.telegram.*`, v.v.) để bật tắt caps và truyền phát theo khối.

Xem [Cấu hình](/gateway/configuration) để xem đầy đủ schema.
## Khử trùng lặp đến

Các kênh có thể gửi lại cùng một tin nhắn sau khi kết nối lại. OpenClaw duy trì
một bộ nhớ cache tồn tại ngắn hạn được khóa theo kênh/tài khoản/đối tác/phiên/id tin nhắn để các lần gửi trùng lặp không kích hoạt chạy agent khác.
## Khử rung đầu vào

Các tin nhắn liên tiếp nhanh chóng từ **cùng một người gửi** có thể được gộp thành một lượt agent duy nhất thông qua `messages.inbound`. Khử rung được phạm vi hóa theo kênh + cuộc trò chuyện và sử dụng tin nhắn gần đây nhất để luồng trả lời/ID.

Cấu hình (mặc định toàn cục + ghi đè theo kênh):

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

Ghi chú:

- Khử rung chỉ áp dụng cho tin nhắn **chỉ văn bản**; phương tiện/tệp đính kèm được xóa ngay lập tức.
- Các lệnh điều khiển bỏ qua khử rung để chúng vẫn độc lập.
## Phiên và thiết bị

Các phiên được sở hữu bởi Gateway, không phải bởi các client.

- Các cuộc trò chuyện trực tiếp sẽ được gộp vào khóa phiên chính của agent.
- Các nhóm/kênh sẽ có các khóa phiên riêng của chúng.
- Kho phiên và bản ghi lại nằm trên máy chủ Gateway.

Nhiều thiết bị/kênh có thể ánh xạ tới cùng một phiên, nhưng lịch sử không được đồng bộ hóa đầy đủ trở lại cho mọi client. Khuyến nghị: sử dụng một thiết bị chính duy nhất cho các cuộc trò chuyện dài để tránh ngữ cảnh khác nhau. Control UI và TUI luôn hiển thị bản ghi lại phiên được hỗ trợ bởi Gateway, vì vậy chúng là nguồn sự thật.

Chi tiết: [Quản lý phiên](/concepts/session).
## Nội dung đến và ngữ cảnh lịch sử

OpenClaw tách biệt **nội dung prompt** với **nội dung lệnh**:

- `Body`: văn bản prompt được gửi đến agent. Điều này có thể bao gồm các bao bọc kênh và các trình bao bọc lịch sử tùy chọn.
- `CommandBody`: văn bản người dùng thô để phân tích chỉ thị/lệnh.
- `RawBody`: bí danh kế thừa cho `CommandBody` (được giữ lại để tương thích).

Khi một kênh cung cấp lịch sử, nó sử dụng một trình bao bọc chung:

- `[Chat messages since your last reply - for context]`
- `[Current message - respond to this]`

Đối với **các cuộc trò chuyện không trực tiếp** (nhóm/kênh/phòng), **nội dung tin nhắn hiện tại** được đặt tiền tố với nhãn người gửi (cùng kiểu được sử dụng cho các mục nhập lịch sử). Điều này giữ cho các tin nhắn thời gian thực và tin nhắn xếp hàng/lịch sử nhất quán trong prompt của agent.

Bộ đệm lịch sử là **chỉ đang chờ xử lý**: chúng bao gồm các tin nhắn nhóm không kích hoạt chạy (ví dụ: các tin nhắn được gated bằng mention) và **loại trừ** các tin nhắn đã có trong bảng ghi âm phiên.

Loại bỏ chỉ thị chỉ áp dụng cho phần **tin nhắn hiện tại** để lịch sử vẫn nguyên vẹn. Các kênh bao bọc lịch sử nên đặt `CommandBody` (hoặc `RawBody`) thành văn bản tin nhắn gốc và giữ `Body` làm prompt kết hợp. Bộ đệm lịch sử có thể cấu hình thông qua `messages.groupChat.historyLimit` (mặc định toàn cầu) và các ghi đè cho từng kênh như `channels.slack.historyLimit` hoặc `channels.telegram.accounts.<id>.historyLimit` (đặt `0` để tắt).
## Xếp hàng và theo dõi

Nếu một lần chạy đã hoạt động, các tin nhắn đến có thể được xếp hàng, điều hướng vào lần chạy hiện tại hoặc thu thập cho một lượt theo dõi.

- Cấu hình thông qua `messages.queue` (và `messages.queue.byChannel`).
- Chế độ: `interrupt`, `steer`, `followup`, `collect`, cộng với các biến thể backlog.

Chi tiết: [Xếp hàng](/concepts/queue).
## Truyền phát theo khối, chia nhỏ và xử lý hàng loạt

Truyền phát theo khối gửi các phản hồi một phần khi mô hình tạo ra các khối văn bản.
Chia nhỏ tôn trọng giới hạn văn bản kênh và tránh chia tách mã được bao quanh.

Cài đặt chính:

- `agents.defaults.blockStreamingDefault` (`on|off`, tắt theo mặc định)
- `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
- `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
- `agents.defaults.blockStreamingCoalesce` (xử lý hàng loạt dựa trên thời gian chờ)
- `agents.defaults.humanDelay` (tạm dừng giống con người giữa các phản hồi khối)
- Ghi đè kênh: `*.blockStreaming` và `*.blockStreamingCoalesce` (các kênh không phải Telegram yêu cầu `*.blockStreaming: true` rõ ràng)

Chi tiết: [Truyền phát + chia nhỏ](/concepts/streaming).
## Khả năng hiển thị lý luận và token

OpenClaw có thể hiển thị hoặc ẩn lý luận của mô hình:

- `/reasoning on|off|stream` kiểm soát khả năng hiển thị.
- Nội dung lý luận vẫn tính vào mức sử dụng token khi được tạo bởi mô hình.
- Telegram hỗ trợ truyền phát lý luận vào bong bóng nháp.

Chi tiết: [Thinking + reasoning directives](/tools/thinking) và [Token use](/reference/token-use).
## Tiền tố, luồng tin nhắn và trả lời

Định dạng tin nhắn gửi đi được tập trung hóa trong `messages`:

- `messages.responsePrefix`, `channels.<channel>.responsePrefix`, và `channels.<channel>.accounts.<id>.responsePrefix` (cascade tiền tố gửi đi), cộng với `channels.whatsapp.messagePrefix` (tiền tố nhận WhatsApp)
- Luồng trả lời thông qua `replyToMode` và các giá trị mặc định theo kênh

Chi tiết: [Cấu hình](/gateway/configuration#messages) và tài liệu kênh.