---
summary: >-
  Công cụ phiên làm việc của Agent để liệt kê các phiên, lấy lịch sử và gửi tin
  nhắn giữa các phiên
read_when:
  - Thêm hoặc sửa đổi công cụ phiên
title: Công cụ Phiên làm việc
x-i18n:
  source_path: concepts\session-tool.md
  source_hash: 0afe11bd9af198eaa3014c93f84828b8c69e95f9c68b2d82999476cdae9f5532
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:59.270Z'
---

# Công cụ Phiên

Mục tiêu: bộ công cụ nhỏ, khó sử dụng sai để các agent có thể liệt kê các phiên, tìm nạp lịch sử và gửi đến một phiên khác.
## Tên công cụ

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`
## Mô hình Khóa

- Thùng trò chuyện trực tiếp chính luôn là khóa theo nghĩa đen `"main"` (được phân giải thành khóa chính của agent hiện tại).
- Trò chuyện nhóm sử dụng `agent:<agentId>:<channel>:group:<id>` hoặc `agent:<agentId>:<channel>:channel:<id>` (truyền khóa đầy đủ).
- Cron jobs sử dụng `cron:<job.id>`.
- Hooks sử dụng `hook:<uuid>` trừ khi được đặt rõ ràng.
- Phiên node sử dụng `node-<nodeId>` trừ khi được đặt rõ ràng.

`global` và `unknown` là các giá trị dành riêng và không bao giờ được liệt kê. Nếu `session.scope = "global"`, chúng tôi sẽ đặt bí danh thành `main` cho tất cả các công cụ để những người gọi không bao giờ thấy `global`.
## sessions_list

Liệt kê các phiên dưới dạng mảng các hàng.

Tham số:

- `kinds?: string[]` filter: bất kỳ `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` số hàng tối đa (mặc định: mặc định máy chủ, giới hạn ví dụ 200)
- `activeMinutes?: number` chỉ các phiên được cập nhật trong N phút
- `messageLimit?: number` 0 = không có tin nhắn (mặc định 0); >0 = bao gồm N tin nhắn cuối cùng

Hành vi:

- `messageLimit > 0` tìm nạp `chat.history` cho mỗi phiên và bao gồm N tin nhắn cuối cùng.
- Kết quả công cụ được lọc ra khỏi đầu ra danh sách; sử dụng `sessions_history` cho tin nhắn công cụ.
- Khi chạy trong phiên agent **sandboxed**, các công cụ phiên mặc định là **spawned-only visibility** (xem bên dưới).

Hình dạng hàng (JSON):

- `key`: khóa phiên (chuỗi)
- `kind`: `main | group | cron | hook | node | other`
- `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName` (nhãn hiển thị nhóm nếu có)
- `updatedAt` (ms)
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy` (ghi đè phiên nếu được đặt)
- `lastChannel`, `lastTo`
- `deliveryContext` (chuẩn hóa `{ channel, to, accountId }` khi có sẵn)
- `transcriptPath` (đường dẫn được suy ra từ thư mục lưu trữ + sessionId)
- `messages?` (chỉ khi `messageLimit > 0`)
## sessions_history

Lấy bản ghi lại cho một phiên.

Tham số:

- `sessionKey` (bắt buộc; chấp nhận khóa phiên hoặc `sessionId` từ `sessions_list`)
- `limit?: number` số lượng tin nhắn tối đa (máy chủ giới hạn)
- `includeTools?: boolean` (mặc định false)

Hành vi:

- `includeTools=false` lọc các tin nhắn `role: "toolResult"`.
- Trả về mảng tin nhắn ở định dạng bản ghi lại thô.
- Khi được cung cấp một `sessionId`, OpenClaw sẽ phân giải nó thành khóa phiên tương ứng (lỗi id bị thiếu).
## sessions_send

Gửi một tin nhắn vào một phiên khác.

Tham số:

- `sessionKey` (bắt buộc; chấp nhận khóa phiên hoặc `sessionId` từ `sessions_list`)
- `message` (bắt buộc)
- `timeoutSeconds?: number` (mặc định >0; 0 = fire-and-forget)

Hành vi:

- `timeoutSeconds = 0`: xếp hàng và trả về `{ runId, status: "accepted" }`.
- `timeoutSeconds > 0`: chờ tối đa N giây để hoàn thành, sau đó trả về `{ runId, status: "ok", reply }`.
- Nếu chờ hết thời gian: `{ runId, status: "timeout", error }`. Lần chạy tiếp tục; gọi `sessions_history` sau.
- Nếu lần chạy không thành công: `{ runId, status: "error", error }`.
- Các lần chạy thông báo giao hàng diễn ra sau khi lần chạy chính hoàn thành và là best-effort; `status: "ok"` không đảm bảo rằng thông báo đã được gửi.
- Chờ qua gateway `agent.wait` (phía máy chủ) để các kết nối lại không làm mất lần chờ.
- Ngữ cảnh tin nhắn agent-to-agent được tiêm cho lần chạy chính.
- Các tin nhắn giữa các phiên được lưu trữ với `message.provenance.kind = "inter_session"` để những người đọc bảng ghi có thể phân biệt các hướng dẫn agent được định tuyến từ đầu vào người dùng bên ngoài.
- Sau khi lần chạy chính hoàn thành, OpenClaw chạy một **vòng lặp trả lời**:
  - Vòng 2+ xen kẽ giữa các agent yêu cầu và mục tiêu.
  - Trả lời chính xác `REPLY_SKIP` để dừng ping‑pong.
  - Số lượt tối đa là `session.agentToAgent.maxPingPongTurns` (0–5, mặc định 5).
