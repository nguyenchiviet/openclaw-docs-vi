---
summary: Webhook đầu vào cho việc đánh thức và chạy agent cô lập
read_when:
  - Thêm hoặc thay đổi các endpoint webhook
  - Kết nối các hệ thống bên ngoài vào OpenClaw
title: Webhooks
x-i18n:
  source_path: automation\webhook.md
  source_hash: 67b138101b1c7d4d13ea9fd611ffaf383dca1e7da4da421e974874dfb889aab0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:12:46.902Z'
---

# Webhooks

Gateway có thể cung cấp một endpoint webhook HTTP nhỏ cho các trigger bên ngoài.
## Kích hoạt

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // Optional: restrict explicit `agentId` routing to this allowlist.
    // Omit or include "*" to allow any agent.
    // Set [] to deny all explicit `agentId` routing.
    allowedAgentIds: ["hooks", "main"],
  },
}
```

Notes:

- `hooks.token` is required when `hooks.enabled=true`.
- `hooks.path` defaults to `/hooks`.
## Xác thực

Mọi yêu cầu phải bao gồm token hook. Ưu tiên sử dụng headers:

- `Authorization: Bearer <token>` (khuyến nghị)
- `x-openclaw-token: <token>`
- Token trong query-string sẽ bị từ chối (`?token=...` trả về `400`).
## Điểm cuối

### `POST /hooks/wake`

Tải trọng:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **required** (string): The description of the event (e.g., "New email received").
- `mode` optional (`now` | `next-heartbeat`): Whether to trigger an immediate heartbeat (default `now`) or wait for the next periodic check.

Effect:

- Enqueues a system event for the **main** session
- If `mode=now`, triggers an immediate heartbeat

### `POST /hooks/agent`

Payload:

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **required** (string): The prompt or message for the agent to process.
- `name` optional (string): Human-readable name for the hook (e.g., "GitHub"), used as a prefix in session summaries.
- `agentId` optional (string): Route this hook to a specific agent. Unknown IDs fall back to the default agent. When set, the hook runs using the resolved agent's workspace and configuration.
- `sessionKey` optional (string): The key used to identify the agent's session. By default this field is rejected unless `hooks.allowRequestSessionKey=true`.
- `wakeMode` optional (`now` | `next-heartbeat`): Whether to trigger an immediate heartbeat (default `now`) or wait for the next periodic check.
- `deliver` optional (boolean): If `true`, the agent's response will be sent to the messaging channel. Defaults to `true`. Responses that are only heartbeat acknowledgments are automatically skipped.
- `channel` optional (string): The messaging channel for delivery. One of: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Defaults to `last`.
- `to` optional (string): The recipient identifier for the channel (e.g., phone number for WhatsApp/Signal, chat ID for Telegram, channel ID for Discord/Slack/Mattermost (plugin), conversation ID for MS Teams). Defaults to the last recipient in the main session.
- `model` optional (string): Model override (e.g., `anthropic/claude-3-5-sonnet` or an alias). Must be in the allowed model list if restricted.
- `thinking` optional (string): Thinking level override (e.g., `low`, `medium`, `high`).
- `timeoutSeconds` optional (number): Maximum duration for the agent run in seconds.

Effect:

- Runs an **isolated** agent turn (own session key)
- Always posts a summary into the **main** session
- If `wakeMode=now`, kích hoạt heartbeat ngay lập tức
## Chính sách khóa phiên (thay đổi đột phá)

`/hooks/agent` payload `sessionKey` ghi đè bị vô hiệu hóa theo mặc định.

- Khuyến nghị: đặt `hooks.defaultSessionKey` cố định và tắt ghi đè yêu cầu.
- Tùy chọn: chỉ cho phép ghi đè yêu cầu khi cần thiết và hạn chế tiền tố.

Cấu hình khuyến nghị:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Compatibility config (legacy behavior):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // strongly recommended
  },
}
```

