---
summary: >-
  Cú pháp chỉ thị cho /think + /verbose và cách chúng ảnh hưởng đến lý luận của
  mô hình


  /think: Kích hoạt chế độ suy nghĩ sâu, cho phép mô hình thực hiện suy luận chi
  tiết trước khi đưa ra câu trả lời cuối cùng.


  /verbose: Bật chế độ chi tiết, hiển thị toàn bộ quá trình suy nghĩ và lý luận
  của mô hình, bao gồm các bước trung gian.


  Khi sử dụng cùng nhau (/think + /verbose):

  - Mô hình sẽ thực hiện suy luận sâu hơn

  - Tất cả các bước lý luận sẽ được hiển thị rõ ràng

  - Người dùng có thể theo dõi quá trình tư duy từng bước

  - Thích hợp cho các vấn đề phức tạp yêu cầu giải thích chi tiết


  Cú pháp sử dụng:

  ```

  /think /verbose [câu hỏi hoặc yêu cầu của bạn]

  ```


  Ảnh hưởng đến lý luận:

  - Tăng độ chính xác của câu trả lời

  - Cải thiện khả năng giải quyết vấn đề phức tạp

  - Cung cấp tính minh bạch cao hơn trong quá trình suy nghĩ

  - Tiêu tốn nhiều tài nguyên tính toán hơn
read_when:
  - Điều chỉnh tư duy hoặc phân tích chỉ thị chi tiết hoặc giá trị mặc định
title: Mức Độ Suy Nghĩ
x-i18n:
  source_path: tools\thinking.md
  source_hash: 98900f36c017606dad591e0615341cecab3f54788718d7e92895c29648deea71
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:22.049Z'
---

# Mức Thinking (/think directives)

## Chức năng

- Inline directive trong bất kỳ body nào: `/t <level>`, `/think:<level>`, hoặc `/thinking <level>`.
- Mức (bí danh): `off | minimal | low | medium | high | xhigh` (chỉ mô hình GPT-5.2 + Codex)
  - minimal → "think"
  - low → "think hard"
  - medium → "think harder"
  - high → "ultrathink" (ngân sách tối đa)
  - xhigh → "ultrathink+" (chỉ mô hình GPT-5.2 + Codex)
  - `x-high`, `x_high`, `extra-high`, `extra high`, và `extra_high` ánh xạ tới `xhigh`.
  - `highest`, `max` ánh xạ tới `high`.
- Ghi chú nhà cung cấp:
  - Z.AI (`zai/*`) chỉ hỗ trợ thinking nhị phân (`on`/`off`). Bất kỳ mức nào không phải `off` được coi là `on` (ánh xạ tới `low`).
## Thứ tự phân giải

1. Chỉ thị nội tuyến trên tin nhắn (chỉ áp dụng cho tin nhắn đó).
2. Ghi đè phiên (được đặt bằng cách gửi tin nhắn chỉ chứa chỉ thị).
3. Mặc định toàn cục (`agents.defaults.thinkingDefault` trong cấu hình).
4. Dự phòng: thấp cho các mô hình có khả năng suy luận; tắt nếu không.
## Đặt mặc định phiên

- Gửi một tin nhắn chỉ chứa chỉ thị (khoảng trắng được phép), ví dụ `/think:medium` hoặc `/t high`.
- Điều đó sẽ được giữ lại cho phiên hiện tại (theo người gửi theo mặc định); được xóa bởi `/think:off` hoặc khi phiên hết thời gian chờ.
- Tin nhắn xác nhận được gửi (`Thinking level set to high.` / `Thinking disabled.`). Nếu mức độ không hợp lệ (ví dụ `/thinking big`), lệnh bị từ chối kèm theo gợi ý và trạng thái phiên không thay đổi.
- Gửi `/think` (hoặc `/think:`) mà không có đối số để xem mức độ suy nghĩ hiện tại.
## Ứng dụng theo agent

- **Embedded Pi**: mức độ được phân giải được chuyển đến runtime agent Pi trong quy trình.
## Các chỉ thị chi tiết (/verbose hoặc /v)

