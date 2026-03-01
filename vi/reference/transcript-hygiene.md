---
summary: >-
  Tham chiếu: các quy tắc vệ sinh và sửa chữa bản ghi lời nói dành riêng cho
  từng nhà cung cấp
read_when:
  - >-
    Bạn đang gỡ lỗi các yêu cầu bị từ chối của nhà cung cấp liên quan đến hình
    dạng bản ghi.
  - >-
    Bạn đang thay đổi logic làm sạch bản ghi âm hoặc logic sửa chữa lệnh gọi
    công cụ
  - >-
    Bạn đang điều tra sự không khớp giữa các ID gọi công cụ trên các nhà cung
    cấp khác nhau
title: Vệ sinh Bản ghi
x-i18n:
  source_path: reference\transcript-hygiene.md
  source_hash: 217afafb693cf89651e8fa361252f7b5c197feb98d20be4697a83e6dedc0ec3f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:22:38.971Z'
---

# Vệ sinh Transcript (Sửa chữa Nhà cung cấp)

Tài liệu này mô tả **các sửa chữa dành riêng cho nhà cung cấp** được áp dụng cho các transcript trước khi chạy
(xây dựng ngữ cảnh mô hình). Đây là các **điều chỉnh trong bộ nhớ** được sử dụng để đáp ứng các yêu cầu nghiêm ngặt của nhà cung cấp. Các bước vệ sinh này **không** viết lại transcript JSONL được lưu trữ
trên đĩa; tuy nhiên, một lần vượt qua sửa chữa tệp phiên riêng biệt có thể viết lại các tệp JSONL bị hỏng
bằng cách loại bỏ các dòng không hợp lệ trước khi phiên được tải. Khi sửa chữa xảy ra, tệp gốc
được sao lưu cùng với tệp phiên.

Phạm vi bao gồm:

- Vệ sinh ID lệnh gọi công cụ
- Xác thực đầu vào lệnh gọi công cụ
- Sửa chữa ghép nối kết quả công cụ
- Xác thực / sắp xếp lượt
- Làm sạch chữ ký suy nghĩ
- Vệ sinh tải trọng hình ảnh
- Gắn thẻ nguồn gốc đầu vào người dùng (cho các lời nhắc được định tuyến giữa các phiên)

Nếu bạn cần chi tiết về lưu trữ transcript, hãy xem:

- [/reference/session-management-compaction](/reference/session-management-compaction)

---
## Nơi chạy

Tất cả vệ sinh bản ghi được tập trung trong trình chạy nhúng:

- Lựa chọn chính sách: `src/agents/transcript-policy.ts`
- Ứng dụng vệ sinh/sửa chữa: `sanitizeSessionHistory` trong `src/agents/pi-embedded-runner/google.ts`

Chính sách sử dụng `provider`, `modelApi` và `modelId` để quyết định những gì cần áp dụng.

Riêng biệt với vệ sinh bản ghi, các tệp phiên được sửa chữa (nếu cần) trước khi tải:

- `repairSessionFileIfNeeded` trong `src/agents/session-file-repair.ts`
- Được gọi từ `run/attempt.ts` và `compact.ts` (trình chạy nhúng)

---
## Quy tắc toàn cục: vệ sinh hình ảnh

Các tải trọng hình ảnh luôn được vệ sinh để ngăn chặn việc từ chối phía nhà cung cấp do giới hạn kích thước (giảm tỷ lệ/nén lại các hình ảnh base64 quá lớn).

Điều này cũng giúp kiểm soát áp lực token do hình ảnh gây ra đối với các mô hình có khả năng nhìn thấy.
Các kích thước tối đa thấp hơn thường giảm mức sử dụng token; các kích thước cao hơn bảo toàn chi tiết.

Triển khai:

- `sanitizeSessionMessagesImages` trong `src/agents/pi-embedded-helpers/images.ts`
- `sanitizeContentBlocksImages` trong `src/agents/tool-images.ts`
- Cạnh hình ảnh tối đa có thể cấu hình qua `agents.defaults.imageMaxDimensionPx` (mặc định: `1200`).

---
## Quy tắc toàn cục: lệnh gọi công cụ không hợp lệ

Các khối lệnh gọi công cụ Assistant bị thiếu cả `input` và `arguments` sẽ bị loại bỏ
trước khi bối cảnh mô hình được xây dựng. Điều này ngăn chặn các lỗi từ nhà cung cấp do các lệnh gọi công cụ được lưu trữ một phần (ví dụ: sau khi vượt quá giới hạn tốc độ).

Triển khai:

