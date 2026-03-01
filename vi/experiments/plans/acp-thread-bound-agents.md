---
summary: >-
  Tích hợp các agent mã hóa ACP thông qua một mặt phẳng điều khiển ACP hạng nhất
  trong các runtime cốt lõi và được hỗ trợ bởi plugin (acpx trước tiên)
owner: onutc
status: draft
last_updated: '2026-02-25'
title: Các Agents Liên Kết với Luồng ACP
x-i18n:
  source_path: experiments\plans\acp-thread-bound-agents.md
  source_hash: c98e5e02f369fb4a849906d02b9c8e58f227effd6383aea58c2e3d5a75348e55
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:54:58.409Z'
---

# Các Agent Liên kết ACP Thread

## Tổng quan

Kế hoạch này định nghĩa cách OpenClaw nên hỗ trợ các agent mã hóa ACP trong các kênh có khả năng thread (Discord trước tiên) với vòng đời cấp sản xuất và khôi phục.

Tài liệu liên quan:

- [Unified Runtime Streaming Refactor Plan](/experiments/plans/acp-unified-streaming-refactor)

Trải nghiệm người dùng mục tiêu:

- người dùng tạo hoặc tập trung một phiên ACP vào một thread
- tin nhắn của người dùng trong thread đó được định tuyến đến phiên ACP liên kết
- đầu ra agent truyền phát trở lại cùng một persona thread
- phiên có thể là persistent hoặc one shot với các điều khiển dọn dẹp rõ ràng
## Tóm tắt quyết định

Khuyến nghị dài hạn là một kiến trúc hybrid:

- OpenClaw core sở hữu các mối quan tâm của mặt phẳng điều khiển ACP
  - nhận dạng phiên và siêu dữ liệu
  - quyết định ràng buộc luồng và định tuyến
  - bất biến giao hàng và loại bỏ trùng lặp
  - ngữ nghĩa dọn dẹp vòng đời và phục hồi
- ACP runtime backend có thể cắm được
  - backend đầu tiên là dịch vụ plugin được hỗ trợ bởi acpx
  - runtime thực hiện vận chuyển ACP, xếp hàng, hủy, kết nối lại

OpenClaw không nên triển khai lại các nội dung vận chuyển ACP trong core.
OpenClaw không nên dựa vào một đường dẫn chặn plugin thuần túy cho định tuyến.
## Kiến trúc hướng tới mục tiêu (thánh grail)

Coi ACP là một control plane hạng nhất trong OpenClaw, với các adapter runtime có thể cắm được.

Các bất biến không thể thương lượng:

- mỗi ràng buộc thread ACP tham chiếu một bản ghi phiên ACP hợp lệ
- mỗi phiên ACP có trạng thái vòng đời rõ ràng (`creating`, `idle`, `running`, `cancelling`, `closed`, `error`)
- mỗi lần chạy ACP có trạng thái chạy rõ ràng (`queued`, `running`, `completed`, `failed`, `cancelled`)
- spawn, bind, và initial enqueue là nguyên tử
- các lần thử lại lệnh là idempotent (không có lần chạy trùng lặp hoặc đầu ra Discord trùng lặp)
- đầu ra kênh bound-thread là một phép chiếu của các sự kiện chạy ACP, không bao giờ là các tác dụng phụ ad-hoc

Mô hình sở hữu dài hạn:

- `AcpSessionManager` là người viết và nhà điều phối ACP duy nhất
- manager sống trong quy trình gateway trước tiên; có thể được chuyển đến một sidecar chuyên dụng sau đó phía sau cùng một giao diện
- cho mỗi khóa phiên ACP, manager sở hữu một actor trong bộ nhớ (thực thi lệnh được tuần tự hóa)
- các adapter (`acpx`, các backend trong tương lai) chỉ là các triển khai transport/runtime

Mô hình lưu trữ dài hạn:

- chuyển trạng thái control-plane ACP sang một kho SQLite chuyên dụng (chế độ WAL) dưới thư mục trạng thái OpenClaw
- giữ `SessionEntry.acp` làm phép chiếu tương thích trong quá trình di chuyển, không phải nguồn sự thật
- lưu trữ các sự kiện ACP theo kiểu append-only để hỗ trợ replay, khôi phục sự cố và giao hàng xác định

### Chiến lược giao hàng (cầu nối tới thánh grail)

- cầu nối ngắn hạn
  - giữ cơ chế ràng buộc thread hiện tại và bề mặt cấu hình ACP hiện có
  - sửa các lỗi metadata-gap và định tuyến các lượt ACP thông qua một nhánh ACP cốt lõi duy nhất
  - thêm các khóa idempotency và kiểm tra định tuyến fail-closed ngay lập tức
- chuyển đổi dài hạn
  - chuyển nguồn sự thật ACP sang control-plane DB + actors
  - làm cho giao hàng bound-thread hoàn toàn dựa trên phép chiếu sự kiện
  - loại bỏ hành vi fallback kế thừa phụ thuộc vào metadata mục nhập phiên cơ hội
## Tại sao không chỉ dùng plugin thuần túy