- Mức độ: `on` (tối thiểu) | `full` | `off` (mặc định).
- Chỉ thị duy nhất chuyển đổi verbose của phiên và trả lời `Verbose logging enabled.` / `Verbose logging disabled.`; các mức không hợp lệ trả về gợi ý mà không thay đổi trạng thái.
- `/verbose off` lưu trữ ghi đè phiên rõ ràng; xóa nó thông qua Giao diện người dùng Phiên bằng cách chọn `inherit`.
- Chỉ thị nội tuyến chỉ ảnh hưởng đến tin nhắn đó; các giá trị mặc định phiên/toàn cục áp dụng nếu không.
- Gửi `/verbose` (hoặc `/verbose:`) mà không có đối số để xem mức verbose hiện tại.
- Khi verbose bật, các agent phát ra kết quả công cụ có cấu trúc (Pi, các agent JSON khác) gửi lại từng lệnh gọi công cụ dưới dạng tin nhắn chỉ siêu dữ liệu, có tiền tố là `<emoji> <tool-name>: <arg>` khi có sẵn (đường dẫn/lệnh). Các bản tóm tắt công cụ này được gửi ngay khi mỗi công cụ bắt đầu (các bong bóng riêng biệt), không phải là delta truyền phát.
- Các bản tóm tắt lỗi công cụ vẫn hiển thị ở chế độ bình thường, nhưng các hậu tố chi tiết lỗi thô được ẩn trừ khi verbose là `on` hoặc `full`.
- Khi verbose là `full`, các đầu ra công cụ cũng được chuyển tiếp sau khi hoàn thành (bong bóng riêng biệt, cắt ngắn thành độ dài an toàn). Nếu bạn chuyển đổi `/verbose on|full|off` trong khi một lần chạy đang diễn ra, các bong bóng công cụ tiếp theo sẽ tuân theo cài đặt mới.
## Khả năng hiển thị lý luận (/reasoning)

- Mức độ: `on|off|stream`.
- Chỉ thị tin nhắn chuyển đổi xem các khối suy nghĩ có được hiển thị trong câu trả lời hay không.
- Khi được bật, lý luận được gửi dưới dạng **tin nhắn riêng biệt** có tiền tố `Reasoning:`.
- `stream` (chỉ Telegram): truyền phát lý luận vào bong bóng nháp Telegram trong khi câu trả lời đang được tạo, sau đó gửi câu trả lời cuối cùng mà không có lý luận.
- Bí danh: `/reason`.
- Gửi `/reasoning` (hoặc `/reasoning:`) mà không có đối số để xem mức lý luận hiện tại.
## Liên quan

- Tài liệu chế độ nâng cao nằm trong [Chế độ nâng cao](/tools/elevated).
## Nhịp tim

- Nội dung kiểm tra nhịp tim là heartbeat prompt được cấu hình (mặc định: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Các chỉ thị nội tuyến trong một tin nhắn nhịp tim được áp dụng như bình thường (nhưng tránh thay đổi các giá trị mặc định của phiên từ các nhịp tim).
- Gửi nhịp tim mặc định chỉ gửi payload cuối cùng. Để cũng gửi tin nhắn `Reasoning:` riêng biệt (khi có sẵn), hãy đặt `agents.defaults.heartbeat.includeReasoning: true` hoặc `agents.list[].heartbeat.includeReasoning: true` cho từng agent.
## Giao diện trò chuyện web

- Bộ chọn suy nghĩ của trò chuyện web phản ánh mức độ được lưu trữ của phiên từ kho lưu trữ/cấu hình phiên inbound khi trang tải.
- Chọn một mức độ khác chỉ áp dụng cho tin nhắn tiếp theo (`thinkingOnce`); sau khi gửi, bộ chọn sẽ quay lại mức độ phiên được lưu trữ.
- Để thay đổi mặc định phiên, gửi một chỉ thị `/think:<level>` (như trước); bộ chọn sẽ phản ánh nó sau lần tải lại tiếp theo.