- `sanitizeToolCallInputs` trong `src/agents/session-transcript-repair.ts`
- Được áp dụng trong `sanitizeSessionHistory` trong `src/agents/pi-embedded-runner/google.ts`

---
## Quy tắc toàn cục: Nguồn gốc đầu vào giữa các phiên

Khi một agent gửi prompt vào một phiên khác thông qua `sessions_send` (bao gồm
các bước trả lời/thông báo agent-to-agent), OpenClaw lưu giữ lượt người dùng được tạo với:

- `message.provenance.kind = "inter_session"`

Siêu dữ liệu này được ghi tại thời điểm nối thêm transcript và không thay đổi vai trò
(`role: "user"` vẫn được giữ để tương thích với nhà cung cấp). Những người đọc transcript có thể sử dụng
điều này để tránh coi các prompt được định tuyến nội bộ là hướng dẫn do người dùng cuối tạo.

Trong quá trình xây dựng lại ngữ cảnh, OpenClaw cũng thêm vào đầu một `[Inter-session message]`
marker ngắn cho những lượt người dùng đó trong bộ nhớ để mô hình có thể phân biệt chúng với
các hướng dẫn của người dùng cuối bên ngoài.

---
## Ma trận nhà cung cấp (hành vi hiện tại)

**OpenAI / OpenAI Codex**

- Chỉ vệ sinh hình ảnh.
- Loại bỏ chữ ký lý luận mồ côi (các mục lý luận độc lập mà không có khối nội dung theo sau) cho Phản hồi OpenAI/Bản ghi Codex.
- Không vệ sinh ID lệnh gọi công cụ.
- Không sửa chữa ghép nối kết quả công cụ.
- Không xác thực hoặc sắp xếp lại lượt.
- Không có kết quả công cụ tổng hợp.
- Không loại bỏ chữ ký suy nghĩ.

**Google (Generative AI / Gemini CLI / Antigravity)**

- Vệ sinh ID lệnh gọi công cụ: chỉ chữ và số.
- Sửa chữa ghép nối kết quả công cụ và kết quả công cụ tổng hợp.
- Xác thực lượt (luân phiên lượt theo kiểu Gemini).
- Sửa chữa thứ tự lượt Google (thêm một bootstrap người dùng nhỏ nếu lịch sử bắt đầu bằng trợ lý).
- Antigravity Claude: chuẩn hóa chữ ký suy nghĩ; loại bỏ các khối suy nghĩ không ký.

**Anthropic / Minimax (tương thích Anthropic)**

- Sửa chữa ghép nối kết quả công cụ và kết quả công cụ tổng hợp.
- Xác thực lượt (hợp nhất các lượt người dùng liên tiếp để thỏa mãn luân phiên nghiêm ngặt).

**Mistral (bao gồm phát hiện dựa trên model-id)**

- Vệ sinh ID lệnh gọi công cụ: strict9 (chữ và số độ dài 9).

**OpenRouter Gemini**

- Dọn dẹp chữ ký suy nghĩ: loại bỏ các giá trị `thought_signature` không phải base64 (giữ base64).

**Mọi thứ khác**

- Chỉ vệ sinh hình ảnh.
## Hành vi lịch sử (trước 2026.1.22)

Trước bản phát hành 2026.1.22, OpenClaw áp dụng nhiều lớp vệ sinh bảng ghi:

- Một **tiện ích mở rộng transcript-sanitize** chạy trên mỗi bản dựng ngữ cảnh và có thể:
  - Sửa chữa ghép nối công cụ sử dụng/kết quả.
  - Vệ sinh các id lệnh gọi công cụ (bao gồm chế độ không nghiêm ngặt bảo tồn `_`/`-`).
- Runner cũng thực hiện vệ sinh dành riêng cho nhà cung cấp, điều này trùng lặp công việc.
- Các đột biến bổ sung xảy ra bên ngoài chính sách nhà cung cấp, bao gồm:
  - Loại bỏ các thẻ `<final>` khỏi văn bản trợ lý trước khi lưu trữ.
  - Bỏ các lượt lỗi trợ lý trống.
  - Cắt ngắn nội dung trợ lý sau các lệnh gọi công cụ.

Sự phức tạp này gây ra các hồi quy giữa các nhà cung cấp (đáng chú ý là `openai-responses`
`call_id|fc_id` ghép nối). Bản dọn dẹp 2026.1.22 đã loại bỏ tiện ích mở rộng, tập trung hóa logic trong runner, và làm cho OpenAI **không chạm** ngoài vệ sinh hình ảnh.