Các hook plugin hiện tại không đủ để định tuyến phiên ACP từ đầu đến cuối mà không thay đổi core.

- định tuyến inbound từ thread binding được phân giải thành session key trong core dispatch trước tiên
- message hooks là fire-and-forget và không thể ngắt đường dẫn reply chính
- plugin commands rất tốt cho các hoạt động điều khiển nhưng không thích hợp để thay thế luồng dispatch per-turn của core

Kết quả:

- ACP runtime có thể được pluginized
- nhánh định tuyến ACP phải tồn tại trong core
## Nền tảng hiện có để tái sử dụng

Đã được triển khai và nên giữ nguyên:

- thread binding target hỗ trợ `subagent` và `acp`
- inbound thread routing override phân giải bằng binding trước khi dispatch bình thường
- outbound thread identity thông qua webhook trong reply delivery
- `/focus` và `/unfocus` flow với khả năng tương thích ACP target
- persistent binding store với restore khi khởi động
- unbind lifecycle trên archive, delete, unfocus, reset, và delete

Kế hoạch này mở rộng nền tảng đó thay vì thay thế nó.
## Kiến trúc

### Mô hình ranh giới

Lõi (phải có trong lõi OpenClaw):

- Nhánh điều phối chế độ phiên ACP trong đường ống trả lời
- Phân bổ giao hàng để tránh trùng lặp luồng cha
- Tính bền vững mặt phẳng điều khiển ACP (với phép chiếu tương thích `SessionEntry.acp` trong quá trình di chuyển)
- Ngữ nghĩa tách biệt vòng đời và tách rời thời gian chạy được liên kết với đặt lại/xóa phiên

Plugin backend (triển khai acpx):

- Giám sát công nhân thời gian chạy ACP
- Gọi quy trình acpx và phân tích cú pháp sự kiện
- Trình xử lý lệnh ACP (`/acp ...`) và UX toán tử
- Mặc định cấu hình cụ thể backend và chẩn đoán

### Mô hình quyền sở hữu thời gian chạy

- một quy trình gateway sở hữu trạng thái điều phối ACP
- Thực thi ACP chạy trong các quy trình con được giám sát thông qua backend acpx
- Chiến lược quy trình tồn tại lâu dài cho mỗi khóa phiên ACP hoạt động, không phải cho mỗi tin nhắn

Điều này tránh chi phí khởi động trên mỗi lời nhắc và giữ cho ngữ nghĩa hủy và kết nối lại đáng tin cậy.

### Hợp đồng thời gian chạy cốt lõi

Thêm hợp đồng thời gian chạy ACP cốt lõi để mã định tuyến không phụ thuộc vào chi tiết CLI và có thể chuyển đổi backend mà không thay đổi logic điều phối:

```ts
export type AcpRuntimePromptMode = "prompt" | "steer";

export type AcpRuntimeHandle = {
  sessionKey: string;
  backend: string;
  runtimeSessionName: string;
};

export type AcpRuntimeEvent =
  | { type: "text_delta"; stream: "output" | "thought"; text: string }
  | { type: "tool_call"; name: string; argumentsText: string }
  | { type: "done"; usage?: Record<string, number> }
  | { type: "error"; code: string; message: string; retryable?: boolean };

export interface AcpRuntime {
  ensureSession(input: {
    sessionKey: string;
    agent: string;
    mode: "persistent" | "oneshot";
    cwd?: string;
    env?: Record<string, string>;
    idempotencyKey: string;
  }): Promise<AcpRuntimeHandle>;

  submit(input: {
    handle: AcpRuntimeHandle;
    text: string;
    mode: AcpRuntimePromptMode;
    idempotencyKey: string;
  }): Promise<{ runtimeRunId: string }>;

  stream(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId: string;
    onEvent: (event: AcpRuntimeEvent) => Promise<void> | void;
    signal?: AbortSignal;
  }): Promise<void>;

  cancel(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId?: string;
    reason?: string;
    idempotencyKey: string;
  }): Promise<void>;

  close(input: { handle: AcpRuntimeHandle; reason: string; idempotencyKey: string }): Promise<void>;

  health?(): Promise<{ ok: boolean; details?: string }>;
}
```
Chi tiết triển khai:

- backend đầu tiên: `AcpxRuntime` được cung cấp dưới dạng dịch vụ plugin
- core giải quyết runtime thông qua registry và thất bại với lỗi toán tử rõ ràng khi không có backend runtime ACP nào khả dụng

### Mô hình dữ liệu control-plane và tính bền vững

Nguồn sự thật dài hạn là cơ sở dữ liệu ACP SQLite chuyên dụng (chế độ WAL), để cập nhật giao dịch và khôi phục an toàn khi gặp sự cố:

- `acp_sessions`
  - `session_key` (pk), `backend`, `agent`, `mode`, `cwd`, `state`, `created_at`, `updated_at`, `last_error`
- `acp_runs`
  - `run_id` (pk), `session_key` (fk), `state`, `requester_message_id`, `idempotency_key`, `started_at`, `ended_at`, `error_code`, `error_message`
