---
summary: >-
  Sub-agents: tạo các agent runs cách ly mà công bố kết quả trở lại chat của
  người yêu cầu
read_when:
  - Bạn muốn công việc nền/song song thông qua agent
  - Bạn đang thay đổi chính sách công cụ sessions_spawn hoặc sub-agent
  - >-
    Bạn đang triển khai hoặc khắc phục sự cố các phiên subagent được ràng buộc
    với luồng
title: Các Tác Nhân Phụ
x-i18n:
  source_path: tools\subagents.md
  source_hash: bec64e8511107851c409a2f30071dc321ef5bfc5e389a52ba70ad3f1b81bf33e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:41.440Z'
---

# Các agent phụ

Các agent phụ là các lần chạy agent nền được tạo từ một lần chạy agent hiện có. Chúng chạy trong phiên riêng của chúng (`agent:<agentId>:subagent:<uuid>`) và khi hoàn thành, **thông báo** kết quả của chúng trở lại kênh trò chuyện của người yêu cầu.
## Lệnh slash

Sử dụng `/subagents` để kiểm tra hoặc kiểm soát các lần chạy sub-agent cho **phiên hiện tại**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`
- `/subagents steer <id|#> <message>`
- `/subagents spawn <agentId> <task> [--model <model>] [--thinking <level>]`

Điều khiển ràng buộc luồng:

Các lệnh này hoạt động trên các kênh hỗ trợ ràng buộc luồng bền vững. Xem **Các kênh hỗ trợ luồng** dưới đây.

- `/focus <subagent-label|session-key|session-id|session-label>`
- `/unfocus`
- `/agents`
- `/session idle <duration|off>`
- `/session max-age <duration|off>`

`/subagents info` hiển thị siêu dữ liệu chạy (trạng thái, dấu thời gian, id phiên, đường dẫn bản ghi, dọn dẹp).

### Hành vi sinh sản

`/subagents spawn` bắt đầu một sub-agent nền như một lệnh người dùng, không phải một relay nội bộ, và nó gửi một bản cập nhật hoàn thành cuối cùng trở lại kênh trò chuyện của người yêu cầu khi lần chạy kết thúc.

- Lệnh sinh sản không chặn; nó trả về một id chạy ngay lập tức.
- Khi hoàn thành, sub-agent thông báo một tin nhắn tóm tắt/kết quả trở lại kênh trò chuyện của người yêu cầu.
- Đối với các lần sinh sản thủ công, việc gửi là có khả năng phục hồi:
  - OpenClaw cố gắng gửi `agent` trực tiếp trước tiên với một khóa idempotency ổn định.
  - Nếu gửi trực tiếp không thành công, nó sẽ quay lại định tuyến hàng đợi.
  - Nếu định tuyến hàng đợi vẫn không khả dụng, thông báo sẽ được thử lại với một backoff theo cấp số nhân ngắn trước khi từ bỏ cuối cùng.
- Tin nhắn hoàn thành là một tin nhắn hệ thống và bao gồm:
  - `Result` (văn bản trả lời `assistant`, hoặc `toolResult` mới nhất nếu trả lời của trợ lý trống)
  - `Status` (`completed successfully` / `failed` / `timed out`)
  - thống kê thời gian chạy/token gọn gàng
- `--model` và `--thinking` ghi đè các giá trị mặc định cho lần chạy cụ thể đó.
- Sử dụng `info`/`log` để kiểm tra chi tiết và đầu ra sau khi hoàn thành.
- `/subagents spawn` là chế độ một lần (`mode: "run"`). Đối với các phiên ràng buộc luồng bền vững, sử dụng `sessions_spawn` với `thread: true` và `mode: "session"`.
- Đối với các phiên harness ACP (Codex, Claude Code, Gemini CLI), sử dụng `sessions_spawn` với `runtime: "acp"` và xem [ACP Agents](/tools/acp-agents).

Mục tiêu chính:

- Song song hóa công việc "nghiên cứu / tác vụ dài / công cụ chậm" mà không chặn lần chạy chính.
- Giữ sub-agent bị cô lập theo mặc định (tách phiên + sandboxing tùy chọn).
- Giữ bề mặt công cụ khó sử dụng sai: sub-agent **không** nhận được các công cụ phiên theo mặc định.
- Hỗ trợ độ sâu lồng nhau có thể cấu hình cho các mẫu orchestrator.

