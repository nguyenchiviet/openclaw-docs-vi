---
summary: 'Vòng đời vòng lặp Agent, streams, và ngữ nghĩa chờ đợi'
read_when:
  - Bạn cần một hướng dẫn chính xác về vòng lặp agent hoặc các sự kiện vòng đời.
title: Vòng Lặp Agent
x-i18n:
  source_path: concepts\agent-loop.md
  source_hash: 2bc487fd8d01e45526c0615e9b18672c324fe53d416e4e13cfd3c7860d1784d5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:38:26.068Z'
---

# Vòng lặp Agent (OpenClaw)

Một vòng lặp agentic là lần chạy "thực" đầy đủ của một agent: tiếp nhận → lắp ráp ngữ cảnh → suy luận mô hình →
thực thi công cụ → truyền phát phản hồi → lưu trữ. Đó là đường dẫn có thẩm quyền biến một tin nhắn
thành các hành động và phản hồi cuối cùng, đồng thời giữ trạng thái phiên nhất quán.

Trong OpenClaw, một vòng lặp là một lần chạy duy nhất, được tuần tự hóa cho mỗi phiên, phát ra các sự kiện vòng đời và truyền phát
khi mô hình suy nghĩ, gọi công cụ và truyền phát đầu ra. Tài liệu này giải thích cách vòng lặp xác thực đó được
kết nối từ đầu đến cuối.
## Điểm vào

- Gateway RPC: `agent` và `agent.wait`.
- CLI: lệnh `agent`.
## Cách hoạt động (cấp cao)

1. `agent` xác thực các tham số, giải quyết phiên (sessionKey/sessionId), lưu trữ siêu dữ liệu phiên, trả về `{ runId, acceptedAt }` ngay lập tức.
2. `agentCommand` chạy agent:
   - giải quyết mô hình + các giá trị mặc định thinking/verbose
   - tải snapshot Skills
   - gọi `runEmbeddedPiAgent` (runtime pi-agent-core)
   - phát ra **lifecycle end/error** nếu vòng lặp nhúng không phát ra
3. `runEmbeddedPiAgent`:
   - tuần tự hóa các lần chạy thông qua hàng đợi theo phiên + toàn cầu
   - giải quyết mô hình + hồ sơ xác thực và xây dựng phiên pi
   - đăng ký các sự kiện pi và truyền phát các delta trợ lý/công cụ
   - thực thi timeout -> hủy bỏ lần chạy nếu vượt quá
   - trả về payloads + siêu dữ liệu sử dụng