- `acp_bindings`
  - `binding_key` (pk), `thread_id`, `channel_id`, `account_id`, `session_key` (fk), `expires_at`, `bound_at`
- `acp_events`
  - `event_id` (pk), `run_id` (fk), `seq`, `kind`, `payload_json`, `created_at`
- `acp_delivery_checkpoint`
  - `run_id` (pk/fk), `last_event_seq`, `last_discord_message_id`, `updated_at`
- `acp_idempotency`
  - `scope`, `idempotency_key`, `result_json`, `created_at`, `(scope, idempotency_key)` duy nhất

```ts
export type AcpSessionMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

Storage rules:

- keep `SessionEntry.acp` as a compatibility projection during migration
- process ids and sockets stay in memory only
- durable lifecycle and run status live in ACP DB, not generic session JSON
- if runtime owner dies, gateway rehydrates from ACP DB and resumes from checkpoints

### Routing and delivery

Inbound:

- keep current thread binding lookup as first routing step
- if bound target is ACP session, route to ACP runtime branch instead of `getReplyFromConfig`
- explicit `/acp steer` command uses `mode: "steer"`

Đi ra:

- luồng sự kiện ACP được chuẩn hóa thành các khối trả lời OpenClaw
- đích giao hàng được giải quyết thông qua đường dẫn đích ràng buộc hiện có
- khi một luồng ràng buộc hoạt động cho lượt phiên đó, hoàn thành kênh cha bị triệt tiêu

Chính sách truyền phát:

- truyền phát đầu ra một phần với cửa sổ hợp nhất
- khoảng thời gian tối thiểu có thể cấu hình và số byte khối tối đa để tuân thủ giới hạn tốc độ Discord
- tin nhắn cuối cùng luôn được phát hành khi hoàn thành hoặc thất bại

### Máy trạng thái và ranh giới giao dịch

Máy trạng thái phiên:

- `creating -> idle -> running -> idle`
- `running -> cancelling -> idle | error`
- `idle -> closed`
- `error -> idle | closed`

Máy trạng thái chạy:

- `queued -> running -> completed`
- `running -> failed | cancelled`
- `queued -> cancelled`

Ranh giới giao dịch bắt buộc:

- giao dịch sinh
  - tạo hàng phiên ACP
  - tạo/cập nhật hàng ràng buộc luồng ACP
  - xếp hàng hàng chạy ban đầu
- giao dịch đóng
  - đánh dấu phiên đã đóng
  - xóa/hết hạn hàng ràng buộc
  - ghi sự kiện đóng cuối cùng
- giao dịch hủy
  - đánh dấu chạy mục tiêu đang hủy/đã hủy bằng khóa idempotency

Không cho phép thành công một phần trên các ranh giới này.

### Mô hình actor theo phiên

`AcpSessionManager` chạy một actor cho mỗi khóa phiên ACP:

- hộp thư actor tuần tự hóa `submit`, `cancel`, `close`, và `stream` tác dụng phụ
- actor sở hữu hydration xử lý runtime và vòng đời quy trình bộ điều hợp runtime cho phiên đó
- actor ghi sự kiện chạy theo thứ tự (`seq`) trước bất kỳ lần gửi Discord nào
- actor cập nhật điểm kiểm tra gửi sau khi gửi đi thành công

Điều này loại bỏ các cuộc đua giữa các lượt và ngăn chặn đầu ra luồng trùng lặp hoặc không theo thứ tự.

### Idempotency và phép chiếu gửi

Tất cả các hành động ACP bên ngoài phải mang khóa idempotency:

- khóa idempotency sinh
- khóa idempotency nhắc nhở/điều hướng
- khóa idempotency hủy
- khóa idempotency đóng

Quy tắc gửi:

- tin nhắn Discord được lấy từ `acp_events` cộng với `acp_delivery_checkpoint`
- các lần thử lại tiếp tục từ điểm kiểm tra mà không gửi lại các khối đã gửi
- phát hành trả lời cuối cùng là chính xác một lần cho mỗi lần chạy từ logic phép chiếu

### Khôi phục và tự chữa lành
Khi Gateway khởi động:

- tải các phiên ACP không phải terminal (`creating`, `idle`, `running`, `cancelling`, `error`)
- tạo lại các actor một cách lười biếng khi có sự kiện inbound đầu tiên hoặc tích cực dưới giới hạn được cấu hình
- điều hòa bất kỳ lần chạy `running` nào bị thiếu nhịp tim và đánh dấu `failed` hoặc khôi phục thông qua adapter

Khi nhận tin nhắn luồng Discord:

- nếu binding tồn tại nhưng phiên ACP bị thiếu, thất bại đóng với thông báo binding cũ rõ ràng
- tùy chọn tự động hủy binding cũ sau xác thực an toàn cho nhà điều hành
- không bao giờ định tuyến im lặng các binding ACP cũ đến đường dẫn LLM bình thường

### Vòng đời và an toàn

Các hoạt động được hỗ trợ:

- hủy lần chạy hiện tại: `/acp cancel`
- hủy binding luồng: `/unfocus`
- đóng phiên ACP: `/acp close`
- tự động đóng các phiên không hoạt động theo TTL hiệu quả

Chính sách TTL:

- TTL hiệu quả là giá trị tối thiểu của
  - TTL toàn cầu/phiên
  - TTL binding luồng Discord
  - TTL chủ sở hữu runtime ACP

Các điều khiển an toàn:

- danh sách cho phép các agent ACP theo tên
- hạn chế các gốc không gian làm việc cho các phiên ACP
- passthrough danh sách cho phép env
- số phiên ACP đồng thời tối đa trên mỗi tài khoản và toàn cầu
- backoff khởi động lại có giới hạn cho các sự cố runtime
## Bề mặt cấu hình

Các khóa cốt lõi:

- `acp.enabled`
- `acp.dispatch.enabled` (công tắc tắt định tuyến ACP độc lập)
- `acp.backend` (mặc định `acpx`)
- `acp.defaultAgent`
- `acp.allowedAgents[]`
- `acp.maxConcurrentSessions`
- `acp.stream.coalesceIdleMs`
- `acp.stream.maxChunkChars`
- `acp.runtime.ttlMinutes`
- `acp.controlPlane.store` (mặc định `sqlite`)
- `acp.controlPlane.storePath`
- `acp.controlPlane.recovery.eagerActors`
- `acp.controlPlane.recovery.reconcileRunningAfterMs`
- `acp.controlPlane.checkpoint.flushEveryEvents`
- `acp.controlPlane.checkpoint.flushEveryMs`
- `acp.idempotency.ttlHours`
- `channels.discord.threadBindings.spawnAcpSessions`

Các khóa plugin/backend (phần plugin acpx):

- ghi đè lệnh/đường dẫn backend
- danh sách cho phép biến môi trường backend
- cài đặt trước mỗi agent backend
- thời gian chờ khởi động/dừng backend
- số lần chạy tối đa trong chuyến bay trên mỗi phiên
## Thông số kỹ thuật triển khai

### Các mô-đun control-plane (mới)

Thêm các mô-đun control-plane ACP chuyên dụng trong core:

- `src/acp/control-plane/manager.ts`
  - sở hữu các actor ACP, chuyển đổi vòng đời, tuần tự hóa lệnh
- `src/acp/control-plane/store.ts`
  - quản lý schema SQLite, giao dịch, trợ giúp truy vấn
- `src/acp/control-plane/events.ts`
  - định nghĩa sự kiện ACP được gõ và tuần tự hóa
- `src/acp/control-plane/checkpoint.ts`
  - điểm kiểm tra giao hàng bền vững và con trỏ phát lại
- `src/acp/control-plane/idempotency.ts`
  - đặt trữ khóa idempotency và phát lại phản hồi
- `src/acp/control-plane/recovery.ts`
  - đối sánh thời gian khởi động và kế hoạch tái hydrate actor

Các mô-đun cầu nối tương thích:

- `src/acp/runtime/session-meta.ts`
  - vẫn tạm thời dành cho phép chiếu vào `SessionEntry.acp`
  - phải ngừng là nguồn sự thật sau khi cắt chuyển đổi

### Các bất biến bắt buộc (phải thực thi trong mã)

- Tạo phiên ACP và ràng buộc luồng là nguyên tử (giao dịch đơn)
- có tối đa một lần chạy hoạt động trên mỗi actor phiên ACP tại một thời điểm
- sự kiện `seq` tăng nghiêm ngặt trên mỗi lần chạy
- điểm kiểm tra giao hàng không bao giờ vượt quá sự kiện được cam kết cuối cùng
- phát lại idempotency trả về tải trọng thành công trước đó cho các khóa lệnh trùng lặp
- siêu dữ liệu ACP cũ/bị thiếu không thể định tuyến vào đường dẫn trả lời không phải ACP bình thường

### Các điểm chạm cốt lõi

Các tệp core cần thay đổi:

- `src/auto-reply/reply/dispatch-from-config.ts`
  - các lệnh nhánh ACP `AcpSessionManager.submit` và giao hàng phép chiếu sự kiện
  - loại bỏ dự phòng ACP trực tiếp bỏ qua các bất biến control-plane
- `src/auto-reply/reply/inbound-context.ts` (hoặc ranh giới bối cảnh chuẩn hóa gần nhất)
  - hiển thị các khóa định tuyến chuẩn hóa và hạt giống idempotency cho control plane ACP
- `src/config/sessions/types.ts`
  - giữ `SessionEntry.acp` làm trường tương thích chỉ phép chiếu
- `src/gateway/server-methods/sessions.ts`
  - đặt lại/xóa/lưu trữ phải gọi đường dẫn giao dịch đóng/unbind của trình quản lý ACP
- `src/infra/outbound/bound-delivery-router.ts`
  - thực thi hành vi đích đóng không thành công cho các lần quay phiên ràng buộc ACP
- `src/discord/monitor/thread-bindings.ts`
  - thêm trợ giúp xác thực ràng buộc cũ ACP được kết nối với các tra cứu control-plane
- `src/auto-reply/reply/commands-acp.ts`
  - định tuyến spawn/cancel/close/steer thông qua các API trình quản lý ACP
- `src/agents/acp-spawn.ts`
  - ngừng ghi siêu dữ liệu ad-hoc; gọi giao dịch spawn của trình quản lý ACP
- `src/plugin-sdk/**` và cầu nối thời gian chạy plugin
  - hiển thị đăng ký backend ACP và ngữ nghĩa sức khỏe một cách sạch sẽ

Các tệp core không được thay thế rõ ràng:
- `src/discord/monitor/message-handler.preflight.ts`
  - giữ hành vi ghi đè ràng buộc luồng làm bộ phân giải khóa phiên chính tắc

### API đăng ký runtime ACP

Thêm mô-đun đăng ký cốt lõi:

- `src/acp/runtime/registry.ts`

API bắt buộc:

```ts
export type AcpRuntimeBackend = {
  id: string;
  runtime: AcpRuntime;
  healthy?: () => boolean;
};

export function registerAcpRuntimeBackend(backend: AcpRuntimeBackend): void;
export function unregisterAcpRuntimeBackend(id: string): void;
export function getAcpRuntimeBackend(id?: string): AcpRuntimeBackend | null;
export function requireAcpRuntimeBackend(id?: string): AcpRuntimeBackend;
```

Behavior:

- `requireAcpRuntimeBackend` throws a typed ACP backend missing error when unavailable
- plugin service registers backend on `start` and unregisters on `stop`
- runtime lookups are read-only and process-local

### acpx runtime plugin contract (implementation detail)

For the first production backend (`extensions/acpx`), OpenClaw and acpx are
connected with a strict command contract:

- backend id: `acpx`
- plugin service id: `acpx-runtime`
- runtime handle encoding: `runtimeSessionName = acpx:v1:<base64url(json)>`
- encoded payload fields:
  - `name` (acpx named session; uses OpenClaw `sessionKey`)
  - `agent` (acpx agent command)
  - `cwd` (session workspace root)
  - `mode` (`persistent | oneshot`)

Command mapping:

- ensure session:
  - `acpx --format json --json-strict --cwd <cwd> <agent> sessions ensure --name <name>`
- prompt turn:
  - `acpx --format json --json-strict --cwd <cwd> <agent> prompt --session <name> --file -`
- cancel:
  - `acpx --format json --json-strict --cwd <cwd> <agent> cancel --session <name>`
- close:
  - `acpx --format json --json-strict --cwd <cwd> <agent> sessions close <name>`

Streaming:

- OpenClaw consumes ndjson events from `acpx --format json --json-strict`
- `text` => `text_delta/output`
- `thought` => `text_delta/thought`
- `tool_call` => `tool_call`
- `done` => `done`
- `error` => `error`

### Bản vá lược đồ phiên

Bản vá `SessionEntry` trong `src/config/sessions/types.ts`:

```ts
type SessionAcpMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

Persisted field:

- `SessionEntry.acp?: SessionAcpMeta`

Migration rules:

- phase A: dual-write (`acp` projection + ACP SQLite source-of-truth)
- phase B: read-primary from ACP SQLite, fallback-read from legacy `SessionEntry.acp`
- phase C: migration command backfills missing ACP rows from valid legacy entries
- phase D: remove fallback-read and keep projection optional for UX only
- legacy fields (`cliSessionIds`, `claudeCliSessionId`) remain untouched

### Error contract

Add stable ACP error codes and user-facing messages:

- `ACP_BACKEND_MISSING`
  - message: `ACP runtime backend is not configured. Install and enable the acpx runtime plugin.`
- `ACP_BACKEND_UNAVAILABLE`
  - message: `ACP runtime backend is currently unavailable. Try again in a moment.`
- `ACP_SESSION_INIT_FAILED`
  - message: `Could not initialize ACP session runtime.`
- `ACP_TURN_FAILED`
  - message: `ACP turn failed before completion.`

Quy tắc:

- trả về thông báo hữu ích và an toàn cho người dùng trong luồng
- chỉ ghi lỗi chi tiết về backend/hệ thống trong nhật ký runtime
- không bao giờ im lặng quay lại đường dẫn LLM bình thường khi định tuyến ACP được chọn rõ ràng

### Trọng tài giao hàng trùng lặp

Quy tắc định tuyến duy nhất cho các lượt ACP ràng buộc:

- nếu tồn tại ràng buộc luồng hoạt động cho phiên ACP đích và ngữ cảnh yêu cầu, chỉ giao hàng cho luồng ràng buộc đó
- không gửi đến kênh cha cho cùng một lượt
- nếu lựa chọn đích ràng buộc không rõ ràng, thất bại với lỗi rõ ràng (không quay lại cha ngầm)
- nếu không có ràng buộc hoạt động, sử dụng hành vi đích phiên bình thường
### Khả năng quan sát và sẵn sàng hoạt động

Các chỉ số bắt buộc:

- Số lần thành công/thất bại khi tạo ACP theo backend và mã lỗi
- Phần trăm độ trễ chạy ACP (thời gian chờ hàng đợi, thời gian chạy, thời gian dự báo giao hàng)
- Số lần khởi động lại actor ACP và lý do khởi động lại
- Số lần phát hiện binding cũ
- Tỷ lệ hit phát lại idempotency
- Bộ đếm giao hàng Discord và bộ đếm giới hạn tốc độ

Nhật ký bắt buộc:

- nhật ký có cấu trúc được khóa bằng `sessionKey`, `runId`, `backend`, `threadId`, `idempotencyKey`
- nhật ký chuyển đổi trạng thái rõ ràng cho máy trạng thái phiên và chạy
- nhật ký lệnh adapter với các đối số an toàn redaction và tóm tắt thoát

Chẩn đoán bắt buộc:

- `/acp sessions` bao gồm trạng thái, chạy hoạt động, lỗi cuối cùng và trạng thái binding
- `/acp doctor` (hoặc tương đương) xác thực đăng ký backend, tình trạng kho lưu trữ và binding cũ

### Ưu tiên cấu hình và giá trị hiệu quả

Ưu tiên bật ACP:

- ghi đè tài khoản: `channels.discord.accounts.<id>.threadBindings.spawnAcpSessions`
- ghi đè kênh: `channels.discord.threadBindings.spawnAcpSessions`
- cổng ACP toàn cầu: `acp.enabled`
- cổng dispatch: `acp.dispatch.enabled`
- tính khả dụng backend: backend đã đăng ký cho `acp.backend`

Hành vi tự động bật:

- khi ACP được cấu hình (`acp.enabled=true`, `acp.dispatch.enabled=true`, hoặc
  `acp.backend=acpx`), tự động bật plugin đánh dấu `plugins.entries.acpx.enabled=true`
  trừ khi bị từ chối hoặc bị vô hiệu hóa rõ ràng

Giá trị TTL hiệu quả:

- `min(session ttl, discord thread binding ttl, acp runtime ttl)`

### Bản đồ kiểm tra

Kiểm tra đơn vị:

- `src/acp/runtime/registry.test.ts` (mới)
- `src/auto-reply/reply/dispatch-from-config.acp.test.ts` (mới)
- `src/infra/outbound/bound-delivery-router.test.ts` (mở rộng các trường hợp ACP fail-closed)
- `src/config/sessions/types.test.ts` hoặc các kiểm tra session-store gần nhất (tính bền vững siêu dữ liệu ACP)

Kiểm tra tích hợp:

- `src/discord/monitor/reply-delivery.test.ts` (hành vi mục tiêu giao hàng ACP được ràng buộc)
- `src/discord/monitor/message-handler.preflight*.test.ts` (tính liên tục định tuyến khóa phiên ACP được ràng buộc)
- kiểm tra runtime plugin acpx trong gói backend (đăng ký/bắt đầu/dừng dịch vụ + chuẩn hóa sự kiện)

Kiểm tra e2e Gateway:
- `src/gateway/server.sessions.gateway-server-sessions-a.e2e.test.ts` (mở rộng phạm vi bảo vệ vòng đời đặt lại/xóa ACP)
- ACP thread turn roundtrip e2e cho spawn, message, stream, cancel, unfocus, restart recovery

### Rollout guard

Thêm công tắc kill dispatch ACP độc lập:

- `acp.dispatch.enabled` mặc định `false` cho bản phát hành đầu tiên
- khi bị vô hiệu hóa:
  - các lệnh điều khiển spawn/focus ACP vẫn có thể liên kết các phiên
  - đường dẫn dispatch ACP không được kích hoạt
  - người dùng nhận được thông báo rõ ràng rằng dispatch ACP bị vô hiệu hóa theo chính sách
- sau khi xác thực canary, mặc định có thể được chuyển sang `true` trong bản phát hành sau
## Kế hoạch lệnh và UX

### Lệnh mới

- `/acp spawn <agent-id> [--mode persistent|oneshot] [--thread auto|here|off]`
- `/acp cancel [session]`
- `/acp steer <instruction>`
- `/acp close [session]`
- `/acp sessions`

### Tương thích lệnh hiện có

- `/focus <sessionKey>` tiếp tục hỗ trợ các mục tiêu ACP
- `/unfocus` giữ ngữ nghĩa hiện tại
- `/session idle` và `/session max-age` thay thế ghi đè TTL cũ
## Triển khai theo giai đoạn

### Giai đoạn 0 ADR và đóng băng schema

- gửi ADR cho quyền sở hữu control-plane ACP và ranh giới adapter
- đóng băng DB schema (`acp_sessions`, `acp_runs`, `acp_bindings`, `acp_events`, `acp_delivery_checkpoint`, `acp_idempotency`)
- định nghĩa mã lỗi ACP ổn định, hợp đồng sự kiện và bảo vệ chuyển đổi trạng thái

### Giai đoạn 1 Nền tảng control-plane trong core

- triển khai `AcpSessionManager` và runtime actor cho mỗi phiên
- triển khai kho lưu trữ ACP SQLite và trợ giúp giao dịch
- triển khai kho lưu trữ idempotency và trợ giúp phát lại
- triển khai các mô-đun append sự kiện + checkpoint giao hàng
- kết nối các API spawn/cancel/close với manager với đảm bảo giao dịch

### Giai đoạn 2 Định tuyến core và tích hợp vòng đời

- định tuyến các lượt ACP ràng buộc thread từ pipeline dispatch vào ACP manager
- thực thi định tuyến fail-closed khi các bất biến ràng buộc/phiên ACP không thành công
- tích hợp vòng đời reset/delete/archive/unfocus với các giao dịch ACP close/unbind
- thêm phát hiện ràng buộc cũ và chính sách auto-unbind tùy chọn

### Giai đoạn 3 adapter/plugin backend acpx

- triển khai adapter `acpx` dựa trên hợp đồng runtime (`ensureSession`, `submit`, `stream`, `cancel`, `close`)
- thêm kiểm tra sức khỏe backend và đăng ký startup/teardown
- chuẩn hóa các sự kiện acpx ndjson thành các sự kiện runtime ACP
- thực thi timeout backend, giám sát quy trình và chính sách restart/backoff

### Giai đoạn 4 Phép chiếu giao hàng và UX kênh (Discord trước tiên)

- triển khai phép chiếu kênh được điều khiển bởi sự kiện với checkpoint resume (Discord trước tiên)
- gộp các khối truyền phát với chính sách flush nhận thức giới hạn tốc độ
- đảm bảo thông báo hoàn thành cuối cùng chính xác một lần cho mỗi lần chạy
- gửi `/acp spawn`, `/acp cancel`, `/acp steer`, `/acp close`, `/acp sessions`

### Giai đoạn 5 Di chuyển và chuyển đổi

- giới thiệu dual-write cho phép chiếu `SessionEntry.acp` cộng với nguồn sự thật ACP SQLite
- thêm tiện ích di chuyển cho các hàng siêu dữ liệu ACP kế thừa
- lật đường dẫn đọc thành ACP SQLite chính
- xóa định tuyến fallback kế thừa phụ thuộc vào `SessionEntry.acp` bị thiếu

### Giai đoạn 6 Cứng hóa, SLO và giới hạn quy mô

- thực thi giới hạn đồng thời (toàn cầu/tài khoản/phiên), chính sách hàng đợi và ngân sách timeout
- thêm telemetry đầy đủ, bảng điều khiển và ngưỡng cảnh báo
- kiểm tra chaos cho khôi phục sự cố và triệt tiêu giao hàng trùng lặp
- xuất bản runbook cho tình trạng mất điện backend, hỏng DB và khắc phục ràng buộc cũ

### Danh sách kiểm tra triển khai đầy đủ

- các mô-đun control-plane core và bài kiểm tra
- di chuyển DB và kế hoạch rollback
- tích hợp API ACP manager trên dispatch và commands
- giao diện đăng ký adapter trong plugin runtime bridge
- triển khai adapter acpx và bài kiểm tra
- logic phép chiếu giao hàng kênh có khả năng thread với replay checkpoint (Discord trước tiên)
- hook vòng đời cho reset/delete/archive/unfocus
- trình phát hiện liên kết cũ và chẩn đoán hướng tới nhà điều hành
- kiểm tra xác thực cấu hình và kiểm tra ưu tiên cho tất cả các khóa ACP mới
- tài liệu hoạt động và sổ tay khắc phục sự cố
## Kế hoạch kiểm tra

Kiểm tra đơn vị:

- Ranh giới giao dịch ACP DB (spawn/bind/enqueue tính nguyên tử, cancel, close)
- Bảo vệ máy trạng thái ACP cho các phiên và chạy
- Ngữ nghĩa idempotency reservation/replay trên tất cả các lệnh ACP
- Tuần tự hóa actor và sắp xếp hàng đợi cho mỗi phiên
- Trình phân tích sự kiện acpx và coalescer khối
- Chính sách khởi động lại và backoff của giám sát runtime
- Ưu tiên cấu hình và tính toán TTL hiệu quả
- Lựa chọn nhánh định tuyến ACP cốt lõi và hành vi fail-closed khi backend/phiên không hợp lệ

Kiểm tra tích hợp:

- Quy trình bộ điều hợp ACP giả để truyền phát và hành vi cancel xác định
- Tích hợp ACP manager + dispatch với tính bền vững giao dịch
- Định tuyến inbound liên kết thread đến khóa phiên ACP
- Phân phối outbound liên kết thread loại bỏ nhân bản kênh cha
- Replay checkpoint phục hồi sau lỗi phân phối và tiếp tục từ sự kiện cuối cùng
- Đăng ký dịch vụ plugin và tháo dỡ backend runtime ACP

Kiểm tra e2e Gateway:

- Spawn ACP với thread, trao đổi các lời nhắc đa lần, unfocus
- Khởi động lại gateway với ACP DB và liên kết được lưu giữ, sau đó tiếp tục cùng một phiên
- Các phiên ACP đồng thời trong nhiều thread không có cross-talk
- Thử lại lệnh trùng lặp (cùng khóa idempotency) không tạo chạy hoặc trả lời trùng lặp
- Kịch bản liên kết cũ mang lại lỗi rõ ràng và hành vi tự động làm sạch tùy chọn
## Rủi ro và biện pháp giảm thiểu

- Giao hàng trùng lặp trong quá trình chuyển đổi
  - Biện pháp giảm thiểu: bộ phân giải đích duy nhất và điểm kiểm tra sự kiện idempotent
- Quá trình runtime bị thay đổi dưới tải
  - Biện pháp giảm thiểu: chủ sở hữu phiên lâu dài + giới hạn đồng thời + backoff
- Plugin vắng mặt hoặc cấu hình sai
  - Biện pháp giảm thiểu: lỗi rõ ràng hướng tới nhà điều hành và định tuyến ACP đóng lại (không có fallback ngầm định tới đường dẫn phiên bình thường)
- Nhầm lẫn cấu hình giữa subagent và cổng ACP
  - Biện pháp giảm thiểu: các khóa ACP rõ ràng và phản hồi lệnh bao gồm nguồn chính sách hiệu quả
- Hỏng hóc kho lưu trữ mặt phẳng điều khiển hoặc lỗi di chuyển
  - Biện pháp giảm thiểu: chế độ WAL, móc sao lưu/khôi phục, kiểm tra khói di chuyển và chẩn đoán fallback chỉ đọc
- Deadlock actor hoặc starvation hộp thư
  - Biện pháp giảm thiểu: bộ đếm thời gian watchdog, kiểm tra sức khỏe actor và độ sâu hộp thư bị giới hạn với telemetry từ chối
## Danh sách kiểm tra chấp nhận

- Phiên ACP spawn có thể tạo hoặc liên kết một thread trong bộ điều hợp kênh được hỗ trợ (hiện tại là Discord)
- tất cả các tin nhắn thread định tuyến đến phiên ACP được liên kết duy nhất
- các đầu ra ACP xuất hiện trong cùng một định danh thread với truyền phát hoặc lô
- không có đầu ra trùng lặp trong kênh cha cho các lượt được liên kết
- spawn+bind+initial enqueue là nguyên tử trong kho lưu trữ bền vững
- các lần thử lại lệnh ACP là idempotent và không tạo ra các lần chạy hoặc đầu ra trùng lặp
- cancel, close, unfocus, archive, reset, và delete thực hiện dọn dẹp xác định
- khởi động lại sau sự cố bảo toàn ánh xạ và tiếp tục tính liên tục nhiều lượt
- các phiên ACP được liên kết thread đồng thời hoạt động độc lập
- trạng thái bị thiếu của backend ACP tạo ra lỗi rõ ràng và có thể hành động
- các liên kết cũ được phát hiện và hiển thị rõ ràng (với tùy chọn tự động dọn dẹp an toàn)
- các chỉ số mặt phẳng điều khiển và chẩn đoán có sẵn cho các nhà điều hành
- bảo phủ đơn vị, tích hợp và e2e mới vượt qua
## Phụ lục: Tái cấu trúc có mục tiêu cho triển khai hiện tại (trạng thái)

Đây là những công việc tiếp theo không chặn để giữ cho đường dẫn ACP dễ bảo trì sau khi bộ tính năng hiện tại được triển khai.

### 1) Tập trung đánh giá chính sách phân phối ACP (hoàn thành)

- được triển khai thông qua các trình trợ giúp chính sách ACP được chia sẻ trong `src/acp/policy.ts`
- các trình xử lý vòng đời lệnh phân phối, ACP và đường dẫn sinh ACP hiện tiêu thụ logic chính sách được chia sẻ

### 2) Chia trình xử lý lệnh ACP theo miền lệnh con (hoàn thành)

- `src/auto-reply/reply/commands-acp.ts` hiện là một bộ định tuyến mỏng
- hành vi lệnh con được chia thành:
  - `src/auto-reply/reply/commands-acp/lifecycle.ts`
  - `src/auto-reply/reply/commands-acp/runtime-options.ts`
  - `src/auto-reply/reply/commands-acp/diagnostics.ts`
  - các trình trợ giúp được chia sẻ trong `src/auto-reply/reply/commands-acp/shared.ts`

### 3) Chia trình quản lý phiên ACP theo trách nhiệm (hoàn thành)

- trình quản lý được chia thành:
  - `src/acp/control-plane/manager.ts` (واجهة عامة + singleton)
  - `src/acp/control-plane/manager.core.ts` (triển khai trình quản lý)
  - `src/acp/control-plane/manager.types.ts` (loại trình quản lý/phụ thuộc)
  - `src/acp/control-plane/manager.utils.ts` (chuẩn hóa + các hàm trợ giúp)

### 4) Dọn dẹp bộ điều hợp thời gian chạy acpx tùy chọn

- `extensions/acpx/src/runtime.ts` có thể được chia thành:
- thực thi/giám sát quy trình
- phân tích cú pháp/chuẩn hóa sự kiện ndjson
- bề mặt API thời gian chạy (`submit`, `cancel`, `close`, v.v.)
- cải thiện khả năng kiểm tra và giúp dễ dàng kiểm toán hành vi backend