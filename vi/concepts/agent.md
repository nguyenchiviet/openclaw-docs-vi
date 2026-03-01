---
summary: >-
  Thời gian chạy Agent (pi-mono nhúng), hợp đồng không gian làm việc và khởi
  động phiên
read_when:
  - 'Thay đổi runtime của agent, khởi động workspace, hoặc hành vi phiên'
title: Thời gian chạy Agent
x-i18n:
  source_path: concepts\agent.md
  source_hash: 121103fda29a5481cb43234a39494f038e5dba89d0257fd3f7150c896b142bca
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:39:00.449Z'
---

# Agent Runtime 🤖

OpenClaw chạy một agent runtime nhúng duy nhất được lấy từ **pi-mono**.
## Workspace (bắt buộc)

OpenClaw sử dụng một thư mục workspace agent duy nhất (`agents.defaults.workspace`) làm thư mục làm việc **duy nhất** (`cwd`) cho các công cụ và ngữ cảnh.

Khuyến nghị: sử dụng `openclaw setup` để tạo `~/.openclaw/openclaw.json` nếu bị thiếu và khởi tạo các tệp workspace.

Bố cục workspace đầy đủ + hướng dẫn sao lưu: [Agent workspace](/concepts/agent-workspace)

Nếu `agents.defaults.sandbox` được bật, các phiên không phải phiên chính có thể ghi đè điều này bằng
các workspace cho từng phiên dưới `agents.defaults.sandbox.workspaceRoot` (xem
[Gateway configuration](/gateway/configuration)).
## Các tệp Bootstrap (được tiêm)

Bên trong `agents.defaults.workspace`, OpenClaw mong đợi các tệp có thể chỉnh sửa của người dùng này:

- `AGENTS.md` — hướng dẫn hoạt động + "bộ nhớ"
- `SOUL.md` — nhân cách, ranh giới, tông điệu
- `TOOLS.md` — ghi chú công cụ do người dùng duy trì (ví dụ: `imsg`, `sag`, quy ước)
- `BOOTSTRAP.md` — nghi thức chạy lần đầu tiên một lần (xóa sau khi hoàn thành)
- `IDENTITY.md` — tên/vibe/emoji của agent
- `USER.md` — hồ sơ người dùng + địa chỉ ưa thích

Ở lượt đầu tiên của một phiên mới, OpenClaw tiêm nội dung của các tệp này trực tiếp vào ngữ cảnh agent.

Các tệp trống bị bỏ qua. Các tệp lớn bị cắt ngắn và cắt bớt bằng một dấu hiệu để các prompt vẫn gọn gàng (đọc tệp để xem nội dung đầy đủ).

Nếu một tệp bị thiếu, OpenClaw tiêm một dòng dấu hiệu "tệp bị thiếu" duy nhất (và `openclaw setup` sẽ tạo một mẫu mặc định an toàn).

`BOOTSTRAP.md` chỉ được tạo cho một **không gian làm việc hoàn toàn mới** (không có tệp bootstrap khác). Nếu bạn xóa nó sau khi hoàn thành nghi thức, nó không nên được tạo lại khi khởi động lại sau này.

Để vô hiệu hóa hoàn toàn việc tạo tệp bootstrap (cho các không gian làm việc được cấy sẵn), hãy đặt:

```json5
{ agent: { skipBootstrap: true } }
```
## Công cụ tích hợp

Các công cụ cốt lõi (read/exec/edit/write và các công cụ hệ thống liên quan) luôn khả dụng,
tuân theo chính sách công cụ. `apply_patch` là tùy chọn và được kiểm soát bởi
`tools.exec.applyPatch`. `TOOLS.md` **không** kiểm soát công cụ nào tồn tại; đó là
hướng dẫn cho cách bạn muốn sử dụng chúng.
## Skills

OpenClaw tải Skills từ ba vị trí (workspace sẽ được ưu tiên nếu có xung đột tên):

