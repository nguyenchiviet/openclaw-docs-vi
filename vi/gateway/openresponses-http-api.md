---
summary: >-
  Expose một /v1/responses HTTP endpoint tương thích với OpenResponses từ
  Gateway
read_when:
  - Tích hợp các client sử dụng OpenResponses API
  - >-
    Bạn muốn các đầu vào dựa trên mục, lệnh gọi công cụ của khách hàng, hoặc sự
    kiện SSE
title: API OpenResponses
x-i18n:
  source_path: gateway\openresponses-http-api.md
  source_hash: 5a66ae563c5cde78e9d4629d3e792bb8bd6fb8cd33b4e2041e4acb5e97238d44
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:58:43.056Z'
---

# OpenResponses API (HTTP)

Gateway của OpenClaw có thể phục vụ một endpoint tương thích với OpenResponses `POST /v1/responses`.

Endpoint này **bị vô hiệu hóa theo mặc định**. Hãy bật nó trong cấu hình trước.

- `POST /v1/responses`
- Cùng cổng với Gateway (WS + HTTP multiplex): `http://<gateway-host>:<port>/v1/responses`

Dưới nắp, các yêu cầu được thực thi như một lần chạy agent Gateway bình thường (cùng đường mã với
`openclaw agent`), vì vậy định tuyến/quyền/cấu hình phù hợp với Gateway của bạn.
## Xác thực

Sử dụng cấu hình xác thực của Gateway. Gửi một bearer token:

- `Authorization: Bearer <token>`

Ghi chú:

- Khi `gateway.auth.mode="token"`, sử dụng `gateway.auth.token` (hoặc `OPENCLAW_GATEWAY_TOKEN`).
- Khi `gateway.auth.mode="password"`, sử dụng `gateway.auth.password` (hoặc `OPENCLAW_GATEWAY_PASSWORD`).
- Nếu `gateway.auth.rateLimit` được cấu hình và quá nhiều lỗi xác thực xảy ra, endpoint trả về `429` với `Retry-After`.
## Chọn một agent

Không cần tiêu đề tùy chỉnh: mã hóa id agent trong trường OpenResponses `model`:

