---
summary: 'Sâu sắc: session store + transcripts, lifecycle, và nội bộ (auto)compaction'
read_when:
  - 'Bạn cần gỡ lỗi các trường session ids, transcript JSONL, hoặc sessions.json'
  - >-
    Bạn đang thay đổi hành vi tự động nén hoặc thêm công việc dọn dẹp "trước khi
    nén"
  - Bạn muốn triển khai xóa bộ nhớ hoặc tắt hệ thống im lặng
title: Quản Lý Phiên Làm Việc - Tìm Hiểu Sâu
x-i18n:
  source_path: reference\session-management-compaction.md
  source_hash: 165198b4d850d95eec4bbf86008fe0e86c66191074ac7ed5b5b6b682efd422d0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:20:44.784Z'
---

# Quản lý phiên & Nén dữ liệu (Tìm hiểu sâu)

Tài liệu này giải thích cách OpenClaw quản lý các phiên từ đầu đến cuối:

- **Định tuyến phiên** (cách các tin nhắn đến được ánh xạ tới một `sessionKey`)
- **Kho phiên** (`sessions.json`) và những gì nó theo dõi
- **Lưu trữ bảng ghi** (`*.jsonl`) và cấu trúc của nó
- **Vệ sinh bảng ghi** (các sửa chữa cụ thể của nhà cung cấp trước khi chạy)
- **Giới hạn ngữ cảnh** (cửa sổ ngữ cảnh so với các token được theo dõi)
- **Nén dữ liệu** (nén thủ công + tự động) và nơi để kết nối công việc trước nén
- **Dọn dẹp im lặng** (ví dụ: ghi vào bộ nhớ không nên tạo ra kết quả hiển thị cho người dùng)

Nếu bạn muốn có cái nhìn tổng quan ở mức cao hơn trước tiên, hãy bắt đầu với:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---
## Nguồn thông tin chính: Gateway

OpenClaw được thiết kế xung quanh một **quy trình Gateway** duy nhất sở hữu trạng thái phiên.

- Các giao diện người dùng (ứng dụng macOS, Control UI web, TUI) nên truy vấn Gateway để lấy danh sách phiên và số lượng token.
- Ở chế độ từ xa, các tệp phiên nằm trên máy chủ từ xa; "kiểm tra các tệp Mac cục bộ của bạn" sẽ không phản ánh những gì Gateway đang sử dụng.

---
## Hai lớp lưu trữ dữ liệu

OpenClaw lưu trữ các phiên trong hai lớp:

1. **Kho phiên (`sessions.json`)**
   - Bản đồ khóa/giá trị: `sessionKey -> SessionEntry`
   - Nhỏ, có thể thay đổi, an toàn để chỉnh sửa (hoặc xóa các mục)
   - Theo dõi siêu dữ liệu phiên (id phiên hiện tại, hoạt động cuối cùng, các công tắc, bộ đếm token, v.v.)

2. **Bảng ghi chép (`<sessionId>.jsonl`)**
   - Bảng ghi chép chỉ nối thêm với cấu trúc cây (các mục có `id` + `parentId`)
   - Lưu trữ cuộc trò chuyện thực tế + lệnh gọi công cụ + tóm tắt nén
   - Được sử dụng để xây dựng lại ngữ cảnh mô hình cho các lượt tiếp theo

---
## Vị trí trên đĩa

Trên mỗi agent, trên máy chủ Gateway:

- Store: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transcripts: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram topic sessions: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw giải quyết những vị trí này thông qua `src/config/sessions.ts`.

---
## Bảo trì kho lưu trữ và kiểm soát đĩa

Tính bền vững của phiên có các kiểm soát bảo trì tự động (`session.maintenance`) cho `sessions.json` và các tạo tác bản ghi:

- `mode`: `warn` (mặc định) hoặc `enforce`
- `pruneAfter`: ngưỡng tuổi mục cũ (mặc định `30d`)
- `maxEntries`: giới hạn mục trong `sessions.json` (mặc định `500`)
- `rotateBytes`: xoay vòng `sessions.json` khi quá lớn (mặc định `10mb`)
- `resetArchiveRetention`: thời gian lưu giữ cho các kho lưu trữ bản ghi `*.reset.<timestamp>` (mặc định: giống như `pruneAfter`; `false` vô hiệu hóa dọn dẹp)
- `maxDiskBytes`: ngân sách thư mục phiên tùy chọn
- `highWaterBytes`: mục tiêu tùy chọn sau khi dọn dẹp (mặc định `80%` của `maxDiskBytes`)