### `POST /hooks/<name>` (mapped)

Custom hook names are resolved via `hooks.mappings` (see configuration). A mapping can
turn arbitrary payloads into `wake` or `agent` actions, with optional templates or
code transforms.

Mapping options (summary):

- `hooks.presets: ["gmail"]` enables the built-in Gmail mapping.
- `hooks.mappings` lets you define `match`, `action`, and templates in config.
- `hooks.transformsDir` + `transform.module` loads a JS/TS module for custom logic.
  - `hooks.transformsDir` (if set) must stay within the transforms root under your OpenClaw config directory (typically `~/.openclaw/hooks/transforms`).
  - `transform.module` must resolve within the effective transforms directory (traversal/escape paths are rejected).
- Use `match.source` to keep a generic ingest endpoint (payload-driven routing).
- TS transforms require a TS loader (e.g. `bun` or `tsx`) or precompiled `.js` at runtime.
- Set `deliver: true` + `channel`/`to` on mappings to route replies to a chat surface
  (`channel` defaults to `last` and falls back to WhatsApp).
- `agentId` routes the hook to a specific agent; unknown IDs fall back to the default agent.
- `hooks.allowedAgentIds` restricts explicit `agentId` routing. Omit it (or include `*`) to allow any agent. Set `[]` to deny explicit `agentId` routing.
- `hooks.defaultSessionKey` sets the default session for hook agent runs when no explicit key is provided.
- `hooks.allowRequestSessionKey` controls whether `/hooks/agent` payloads may set `sessionKey` (default: `false`).
- `hooks.allowedSessionKeyPrefixes` optionally restricts explicit `sessionKey` values from request payloads and mappings.
- `allowUnsafeExternalContent: true` disables the external content safety wrapper for that hook
  (dangerous; only for trusted internal sources).
- `openclaw webhooks gmail setup` writes `hooks.gmail` config for `openclaw webhooks gmail run`.
  Xem [Gmail Pub/Sub](/automation/gmail-pubsub) để biết quy trình theo dõi Gmail đầy đủ.
## Phản hồi

- `200` cho `/hooks/wake`
- `202` cho `/hooks/agent` (bắt đầu chạy bất đồng bộ)
- `401` khi xác thực thất bại
- `429` sau nhiều lần xác thực thất bại từ cùng một client (kiểm tra `Retry-After`)
- `400` khi payload không hợp lệ
- `413` khi payload quá lớn
## Ví dụ

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Use a different model

Add `model` to the agent payload (or mapping) to override the model for that run:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

If you enforce `agents.defaults.models`, make sure the override model is included there.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```
## Bảo mật

- Giữ các endpoint hook phía sau local loopback, tailnet, hoặc reverse proxy đáng tin cậy.
- Sử dụng token hook chuyên dụng; không tái sử dụng token xác thực gateway.
- Các lần xác thực thất bại liên tiếp sẽ bị giới hạn tốc độ theo địa chỉ client để làm chậm các cuộc tấn công brute-force.
- Nếu bạn sử dụng định tuyến đa agent, hãy đặt `hooks.allowedAgentIds` để giới hạn việc lựa chọn `agentId` rõ ràng.
- Giữ `hooks.allowRequestSessionKey=false` trừ khi bạn yêu cầu phiên được chọn bởi người gọi.
- Nếu bạn bật `sessionKey` yêu cầu, hãy hạn chế `hooks.allowedSessionKeyPrefixes` (ví dụ, `["hook:"]`).
- Tránh bao gồm các payload thô nhạy cảm trong log webhook.
- Các payload hook được coi là không đáng tin cậy và được bao bọc với các ranh giới an toàn theo mặc định.
  Nếu bạn phải vô hiệu hóa điều này cho một hook cụ thể, hãy đặt `allowUnsafeExternalContent: true`
  trong mapping của hook đó (nguy hiểm).