- Khi vòng lặp kết thúc, OpenClaw chạy **bước thông báo agent‑to‑agent** (chỉ agent mục tiêu):
  - Trả lời chính xác `ANNOUNCE_SKIP` để im lặng.
  - Bất kỳ trả lời nào khác được gửi đến kênh mục tiêu.
  - Bước thông báo bao gồm yêu cầu ban đầu + trả lời vòng 1 + trả lời ping‑pong mới nhất.
## Trường Kênh

- Đối với các nhóm, `channel` là kênh được ghi lại trong mục nhập phiên.
- Đối với các cuộc trò chuyện trực tiếp, `channel` ánh xạ từ `lastChannel`.
- Đối với cron/hook/node, `channel` là `internal`.
- Nếu bị thiếu, `channel` là `unknown`.
## Bảo mật / Chính sách gửi

Chặn dựa trên chính sách theo kênh/loại trò chuyện (không phải theo session id).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Runtime override (per session entry):

- `sendPolicy: "allow" | "deny"` (unset = inherit config)
- Settable via `sessions.patch` or owner-only `/send on|off|inherit` (standalone message).

Enforcement points:

- `chat.send` / `agent` (gateway)
- logic phân phối tự động trả lời
## sessions_spawn

Tạo một phiên chạy agent phụ trong một phiên cô lập và thông báo kết quả trở lại kênh trò chuyện của người yêu cầu.

Tham số:

- `task` (bắt buộc)
- `label?` (tùy chọn; được sử dụng cho nhật ký/giao diện người dùng)
- `agentId?` (tùy chọn; tạo dưới một id agent khác nếu được phép)
- `model?` (tùy chọn; ghi đè mô hình agent phụ; các giá trị không hợp lệ sẽ gây lỗi)
- `thinking?` (tùy chọn; ghi đè mức độ suy nghĩ cho lần chạy agent phụ)
- `runTimeoutSeconds?` (mặc định là `agents.defaults.subagents.runTimeoutSeconds` khi được đặt, nếu không là `0`; khi được đặt, hủy bỏ lần chạy agent phụ sau N giây)
- `thread?` (mặc định false; yêu cầu định tuyến liên kết với luồng cho lần tạo này khi được hỗ trợ bởi kênh/plugin)
- `mode?` (`run|session`; mặc định là `run`, nhưng mặc định là `session` khi `thread=true`; `mode="session"` yêu cầu `thread=true`)
- `cleanup?` (`delete|keep`, mặc định `keep`)

Danh sách cho phép:

- `agents.list[].subagents.allowAgents`: danh sách các id agent được phép thông qua `agentId` (`["*"]` để cho phép bất kỳ). Mặc định: chỉ agent yêu cầu.

Khám phá:

- Sử dụng `agents_list` để khám phá những id agent nào được phép cho `sessions_spawn`.

Hành vi:

- Bắt đầu một phiên `agent:<agentId>:subagent:<uuid>` mới với `deliver: false`.
- Các agent phụ mặc định sử dụng bộ công cụ đầy đủ **trừ các công cụ phiên** (có thể cấu hình thông qua `tools.subagents.tools`).
- Các agent phụ không được phép gọi `sessions_spawn` (không có agent phụ → tạo agent phụ).
- Luôn không chặn: trả về `{ status: "accepted", runId, childSessionKey }` ngay lập tức.
- Với `thread=true`, các plugin kênh có thể liên kết giao hàng/định tuyến với mục tiêu luồng (hỗ trợ Discord được kiểm soát bởi `session.threadBindings.*` và `channels.discord.threadBindings.*`).
- Sau khi hoàn thành, OpenClaw chạy một **bước thông báo agent phụ** và đăng kết quả vào kênh trò chuyện của người yêu cầu.
  - Nếu phản hồi cuối cùng của trợ lý trống, `toolResult` mới nhất từ lịch sử agent phụ được đưa vào dưới dạng `Result`.
- Trả lời chính xác `ANNOUNCE_SKIP` trong bước thông báo để giữ im lặng.
- Các phản hồi thông báo được chuẩn hóa thành `Status`/`Result`/`Notes`; `Status` đến từ kết quả thời gian chạy (không phải văn bản mô hình).
- Các phiên agent phụ được lưu trữ tự động sau `agents.defaults.subagents.archiveAfterMinutes` (mặc định: 60).
- Các phản hồi thông báo bao gồm một dòng thống kê (thời gian chạy, token, sessionKey/sessionId, đường dẫn bản ghi, và chi phí tùy chọn).
## Khả Năng Hiển Thị Phiên Sandbox

Các công cụ phiên có thể được giới hạn phạm vi để giảm quyền truy cập giữa các phiên.

Hành vi mặc định:

- `tools.sessions.visibility` mặc định là `tree` (phiên hiện tại + phiên subagent được tạo).
- Đối với các phiên sandboxed, `agents.defaults.sandbox.sessionToolsVisibility` có thể cố định giới hạn khả năng hiển thị.

Cấu hình:

```json5
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // default: "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // default: "spawned"
        sessionToolsVisibility: "spawned", // or "all"
      },
    },
  },
}
```

Notes:

- `self`: only the current session key.
- `tree`: current session + sessions spawned by the current session.
- `agent`: any session belonging to the current agent id.
- `all`: any session (cross-agent access still requires `tools.agentToAgent`).
- When a session is sandboxed and `sessionToolsVisibility="spawned"`, OpenClaw clamps visibility to `tree` even if you set `tools.sessions.visibility="all"`.