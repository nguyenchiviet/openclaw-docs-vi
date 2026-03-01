---
summary: >-
  Bề mặt công cụ Agent cho OpenClaw (browser, canvas, nodes, message, cron) thay
  thế các skills `openclaw-*` cũ
read_when:
  - Thêm hoặc sửa đổi công cụ agent
  - Loại bỏ hoặc thay đổi các `openclaw-*` Skills
title: Công cụ
x-i18n:
  source_path: tools\index.md
  source_hash: 77f9e25ade41869156fcb71507ef5b010e3ee51da06840a68613998d954c30a5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:31:40.598Z'
---

# Công cụ (OpenClaw)

OpenClaw cung cấp **công cụ agent hạng nhất** cho trình duyệt, canvas, node và cron.
Những công cụ này thay thế các `openclaw-*` Skills cũ: các công cụ được gõ kiểu, không cần shell,
và agent nên dựa vào chúng trực tiếp.
## Vô hiệu hóa công cụ

Bạn có thể cho phép/từ chối công cụ trên toàn cầu thông qua `tools.allow` / `tools.deny` trong `openclaw.json`
(từ chối có ưu tiên). Điều này ngăn chặn các công cụ không được phép được gửi đến các nhà cung cấp mô hình.

```json5
{
  tools: { deny: ["browser"] },
}
```

Notes:

- Matching is case-insensitive.
- `*` wildcards are supported (`"*"` means all tools).
- If `tools.allow` chỉ tham chiếu đến các tên công cụ plugin không xác định hoặc chưa được tải, OpenClaw ghi lại cảnh báo và bỏ qua danh sách cho phép để các công cụ cốt lõi vẫn khả dụng.
## Hồ sơ công cụ (danh sách cho phép cơ bản)

`tools.profile` đặt **danh sách cho phép công cụ cơ bản** trước `tools.allow`/`tools.deny`.
Ghi đè cho từng agent: `agents.list[].tools.profile`.

Hồ sơ:

- `minimal`: `session_status` chỉ
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: không có hạn chế (giống như chưa đặt)

