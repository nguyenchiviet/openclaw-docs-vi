---
summary: >-
  Hành vi Streaming + chunking (block replies, channel preview streaming, mode
  mapping)
read_when:
  - Giải thích cách hoạt động của streaming hoặc chunking trên các channels
  - Thay đổi hành vi streaming khối hoặc chunking kênh
  - Gỡ lỗi các phản hồi khối trùng lặp/sớm hoặc phát trực tuyến xem trước kênh
title: Streaming và Chunking
x-i18n:
  source_path: concepts\streaming.md
  source_hash: da42317edbcd3a6af56087aeaf5959f162b3f5a5fd8a7cc99d5777f49d833023
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:06.208Z'
---

# Truyền phát + chia nhỏ

OpenClaw có hai lớp truyền phát riêng biệt:

- **Truyền phát theo khối (kênh):** phát hành các **khối** đã hoàn thành khi trợ lý viết. Đây là các tin nhắn kênh bình thường (không phải delta token).
- **Truyền phát xem trước (Telegram/Discord/Slack):** cập nhật một **tin nhắn xem trước** tạm thời trong khi tạo.

Hiện tại **không có truyền phát delta token thực sự** cho các tin nhắn kênh. Truyền phát xem trước dựa trên tin nhắn (gửi + chỉnh sửa/nối thêm).
## Truyền phát theo khối (tin nhắn kênh)

Truyền phát theo khối gửi đầu ra của trợ lý trong các khối thô khi chúng có sẵn.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Legend:

- `text_delta/events`: model stream events (may be sparse for non-streaming models).
- `chunker`: `EmbeddedBlockChunker` applying min/max bounds + break preference.
- `channel send`: actual outbound messages (block replies).

**Controls:**

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (default off).
- Channel overrides: `*.blockStreaming` (and per-account variants) to force `"on"`/`"off"` per channel.
- `agents.defaults.blockStreamingBreak`: `"text_end"` or `"message_end"`.
- `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
- `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (merge streamed blocks before send).
- Channel hard cap: `*.textChunkLimit` (e.g., `channels.whatsapp.textChunkLimit`).
- Channel chunk mode: `*.chunkMode` (`length` default, `newline` splits on blank lines (paragraph boundaries) before length chunking).
- Discord soft cap: `channels.discord.maxLinesPerMessage` (default 17) splits tall replies to avoid UI clipping.

**Boundary semantics:**

- `text_end`: stream blocks as soon as chunker emits; flush on each `text_end`.
- `message_end`: wait until assistant message finishes, then flush buffered output.

`message_end` still uses the chunker if the buffered text exceeds `maxChars`, vì vậy nó có thể phát ra nhiều khối ở cuối.
## Thuật toán chia nhỏ (giới hạn thấp/cao)

Truyền phát theo khối được triển khai bởi `EmbeddedBlockChunker`:

- **Giới hạn thấp:** không phát ra cho đến khi bộ đệm >= `minChars` (trừ khi bị buộc).
- **Giới hạn cao:** ưu tiên chia tách trước `maxChars`; nếu bị buộc, chia tách tại `maxChars`.
- **Ưu tiên ngắt:** `paragraph` → `newline` → `sentence` → `whitespace` → ngắt cứng.
- **Hàng rào mã:** không bao giờ chia tách bên trong hàng rào; khi bị buộc tại `maxChars`, đóng + mở lại hàng rào để giữ Markdown hợp lệ.

`maxChars` được giới hạn trong kênh `textChunkLimit`, vì vậy bạn không thể vượt quá giới hạn trên mỗi kênh.
## Gộp (merge các khối được truyền phát)

Khi truyền phát theo khối được bật, OpenClaw có thể **gộp các khối liên tiếp**
trước khi gửi chúng. Điều này giảm "spam một dòng" trong khi vẫn cung cấp
đầu ra tiến bộ.

- Gộp chờ **khoảng trống nhàn rỗi** (`idleMs`) trước khi xả.
- Bộ đệm được giới hạn bởi `maxChars` và sẽ xả nếu vượt quá nó.
- `minChars` ngăn các mảnh nhỏ gửi đi cho đến khi có đủ văn bản tích lũy
  (xả cuối cùng luôn gửi văn bản còn lại).
