---
summary: Các schema TypeBox làm nguồn sự thật duy nhất cho giao thức gateway
read_when:
  - Cập nhật các schema giao thức hoặc codegen
title: TypeBox
x-i18n:
  source_path: concepts\typebox.md
  source_hash: 72fb8a1244edd84bbf50359722c73c00aef79b744c7b17e2a68122cebb055dc0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:03.129Z'
---

# TypeBox làm nguồn sự thật của giao thức

Cập nhật lần cuối: 2026-01-10

TypeBox là một thư viện schema hướng TypeScript. Chúng tôi sử dụng nó để định nghĩa **giao thức Gateway WebSocket** (bắt tay, yêu cầu/phản hồi, sự kiện máy chủ). Những schema đó thúc đẩy **xác thực thời gian chạy**, **xuất JSON Schema**, và **codegen Swift** cho ứng dụng macOS. Một nguồn sự thật duy nhất; mọi thứ khác được tạo ra.

Nếu bạn muốn ngữ cảnh giao thức cấp cao hơn, hãy bắt đầu với [Kiến trúc Gateway](/concepts/architecture).
## Mô hình tư duy (30 giây)

Mỗi thông báo Gateway WS là một trong ba khung:

- **Request**: `{ type: "req", id, method, params }`
- **Response**: `{ type: "res", id, ok, payload | error }`
- **Event**: `{ type: "event", event, payload, seq?, stateVersion? }`

Khung đầu tiên **phải** là một yêu cầu `connect`. Sau đó, các client có thể gọi
các phương thức (ví dụ: `health`, `send`, `chat.send`) và đăng ký các sự kiện (ví dụ:
`presence`, `tick`, `agent`).

Luồng kết nối (tối thiểu):

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Common methods + events:

| Category  | Examples                                                  | Notes                              |
| --------- | --------------------------------------------------------- | ---------------------------------- |
| Core      | `connect`, `health`, `status`                             | `connect` must be first            |
| Messaging | `send`, `poll`, `agent`, `agent.wait`                     | side-effects need `idempotencyKey` |
| Chat      | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat uses these                 |
| Sessions  | `sessions.list`, `sessions.patch`, `sessions.delete`      | session admin                      |
| Nodes     | `node.list`, `node.invoke`, `node.pair.*`                 | Gateway WS + node actions          |
| Events    | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | server push                        |

Authoritative list lives in `src/gateway/server.ts` (`METHODS`, `EVENTS`).
## Vị trí lưu trữ các schema

- Nguồn: `src/gateway/protocol/schema.ts`
- Trình xác thực runtime (AJV): `src/gateway/protocol/index.ts`
- Bắt tay máy chủ + phân phối phương thức: `src/gateway/server.ts`
- Node client: `src/gateway/client.ts`
- JSON Schema được tạo: `dist/protocol.schema.json`
- Mô hình Swift được tạo: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`
## Đường ống hiện tại

- `pnpm protocol:gen`
  - ghi JSON Schema (draft‑07) vào `dist/protocol.schema.json`
- `pnpm protocol:gen:swift`
  - tạo các mô hình gateway Swift
- `pnpm protocol:check`
  - chạy cả hai trình tạo và xác minh rằng đầu ra được cam kết
## Cách các schema được sử dụng tại thời gian chạy

- **Phía máy chủ**: mỗi frame đến được xác thực bằng AJV. Bắt tay chỉ
  chấp nhận một yêu cầu `connect` có các tham số khớp với `ConnectParams`.
- **Phía máy khách**: máy khách JS xác thực các frame sự kiện và phản hồi trước
  khi sử dụng chúng.
- **Bề mặt phương thức**: Gateway quảng cáo các `methods` và
  `events` được hỗ trợ trong `hello-ok`.
## Các khung hình ví dụ

Kết nối (tin nhắn đầu tiên):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Hello-ok response:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Request + response:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Event:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```
## Client tối thiểu (Node.js)

Luồng hữu ích nhỏ nhất: kết nối + kiểm tra sức khỏe.

```ts
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```
## Ví dụ thực tế: thêm một phương thức từ đầu đến cuối

Ví dụ: thêm một yêu cầu `system.echo` mới trả về `{ ok: true, text }`.

1. **Schema (nguồn sự thật)**

Thêm vào `src/gateway/protocol/schema.ts`:

```ts
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Add both to `ProtocolSchemas` and export types:

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **Validation**

In `src/gateway/protocol/index.ts`, export an AJV validator:

```ts
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **Server behavior**

Add a handler in `src/gateway/server-methods/system.ts`:

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Register it in `src/gateway/server-methods.ts` (already merges `systemHandlers`),
then add `"system.echo"` to `METHODS` in `src/gateway/server.ts`.

4. **Regenerate**

```bash
pnpm protocol:check
```
5. **Kiểm tra + tài liệu**

Thêm một bài kiểm tra máy chủ trong `src/gateway/server.*.test.ts` và ghi chú phương thức trong tài liệu.
## Hành vi codegen Swift

Trình tạo Swift phát ra:

- `GatewayFrame` enum với các trường hợp `req`, `res`, `event`, và `unknown`
- Các struct/enum payload được gõ mạnh
- Các giá trị `ErrorCode` và `GATEWAY_PROTOCOL_VERSION`

Các loại khung không xác định được bảo toàn dưới dạng payload thô để tương thích với các phiên bản trong tương lai.
## Phiên bản + tương thích

- `PROTOCOL_VERSION` nằm trong `src/gateway/protocol/schema.ts`.
- Các client gửi `minProtocol` + `maxProtocol`; máy chủ từ chối các không khớp.
- Các mô hình Swift giữ lại các loại khung không xác định để tránh làm hỏng các client cũ hơn.
## Các mẫu và quy ước lược đồ

- Hầu hết các đối tượng sử dụng `additionalProperties: false` cho các payload nghiêm ngặt.
- `NonEmptyString` là mặc định cho các ID và tên phương thức/sự kiện.
- `GatewayFrame` cấp cao nhất sử dụng một **discriminator** trên `type`.
- Các phương thức có tác dụng phụ thường yêu cầu một `idempotencyKey` trong params
  (ví dụ: `send`, `poll`, `agent`, `chat.send`).
## JSON Schema trực tiếp

JSON Schema được tạo ra có sẵn trong repo tại `dist/protocol.schema.json`. Tệp thô được xuất bản thường có sẵn tại:

- [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)
## Khi bạn thay đổi schemas

1. Cập nhật các schemas TypeBox.
2. Chạy `pnpm protocol:check`.
3. Commit các schema được tạo lại + Swift models.