Thứ tự thực thi cho dọn dẹp ngân sách đĩa (`mode: "enforce"`):

1. Xóa các tạo tác bản ghi được lưu trữ hoặc mồ côi cũ nhất trước tiên.
2. Nếu vẫn vượt quá mục tiêu, loại bỏ các mục phiên cũ nhất và các tệp bản ghi của chúng.
3. Tiếp tục cho đến khi mức sử dụng ở mức hoặc dưới `highWaterBytes`.

Trong `mode: "warn"`, OpenClaw báo cáo các lần loại bỏ tiềm năng nhưng không thay đổi kho lưu trữ/tệp.

Chạy bảo trì theo yêu cầu:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

---
## Phiên cron và nhật ký chạy

Các lần chạy cron cô lập cũng tạo các mục nhập phiên/bản ghi, và chúng có các điều khiển lưu giữ dành riêng:

- `cron.sessionRetention` (mặc định `24h`) xóa các phiên chạy cron cô lập cũ khỏi kho phiên (`false` vô hiệu hóa).
- `cron.runLog.maxBytes` + `cron.runLog.keepLines` xóa các tệp `~/.openclaw/cron/runs/<jobId>.jsonl` (mặc định: `2_000_000` byte và `2000` dòng).

---
## Khóa phiên (`sessionKey`)

Một `sessionKey` xác định _bạn đang ở trong nhóm cuộc trò chuyện nào_ (định tuyến + cách ly).

Các mẫu phổ biến:

