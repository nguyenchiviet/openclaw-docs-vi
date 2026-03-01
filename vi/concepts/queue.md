---
summary: Thiết kế hàng đợi lệnh để tuần tự hóa các lần chạy tự động trả lời đến
read_when:
  - Thay đổi thực thi tự động trả lời hoặc đồng thời
title: Hàng đợi Lệnh
x-i18n:
  source_path: concepts\queue.md
  source_hash: 2104c24d200fb4f9620e52a19255cd614ababe19d78f3ee42936dc6d0499b73b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:05.221Z'
---

# Hàng đợi lệnh (2026-01-16)

Chúng tôi tuần tự hóa các lần chạy tự động trả lời đến (tất cả các kênh) thông qua một hàng đợi trong quy trình nhỏ để ngăn chặn nhiều lần chạy agent va chạm, đồng thời vẫn cho phép song song hóa an toàn trên các phiên.
## Tại sao

- Các lần chạy tự động trả lời có thể tốn kém (các lệnh gọi LLM) và có thể xung đột khi nhiều tin nhắn đến gần nhau.
- Tuần tự hóa tránh cạnh tranh cho các tài nguyên được chia sẻ (tệp phiên, nhật ký, CLI stdin) và giảm khả năng giới hạn tốc độ thượng nguồn.
## Cách hoạt động

- Một hàng đợi FIFO nhận biết lane xả từng lane với giới hạn đồng thời có thể cấu hình (mặc định 1 cho các lane không được cấu hình; main mặc định là 4, subagent là 8).
- `runEmbeddedPiAgent` xếp hàng theo **session key** (lane `session:<key>`) để đảm bảo chỉ có một lần chạy hoạt động trên mỗi phiên.
- Mỗi lần chạy phiên sau đó được xếp hàng vào một **lane toàn cục** (`main` theo mặc định) để tổng thể song song hóa bị giới hạn bởi `agents.defaults.maxConcurrent`.
- Khi bật ghi nhật ký chi tiết, các lần chạy được xếp hàng sẽ phát ra một thông báo ngắn nếu chúng chờ đợi hơn ~2 giây trước khi bắt đầu.
- Các chỉ báo gõ phím vẫn kích hoạt ngay lập tức khi xếp hàng (khi được hỗ trợ bởi kênh) để trải nghiệm người dùng không thay đổi trong khi chúng ta chờ lượt của mình.
## Chế độ hàng đợi (mỗi kênh)

Các tin nhắn đến có thể điều khiển lần chạy hiện tại, chờ lượt tiếp theo, hoặc làm cả hai:

- `steer`: tiêm ngay vào lần chạy hiện tại (hủy các lệnh gọi công cụ đang chờ sau ranh giới công cụ tiếp theo). Nếu không truyền phát, quay lại lượt tiếp theo.
- `followup`: xếp hàng cho lượt agent tiếp theo sau khi lần chạy hiện tại kết thúc.
- `collect`: hợp nhất tất cả các tin nhắn xếp hàng thành **một** lượt tiếp theo duy nhất (mặc định). Nếu các tin nhắn nhắm đến các kênh/luồng khác nhau, chúng sẽ được xử lý riêng lẻ để bảo toàn định tuyến.
- `steer-backlog` (hay `steer+backlog`): điều khiển ngay **và** bảo toàn tin nhắn cho lượt tiếp theo.
- `interrupt` (cũ): hủy lần chạy hoạt động cho phiên đó, sau đó chạy tin nhắn mới nhất.
- `queue` (bí danh cũ): giống như `steer`.

Điều khiển-backlog có nghĩa là bạn có thể nhận được phản hồi lượt tiếp theo sau lần chạy được điều khiển, vì vậy
các bề mặt truyền phát có thể trông giống như bản sao. Ưu tiên `collect`/`steer` nếu bạn muốn
một phản hồi cho mỗi tin nhắn đến.
Gửi `/queue collect` dưới dạng lệnh độc lập (mỗi phiên) hoặc đặt `messages.queue.byChannel.discord: "collect"`.

Giá trị mặc định (khi không được đặt trong cấu hình):

- Tất cả các bề mặt → `collect`

Cấu hình toàn cục hoặc mỗi kênh thông qua `messages.queue`:

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```
## Tùy chọn hàng đợi

Các tùy chọn áp dụng cho `followup`, `collect`, và `steer-backlog` (và cho `steer` khi nó quay lại followup):

- `debounceMs`: chờ yên tĩnh trước khi bắt đầu lượt followup (ngăn chặn "tiếp tục, tiếp tục").
- `cap`: số lượng tin nhắn tối đa được xếp hàng đợi trên mỗi phiên.
- `drop`: chính sách tràn (`old`, `new`, `summarize`).

Summarize giữ một danh sách dấu đầu dòng ngắn các tin nhắn bị loại bỏ và chèn nó dưới dạng lời nhắc followup tổng hợp.
Giá trị mặc định: `debounceMs: 1000`, `cap: 20`, `drop: summarize`.
## Ghi đè theo phiên

- Gửi `/queue <mode>` dưới dạng lệnh độc lập để lưu trữ chế độ cho phiên hiện tại.
- Các tùy chọn có thể được kết hợp: `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` hoặc `/queue reset` xóa ghi đè phiên.
## Phạm vi và đảm bảo

- Áp dụng cho các lần chạy agent tự động trả lời trên tất cả các kênh inbound sử dụng pipeline trả lời gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, v.v.).
- Lane mặc định (`main`) là quy trình toàn cục cho inbound + heartbeat chính; đặt `agents.defaults.maxConcurrent` để cho phép nhiều phiên song song.
- Các lane bổ sung có thể tồn tại (ví dụ: `cron`, `subagent`) để các công việc nền có thể chạy song song mà không chặn các trả lời inbound.
- Các lane theo phiên đảm bảo rằng chỉ một lần chạy agent chạm vào một phiên nhất định tại một thời điểm.
- Không có phụ thuộc bên ngoài hoặc luồng worker nền; TypeScript thuần + promises.
## Khắc phục sự cố

- Nếu các lệnh dường như bị treo, hãy bật nhật ký chi tiết và tìm các dòng "queued for …ms" để xác nhận hàng đợi đang được xử lý.
- Nếu bạn cần độ sâu của hàng đợi, hãy bật nhật ký chi tiết và theo dõi các dòng thời gian hàng đợi.