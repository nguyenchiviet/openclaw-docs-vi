---
summary: 'Giao thức WebSocket của Gateway: bắt tay, khung, quản lý phiên bản'
read_when:
  - Triển khai hoặc cập nhật các ứng dụng khách WS của gateway
  - Gỡ lỗi không khớp giao thức hoặc lỗi kết nối
  - Tái tạo các lược đồ/mô hình giao thức
title: Giao thức Gateway
x-i18n:
  source_path: gateway\protocol.md
  source_hash: c97074d5b52e561afe2a5c70f7ae7817d943557987b328b26ae0e203503099b9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:08:59.144Z'
---

# Giao thức Gateway (WebSocket)

Giao thức Gateway WS là **mặt phẳng điều khiển duy nhất + giao thức truyền tải node** cho OpenClaw. Tất cả các client (CLI, giao diện web, ứng dụng macOS, node iOS/Android, node không đầu) kết nối qua WebSocket và khai báo **vai trò** + **phạm vi** của chúng tại thời điểm bắt tay.

## Giao thức truyền tải

- WebSocket, các khung văn bản với tải trọng JSON.
- Khung đầu tiên **phải** là một yêu cầu `connect`.

## Bắt tay (kết nối)

Gateway → Client (thử thách trước kết nối):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Client → Gateway:

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```
Gateway → Client: ``__OC_I19N_0000__`__OC_I19N_0001__hello-ok__OC_I19N_0002__`__OC_I19N_0003__`__OC_I19N_0004__`__OC_I19N_0005__``
## Định khung

- **Yêu cầu**: `{type:"req", id, method, params}`
- **Phản hồi**: `{type:"res", id, ok, payload|error}`
- **Sự kiện**: `{type:"event", event, payload, seq?, stateVersion?}`

Các phương thức gây tác dụng phụ yêu cầu **khóa bất biến** (xem lược đồ).

## Vai trò + phạm vi

### Vai trò

- `operator` = máy khách mặt phẳng điều khiển (CLI/UI/tự động hóa).
- `node` = máy chủ khả năng (camera/màn hình/canvas/system.run).

### Phạm vi (người vận hành)

Các phạm vi phổ biến:

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

### Khả năng/lệnh/quyền (node)

Các node khai báo yêu cầu khả năng khi kết nối:

- `caps`: các danh mục khả năng cấp cao.
- `commands`: danh sách cho phép lệnh để gọi.
- `permissions`: các công tắc chi tiết (ví dụ: `screen.record`, `camera.capture`).

Gateway coi đây là **yêu cầu** và thực thi danh sách cho phép phía máy chủ.
## Trạng thái hiện diện

- `system-presence` trả về các mục được khóa bằng định danh thiết bị.
- Các mục trạng thái hiện diện bao gồm `deviceId`, `roles`, và `scopes` để giao diện người dùng có thể hiển thị một hàng duy nhất cho mỗi thiết bị ngay cả khi nó kết nối với tư cách là cả **operator** và **node**.

### Phương thức hỗ trợ node

- Các node có thể gọi `skills.bins` để lấy danh sách hiện tại các tệp thực thi Skills cho các kiểm tra tự động cho phép.

### Phương thức hỗ trợ operator

- Các operator có thể gọi `tools.catalog` (`operator.read`) để lấy danh mục công cụ thời gian chạy cho một agent. Phản hồi bao gồm các công cụ được nhóm và siêu dữ liệu nguồn gốc:
  - `source`: `core` hoặc `plugin`
  - `pluginId`: chủ sở hữu plugin khi `source="plugin"`
  - `optional`: liệu một công cụ plugin có tùy chọn hay không
## Phê duyệt thực thi

- Khi một yêu cầu thực thi cần phê duyệt, Gateway sẽ phát sóng `exec.approval.requested`.
- Các client của nhà điều hành giải quyết bằng cách gọi `exec.approval.resolve` (yêu cầu phạm vi `operator.approvals`).
## Lập phiên bản

- `PROTOCOL_VERSION` nằm trong `src/gateway/protocol/schema.ts`.
- Máy khách gửi `minProtocol` + `maxProtocol`; máy chủ từ chối các trường hợp không khớp.
- Các lược đồ + mô hình được tạo từ định nghĩa TypeBox:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

## Xác thực

- Nếu `OPENCLAW_GATEWAY_TOKEN` (hoặc `--token`) được đặt, `connect.params.auth.token` phải khớp hoặc socket sẽ bị đóng.
- Sau khi ghép nối, Gateway cấp một **mã thông báo thiết bị** được giới hạn theo vai trò kết nối + phạm vi. Nó được trả về trong `hello-ok.auth.deviceToken` và nên được client lưu trữ để kết nối trong tương lai.
- Mã thông báo thiết bị có thể được xoay vòng/thu hồi thông qua `device.token.rotate` và `device.token.revoke` (yêu cầu phạm vi `operator.pairing`).
## Định danh thiết bị + ghép nối

- Các node nên bao gồm một định danh thiết bị ổn định (__OC_I118N_0000__) được tạo ra từ dấu vân tay của cặp khóa.
- Gateway cấp token cho từng thiết bị + vai trò.
- Cần có sự chấp thuận ghép nối cho các ID thiết bị mới trừ khi tính năng tự động chấp thuận cục bộ được bật.
- Các kết nối **cục bộ** bao gồm local loopback và địa chỉ tailnet của chính máy chủ gateway (để các liên kết tailnet cùng máy chủ vẫn có thể tự động chấp thuận).
- Tất cả các client WS phải bao gồm định danh `device` trong quá trình `connect` (người vận hành + node). Giao diện người dùng điều khiển có thể bỏ qua nó **chỉ** khi `gateway.controlUi.dangerouslyDisableDeviceAuth` được bật để sử dụng trong trường hợp khẩn cấp.
- Tất cả các kết nối phải ký vào nonce `connect.challenge` do máy chủ cung cấp.

### Chẩn đoán di chuyển xác thực thiết bị

Đối với các client cũ vẫn sử dụng hành vi ký trước thử thách, `connect` hiện trả về các mã chi tiết `DEVICE_AUTH_*` dưới `error.details.code` với một `error.details.reason` ổn định.

Các lỗi di chuyển phổ biến:

| Thông báo                     | details.code                     | details.reason           | Ý nghĩa                                            |
| --------------------------- | -------------------------------- | ------------------------ | -------------------------------------------------- |
| `device nonce required`     | `DEVICE_AUTH_NONCE_REQUIRED`     | `device-nonce-missing`   | Client đã bỏ qua `device.nonce` (hoặc gửi trống).     |
| `device nonce mismatch`     | `DEVICE_AUTH_NONCE_MISMATCH`     | `device-nonce-mismatch`  | Client đã ký bằng một nonce cũ/sai.            |
| `device signature invalid`  | `DEVICE_AUTH_SIGNATURE_INVALID`  | `device-signature`       | Tải trọng chữ ký không khớp với tải trọng v2.       |
| `device signature expired`  | `DEVICE_AUTH_SIGNATURE_EXPIRED`  | `device-signature-stale` | Dấu thời gian đã ký nằm ngoài độ lệch cho phép.          |
| `device identity mismatch`  | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch`     | `device.id` không khớp với dấu vân tay khóa công khai. |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key`      | Định dạng/chuẩn hóa khóa công khai không thành công.         |

Mục tiêu di chuyển:

- Luôn chờ `connect.challenge`.
- Ký tải trọng v2 bao gồm nonce của máy chủ.
- Gửi cùng một nonce trong `connect.params.device.nonce`.
- Tải trọng chữ ký ưu tiên là `v3`, liên kết `platform` và `deviceFamily`
  ngoài các trường device/client/role/scopes/token/nonce.
- Chữ ký `v2` cũ vẫn được chấp nhận để tương thích, nhưng việc ghim siêu dữ liệu thiết bị đã ghép nối vẫn kiểm soát chính sách lệnh khi kết nối lại.
## TLS + ghim chứng chỉ

- TLS được hỗ trợ cho các kết nối WS.
- Các máy khách có thể tùy chọn ghim dấu vân tay chứng chỉ gateway (xem cấu hình `gateway.tls` cộng với `gateway.remote.tlsFingerprint` hoặc CLI `--tls-fingerprint`).
## Phạm vi

Giao thức này cung cấp **API Gateway đầy đủ** (trạng thái, kênh, mô hình, trò chuyện, agent, phiên, node, phê duyệt, v.v.). Bề mặt chính xác được định nghĩa bởi các lược đồ TypeBox trong `src/gateway/protocol/schema.ts`.
