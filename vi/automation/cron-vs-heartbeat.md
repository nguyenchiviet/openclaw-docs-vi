---
summary: Hướng dẫn lựa chọn giữa heartbeat và cron jobs cho tự động hóa
read_when:
  - Quyết định cách lên lịch các tác vụ định kỳ
  - Thiết lập giám sát chạy nền hoặc thông báo
  - Tối ưu hóa việc sử dụng token cho các kiểm tra định kỳ
title: Cron so với Heartbeat
x-i18n:
  source_path: automation\cron-vs-heartbeat.md
  source_hash: f41e6321e67971407b9e51e8288b6215d56f0008ee5a58713789eadb6e56cba9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:08:07.936Z'
---

# Cron vs Heartbeat: Khi nào nên sử dụng từng loại

Cả heartbeat và cron job đều cho phép bạn chạy các tác vụ theo lịch trình. Hướng dẫn này giúp bạn chọn cơ chế phù hợp cho trường hợp sử dụng của mình.

## Hướng dẫn ra quyết định nhanh

| Trường hợp sử dụng                          | Đề xuất             | Lý do                                      |
| :----------------------------------------- | :------------------ | :----------------------------------------- |
| Kiểm tra hộp thư đến mỗi 30 phút            | Heartbeat           | Xử lý theo lô với các kiểm tra khác, nhận biết ngữ cảnh |
| Gửi báo cáo hàng ngày đúng 9 giờ sáng       | Cron (cô lập)       | Cần thời gian chính xác                   |
| Theo dõi lịch cho các sự kiện sắp tới       | Heartbeat           | Phù hợp tự nhiên cho nhận thức định kỳ    |
| Chạy phân tích chuyên sâu hàng tuần         | Cron (cô lập)       | Tác vụ độc lập, có thể sử dụng mô hình khác |
| Nhắc nhở tôi trong 20 phút                  | Cron (chính, `--at`) | Một lần với thời gian chính xác           |
| Kiểm tra tình trạng dự án nền              | Heartbeat           | Tận dụng chu kỳ hiện có                   |
## Heartbeat: Nhận biết định kỳ

Heartbeat chạy trong **phiên chính** theo một khoảng thời gian đều đặn (mặc định: 30 phút). Chúng được thiết kế để agent kiểm tra mọi thứ và đưa ra bất kỳ điều gì quan trọng.

### Khi nào nên sử dụng heartbeat

- **Nhiều kiểm tra định kỳ**: Thay vì 5 cron job riêng biệt kiểm tra hộp thư đến, lịch, thời tiết, thông báo và trạng thái dự án, một heartbeat duy nhất có thể gộp tất cả các tác vụ này.
- **Quyết định dựa trên ngữ cảnh**: Agent có toàn bộ ngữ cảnh của phiên chính, vì vậy nó có thể đưa ra các quyết định thông minh về việc điều gì khẩn cấp và điều gì có thể chờ đợi.
- **Tính liên tục của cuộc trò chuyện**: Các lần chạy heartbeat chia sẻ cùng một phiên, vì vậy agent ghi nhớ các cuộc trò chuyện gần đây và có thể theo dõi một cách tự nhiên.
- **Giám sát chi phí thấp**: Một heartbeat thay thế nhiều tác vụ thăm dò nhỏ.

### Ưu điểm của heartbeat

- **Gộp nhiều kiểm tra**: Một lượt của agent có thể xem xét hộp thư đến, lịch và thông báo cùng lúc.
- **Giảm các lệnh gọi API**: Một heartbeat duy nhất rẻ hơn 5 cron job riêng biệt.
- **Nhận biết ngữ cảnh**: Agent biết bạn đang làm gì và có thể ưu tiên phù hợp.
- **Ngăn chặn thông minh**: Nếu không có gì cần chú ý, agent trả lời `HEARTBEAT_OK` và không có tin nhắn nào được gửi.
- **Thời gian tự nhiên**: Thay đổi nhẹ dựa trên tải hàng đợi, điều này phù hợp với hầu hết các tác vụ giám sát.

### Ví dụ về heartbeat: Danh sách kiểm tra HEARTBEAT.md

```md
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

Agent đọc thông tin này trong mỗi nhịp tim và xử lý tất cả các mục trong một lượt.

### Cấu hình nhịp tim

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // interval
        target: "last", // explicit alert delivery target (default is "none")
        activeHours: { start: "08:00", end: "22:00" }, // optional
      },
    },
  },
}
```

Xem [Nhịp tim](/gateway/heartbeat) để biết cấu hình đầy đủ.

## Cron: Lập lịch chính xác

