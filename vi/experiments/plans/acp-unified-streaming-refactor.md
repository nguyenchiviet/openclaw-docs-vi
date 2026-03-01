---
summary: >-
  Kế hoạch tái cấu trúc "Holy Grail" cho một unified runtime streaming pipeline
  trên main, subagent, và ACP
owner: onutc
status: draft
last_updated: '2026-02-25'
title: Kế hoạch Tái cấu trúc Unified Runtime Streaming
x-i18n:
  source_path: experiments\plans\acp-unified-streaming-refactor.md
  source_hash: 298025b10056c75136ba4c0935ea27d957df1ad5a41a78ce2942b72030662274
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:53:11.731Z'
---

# Kế hoạch Tái cấu trúc Truyền phát Thời gian chạy Thống nhất

## Mục tiêu

Cung cấp một đường ống truyền phát được chia sẻ cho `main`, `subagent` và `acp` để tất cả các thời gian chạy có hành vi gộp, chia khối, thứ tự giao hàng và khôi phục sự cố giống hệt nhau.
## Tại sao tồn tại điều này

- Hành vi hiện tại được chia tách trên nhiều đường dẫn định hình cụ thể cho từng runtime.
- Các lỗi định dạng/hợp nhất có thể được sửa trong một đường dẫn nhưng vẫn tồn tại trong các đường dẫn khác.
- Tính nhất quán trong giao付, loại bỏ trùng lặp và ngữ nghĩa phục hồi khó có thể được suy luận.
## Kiến trúc mục tiêu

Pipeline duy nhất, adapter dành riêng cho runtime:

1. Các adapter runtime chỉ phát ra các sự kiện chính tắc.
2. Bộ lắp ráp luồng dùng chung hợp nhất và hoàn thiện các sự kiện văn bản/công cụ/trạng thái.
3. Bộ chiếu kênh dùng chung áp dụng chunking/định dạng dành riêng cho kênh một lần.
4. Sổ cái giao hàng dùng chung thực thi ngữ nghĩa gửi/phát lại idempotent.
5. Adapter kênh gửi đi thực thi các lần gửi và ghi lại các điểm kiểm tra giao hàng.

Hợp đồng sự kiện chính tắc:

- `turn_started`
- `text_delta`
- `block_final`
- `tool_started`
- `tool_finished`
- `status`
- `turn_completed`
- `turn_failed`
- `turn_cancelled`
## Luồng công việc

### 1) Hợp đồng truyền phát chính tắc

- Xác định lược đồ sự kiện nghiêm ngặt + xác thực trong lõi.
- Thêm bài kiểm tra hợp đồng adapter để đảm bảo mỗi runtime phát ra các sự kiện tương thích.
- Từ chối các sự kiện runtime không đúng định dạng sớm và cung cấp chẩn đoán có cấu trúc.

### 2) Bộ xử lý luồng dùng chung

- Thay thế logic coalescer/projector dành riêng cho runtime bằng một bộ xử lý.
- Bộ xử lý sở hữu đệm delta văn bản, xả khi rảnh, chia tách khối tối đa và xả hoàn thành.
- Di chuyển phân giải cấu hình ACP/main/subagent vào một trình trợ giúp để ngăn chặn sự trôi dạt.

### 3) Phép chiếu kênh dùng chung

- Giữ các adapter kênh đơn giản: chấp nhận các khối hoàn thiện và gửi.
- Di chuyển các điểm lạ về chunking dành riêng cho Discord sang bộ chiếu kênh chỉ.
- Giữ pipeline không phụ thuộc vào kênh trước khi chiếu.

### 4) Sổ cái giao hàng + phát lại

- Thêm ID giao hàng cho mỗi lượt/mỗi khối.
- Ghi lại các điểm kiểm tra trước và sau khi gửi vật lý.
- Khi khởi động lại, phát lại các khối đang chờ xử lý một cách idempotent và tránh trùng lặp.

### 5) Di chuyển và chuyển đổi

- Giai đoạn 1: chế độ bóng (pipeline mới tính toán đầu ra nhưng đường dẫn cũ gửi; so sánh).
- Giai đoạn 2: chuyển đổi từng runtime (`acp`, sau đó `subagent`, sau đó `main` hoặc ngược lại theo rủi ro).
- Giai đoạn 3: xóa mã truyền phát dành riêng cho runtime cũ.
## Mục tiêu không đạt được

- Không thay đổi mô hình chính sách/quyền ACP trong quá trình tái cấu trúc này.
- Không mở rộng tính năng cụ thể cho kênh ngoài các bản sửa lỗi tương thích chiếu.
- Không thiết kế lại giao thức truyền tải/backend (hợp đồng plugin acpx vẫn giữ nguyên trừ khi cần thiết để đạt được tính chẵn của sự kiện).
## Rủi ro và biện pháp giảm thiểu

- Rủi ro: các hồi quy hành vi trong các đường dẫn agent chính/phụ hiện có.
  Biện pháp giảm thiểu: so sánh chế độ bóng + bài kiểm tra hợp đồng adapter + bài kiểm tra e2e kênh.
- Rủi ro: gửi trùng lặp trong quá trình phục hồi sau sự cố.
  Biện pháp giảm thiểu: ID giao hàng bền vững + phát lại idempotent trong adapter giao hàng.
- Rủi ro: các adapter runtime phân kỳ lại.
  Biện pháp giảm thiểu: bộ kiểm tra hợp đồng chung bắt buộc cho tất cả các adapter.
## Tiêu chí chấp nhận

- Tất cả các runtime vượt qua các bài kiểm tra hợp đồng truyền phát dùng chung.
- Discord ACP/main/subagent tạo ra hành vi khoảng cách/phân đoạn tương đương cho các delta nhỏ.
- Phát lại crash/restart không gửi chunk trùng lặp cho cùng một ID giao hàng.
- Đường dẫn ACP projector/coalescer cũ được loại bỏ.
- Phân giải cấu hình truyền phát được chia sẻ và độc lập với runtime.