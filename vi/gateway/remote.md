---
summary: Truy cập từ xa bằng SSH tunnels (Gateway WS) và tailnets
read_when:
  - Chạy hoặc khắc phục sự cố thiết lập gateway từ xa
title: Truy cập từ xa
x-i18n:
  source_path: gateway\remote.md
  source_hash: afe5862cc4181115dab3c3b69aac71ba0b88e53501def68b0addcb17c29c7aab
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:13.588Z'
---

# Truy cập từ xa (SSH, tunnel và tailnet)

Repo này hỗ trợ "remote over SSH" bằng cách giữ một Gateway duy nhất (master) chạy trên một máy chủ chuyên dụng (desktop/server) và kết nối các client đến nó.

- Đối với **operators (bạn / ứng dụng macOS)**: SSH tunneling là giải pháp dự phòng phổ quát.
- Đối với **nodes (iOS/Android và các thiết bị trong tương lai)**: kết nối đến Gateway **WebSocket** (LAN/tailnet hoặc SSH tunnel khi cần).
## Ý tưởng cốt lõi

- Gateway WebSocket liên kết với **loopback** trên cổng được cấu hình của bạn (mặc định là 18789).
- Để sử dụng từ xa, bạn chuyển tiếp cổng loopback đó qua SSH (hoặc sử dụng tailnet/VPN và tunnel ít hơn).
## Các thiết lập VPN/tailnet phổ biến (nơi agent sống)

Hãy coi **Gateway host** là "nơi agent sống." Nó sở hữu các phiên, hồ sơ xác thực, kênh và trạng thái.
Laptop/desktop của bạn (và các node) kết nối với host đó.

### 1) Gateway luôn bật trong tailnet của bạn (VPS hoặc máy chủ nhà)

Chạy Gateway trên một host liên tục và truy cập nó qua **Tailscale** hoặc SSH.

- **UX tốt nhất:** giữ `gateway.bind: "loopback"` và sử dụng **Tailscale Serve** cho Control UI.
- **Dự phòng:** giữ local loopback + SSH tunnel từ bất kỳ máy nào cần truy cập.
- **Ví dụ:** [exe.dev](/install/exe-dev) (VM dễ dàng) hoặc [Hetzner](/install/hetzner) (VPS sản xuất).

Điều này lý tưởng khi laptop của bạn thường xuyên ngủ nhưng bạn muốn agent luôn bật.

### 2) Desktop nhà chạy Gateway, laptop là điều khiển từ xa

Laptop **không** chạy agent. Nó kết nối từ xa:

- Sử dụng chế độ **Remote over SSH** của ứng dụng macOS (Settings → General → "OpenClaw runs").
- Ứng dụng mở và quản lý tunnel, vì vậy WebChat + kiểm tra sức khỏe "hoạt động bình thường."

Runbook: [truy cập từ xa macOS](/platforms/mac/remote).

### 3) Laptop chạy Gateway, truy cập từ xa từ các máy khác

Giữ Gateway ở local nhưng tiếp xúc nó một cách an toàn:

- SSH tunnel đến laptop từ các máy khác, hoặc
- Tailscale Serve Control UI và giữ Gateway chỉ local loopback.

Hướng dẫn: [Tailscale](/gateway/tailscale) và [Tổng quan Web](/web).
## Luồng lệnh (những gì chạy ở đâu)

Một dịch vụ Gateway sở hữu trạng thái + kênh. Các node là các thiết bị ngoại vi.

Ví dụ luồng (Telegram → node):

- Tin nhắn Telegram đến **Gateway**.
- Gateway chạy **agent** và quyết định có gọi công cụ node hay không.
- Gateway gọi **node** qua Gateway WebSocket (`node.*` RPC).
- Node trả về kết quả; Gateway trả lời lại cho Telegram.

Ghi chú:

- **Các node không chạy dịch vụ gateway.** Chỉ một gateway nên chạy trên mỗi máy chủ trừ khi bạn cố ý chạy các hồ sơ cô lập (xem [Multiple gateways](/gateway/multiple-gateways)).
- Ứng dụng macOS "node mode" chỉ là một node client qua Gateway WebSocket.
## SSH tunnel (CLI + tools)