Ghi chú chi phí: mỗi sub-agent có **riêng** ngữ cảnh và sử dụng token của nó. Đối với các tác vụ nặng hoặc lặp lại, hãy đặt một mô hình rẻ hơn cho sub-agent và giữ agent chính của bạn trên một mô hình chất lượng cao hơn.
Bạn có thể cấu hình điều này thông qua `agents.defaults.subagents.model` hoặc ghi đè cho từng agent.
## Công cụ

Sử dụng `sessions_spawn`:

- Bắt đầu chạy sub-agent (`deliver: false`, global lane: `subagent`)
- Sau đó chạy bước announce và đăng reply announce vào kênh chat của người yêu cầu
- Mô hình mặc định: kế thừa từ người gọi trừ khi bạn đặt `agents.defaults.subagents.model` (hoặc per-agent `agents.list[].subagents.model`); một `sessions_spawn.model` rõ ràng vẫn có ưu tiên.
- Thinking mặc định: kế thừa từ người gọi trừ khi bạn đặt `agents.defaults.subagents.thinking` (hoặc per-agent `agents.list[].subagents.thinking`); một `sessions_spawn.thinking` rõ ràng vẫn có ưu tiên.
- Timeout chạy mặc định: nếu `sessions_spawn.runTimeoutSeconds` bị bỏ qua, OpenClaw sử dụng `agents.defaults.subagents.runTimeoutSeconds` khi được đặt; nếu không, nó quay lại `0` (không timeout).

Tham số công cụ:

- `task` (bắt buộc)
- `label?` (tùy chọn)
- `agentId?` (tùy chọn; spawn dưới một agent id khác nếu được phép)
- `model?` (tùy chọn; ghi đè mô hình sub-agent; các giá trị không hợp lệ bị bỏ qua và sub-agent chạy trên mô hình mặc định với cảnh báo trong kết quả công cụ)
- `thinking?` (tùy chọn; ghi đè mức thinking cho lần chạy sub-agent)
- `runTimeoutSeconds?` (mặc định là `agents.defaults.subagents.runTimeoutSeconds` khi được đặt, nếu không là `0`; khi được đặt, lần chạy sub-agent bị hủy sau N giây)
- `thread?` (mặc định `false`; khi `true`, yêu cầu ràng buộc thread kênh cho phiên sub-agent này)
- `mode?` (`run|session`)
  - mặc định là `run`
  - nếu `thread: true` và `mode` bị bỏ qua, mặc định trở thành `session`
  - `mode: "session"` yêu cầu `thread: true`
- `cleanup?` (`delete|keep`, mặc định `keep`)
## Phiên làm việc được liên kết với luồng

Khi liên kết luồng được bật cho một kênh, một sub-agent có thể được liên kết với một luồng để các tin nhắn tiếp theo của người dùng trong luồng đó tiếp tục được định tuyến đến cùng một phiên sub-agent.

### Các kênh hỗ trợ luồng