Các tác vụ Cron chạy vào những thời điểm chính xác và có thể chạy trong các phiên riêng biệt mà không ảnh hưởng đến ngữ cảnh chính. Các lịch trình định kỳ vào đầu giờ được tự động phân tán bằng một độ lệch xác định cho mỗi tác vụ trong khoảng thời gian 0-5 phút.

### Khi nào nên sử dụng cron

- **Yêu cầu thời gian chính xác**: "Gửi cái này vào 9:00 sáng mỗi thứ Hai" (không phải "khoảng 9 giờ").
- **Tác vụ độc lập**: Các tác vụ không cần ngữ cảnh hội thoại.
- **Mô hình/tư duy khác**: Phân tích chuyên sâu yêu cầu một mô hình mạnh mẽ hơn.
- **Nhắc nhở một lần**: "Nhắc tôi trong 20 phút" với `--at`.
- **Tác vụ ồn ào/thường xuyên**: Các tác vụ có thể làm lộn xộn lịch sử phiên chính.
- **Kích hoạt bên ngoài**: Các tác vụ nên chạy độc lập với việc agent có đang hoạt động hay không.

### Ưu điểm của Cron

- **Thời gian chính xác**: Biểu thức cron 5 trường hoặc 6 trường (giây) có hỗ trợ múi giờ.
- **Phân tán tải tích hợp**: các lịch trình định kỳ vào đầu giờ được mặc định phân tán tối đa 5 phút.
- **Kiểm soát từng tác vụ**: ghi đè phân tán bằng `--stagger <duration>` hoặc buộc thời gian chính xác bằng `--exact`.
- **Cách ly phiên**: Chạy trong `cron:<jobId>` mà không làm ô nhiễm lịch sử chính.
- **Ghi đè mô hình**: Sử dụng mô hình rẻ hơn hoặc mạnh mẽ hơn cho mỗi tác vụ.
- **Kiểm soát phân phối**: Các tác vụ riêng biệt mặc định là `announce` (tóm tắt); chọn `none` khi cần.
- **Phân phối ngay lập tức**: Chế độ thông báo đăng trực tiếp mà không cần chờ nhịp tim.
- **Không cần ngữ cảnh agent**: Chạy ngay cả khi phiên chính không hoạt động hoặc đã được nén.
- **Hỗ trợ một lần**: `--at` cho các dấu thời gian tương lai chính xác.

### Ví dụ Cron: Bản tóm tắt buổi sáng hàng ngày

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```
Điều này chạy chính xác vào lúc 7:00 sáng giờ New York, sử dụng Opus để đảm bảo chất lượng và thông báo tóm tắt trực tiếp đến WhatsApp.

### Ví dụ Cron: Nhắc nhở một lần

``__OC_I19N_0000__``

Xem [Cron jobs](__OC_I19N_0001__) để biết tài liệu tham khảo CLI đầy đủ.
## Lưu đồ Quyết định

```
Does the task need to run at an EXACT time?
  YES -> Use cron
  NO  -> Continue...

Does the task need isolation from main session?
  YES -> Use cron (isolated)
  NO  -> Continue...

Can this task be batched with other periodic checks?
  YES -> Use heartbeat (add to HEARTBEAT.md)
  NO  -> Use cron

Is this a one-shot reminder?
  YES -> Use cron with --at
  NO  -> Continue...

Does it need a different model or thinking level?
  YES -> Use cron (isolated) with --model/--thinking
  NO  -> Use heartbeat
```
## Kết hợp cả hai

Thiết lập hiệu quả nhất sử dụng **cả hai**:

1. **Heartbeat** xử lý việc giám sát định kỳ (hộp thư đến, lịch, thông báo) trong một lượt xử lý theo lô cứ sau 30 phút.
2. **Cron** xử lý các lịch trình chính xác (báo cáo hàng ngày, đánh giá hàng tuần) và các lời nhắc một lần.

### Ví dụ: Thiết lập tự động hóa hiệu quả

**HEARTBEAT.md** (kiểm tra mỗi 30 phút):

```md
# Heartbeat checklist

- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Cron jobs** (precise timing):

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --announce