Tạo một tunnel cục bộ đến Gateway WS từ xa:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

With the tunnel up:

- `openclaw health` and `openclaw status --deep` now reach the remote gateway via `ws://127.0.0.1:18789`.
- `openclaw gateway {status,health,send,agent,call}` can also target the forwarded URL via `--url` when needed.

Note: replace `18789` with your configured `gateway.port` (or `--port`/`OPENCLAW_GATEWAY_PORT`).
Note: when you pass `--url`, the CLI does not fall back to config or environment credentials.
Include `--token` or `--password` một cách rõ ràng. Thiếu thông tin xác thực rõ ràng là một lỗi.
## Mặc định từ xa của CLI

Bạn có thể lưu trữ một mục tiêu từ xa để các lệnh CLI sử dụng nó theo mặc định:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

When the gateway is loopback-only, keep the URL at `ws://127.0.0.1:18789` và mở đường hầm SSH trước.
## Ưu tiên thông tin xác thực

Phân giải thông tin xác thực cho lệnh gọi/kiểm tra Gateway hiện tuân theo một hợp đồng chung:

- Thông tin xác thực rõ ràng (`--token`, `--password`, hoặc công cụ `gatewayToken`) luôn có ưu tiên.
- Mặc định chế độ local:
  - token: `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` -> `gateway.remote.token`
  - password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.auth.password` -> `gateway.remote.password`
- Mặc định chế độ remote:
  - token: `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token`
  - password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.remote.password` -> `gateway.auth.password`
- Kiểm tra token probe/status remote mặc định là nghiêm ngặt: chúng chỉ sử dụng `gateway.remote.token` (không có fallback token local) khi nhắm mục tiêu chế độ remote.
- Biến môi trường `CLAWDBOT_GATEWAY_*` cũ chỉ được sử dụng bởi các đường dẫn lệnh gọi tương thích; phân giải probe/status/auth chỉ sử dụng `OPENCLAW_GATEWAY_*`.
## Giao diện Chat qua SSH

WebChat không còn sử dụng cổng HTTP riêng biệt. Giao diện chat SwiftUI kết nối trực tiếp với Gateway WebSocket.

- Chuyển tiếp `18789` qua SSH (xem ở trên), sau đó kết nối các client với `ws://127.0.0.1:18789`.
- Trên macOS, ưu tiên chế độ "Remote over SSH" của ứng dụng, chế độ này quản lý tunnel tự động.
## Ứng dụng macOS "Remote over SSH"

Ứng dụng thanh menu macOS có thể điều khiển cùng một thiết lập từ đầu đến cuối (kiểm tra trạng thái từ xa, WebChat và chuyển tiếp Voice Wake).

Runbook: [macOS remote access](/platforms/mac/remote).
## Quy tắc bảo mật (remote/VPN)

Phiên bản ngắn: **giữ Gateway chỉ loopback** trừ khi bạn chắc chắn cần một bind.

- **Loopback + SSH/Tailscale Serve** là mặc định an toàn nhất (không tiếp xúc công khai).
- **Non-loopback binds** (`lan`/`tailnet`/`custom`, hoặc `auto` khi loopback không khả dụng) phải sử dụng auth tokens/passwords.
- `gateway.remote.token` / `.password` là các nguồn thông tin xác thực của client. Chúng **không** cấu hình xác thực máy chủ bằng chính nó.
- Các đường dẫn gọi cục bộ có thể sử dụng `gateway.remote.*` làm fallback khi `gateway.auth.*` không được đặt.
- `gateway.remote.tlsFingerprint` ghim chứng chỉ TLS từ xa khi sử dụng `wss://`.
- **Tailscale Serve** có thể xác thực lưu lượng Control UI/WebSocket thông qua các tiêu đề nhận dạng khi `gateway.auth.allowTailscale: true`; các điểm cuối HTTP API vẫn yêu cầu xác thực token/password. Luồng không token này giả định rằng máy chủ gateway được tin tưởng. Đặt nó thành `false` nếu bạn muốn tokens/passwords ở mọi nơi.
- Coi quyền kiểm soát trình duyệt như quyền truy cập của nhà điều hành: chỉ tailnet + ghép nối node có chủ ý.

Tìm hiểu sâu: [Security](/gateway/security).