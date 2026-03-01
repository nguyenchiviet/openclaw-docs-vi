---
summary: Gọi một công cụ duy nhất trực tiếp thông qua điểm cuối HTTP của Gateway
read_when:
  - Gọi các công cụ mà không chạy một lượt agent đầy đủ
  - Xây dựng các tự động hóa cần thực thi chính sách công cụ
title: Gọi API Công cụ
x-i18n:
  source_path: gateway\tools-invoke-http-api.md
  source_hash: ca7c696724c227de97279c74fedfd169d30e04dbc60732d2683b8f3b0d1ab691
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:36.248Z'
---

# Gọi Tools (HTTP)

Gateway của OpenClaw cung cấp một endpoint HTTP đơn giản để gọi một công cụ duy nhất trực tiếp. Nó luôn được bật, nhưng được kiểm soát bởi xác thực Gateway và chính sách công cụ.

- `POST /tools/invoke`
- Cùng cổng với Gateway (WS + HTTP multiplex): `http://<gateway-host>:<port>/tools/invoke`

Kích thước tải trọng tối đa mặc định là 2 MB.
## Xác thực

Sử dụng cấu hình xác thực của Gateway. Gửi một bearer token:

- `Authorization: Bearer <token>`

Ghi chú:

- Khi `gateway.auth.mode="token"`, sử dụng `gateway.auth.token` (hoặc `OPENCLAW_GATEWAY_TOKEN`).
- Khi `gateway.auth.mode="password"`, sử dụng `gateway.auth.password` (hoặc `OPENCLAW_GATEWAY_PASSWORD`).
- Nếu `gateway.auth.rateLimit` được cấu hình và quá nhiều lỗi xác thực xảy ra, endpoint trả về `429` với `Retry-After`.
## Nội dung yêu cầu

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
`tool`

Fields:

- `action` (string, required): tool name to invoke.
- `action` (string, optional): mapped into args if the tool schema supports `args` and the args payload omitted it.
- `sessionKey` (object, optional): tool-specific arguments.
- `"main"` (string, optional): target session key. If omitted or `session.mainKey`, the Gateway uses the configured main session key (honors `global` and default agent, or `dryRun` (boolean, tùy chọn): dành riêng cho việc sử dụng trong tương lai; hiện tại bị bỏ qua.
## Chính sách + hành vi định tuyến

Tính khả dụng của công cụ được lọc thông qua chuỗi chính sách giống như được sử dụng bởi các agent Gateway:

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- chính sách nhóm (nếu khóa phiên ánh xạ tới một nhóm hoặc kênh)
- chính sách subagent (khi gọi với khóa phiên subagent)

Nếu một công cụ không được phép bởi chính sách, điểm cuối trả về **404**.

Gateway HTTP cũng áp dụng danh sách từ chối cứng theo mặc định (ngay cả khi chính sách phiên cho phép công cụ):

- `sessions_spawn`
- `sessions_send`
- `gateway`
- `whatsapp_login`

Bạn có thể tùy chỉnh danh sách từ chối này thông qua `gateway.tools`:

```json5
{
  gateway: {
    tools: {
      // Additional tools to block over HTTP /tools/invoke
      deny: ["browser"],
      // Remove tools from the default deny list
      allow: ["gateway"],
    },
  },
}
```

To help group policies resolve context, you can optionally set:

- `x-openclaw-message-channel: <channel>` (example: `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (khi tồn tại nhiều tài khoản)
## Phản hồi

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (yêu cầu không hợp lệ hoặc lỗi đầu vào công cụ)
- `401` → không được phép truy cập
- `429` → giới hạn tốc độ xác thực (`Retry-After` được đặt)
- `404` → công cụ không khả dụng (không tìm thấy hoặc không được phép)
- `405` → phương thức không được phép
- `500` → `{ ok: false, error: { type, message } }` (lỗi thực thi công cụ không mong đợi; tin nhắn được làm sạch)
## Ví dụ

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```