# Weekly project review on Mondays at 9am
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```
## Lobster: Quy trình làm việc xác định có phê duyệt

Lobster là môi trường chạy quy trình làm việc dành cho **chuỗi công cụ nhiều bước** cần thực thi xác định và phê duyệt rõ ràng. Sử dụng nó khi tác vụ không chỉ là một lượt agent duy nhất và bạn muốn một quy trình làm việc có thể tiếp tục với các điểm kiểm tra của con người.

### Khi Lobster phù hợp

-   **Tự động hóa nhiều bước**: Bạn cần một chuỗi lệnh gọi công cụ cố định, chứ không phải một lời nhắc một lần.
-   **Cổng phê duyệt**: Các tác dụng phụ nên tạm dừng cho đến khi bạn phê duyệt, sau đó tiếp tục.
-   **Các lần chạy có thể tiếp tục**: Tiếp tục một quy trình làm việc đã tạm dừng mà không cần chạy lại các bước trước đó.

### Cách nó kết hợp với heartbeat và cron

-   **Heartbeat/cron** quyết định _khi nào_ một lần chạy diễn ra.
-   **Lobster** xác định _những bước nào_ diễn ra một khi lần chạy bắt đầu.

Đối với các quy trình làm việc theo lịch trình, sử dụng cron hoặc heartbeat để kích hoạt một lượt agent gọi Lobster.
Đối với các quy trình làm việc đặc biệt, gọi Lobster trực tiếp.

### Lưu ý vận hành (từ mã nguồn)

-   Lobster chạy dưới dạng một **tiến trình con cục bộ** (`lobster` CLI) ở chế độ công cụ và trả về một **JSON envelope**.
-   Nếu công cụ trả về `needs_approval`, bạn tiếp tục với cờ `resumeToken` và __OC_I19N_0003__.
-   Công cụ này là một **plugin tùy chọn**; kích hoạt nó một cách bổ sung thông qua `tools.alsoAllow: ["lobster"]` (được khuyến nghị).
-   Lobster mong đợi CLI `lobster` có sẵn trên `PATH`.

Xem [Lobster](/tools/lobster) để biết cách sử dụng đầy đủ và ví dụ.
## Phiên chính so với Phiên cô lập

Cả heartbeat và cron đều có thể tương tác với phiên chính, nhưng theo những cách khác nhau:

|         | Heartbeat                       | Cron (chính)             | Cron (cô lập)              |
| ------- | ------------------------------- | ------------------------ | -------------------------- |
| Phiên   | Chính                           | Chính (qua sự kiện hệ thống) | `cron:<jobId>`             |
| Lịch sử | Chia sẻ                         | Chia sẻ                  | Mới mỗi lần chạy           |
| Ngữ cảnh | Đầy đủ                          | Đầy đủ                   | Không có (bắt đầu sạch)    |
| Mô hình | Mô hình phiên chính             | Mô hình phiên chính      | Có thể ghi đè              |
| Đầu ra  | Được gửi nếu không `HEARTBEAT_OK` | Lời nhắc heartbeat + sự kiện | Thông báo tóm tắt (mặc định) |

### Khi nào nên sử dụng cron phiên chính

Sử dụng `--session main` với `--system-event` khi bạn muốn:

- Lời nhắc/sự kiện xuất hiện trong ngữ cảnh phiên chính
- agent xử lý nó trong heartbeat tiếp theo với ngữ cảnh đầy đủ
- Không có chạy cô lập riêng biệt

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

### Khi nào nên sử dụng cron cô lập
Sử dụng `--session isolated` khi bạn muốn:

- Một khởi đầu mới không có ngữ cảnh trước đó
- Các cài đặt mô hình hoặc tư duy khác
- Thông báo tóm tắt trực tiếp đến một kênh
- Lịch sử không làm lộn xộn phiên chính

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --announce
```
## Các Yếu Tố Chi Phí

| Cơ Chế          | Đặc Điểm Chi Phí                                            |
| --------------- | ------------------------------------------------------- |
| Heartbeat       | Một lượt mỗi N phút; mở rộng theo kích thước của HEARTBEAT.md |
| Cron (chính)    | Thêm sự kiện vào heartbeat tiếp theo (không có lượt riêng biệt)         |
| Cron (riêng biệt) | Một lượt agent đầy đủ cho mỗi tác vụ; có thể sử dụng mô hình rẻ hơn          |

**Mẹo**:

- Giữ __OC_I19N_0000__ nhỏ để giảm thiểu chi phí token.
- Gộp các kiểm tra tương tự vào heartbeat thay vì nhiều cron job.
- Sử dụng __OC_I19N_0001__ trên heartbeat nếu bạn chỉ muốn xử lý nội bộ.
- Sử dụng cron riêng biệt với mô hình rẻ hơn cho các tác vụ định kỳ.
## Liên quan

- [Nhịp tim](/gateway/heartbeat) - cấu hình nhịp tim đầy đủ
- [Tác vụ định kỳ](/automation/cron-jobs) - tham chiếu CLI và API cron đầy đủ
- [Hệ thống](/cli/system) - sự kiện hệ thống + điều khiển nhịp tim