- `model: "openclaw:<agentId>"` (ví dụ: `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (bí danh)

Hoặc nhắm mục tiêu một agent OpenClaw cụ thể theo tiêu đề:

- `x-openclaw-agent-id: <agentId>` (mặc định: `main`)

Nâng cao:

- `x-openclaw-session-key: <sessionKey>` để kiểm soát hoàn toàn định tuyến phiên.
## Bật điểm cuối

Đặt `gateway.http.endpoints.responses.enabled` thành `true`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```
## Vô hiệu hóa điểm cuối

Đặt `gateway.http.endpoints.responses.enabled` thành `false`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```
## Hành vi phiên

Theo mặc định, endpoint là **không trạng thái cho mỗi yêu cầu** (một khóa phiên mới được tạo cho mỗi lần gọi).

Nếu yêu cầu bao gồm chuỗi OpenResponses `user`, Gateway sẽ tạo ra một khóa phiên ổn định
từ nó, để các lần gọi lặp lại có thể chia sẻ một phiên agent.
## Hình dạng yêu cầu (được hỗ trợ)

Yêu cầu tuân theo OpenResponses API với đầu vào dựa trên mục. Hỗ trợ hiện tại:

- `input`: chuỗi hoặc mảng các đối tượng mục.
- `instructions`: được hợp nhất vào lời nhắc hệ thống.
- `tools`: định nghĩa công cụ máy khách (công cụ hàm).
- `tool_choice`: lọc hoặc yêu cầu công cụ máy khách.
- `stream`: bật truyền phát SSE.
- `max_output_tokens`: giới hạn đầu ra nỗ lực tốt nhất (phụ thuộc vào nhà cung cấp).
- `user`: định tuyến phiên ổn định.

Được chấp nhận nhưng **hiện đang bị bỏ qua**:

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`
## Items (input)

### `message`

Roles: `system`, `developer`, `user`, `assistant`.

- `system` và `developer` được thêm vào lời nhắc hệ thống.
- `user` hoặc `function_call_output` item gần đây nhất trở thành "tin nhắn hiện tại."
- Các tin nhắn người dùng/trợ lý trước đó được đưa vào làm lịch sử để tham khảo.

### `function_call_output` (turn-based tools)

Gửi kết quả công cụ trở lại mô hình:

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### `reasoning` and `item_reference`

Được chấp nhận để tương thích schema nhưng bị bỏ qua khi xây dựng lời nhắc.
## Công cụ (công cụ hàm phía máy khách)

Cung cấp công cụ với `tools: [{ type: "function", function: { name, description?, parameters? } }]`.

Nếu agent quyết định gọi một công cụ, phản hồi sẽ trả về một mục đầu ra `function_call`.
Sau đó, bạn gửi một yêu cầu tiếp theo với `function_call_output` để tiếp tục lượt.
## Hình ảnh (`input_image`)

Hỗ trợ các nguồn base64 hoặc URL:

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Allowed MIME types (current): `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Kích thước tối đa (hiện tại): 10MB.
## Tệp (`input_file`)

Hỗ trợ các nguồn base64 hoặc URL:

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Allowed MIME types (current): `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Max size (current): 5MB.

Current behavior:

- File content is decoded and added to the **system prompt**, not the user message,
  so it stays ephemeral (not persisted in session history).
- PDFs are parsed for text. If little text is found, the first pages are rasterized
  into images and passed to the model.

PDF parsing uses the Node-friendly `pdfjs-dist` legacy build (no worker). The modern
PDF.js build expects browser workers/DOM globals, so it is not used in the Gateway.

URL fetch defaults:

- `files.allowUrl`: `true`
- `images.allowUrl`: `true`
- `maxUrlParts`: `8` (total URL-based `input_file` + `input_image` parts per request)
- Requests are guarded (DNS resolution, private IP blocking, redirect caps, timeouts).
- Optional hostname allowlists are supported per input type (`files.urlAllowlist`, `images.urlAllowlist`).
  - Exact host: `"cdn.example.com"`
  - Wildcard subdomains: `"*.assets.example.com"` (không khớp với apex)
## Giới hạn tệp + hình ảnh (cấu hình)

Các giá trị mặc định có thể được điều chỉnh dưới `gateway.http.endpoints.responses`:

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Defaults when omitted:

- `maxBodyBytes`: 20MB
- `maxUrlParts`: 8
- `files.maxBytes`: 5MB
- `files.maxChars`: 200k
- `files.maxRedirects`: 3
- `files.timeoutMs`: 10s
- `files.pdf.maxPages`: 4
- `files.pdf.maxPixels`: 4,000,000
- `files.pdf.minTextChars`: 200
- `images.maxBytes`: 10MB
- `images.maxRedirects`: 3
- `images.timeoutMs`: 10s

Lưu ý bảo mật:

- Danh sách cho phép URL được thực thi trước khi tìm nạp và trên các bước chuyển hướng.
- Thêm tên máy chủ vào danh sách cho phép không bỏ qua chặn IP riêng/nội bộ.
- Đối với các gateway được phơi bày trên internet, hãy áp dụng các điều khiển kiểm soát lưu lượng mạng ngoài cùng với các biện pháp bảo vệ cấp ứng dụng.
  Xem [Bảo mật](/gateway/security).
## Truyền phát (SSE)

Đặt `stream: true` để nhận Server-Sent Events (SSE):

- `Content-Type: text/event-stream`
- Mỗi dòng sự kiện là `event: <type>` và `data: <json>`
- Luồng kết thúc bằng `data: [DONE]`

Các loại sự kiện hiện được phát ra:

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (khi có lỗi)
## Cách sử dụng

`usage` được điền khi nhà cung cấp cơ bản báo cáo số lượng token.
## Lỗi

Lỗi sử dụng một đối tượng JSON như:

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Common cases:

- `401` missing/invalid auth
- `400` invalid request body
- `405` phương thức sai
## Ví dụ

Không truyền phát:

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Streaming:

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```