- Discord (hiện là kênh duy nhất được hỗ trợ): hỗ trợ các phiên sub-agent được liên kết với luồng bền vững (`sessions_spawn` với `thread: true`), điều khiển luồng thủ công (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`), và các khóa adapter `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.idleHours`, `channels.discord.threadBindings.maxAgeHours`, và `channels.discord.threadBindings.spawnSubagentSessions`.

Quy trình nhanh:

1. Tạo với `sessions_spawn` sử dụng `thread: true` (và tùy chọn `mode: "session"`).
2. OpenClaw tạo hoặc liên kết một luồng với mục tiêu phiên đó trong kênh hoạt động.
3. Các tin nhắn trả lời và tiếp theo trong luồng đó được định tuyến đến phiên được liên kết.
4. Sử dụng `/session idle` để kiểm tra/cập nhật tự động hủy tiêu điểm do không hoạt động và `/session max-age` để kiểm soát giới hạn cứng.
5. Sử dụng `/unfocus` để tách rời thủ công.

Điều khiển thủ công:

- `/focus <target>` liên kết luồng hiện tại (hoặc tạo một luồng) với mục tiêu sub-agent/phiên.
- `/unfocus` loại bỏ liên kết cho luồng được liên kết hiện tại.
- `/agents` liệt kê các lần chạy hoạt động và trạng thái liên kết (`thread:<id>` hoặc `unbound`).
- `/session idle` và `/session max-age` chỉ hoạt động cho các luồng được liên kết có tiêu điểm.

Công tắc cấu hình:

- Mặc định toàn cục: `session.threadBindings.enabled`, `session.threadBindings.idleHours`, `session.threadBindings.maxAgeHours`
- Ghi đè kênh và các khóa tự động liên kết tạo là dành riêng cho adapter. Xem **Các kênh hỗ trợ luồng** ở trên.

Xem [Tham chiếu cấu hình](/gateway/configuration-reference) và [Lệnh gạch chéo](/tools/slash-commands) để biết chi tiết adapter hiện tại.

Danh sách cho phép:

- `agents.list[].subagents.allowAgents`: danh sách các id agent có thể được nhắm mục tiêu thông qua `agentId` (`["*"]` để cho phép bất kỳ). Mặc định: chỉ agent yêu cầu.

Khám phá thiết bị:

- Sử dụng `agents_list` để xem những id agent nào hiện được cho phép cho `sessions_spawn`.

Lưu trữ tự động:

- Các phiên sub-agent được lưu trữ tự động sau `agents.defaults.subagents.archiveAfterMinutes` (mặc định: 60).
- Lưu trữ sử dụng `sessions.delete` và đổi tên bản ghi lại thành `*.deleted.<timestamp>` (cùng thư mục).
- `cleanup: "delete"` lưu trữ ngay sau khi thông báo (vẫn giữ bản ghi lại thông qua đổi tên).
- Lưu trữ tự động là nỗ lực tốt nhất; các bộ hẹn giờ đang chờ xử lý sẽ bị mất nếu gateway khởi động lại.
- `runTimeoutSeconds` **không** tự động lưu trữ; nó chỉ dừng lần chạy. Phiên vẫn tồn tại cho đến khi lưu trữ tự động.
- Lưu trữ tự động áp dụng như nhau cho các phiên độ sâu-1 và độ sâu-2.
## Agent Con Lồng Nhau

Theo mặc định, các agent con không thể tạo ra các agent con của riêng chúng (`maxSpawnDepth: 1`). Bạn có thể bật một cấp độ lồng nhau bằng cách đặt `maxSpawnDepth: 2`, cho phép **mô hình orchestrator**: main → agent con orchestrator → agent con con worker.

### Cách bật

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
        runTimeoutSeconds: 900, // default timeout for sessions_spawn when omitted (0 = no timeout)
      },
    },
  },
}
```

### Depth levels

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Main agent                                    | Always                       |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (orchestrator when depth 2 allowed) | Only if `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                   | Never                        |

### Announce chain

Results flow back up the chain:

1. Depth-2 worker finishes → announces to its parent (depth-1 orchestrator)
2. Depth-1 orchestrator receives the announce, synthesizes results, finishes → announces to main
3. Main agent receives the announce and delivers to the user

Each level only sees announces from its direct children.

### Tool policy by depth

- **Depth 1 (orchestrator, when `maxSpawnDepth >= 2`)**: Gets `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` so it can manage its children. Other session/system tools remain denied.
- **Depth 1 (leaf, when `maxSpawnDepth == 1`)**: No session tools (current default behavior).
- **Depth 2 (leaf worker)**: No session tools — `sessions_spawn` is always denied at depth 2. Cannot spawn further children.

### Per-agent spawn limit

Each agent session (at any depth) can have at most `maxChildrenPerAgent` (default: 5) active children at a time. This prevents runaway fan-out from a single orchestrator.

### Cascade stop

Stopping a depth-1 orchestrator automatically stops all its depth-2 children:

- `/stop` in the main chat stops all depth-1 agents and cascades to their depth-2 children.
- `/subagents kill <id>` stops a specific sub-agent and cascades to its children.
- `/subagents kill all` dừng tất cả các agent con cho người yêu cầu và lan tỏa.
## Xác thực

Xác thực sub-agent được giải quyết bằng **agent id**, không phải theo loại phiên:

- Khóa phiên sub-agent là `agent:<agentId>:subagent:<uuid>`.
- Kho xác thực được tải từ `agentDir` của agent đó.
- Các hồ sơ xác thực của agent chính được hợp nhất như một **fallback**; hồ sơ agent ghi đè hồ sơ chính khi có xung đột.

Lưu ý: việc hợp nhất là cộng dồn, vì vậy các hồ sơ chính luôn có sẵn như fallback. Xác thực hoàn toàn cô lập cho mỗi agent chưa được hỗ trợ.
## Thông báo

Các sub-agent báo cáo lại thông qua bước announce:

- Bước announce chạy bên trong phiên sub-agent (không phải phiên requester).
- Nếu sub-agent trả lời chính xác `ANNOUNCE_SKIP`, không có gì được đăng.
- Nếu không, câu trả lời announce được đăng vào kênh chat của requester thông qua lệnh gọi follow-up `agent` (`deliver=true`).
- Các câu trả lời announce bảo toàn định tuyến thread/topic khi có sẵn trên các bộ điều hợp kênh.
- Các tin nhắn announce được chuẩn hóa thành một mẫu ổn định:
  - `Status:` được lấy từ kết quả chạy (`success`, `error`, `timeout`, hoặc `unknown`).
  - `Result:` nội dung tóm tắt từ bước announce (hoặc `(not available)` nếu thiếu).
  - `Notes:` chi tiết lỗi và ngữ cảnh hữu ích khác.
- `Status` không được suy ra từ đầu ra mô hình; nó đến từ các tín hiệu kết quả runtime.

Các payload announce bao gồm một dòng thống kê ở cuối (ngay cả khi được bao):

- Runtime (ví dụ: `runtime 5m12s`)
- Sử dụng token (input/output/total)
- Chi phí ước tính khi định giá mô hình được cấu hình (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId`, và đường dẫn transcript (để agent chính có thể tìm nạp lịch sử thông qua `sessions_history` hoặc kiểm tra tệp trên đĩa)
## Chính sách công cụ (công cụ sub-agent)

Theo mặc định, các sub-agent nhận được **tất cả công cụ ngoại trừ công cụ phiên và công cụ hệ thống**:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Khi `maxSpawnDepth >= 2`, các sub-agent orchestrator độ sâu-1 sẽ nhận thêm `sessions_spawn`, `subagents`, `sessions_list`, và `sessions_history` để họ có thể quản lý các con của họ.

Ghi đè qua cấu hình:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```
## Tính năng đồng thời

Các sub-agent sử dụng một làn hàng đợi trong quy trình chuyên dụng:

- Tên làn: `subagent`
- Tính năng đồng thời: `agents.defaults.subagents.maxConcurrent` (mặc định `8`)
## Dừng lại

- Gửi `/stop` trong chat của người yêu cầu sẽ hủy bỏ phiên của người yêu cầu và dừng bất kỳ lần chạy sub-agent nào được tạo từ nó, lan tỏa đến các phần tử con lồng nhau.
- `/subagents kill <id>` dừng một sub-agent cụ thể và lan tỏa đến các phần tử con của nó.
## Giới hạn

- Thông báo sub-agent là **best-effort**. Nếu gateway khởi động lại, công việc "announce back" đang chờ xử lý sẽ bị mất.
- Sub-agents vẫn chia sẻ các tài nguyên quy trình gateway giống nhau; coi `maxConcurrent` như một van an toàn.
- `sessions_spawn` luôn không chặn: nó trả về `{ status: "accepted", runId, childSessionKey }` ngay lập tức.
- Ngữ cảnh sub-agent chỉ tiêm `AGENTS.md` + `TOOLS.md` (không có `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, hoặc `BOOTSTRAP.md`).
- Độ sâu lồng tối đa là 5 (`maxSpawnDepth` phạm vi: 1–5). Độ sâu 2 được khuyến nghị cho hầu hết các trường hợp sử dụng.
- `maxChildrenPerAgent` giới hạn các con hoạt động trên mỗi phiên (mặc định: 5, phạm vi: 1–20).