- Joiner được lấy từ `blockStreamingChunk.breakPreference`
  (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → dấu cách).
- Ghi đè kênh có sẵn qua `*.blockStreamingCoalesce` (bao gồm cấu hình theo tài khoản).
- Thời gian gộp mặc định `minChars` được tăng lên 1500 cho Signal/Slack/Discord trừ khi bị ghi đè.
## Tốc độ giống con người giữa các khối

Khi truyền phát theo khối được bật, bạn có thể thêm một **tạm dừng ngẫu nhiên** giữa các phản hồi khối (sau khối đầu tiên). Điều này làm cho các phản hồi đa bong bóng cảm thấy tự nhiên hơn.

- Cấu hình: `agents.defaults.humanDelay` (ghi đè cho từng agent thông qua `agents.list[].humanDelay`).
- Chế độ: `off` (mặc định), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
- Chỉ áp dụng cho **phản hồi khối**, không áp dụng cho phản hồi cuối cùng hoặc tóm tắt công cụ.
## "Truyền phát theo khối hoặc toàn bộ"

Điều này ánh xạ tới:

- **Truyền phát theo khối:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (phát ra khi tiến hành). Các kênh không phải Telegram cũng cần `*.blockStreaming: true`.
- **Truyền phát toàn bộ ở cuối:** `blockStreamingBreak: "message_end"` (xả một lần, có thể nhiều khối nếu rất dài).
- **Không truyền phát theo khối:** `blockStreamingDefault: "off"` (chỉ phản hồi cuối cùng).

**Ghi chú kênh:** Truyền phát theo khối **tắt trừ khi**
`*.blockStreaming` được đặt rõ ràng thành `true`. Các kênh có thể truyền phát bản xem trước trực tiếp
(`channels.<channel>.streaming`) mà không cần phản hồi theo khối.

Nhắc nhở vị trí cấu hình: các giá trị mặc định `blockStreaming*` nằm dưới
`agents.defaults`, không phải cấu hình gốc.
## Chế độ xem trước truyền phát

Khóa chính tắc: `channels.<channel>.streaming`

Chế độ:

- `off`: tắt truyền phát xem trước.
- `partial`: xem trước duy nhất được thay thế bằng văn bản mới nhất.
- `block`: cập nhật xem trước theo các bước khối/nối thêm.
- `progress`: xem trước tiến độ/trạng thái trong quá trình tạo, câu trả lời cuối cùng khi hoàn thành.

### Ánh xạ kênh

| Kênh     | `off` | `partial` | `block` | `progress`        |
| -------- | ----- | --------- | ------- | ----------------- |
| Telegram | ✅    | ✅        | ✅      | ánh xạ tới `partial` |
| Discord  | ✅    | ✅        | ✅      | ánh xạ tới `partial` |
| Slack    | ✅    | ✅        | ✅      | ✅                |

Chỉ Slack:

- `channels.slack.nativeStreaming` bật/tắt các lệnh gọi API truyền phát gốc Slack khi `streaming=partial` (mặc định: `true`).

Di chuyển khóa cũ:

- Telegram: `streamMode` + boolean `streaming` tự động di chuyển tới enum `streaming`.
- Discord: `streamMode` + boolean `streaming` tự động di chuyển tới enum `streaming`.
- Slack: `streamMode` tự động di chuyển tới enum `streaming`; boolean `streaming` tự động di chuyển tới `nativeStreaming`.

### Hành vi thời gian chạy

Telegram:

- Sử dụng Bot API `sendMessage` + `editMessageText`.
- Truyền phát xem trước bị bỏ qua khi truyền phát theo khối Telegram được bật rõ ràng (để tránh truyền phát kép).
- `/reasoning stream` có thể ghi lý do vào xem trước.

Discord:

- Sử dụng gửi + chỉnh sửa tin nhắn xem trước.
- Chế độ `block` sử dụng khối nháp (`draftChunk`).
- Truyền phát xem trước bị bỏ qua khi truyền phát theo khối Discord được bật rõ ràng.

Slack:

- `partial` có thể sử dụng truyền phát gốc Slack (`chat.startStream`/`append`/`stop`) khi có sẵn.
- `block` sử dụng xem trước nháp theo kiểu nối thêm.
- `progress` sử dụng văn bản xem trước trạng thái, sau đó là câu trả lời cuối cùng.