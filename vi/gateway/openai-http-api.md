---
summary: >-
  Expose một HTTP endpoint /v1/chat/completions tương thích với OpenAI từ
  Gateway
read_when:
  - Tích hợp các công cụ mong đợi OpenAI Chat Completions
title: OpenAI Chat Completions
x-i18n:
  source_path: gateway\openai-http-api.md
  source_hash: b1d0a69d2ff683dcfb3b347ec9df458d1afbaa3edebad4c56444822827faa579
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:58:22.477Z'
---

# OpenAI Chat Completions (HTTP)

Gateway của OpenClaw có thể phục vụ một điểm cuối Chat Completions tương thích với OpenAI nhỏ.

Điểm cuối này **bị vô hiệu hóa theo mặc định**. Bật nó trong cấu hình trước.

- `POST /v1/chat/completions`
- Cùng cổng với Gateway (WS + HTTP multiplex): `http://<gateway-host>:<port>/v1/chat/completions`

Dưới nắp, các yêu cầu được thực thi như một lần chạy agent Gateway bình thường (cùng đường mã như `openclaw agent`), vì vậy định tuyến/quyền/cấu hình phù hợp với Gateway của bạn.
## Xác thực

Sử dụng cấu hình xác thực của Gateway. Gửi một bearer token:

- `Authorization: Bearer <token>`

Ghi chú:

- Khi `gateway.auth.mode="token"`, sử dụng `gateway.auth.token` (hoặc `OPENCLAW_GATEWAY_TOKEN`).
- Khi `gateway.auth.mode="password"`, sử dụng `gateway.auth.password` (hoặc `OPENCLAW_GATEWAY_PASSWORD`).
- Nếu `gateway.auth.rateLimit` được cấu hình và quá nhiều lỗi xác thực xảy ra, endpoint trả về `429` với `Retry-After`.
## Chọn một agent

Không cần tiêu đề tùy chỉnh: mã hóa id agent trong trường OpenAI `model`:

- `model: "openclaw:<agentId>"` (ví dụ: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (bí danh)

Hoặc nhắm mục tiêu một agent OpenClaw cụ thể theo tiêu đề:

- `x-openclaw-agent-id: <agentId>` (mặc định: `main`)

Nâng cao:

- `x-openclaw-session-key: <sessionKey>` để kiểm soát hoàn toàn định tuyến phiên.
## Bật điểm cuối

Đặt `gateway.http.endpoints.chatCompletions.enabled` thành `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```
## Vô hiệu hóa điểm cuối

Đặt `gateway.http.endpoints.chatCompletions.enabled` thành `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```
## Hành vi phiên

Theo mặc định, endpoint là **không trạng thái cho mỗi yêu cầu** (một khóa phiên mới được tạo cho mỗi lệnh gọi).

Nếu yêu cầu bao gồm một chuỗi OpenAI `user`, Gateway sẽ tạo ra một khóa phiên ổn định từ nó, để các lệnh gọi lặp lại có thể chia sẻ một phiên agent.
## Truyền phát (SSE)

Đặt `stream: true` để nhận Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Mỗi dòng sự kiện là `data: <json>`
- Luồng kết thúc bằng `data: [DONE]`
## Ví dụ

Không truyền phát:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Streaming:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```