- Trò chuyện chính/trực tiếp (cho mỗi agent): `agent:<agentId>:<mainKey>` (mặc định `main`)
- Nhóm: `agent:<agentId>:<channel>:group:<id>`
- Phòng/kênh (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` hoặc `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (trừ khi bị ghi đè)

Các quy tắc chính thức được ghi lại tại [/concepts/session](/concepts/session).

---
## ID phiên (`sessionId`)

Mỗi `sessionKey` trỏ đến một `sessionId` hiện tại (tệp bảng ghi lại tiếp tục cuộc trò chuyện).

Quy tắc chung:

- **Reset** (`/new`, `/reset`) tạo một `sessionId` mới cho `sessionKey` đó.
- **Reset hàng ngày** (mặc định 4:00 AM giờ địa phương trên máy chủ Gateway) tạo một `sessionId` mới trên tin nhắn tiếp theo sau ranh giới reset.
- **Hết hạn do không hoạt động** (`session.reset.idleMinutes` hoặc `session.idleMinutes` cũ) tạo một `sessionId` mới khi tin nhắn đến sau cửa sổ không hoạt động. Khi cả reset hàng ngày và không hoạt động được cấu hình, cái nào hết hạn trước sẽ thắng.
- **Bảo vệ fork phiên gốc** (`session.parentForkMaxTokens`, mặc định `100000`) bỏ qua fork bảng ghi lại gốc khi phiên gốc đã quá lớn; luồng mới bắt đầu từ đầu. Đặt `0` để tắt.

Chi tiết triển khai: quyết định xảy ra trong `initSessionState()` trong `src/auto-reply/reply/session.ts`.

---
## Lược đồ kho phiên (`sessions.json`)

Kiểu giá trị của kho là `SessionEntry` trong `src/config/sessions.ts`.

Các trường chính (không đầy đủ):

- `sessionId`: ID bảng điểm hiện tại (tên tệp được lấy từ đây trừ khi `sessionFile` được đặt)
- `updatedAt`: dấu thời gian hoạt động cuối cùng
- `sessionFile`: ghi đè đường dẫn bảng điểm rõ ràng tùy chọn
- `chatType`: `direct | group | room` (giúp giao diện người dùng và chính sách gửi)
- `provider`, `subject`, `room`, `space`, `displayName`: siêu dữ liệu để gắn nhãn nhóm/kênh
- Công tắc:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (ghi đè cho mỗi phiên)
- Lựa chọn mô hình:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Bộ đếm token (nỗ lực tốt nhất / phụ thuộc vào nhà cung cấp):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: tần suất nén tự động hoàn thành cho khóa phiên này
- `memoryFlushAt`: dấu thời gian cho lần xóa bộ nhớ trước nén cuối cùng
- `memoryFlushCompactionCount`: số lần nén khi lần xóa cuối cùng chạy

Kho an toàn để chỉnh sửa, nhưng Gateway là cơ quan có thẩm quyền: nó có thể viết lại hoặc tái cấp dữ liệu khi các phiên chạy.

---
## Cấu trúc bản ghi (`*.jsonl`)

Bản ghi được quản lý bởi `@mariozechner/pi-coding-agent` của `SessionManager`.

Tệp là JSONL:

- Dòng đầu tiên: tiêu đề phiên (`type: "session"`, bao gồm `id`, `cwd`, `timestamp`, `parentSession` tùy chọn)
- Sau đó: các mục phiên với `id` + `parentId` (cây)

Các loại mục đáng chú ý:

- `message`: tin nhắn người dùng/trợ lý/kết quả công cụ
- `custom_message`: tin nhắn được tiện ích tiêm vào _có_ nhập vào ngữ cảnh mô hình (có thể ẩn khỏi giao diện người dùng)
- `custom`: trạng thái tiện ích _không_ nhập vào ngữ cảnh mô hình
- `compaction`: tóm tắt nén được lưu trữ với `firstKeptEntryId` và `tokensBefore`
- `branch_summary`: tóm tắt được lưu trữ khi điều hướng nhánh cây

OpenClaw cố ý **không** "sửa chữa" bản ghi; Gateway sử dụng `SessionManager` để đọc/ghi chúng.

---
## Cửa sổ ngữ cảnh so với token được theo dõi

Hai khái niệm khác nhau có ý nghĩa:

1. **Cửa sổ ngữ cảnh của mô hình**: giới hạn cứng cho mỗi mô hình (token hiển thị cho mô hình)
2. **Bộ đếm kho phiên**: thống kê lăn được ghi vào `sessions.json` (được sử dụng cho /status và bảng điều khiển)

Nếu bạn đang điều chỉnh giới hạn:

- Cửa sổ ngữ cảnh đến từ danh mục mô hình (và có thể được ghi đè qua cấu hình).
- `contextTokens` trong kho là giá trị ước tính/báo cáo thời gian chạy; đừng coi nó là một bảo đảm nghiêm ngặt.

Để biết thêm, xem [/token-use](/reference/token-use).

---
## Compaction: nó là gì

Compaction tóm tắt các cuộc trò chuyện cũ hơn thành một mục nhập `compaction` được lưu trữ trong bảng ghi và giữ nguyên các tin nhắn gần đây.

Sau khi compaction, các lượt tiếp theo sẽ thấy:

- Bản tóm tắt compaction
- Các tin nhắn sau `firstKeptEntryId`

Compaction là **persistent** (không giống như session pruning). Xem [/concepts/session-pruning](/concepts/session-pruning).

---
## Khi nào tự động nén xảy ra (Pi runtime)

Trong agent Pi nhúng, tự động nén được kích hoạt trong hai trường hợp:

1. **Khôi phục tràn**: mô hình trả về lỗi tràn ngữ cảnh → nén → thử lại.
2. **Bảo trì ngưỡng**: sau một lượt thành công, khi:

`contextTokens > contextWindow - reserveTokens`

Trong đó:

- `contextWindow` là cửa sổ ngữ cảnh của mô hình
- `reserveTokens` là không gian dự phòng được dành riêng cho các lời nhắc + đầu ra mô hình tiếp theo

Đây là ngữ nghĩa Pi runtime (OpenClaw tiêu thụ các sự kiện, nhưng Pi quyết định khi nào nén).

---
## Cài đặt nén (`reserveTokens`, `keepRecentTokens`)

Cài đặt nén của Pi nằm trong cài đặt Pi:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw also enforces a safety floor for embedded runs:

- If `compaction.reserveTokens < reserveTokensFloor`, OpenClaw bumps it.
- Default floor is `20000` tokens.
- Set `agents.defaults.compaction.reserveTokensFloor: 0` to disable the floor.
- If it’s already higher, OpenClaw leaves it alone.

Why: leave enough headroom for multi-turn “housekeeping” (like memory writes) before compaction becomes unavoidable.

Implementation: `ensurePiCompactionReserveTokens()` in `src/agents/pi-settings.ts`
(called from `src/agents/pi-embedded-runner.ts`).

---
## Các giao diện hiển thị cho người dùng

Bạn có thể quan sát nén dữ liệu và trạng thái phiên thông qua:

- `/status` (trong bất kỳ phiên chat nào)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Chế độ chi tiết: `🧹 Auto-compaction complete` + số lần nén

---
## Dọn dẹp im lặng (`NO_REPLY`)

OpenClaw hỗ trợ các lượt "im lặng" cho các tác vụ nền trong đó người dùng không nên thấy đầu ra trung gian.

Quy ước:

- Trợ lý bắt đầu đầu ra của nó bằng `NO_REPLY` để chỉ ra "không gửi trả lời cho người dùng".
- OpenClaw loại bỏ/ẩn điều này trong lớp gửi.

Kể từ `2026.1.10`, OpenClaw cũng ẩn **truyền phát bản nháp/gõ** khi một phần bắt đầu bằng `NO_REPLY`, do đó các hoạt động im lặng không rò rỉ đầu ra một phần giữa lượt.

---
## "Xả bộ nhớ" trước nén (đã triển khai)

Mục tiêu: trước khi tự động nén xảy ra, chạy một lượt agent im lặng ghi trạng thái bền vững vào đĩa (ví dụ: `memory/YYYY-MM-DD.md` trong không gian làm việc của agent) để nén không thể xóa ngữ cảnh quan trọng.

OpenClaw sử dụng phương pháp **xả trước ngưỡng**:

1. Giám sát mức sử dụng ngữ cảnh phiên.
2. Khi nó vượt quá "ngưỡng mềm" (dưới ngưỡng nén của Pi), chạy chỉ thị "ghi bộ nhớ ngay bây giờ" im lặng cho agent.
3. Sử dụng `NO_REPLY` để người dùng không thấy gì.

Cấu hình (`agents.defaults.compaction.memoryFlush`):

- `enabled` (mặc định: `true`)
- `softThresholdTokens` (mặc định: `4000`)
- `prompt` (tin nhắn người dùng cho lượt xả)
- `systemPrompt` (lời nhắc hệ thống bổ sung được thêm vào cho lượt xả)

Ghi chú:

- Lời nhắc/lời nhắc hệ thống mặc định bao gồm gợi ý `NO_REPLY` để chặn gửi.
- Xả chạy một lần cho mỗi chu kỳ nén (được theo dõi trong `sessions.json`).
- Xả chỉ chạy cho các phiên Pi nhúng (các backend CLI bỏ qua nó).
- Xả bị bỏ qua khi không gian làm việc phiên là chỉ đọc (`workspaceAccess: "ro"` hoặc `"none"`).
- Xem [Bộ nhớ](/concepts/memory) để biết bố cục tệp không gian làm việc và các mẫu ghi.

Pi cũng hiển thị hook `session_before_compact` trong API mở rộng, nhưng logic xả của OpenClaw hiện nằm ở phía Gateway.
## Danh sách kiểm tra khắc phục sự cố

- Session key sai? Bắt đầu với [/concepts/session](/concepts/session) và xác nhận `sessionKey` trong `/status`.
- Store vs transcript không khớp? Xác nhận host Gateway và đường dẫn store từ `openclaw status`.
- Compaction spam? Kiểm tra:
  - model context window (quá nhỏ)
  - cài đặt compaction (`reserveTokens` quá cao so với model window có thể gây ra compaction sớm hơn)
  - tool-result bloat: bật/điều chỉnh session pruning
- Silent turns bị rò rỉ? Xác nhận câu trả lời bắt đầu với `NO_REPLY` (token chính xác) và bạn đang sử dụng bản dựng có bao gồm bản sửa lỗi streaming suppression.