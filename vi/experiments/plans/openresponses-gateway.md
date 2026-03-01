---
summary: >-
  Kế hoạch: Thêm endpoint OpenResponses /v1/responses và loại bỏ chat
  completions một cách sạch sẽ
read_when:
  - Thiết kế hoặc triển khai hỗ trợ gateway `/v1/responses`
  - Lập kế hoạch di chuyển từ khả năng tương thích Chat Completions
owner: openclaw
status: draft
last_updated: '2026-01-19'
title: Kế hoạch OpenResponses Gateway
x-i18n:
  source_path: experiments\plans\openresponses-gateway.md
  source_hash: c112e9eb5077ff7c88b1bd1b28d57f0526280207211e1f19ada88ebf212b51d1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:53:15.950Z'
---

# Kế hoạch tích hợp OpenResponses Gateway

## Bối cảnh

Gateway hiện tại cung cấp một endpoint Chat Completions tương thích với OpenAI tối thiểu tại
`/v1/chat/completions` (xem [OpenAI Chat Completions](/gateway/openai-http-api)).

Open Responses là một tiêu chuẩn suy luận mở dựa trên OpenAI Responses API. Nó được thiết kế
cho các quy trình làm việc của agent và sử dụng đầu vào dựa trên mục cộng với các sự kiện truyền phát ngữ nghĩa. Thông số kỹ thuật OpenResponses định nghĩa `/v1/responses`, không phải `/v1/chat/completions`.
## Mục tiêu

- Thêm một endpoint `/v1/responses` tuân theo ngữ nghĩa OpenResponses.
- Giữ Chat Completions như một lớp tương thích dễ dàng vô hiệu hóa và cuối cùng loại bỏ.
- Chuẩn hóa xác thực và phân tích cú pháp với các schema cô lập, có thể tái sử dụng.
## Mục tiêu không phải

- Tính năng OpenResponses đầy đủ trong lần đầu tiên (hình ảnh, tệp, công cụ được lưu trữ).
- Thay thế logic thực thi agent nội bộ hoặc điều phối công cụ.
- Thay đổi hành vi `/v1/chat/completions` hiện có trong giai đoạn đầu tiên.
## Tóm tắt Nghiên cứu

Nguồn: OpenResponses OpenAPI, trang đặc tả OpenResponses và bài viết trên blog Hugging Face.

Các điểm chính được trích xuất:

- `POST /v1/responses` chấp nhận các trường `CreateResponseBody` như `model`, `input` (chuỗi hoặc
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` và
  `max_tool_calls`.
- `ItemParam` là một liên hợp phân biệt của:
  - các mục `message` với vai trò `system`, `developer`, `user`, `assistant`
  - `function_call` và `function_call_output`
  - `reasoning`
  - `item_reference`
- Các phản hồi thành công trả về một `ResponseResource` với `object: "response"`, `status` và
  các mục `output`.
- Truyền phát sử dụng các sự kiện ngữ nghĩa như:
  - `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  - `response.output_item.added`, `response.output_item.done`
  - `response.content_part.added`, `response.content_part.done`
  - `response.output_text.delta`, `response.output_text.done`
- Đặc tả yêu cầu:
  - `Content-Type: text/event-stream`
  - `event:` phải khớp với trường JSON `type`
  - sự kiện cuối cùng phải là `[DONE]` theo nghĩa đen
- Các mục lý luận có thể hiển thị `content`, `encrypted_content` và `summary`.
- Các ví dụ HF bao gồm `OpenResponses-Version: latest` trong các yêu cầu (tiêu đề tùy chọn).
## Kiến trúc Đề xuất

- Thêm `src/gateway/open-responses.schema.ts` chứa các schema Zod chỉ (không có nhập Gateway).
- Thêm `src/gateway/openresponses-http.ts` (hoặc `open-responses-http.ts`) cho `/v1/responses`.
- Giữ `src/gateway/openai-http.ts` nguyên vẹn như một bộ điều hợp tương thích ngược.
- Thêm cấu hình `gateway.http.endpoints.responses.enabled` (mặc định `false`).
- Giữ `gateway.http.endpoints.chatCompletions.enabled` độc lập; cho phép cả hai điểm cuối được
  bật tắt riêng biệt.
- Phát hành cảnh báo khởi động khi Chat Completions được bật để báo hiệu trạng thái cũ.
## Đường dẫn Loại bỏ cho Chat Completions

- Duy trì ranh giới mô-đun nghiêm ngặt: không có các loại lược đồ được chia sẻ giữa phản hồi và hoàn thành trò chuyện.
- Làm cho Chat Completions tùy chọn theo cấu hình để có thể tắt mà không cần thay đổi mã.
- Cập nhật tài liệu để gắn nhãn Chat Completions là kế thừa sau khi `/v1/responses` ổn định.
- Bước tương lai tùy chọn: ánh xạ các yêu cầu Chat Completions tới trình xử lý Phản hồi để có đường dẫn loại bỏ đơn giản hơn.
## Tập hợp hỗ trợ Giai đoạn 1

- Chấp nhận `input` dưới dạng chuỗi hoặc `ItemParam[]` với các vai trò tin nhắn và `function_call_output`.
- Trích xuất các tin nhắn hệ thống và nhà phát triển vào `extraSystemPrompt`.
- Sử dụng `user` hoặc `function_call_output` gần đây nhất làm tin nhắn hiện tại cho các lần chạy agent.
- Từ chối các phần nội dung không được hỗ trợ (hình ảnh/tệp) bằng `invalid_request_error`.
- Trả về một tin nhắn trợ lý duy nhất với nội dung `output_text`.
- Trả về `usage` với các giá trị bằng không cho đến khi tính toán token được kết nối.
## Chiến lược Xác thực (Không SDK)

- Triển khai các schema Zod cho tập hợp được hỗ trợ của:
  - `CreateResponseBody`
  - `ItemParam` + các liên hợp phần nội dung tin nhắn
  - `ResponseResource`
  - Các hình dạng sự kiện truyền phát được sử dụng bởi Gateway
- Giữ các schema trong một mô-đun duy nhất, cô lập để tránh sai lệch và cho phép codegen trong tương lai.
## Triển khai Truyền phát (Giai đoạn 1)

- Dòng SSE với cả `event:` và `data:`.
- Trình tự bắt buộc (tối thiểu khả thi):
  - `response.created`
  - `response.output_item.added`
  - `response.content_part.added`
  - `response.output_text.delta` (lặp lại khi cần)
  - `response.output_text.done`
  - `response.content_part.done`
  - `response.completed`
  - `[DONE]`
## Kiểm tra và Kế hoạch Xác minh

- Thêm phạm vi e2e cho `/v1/responses`:
  - Xác thực bắt buộc
  - Hình dạng phản hồi không phải stream
  - Sắp xếp sự kiện stream và `[DONE]`
  - Định tuyến phiên với tiêu đề và `user`
- Giữ `src/gateway/openai-http.test.ts` không thay đổi.
- Thủ công: curl tới `/v1/responses` với `stream: true` và xác minh sắp xếp sự kiện và terminal
  `[DONE]`.
## Cập nhật Tài liệu (Tiếp theo)

- Thêm một trang tài liệu mới cho cách sử dụng và ví dụ về `/v1/responses`.
- Cập nhật `/gateway/openai-http-api` với ghi chú về phiên bản cũ và con trỏ đến `/v1/responses`.