Ví dụ (chỉ nhắn tin theo mặc định, cho phép công cụ Slack + Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Example (coding profile, but deny exec/process everywhere):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

Example (global coding profile, messaging-only support agent):

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```
## Chính sách công cụ dành riêng cho nhà cung cấp

Sử dụng `tools.byProvider` để **hạn chế thêm** công cụ cho các nhà cung cấp cụ thể
(hoặc một `provider/model`) mà không thay đổi các cài đặt mặc định toàn cầu của bạn.
Ghi đè cho mỗi agent: `agents.list[].tools.byProvider`.

Điều này được áp dụng **sau** hồ sơ công cụ cơ sở và **trước** danh sách cho phép/từ chối,
vì vậy nó chỉ có thể thu hẹp bộ công cụ.
Các khóa nhà cung cấp chấp nhận `provider` (ví dụ: `google-antigravity`) hoặc
`provider/model` (ví dụ: `openai/gpt-5.2`).

Ví dụ (giữ hồ sơ mã hóa toàn cầu, nhưng công cụ tối thiểu cho Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Example (provider/model-specific allowlist for a flaky endpoint):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

Example (agent-specific override for a single provider):

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```
## Nhóm công cụ (viết tắt)

Chính sách công cụ (toàn cục, agent, sandbox) hỗ trợ các mục `group:*` mở rộng thành nhiều công cụ.
Sử dụng các mục này trong `tools.allow` / `tools.deny`.

Các nhóm có sẵn:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: tất cả các công cụ OpenClaw tích hợp sẵn (không bao gồm plugin nhà cung cấp)

Ví dụ (chỉ cho phép công cụ tệp + trình duyệt):

```json5
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```
## Plugins + công cụ

Plugins có thể đăng ký **các công cụ bổ sung** (và lệnh CLI) ngoài bộ cốt lõi.
Xem [Plugins](/tools/plugin) để cài đặt + cấu hình, và [Skills](/tools/skills) để biết cách
hướng dẫn sử dụng công cụ được đưa vào prompts. Một số plugins đi kèm với các Skills riêng
cùng với công cụ (ví dụ: plugin cuộc gọi thoại).

Công cụ plugin tùy chọn:

- [Lobster](/tools/lobster): runtime quy trình làm việc được gõ với các phê duyệt có thể tiếp tục (yêu cầu Lobster CLI trên máy chủ gateway).
- [LLM Task](/tools/llm-task): bước LLM chỉ JSON cho đầu ra quy trình làm việc có cấu trúc (xác thực lược đồ tùy chọn).
## Kho công cụ

### `apply_patch`

Áp dụng các bản vá có cấu trúc trên một hoặc nhiều tệp. Sử dụng cho các chỉnh sửa đa khối.
Thử nghiệm: bật qua `tools.exec.applyPatch.enabled` (chỉ các mô hình OpenAI).
`tools.exec.applyPatch.workspaceOnly` mặc định là `true` (chứa trong không gian làm việc). Đặt nó thành `false` chỉ khi bạn có ý định muốn `apply_patch` ghi/xóa bên ngoài thư mục không gian làm việc.

### `exec`

Chạy các lệnh shell trong không gian làm việc.

Các tham số cốt lõi:

- `command` (bắt buộc)
- `yieldMs` (tự động chạy nền sau khi hết thời gian, mặc định 10000)
- `background` (chạy nền ngay lập tức)
- `timeout` (giây; kết thúc quá trình nếu vượt quá, mặc định 1800)
- `elevated` (bool; chạy trên máy chủ nếu chế độ nâng cao được bật/cho phép; chỉ thay đổi hành vi khi agent được sandbox)
- `host` (`sandbox | gateway | node`)
- `security` (`deny | allowlist | full`)
- `ask` (`off | on-miss | always`)
- `node` (id/tên node cho `host=node`)
- Cần TTY thực? Đặt `pty: true`.

Ghi chú:

- Trả về `status: "running"` với `sessionId` khi chạy nền.
- Sử dụng `process` để kiểm tra/ghi nhật ký/ghi/kết thúc/xóa các phiên nền.
- Nếu `process` không được phép, `exec` chạy đồng bộ và bỏ qua `yieldMs`/`background`.
- `elevated` được kiểm soát bởi `tools.elevated` cộng với bất kỳ ghi đè `agents.list[].tools.elevated` nào (cả hai phải cho phép) và là bí danh cho `host=gateway` + `security=full`.
- `elevated` chỉ thay đổi hành vi khi agent được sandbox (nếu không nó là không hoạt động).
- `host=node` có thể nhắm mục tiêu đến một ứng dụng đi kèm macOS hoặc một máy chủ node không đầu (`openclaw node run`).
- Phê duyệt gateway/node và danh sách cho phép: [Phê duyệt Exec](/tools/exec-approvals).

### `process`

Quản lý các phiên exec nền.

Các hành động cốt lõi:

- `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Ghi chú:

- `poll` trả về đầu ra mới và trạng thái thoát khi hoàn thành.
- `log` hỗ trợ `offset`/`limit` dựa trên dòng (bỏ qua `offset` để lấy N dòng cuối cùng).
- `process` được phạm vi cho mỗi agent; các phiên từ các agent khác không hiển thị.

### `loop-detection` (guardrails vòng lặp gọi công cụ)

OpenClaw theo dõi lịch sử gọi công cụ gần đây và chặn hoặc cảnh báo khi phát hiện các vòng lặp lặp lại không có tiến bộ.
Bật với `tools.loopDetection.enabled: true` (mặc định là `false`).

```json5
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```
- `genericRepeat`: mẫu gọi công cụ giống nhau + tham số giống nhau được lặp lại.
- `knownPollNoProgress`: lặp lại các công cụ kiểu poll với đầu ra giống hệt nhau.
- `pingPong`: các mẫu `A/B/A/B` không có tiến triển xen kẽ.
- Ghi đè theo agent: `agents.list[].tools.loopDetection`.

### `web_search`

Tìm kiếm trên web bằng Brave Search API.

Các tham số cốt lõi:

- `query` (bắt buộc)
- `count` (1–10; mặc định từ `tools.web.search.maxResults`)

Ghi chú:

- Yêu cầu khóa API Brave (khuyến nghị: `openclaw configure --section web`, hoặc đặt `BRAVE_API_KEY`).
- Bật qua `tools.web.search.enabled`.
- Các phản hồi được lưu vào bộ nhớ đệm (mặc định 15 phút).
- Xem [Web tools](/tools/web) để thiết lập.

### `web_fetch`

Tìm nạp và trích xuất nội dung có thể đọc được từ URL (HTML → markdown/text).

Các tham số cốt lõi:

- `url` (bắt buộc)
- `extractMode` (`markdown` | `text`)
- `maxChars` (cắt ngắn các trang dài)

Ghi chú:

- Bật qua `tools.web.fetch.enabled`.
- `maxChars` bị giới hạn bởi `tools.web.fetch.maxCharsCap` (mặc định 50000).
- Các phản hồi được lưu vào bộ nhớ đệm (mặc định 15 phút).
- Đối với các trang nặng JS, hãy ưu tiên công cụ trình duyệt.
- Xem [Web tools](/tools/web) để thiết lập.
- Xem [Firecrawl](/tools/firecrawl) để tìm hiểu về fallback chống bot tùy chọn.

### `browser`

Kiểm soát trình duyệt được OpenClaw quản lý chuyên dụng.

Các hành động cốt lõi:

- `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
- `snapshot` (aria/ai)
- `screenshot` (trả về khối hình ảnh + `MEDIA:<path>`)
- `act` (hành động UI: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
- `navigate`, `console`, `pdf`, `upload`, `dialog`

Quản lý hồ sơ:

- `profiles` — liệt kê tất cả các hồ sơ trình duyệt với trạng thái
- `create-profile` — tạo hồ sơ mới với cổng được phân bổ tự động (hoặc `cdpUrl`)
- `delete-profile` — dừng trình duyệt, xóa dữ liệu người dùng, xóa khỏi cấu hình (chỉ cục bộ)
- `reset-profile` — kết thúc quy trình mồ côi trên cổng của hồ sơ (chỉ cục bộ)
Các tham số chung:

- `profile` (tùy chọn; mặc định là `browser.defaultProfile`)
- `target` (`sandbox` | `host` | `node`)
- `node` (tùy chọn; chọn một ID/tên node cụ thể)
  Ghi chú:
- Yêu cầu `browser.enabled=true` (mặc định là `true`; đặt `false` để vô hiệu hóa).
- Tất cả các hành động chấp nhận tham số `profile` tùy chọn để hỗ trợ nhiều phiên bản.
- Khi `profile` bị bỏ qua, sử dụng `browser.defaultProfile` (mặc định là "chrome").
- Tên hồ sơ: chữ thường alphanumeric + dấu gạch ngang (tối đa 64 ký tự).
- Phạm vi cổng: 18800-18899 (~100 hồ sơ tối đa).
- Hồ sơ từ xa chỉ có thể đính kèm (không khởi động/dừng/đặt lại).
- Nếu một node có khả năng trình duyệt được kết nối, công cụ có thể tự động định tuyến đến nó (trừ khi bạn ghim `target`).
- `snapshot` mặc định là `ai` khi Playwright được cài đặt; sử dụng `aria` cho cây khả năng truy cập.
- `snapshot` cũng hỗ trợ các tùy chọn role-snapshot (`interactive`, `compact`, `depth`, `selector`) trả về các tham chiếu như `e12`.
- `act` yêu cầu `ref` từ `snapshot` (numeric `12` từ ảnh chụp AI, hoặc `e12` từ ảnh chụp role); sử dụng `evaluate` cho nhu cầu bộ chọn CSS hiếm gặp.
- Tránh `act` → `wait` theo mặc định; chỉ sử dụng nó trong các trường hợp ngoại lệ (không có trạng thái UI đáng tin cậy để chờ).
- `upload` có thể tùy chọn chuyển một `ref` để tự động nhấp sau khi chuẩn bị.
- `upload` cũng hỗ trợ `inputRef` (aria ref) hoặc `element` (bộ chọn CSS) để đặt `<input type="file">` trực tiếp.

### `canvas`

Điều khiển Canvas node (present, eval, snapshot, A2UI).

Các hành động cốt lõi:

- `present`, `hide`, `navigate`, `eval`
- `snapshot` (trả về khối hình ảnh + `MEDIA:<path>`)
- `a2ui_push`, `a2ui_reset`

Ghi chú:

- Sử dụng gateway `node.invoke` dưới nền.
- Nếu không cung cấp `node`, công cụ sẽ chọn mặc định (một node được kết nối hoặc node mac cục bộ).
- A2UI chỉ là v0.8 (không `createSurface`); CLI từ chối v0.9 JSONL với lỗi dòng.
- Kiểm tra nhanh: `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`.

### `nodes`

Khám phá và nhắm mục tiêu các node được ghép nối; gửi thông báo; chụp camera/màn hình.

Các hành động cốt lõi:

- `status`, `describe`
- `pending`, `approve`, `reject` (ghép nối)
- `notify` (macOS `system.notify`)
- `run` (macOS `system.run`)
- `camera_list`, `camera_snap`, `camera_clip`, `screen_record`
- `location_get`, `notifications_list`, `notifications_action`
- `device_status`, `device_info`, `device_permissions`, `device_health`

Ghi chú:

- Các lệnh camera/màn hình yêu cầu ứng dụng node ở phía trước.
- Hình ảnh trả về khối hình ảnh + `MEDIA:<path>`.
- Video trả về `FILE:<path>` (mp4).
- Vị trí trả về tải trọng JSON (lat/lon/accuracy/timestamp).
- Các tham số `run`: `command` mảng argv; `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording` tùy chọn.

Ví dụ (`run`):
```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### `image`

Analyze an image with the configured image model.

Core parameters:

- `image` (required path or URL)
- `prompt` (optional; defaults to "Describe the image.")
- `model` (optional override)
- `maxBytesMb` (optional size cap)

Notes:

- Only available when `agents.defaults.imageModel` is configured (primary or fallbacks), or when an implicit image model can be inferred from your default model + configured auth (best-effort pairing).
- Uses the image model directly (independent of the main chat model).

### `message`

Send messages and channel actions across Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Core actions:

- `send` (text + optional media; MS Teams also supports `card` for Adaptive Cards)
- `poll` (WhatsApp/Discord/MS Teams polls)
- `react` / `reactions` / `read` / `edit` / `delete`
- `pin` / `unpin` / `list-pins`
- `permissions`
- `thread-create` / `thread-list` / `thread-reply`
- `search`
- `sticker`
- `member-info` / `role-info`
- `emoji-list` / `emoji-upload` / `sticker-upload`
- `role-add` / `role-remove`
- `channel-info` / `channel-list`
- `voice-status`
- `event-list` / `event-create`
- `timeout` / `kick` / `ban`

Notes:

- `send` routes WhatsApp via the Gateway; other channels go direct.
- `poll` uses the Gateway for WhatsApp and MS Teams; Discord polls go direct.
- When a message tool call is bound to an active chat session, sends are constrained to that session’s target to avoid cross-context leaks.

### `cron`

Quản lý cron job và đánh thức Gateway.
Các hành động cốt lõi:

- `status`, `list`
- `add`, `update`, `remove`, `run`, `runs`
- `wake` (xếp hàng sự kiện hệ thống + heartbeat tức thời tùy chọn)

Ghi chú:

- `add` mong đợi một đối tượng cron job đầy đủ (cùng schema với `cron.add` RPC).
- `update` sử dụng `{ jobId, patch }` (`id` được chấp nhận để tương thích).

### `gateway`

Khởi động lại hoặc áp dụng cập nhật cho quá trình Gateway đang chạy (tại chỗ).

Các hành động cốt lõi:

- `restart` (ủy quyền + gửi `SIGUSR1` để khởi động lại trong quá trình; `openclaw gateway` khởi động lại tại chỗ)
- `config.get` / `config.schema`
- `config.apply` (xác thực + ghi cấu hình + khởi động lại + đánh thức)
- `config.patch` (hợp nhất cập nhật một phần + khởi động lại + đánh thức)
- `update.run` (chạy cập nhật + khởi động lại + đánh thức)

Ghi chú:

- Sử dụng `delayMs` (mặc định là 2000) để tránh gián đoạn một phản hồi đang diễn ra.
- `restart` được bật theo mặc định; đặt `commands.restart: false` để tắt nó.

### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`

Liệt kê các phiên, kiểm tra lịch sử bảng điểm hoặc gửi đến phiên khác.

Các tham số cốt lõi:

- `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = không có)
- `sessions_history`: `sessionKey` (hoặc `sessionId`), `limit?`, `includeTools?`
- `sessions_send`: `sessionKey` (hoặc `sessionId`), `message`, `timeoutSeconds?` (0 = gửi và quên)
- `sessions_spawn`: `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`
- `session_status`: `sessionKey?` (mặc định hiện tại; chấp nhận `sessionId`), `model?` (`default` xóa ghi đè)

Ghi chú:

- `main` là khóa trò chuyện trực tiếp chính tắc; toàn cầu/không xác định bị ẩn.
- `messageLimit > 0` tìm nạp N tin nhắn cuối cùng cho mỗi phiên (tin nhắn công cụ được lọc).
- Nhắm mục tiêu phiên được kiểm soát bởi `tools.sessions.visibility` (mặc định `tree`: phiên hiện tại + phiên subagent được tạo). Nếu bạn chạy một agent được chia sẻ cho nhiều người dùng, hãy cân nhắc đặt `tools.sessions.visibility: "self"` để ngăn chặn duyệt xuyên phiên.
- `sessions_send` chờ hoàn thành cuối cùng khi `timeoutSeconds > 0`.
- Giao hàng/thông báo xảy ra sau khi hoàn thành và là nỗ lực tốt nhất; `status: "ok"` xác nhận agent run kết thúc, không phải announce được giao hàng.
- `sessions_spawn` hỗ trợ `runtime: "subagent" | "acp"` (`subagent` mặc định). Để biết hành vi runtime ACP, xem [ACP Agents](/tools/acp-agents).
- `sessions_spawn` bắt đầu một sub-agent run và đăng một phản hồi thông báo trở lại kênh yêu cầu.
  - Hỗ trợ chế độ một lần (`mode: "run"`) và chế độ liên kết luồng liên tục (`mode: "session"` với `thread: true`).
  - Nếu `thread: true` và `mode` bị bỏ qua, chế độ mặc định là `session`.
  - `mode: "session"` yêu cầu `thread: true`.
  - Nếu `runTimeoutSeconds` bị bỏ qua, OpenClaw sử dụng `agents.defaults.subagents.runTimeoutSeconds` khi được đặt; nếu không, timeout mặc định là `0` (không có timeout).
  - Luồng liên kết Discord phụ thuộc vào `session.threadBindings.*` và `channels.discord.threadBindings.*`.
  - Định dạng phản hồi bao gồm `Status`, `Result` và thống kê nhỏ gọn.
  - `Result` là văn bản hoàn thành trợ lý; nếu bị thiếu, `toolResult` mới nhất được sử dụng làm dự phòng.
- Các spawn chế độ hoàn thành thủ công gửi trực tiếp trước tiên, với dự phòng hàng đợi và thử lại khi gặp lỗi tạm thời (`status: "ok"` có nghĩa là run kết thúc, không phải announce được giao hàng).
- `sessions_spawn` không chặn và trả về `status: "accepted"` ngay lập tức.
- `sessions_send` chạy một ping-pong trả lời (trả lời `REPLY_SKIP` để dừng; lượt tối đa qua `session.agentToAgent.maxPingPongTurns`, 0–5).
- Sau ping-pong, agent đích chạy một **bước thông báo**; trả lời `ANNOUNCE_SKIP` để chặn thông báo.
- Sandbox clamp: khi phiên hiện tại được sandboxed và `agents.defaults.sandbox.sessionToolsVisibility: "spawned"`, OpenClaw clamps `tools.sessions.visibility` thành `tree`.

### `agents_list`

Liệt kê các id agent mà phiên hiện tại có thể nhắm mục tiêu với `sessions_spawn`.

Ghi chú:

- Kết quả bị giới hạn bởi danh sách cho phép theo agent (`agents.list[].subagents.allowAgents`).
- Khi `["*"]` được cấu hình, công cụ bao gồm tất cả các agent được cấu hình và đánh dấu `allowAny: true`.
## Tham số (chung)

Các công cụ được hỗ trợ bởi Gateway (`canvas`, `nodes`, `cron`):

- `gatewayUrl` (mặc định `ws://127.0.0.1:18789`)
- `gatewayToken` (nếu bật xác thực)
- `timeoutMs`

Lưu ý: khi `gatewayUrl` được đặt, hãy bao gồm `gatewayToken` một cách rõ ràng. Các công cụ không kế thừa cấu hình hoặc thông tin xác thực môi trường để ghi đè, và thiếu thông tin xác thực rõ ràng là một lỗi.

Công cụ trình duyệt:

- `profile` (tùy chọn; mặc định là `browser.defaultProfile`)
- `target` (`sandbox` | `host` | `node`)
- `node` (tùy chọn; ghim một node id/name cụ thể)
## Các luồng agent được khuyến nghị

Tự động hóa trình duyệt:

1. `browser` → `status` / `start`
2. `snapshot` (ai hoặc aria)
3. `act` (click/type/press)
4. `screenshot` nếu bạn cần xác nhận trực quan

Kết xuất Canvas:

1. `canvas` → `present`
2. `a2ui_push` (tùy chọn)
3. `snapshot`

Nhắm mục tiêu nút:

1. `nodes` → `status`
2. `describe` trên nút đã chọn
3. `notify` / `run` / `camera_snap` / `screen_record`
## An toàn

- Tránh `system.run` trực tiếp; chỉ sử dụng `nodes` → `run` với sự đồng ý rõ ràng của người dùng.
- Tôn trọng sự đồng ý của người dùng đối với việc chụp camera/màn hình.
- Sử dụng `status/describe` để đảm bảo quyền trước khi gọi các lệnh phương tiện.
## Cách các công cụ được trình bày cho agent

Các công cụ được hiển thị qua hai kênh song song:

1. **Văn bản system prompt**: danh sách dễ đọc + hướng dẫn.
2. **Tool schema**: các định nghĩa hàm có cấu trúc được gửi đến API mô hình.

Điều đó có nghĩa là agent nhìn thấy cả "những công cụ nào tồn tại" và "cách gọi chúng." Nếu một công cụ không xuất hiện trong system prompt hoặc schema, mô hình không thể gọi nó.