- Bundled (được cung cấp kèm với bản cài đặt)
- Managed/local: `~/.openclaw/skills`
- Workspace: `<workspace>/skills`

Skills có thể được kiểm soát bằng cấu hình/biến môi trường (xem `skills` trong [Cấu hình Gateway](/gateway/configuration)).
## Tích hợp pi-mono

OpenClaw tái sử dụng các phần của codebase pi-mono (models/tools), nhưng **quản lý phiên, khám phá thiết bị và kết nối công cụ được OpenClaw quản lý**.

- Không có runtime agent pi-coding.
- Không có cài đặt `~/.pi/agent` hoặc `<workspace>/.pi` được tham khảo.
## Phiên

Bản ghi phiên được lưu trữ dưới dạng JSONL tại:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

ID phiên là ổn định và được chọn bởi OpenClaw.
Các thư mục phiên Pi/Tau cũ **không** được đọc.
## Điều khiển trong khi truyền phát

Khi chế độ hàng đợi là `steer`, các tin nhắn đến được tiêm vào lần chạy hiện tại.
Hàng đợi được kiểm tra **sau mỗi lần gọi công cụ**; nếu có tin nhắn trong hàng đợi,
các lần gọi công cụ còn lại từ tin nhắn trợ lý hiện tại sẽ bị bỏ qua (kết quả công cụ lỗi
với "Skipped due to queued user message."), sau đó tin nhắn người dùng trong hàng đợi
được tiêm vào trước phản hồi trợ lý tiếp theo.

Khi chế độ hàng đợi là `followup` hoặc `collect`, các tin nhắn đến được giữ lại cho đến khi
lượt hiện tại kết thúc, sau đó một lượt agent mới bắt đầu với các tải trọng trong hàng đợi. Xem
[Queue](/concepts/queue) để biết hành vi chế độ + debounce/cap.

Truyền phát theo khối gửi các khối trợ lý hoàn thành ngay khi chúng hoàn tất; nó
**tắt theo mặc định** (`agents.defaults.blockStreamingDefault: "off"`).
Điều chỉnh ranh giới qua `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; mặc định là text_end).
Kiểm soát chia nhỏ khối mềm với `agents.defaults.blockStreamingChunk` (mặc định là
800–1200 ký tự; ưu tiên ngắt đoạn, sau đó là dòng mới; câu cuối cùng).
Hợp nhất các đoạn được truyền phát với `agents.defaults.blockStreamingCoalesce` để giảm
spam một dòng (hợp nhất dựa trên nhàn rỗi trước khi gửi). Các kênh không phải Telegram yêu cầu
`*.blockStreaming: true` rõ ràng để bật các phản hồi khối.
Các bản tóm tắt công cụ chi tiết được phát ra khi công cụ bắt đầu (không debounce); Control UI
truyền phát đầu ra công cụ qua các sự kiện agent khi có sẵn.
Chi tiết thêm: [Streaming + chunking](/concepts/streaming).
## Tham chiếu mô hình

Tham chiếu mô hình trong cấu hình (ví dụ `agents.defaults.model` và `agents.defaults.models`) được phân tích cú pháp bằng cách chia tách trên `/` **đầu tiên**.

- Sử dụng `provider/model` khi cấu hình mô hình.
- Nếu ID mô hình chính nó chứa `/` (kiểu OpenRouter), hãy bao gồm tiền tố nhà cung cấp (ví dụ: `openrouter/moonshotai/kimi-k2`).
- Nếu bạn bỏ qua nhà cung cấp, OpenClaw sẽ coi đầu vào là một bí danh hoặc một mô hình cho **nhà cung cấp mặc định** (chỉ hoạt động khi không có `/` trong ID mô hình).
## Cấu hình (tối thiểu)

Tối thiểu, hãy đặt:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (được khuyến nghị mạnh mẽ)

---

_Tiếp theo: [Group Chats](/channels/group-messages)_ 🦞