4. `subscribeEmbeddedPiSession` cầu nối các sự kiện pi-agent-core tới luồng OpenClaw `agent`:
   - sự kiện công cụ => `stream: "tool"`
   - delta trợ lý => `stream: "assistant"`
   - sự kiện lifecycle => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` sử dụng `waitForAgentJob`:
   - chờ **lifecycle end/error** cho `runId`
   - trả về `{ status: ok|error|timeout, startedAt, endedAt, error? }`
## Xếp hàng + đồng thời

- Các lần chạy được tuần tự hóa cho mỗi khóa phiên (làn phiên) và tùy chọn thông qua một làn toàn cầu.
- Điều này ngăn chặn các cuộc đua công cụ/phiên và giữ cho lịch sử phiên nhất quán.
- Các kênh nhắn tin có thể chọn các chế độ hàng đợi (collect/steer/followup) cung cấp cho hệ thống làn này.
  Xem [Command Queue](/concepts/queue).
## Chuẩn bị phiên + không gian làm việc

- Không gian làm việc được phân giải và tạo; các lần chạy sandboxed có thể chuyển hướng đến thư mục gốc không gian làm việc sandbox.
- Skills được tải (hoặc tái sử dụng từ một snapshot) và được tiêm vào env và prompt.
- Các tệp bootstrap/context được phân giải và tiêm vào báo cáo system prompt.
- Một khóa ghi phiên được lấy; `SessionManager` được mở và chuẩn bị trước khi truyền phát.
## Lắp ráp prompt + system prompt

- System prompt được xây dựng từ base prompt của OpenClaw, skills prompt, bootstrap context, và các ghi đè cho mỗi lần chạy.
- Các giới hạn cụ thể cho mô hình và token dự phòng nén được thực thi.
- Xem [System prompt](/concepts/system-prompt) để biết mô hình nhìn thấy gì.
## Các điểm hook (nơi bạn có thể chặn)

OpenClaw có hai hệ thống hook:

- **Internal hooks** (Gateway hooks): các script được kích hoạt bởi sự kiện cho các lệnh và sự kiện vòng đời.
- **Plugin hooks**: các điểm mở rộng bên trong vòng đời agent/tool và gateway pipeline.

### Internal hooks (Gateway hooks)

- **`agent:bootstrap`**: chạy trong khi xây dựng các tệp bootstrap trước khi system prompt được hoàn thiện.
  Sử dụng điều này để thêm/xóa các tệp ngữ cảnh bootstrap.
- **Command hooks**: `/new`, `/reset`, `/stop`, và các sự kiện lệnh khác (xem tài liệu Hooks).

Xem [Hooks](/automation/hooks) để biết cách thiết lập và ví dụ.

### Plugin hooks (agent + gateway lifecycle)

Những hook này chạy bên trong vòng lặp agent hoặc gateway pipeline:

- **`before_model_resolve`**: chạy trước phiên (không có `messages`) để ghi đè nhà cung cấp/mô hình một cách xác định trước khi phân giải mô hình.
- **`before_prompt_build`**: chạy sau khi tải phiên (với `messages`) để chèn `prependContext`/`systemPrompt` trước khi gửi prompt.
- **`before_agent_start`**: hook tương thích ngược có thể chạy ở bất kỳ giai đoạn nào; ưu tiên các hook rõ ràng ở trên.
- **`agent_end`**: kiểm tra danh sách tin nhắn cuối cùng và chạy metadata sau khi hoàn thành.
- **`before_compaction` / `after_compaction`**: quan sát hoặc chú thích các chu kỳ nén.
- **`before_tool_call` / `after_tool_call`**: chặn các tham số/kết quả công cụ.
- **`tool_result_persist`**: biến đổi đồng bộ các kết quả công cụ trước khi chúng được ghi vào bảng điểm phiên.
- **`message_received` / `message_sending` / `message_sent`**: các hook tin nhắn đến + đi.
- **`session_start` / `session_end`**: các ranh giới vòng đời phiên.
- **`gateway_start` / `gateway_stop`**: các sự kiện vòng đời gateway.

Xem [Plugins](/tools/plugin#plugin-hooks) để biết API hook và chi tiết đăng ký.
## Truyền phát + phản hồi một phần

- Các delta của trợ lý được truyền phát từ pi-agent-core và phát ra dưới dạng sự kiện `assistant`.
- Truyền phát theo khối có thể phát ra phản hồi một phần trên `text_end` hoặc `message_end`.
- Truyền phát suy luận có thể được phát ra dưới dạng một luồng riêng biệt hoặc dưới dạng phản hồi khối.
- Xem [Truyền phát](/concepts/streaming) để biết hành vi chia khối và phản hồi khối.
## Thực thi công cụ + công cụ nhắn tin

- Các sự kiện bắt đầu/cập nhật/kết thúc công cụ được phát hành trên luồng `tool`.
- Kết quả công cụ được làm sạch theo kích thước và tải trọng hình ảnh trước khi ghi nhật ký/phát hành.
- Các lần gửi công cụ nhắn tin được theo dõi để loại bỏ các xác nhận trợ lý trùng lặp.
## Định hình và loại bỏ phản hồi

- Các payload cuối cùng được lắp ráp từ:
  - văn bản trợ lý (và lý luận tùy chọn)
  - tóm tắt công cụ nội tuyến (khi chi tiết + được phép)
  - văn bản lỗi trợ lý khi mô hình gặp lỗi
- `NO_REPLY` được coi là một token im lặng và được lọc ra khỏi các payload gửi đi.
- Các bản sao công cụ nhắn tin được loại bỏ khỏi danh sách payload cuối cùng.
- Nếu không còn payload có thể hiển thị nào và một công cụ gặp lỗi, một phản hồi lỗi công cụ dự phòng sẽ được phát hành
  (trừ khi một công cụ nhắn tin đã gửi một phản hồi hiển thị cho người dùng).
## Nén dữ liệu + thử lại

- Nén dữ liệu tự động phát ra các sự kiện luồng `compaction` và có thể kích hoạt một lần thử lại.
- Khi thử lại, các bộ đệm trong bộ nhớ và tóm tắt công cụ được đặt lại để tránh đầu ra trùng lặp.
- Xem [Nén dữ liệu](/concepts/compaction) để biết chi tiết về quy trình nén dữ liệu.
## Luồng sự kiện (hôm nay)

- `lifecycle`: được phát ra bởi `subscribeEmbeddedPiSession` (và như một giải pháp dự phòng bởi `agentCommand`)
- `assistant`: các thay đổi được truyền phát từ pi-agent-core
- `tool`: các sự kiện công cụ được truyền phát từ pi-agent-core
## Xử lý kênh trò chuyện

- Các delta của trợ lý được lưu vào bộ đệm thành các tin nhắn `delta` trò chuyện.
- Một `final` trò chuyện được phát hành khi **kết thúc/lỗi vòng đời**.
## Hết thời gian chờ

- `agent.wait` mặc định: 30s (chỉ thời gian chờ). `timeoutMs` tham số ghi đè.
- Thời gian chạy agent: `agents.defaults.timeoutSeconds` mặc định 600s; được thực thi trong `runEmbeddedPiAgent` bộ hẹn giờ hủy bỏ.
## Nơi mọi thứ có thể kết thúc sớm

- Hết thời gian chờ agent (hủy bỏ)
- AbortSignal (hủy)
- Gateway ngắt kết nối hoặc RPC hết thời gian chờ
- `agent.wait` hết thời gian chờ (chỉ chờ, không dừng agent)