---
summary: 'Các bề mặt web Gateway: Giao diện điều khiển, chế độ liên kết và bảo mật'
read_when:
  - Bạn muốn truy cập Gateway qua Tailscale
  - Bạn muốn giao diện điều khiển trình duyệt và chỉnh sửa cấu hình
title: Web
x-i18n:
  source_path: web\index.md
  source_hash: 3e268b325dcd2b0bf28b11eda2fae811a14a138b2935852774a26bf6565e3411
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:58.452Z'
---

# Web (Gateway)

Gateway phục vụ một **Control UI nhỏ** (Vite + Lit) từ cùng cổng với Gateway WebSocket:

- mặc định: `http://<host>:18789/`
- tiền tố tùy chọn: đặt `gateway.controlUi.basePath` (ví dụ `/openclaw`)

Các khả năng nằm trong [Control UI](/web/control-ui).
Trang này tập trung vào các chế độ liên kết, bảo mật và các bề mặt hướng ra web.
## Webhooks

Khi `hooks.enabled=true`, Gateway cũng hiển thị một điểm cuối webhook nhỏ trên cùng máy chủ HTTP.
Xem [Cấu hình Gateway](/gateway/configuration) → `hooks` để biết thêm về xác thực + tải trọng.
## Cấu hình (bật theo mặc định)

Control UI được **bật theo mặc định** khi có tài sản (`dist/control-ui`).
Bạn có thể kiểm soát nó thông qua cấu hình:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath optional
  },
}
```
## Truy cập Tailscale

### Integrated Serve (được khuyến nghị)

Giữ Gateway trên local loopback và để Tailscale Serve proxy nó:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Then start the gateway:

```bash
openclaw gateway
```

Open:

- `https://<magicdns>/` (or your configured `gateway.controlUi.basePath`)

### Tailnet bind + token

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

Then start the gateway (token required for non-loopback binds):

```bash
openclaw gateway
```

Open:

- `http://<tailscale-ip>:18789/` (or your configured `gateway.controlUi.basePath`)

### Public internet (Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // or OPENCLAW_GATEWAY_PASSWORD
  },
}
```
## Ghi chú bảo mật

- Xác thực Gateway được yêu cầu theo mặc định (token/mật khẩu hoặc tiêu đề nhận dạng Tailscale).
- Các liên kết không phải local loopback vẫn **yêu cầu** token/mật khẩu được chia sẻ (`gateway.auth` hoặc env).
- Trình hướng dẫn tạo token gateway theo mặc định (ngay cả trên local loopback).
- Giao diện người dùng gửi `connect.params.auth.token` hoặc `connect.params.auth.password`.
- Đối với các triển khai Control UI không phải local loopback, hãy đặt `gateway.controlUi.allowedOrigins`
  một cách rõ ràng (các nguồn gốc đầy đủ). Nếu không có nó, khởi động gateway bị từ chối theo mặc định.
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` cho phép
  chế độ dự phòng nguồn gốc Host-header, nhưng là một hạ cấp bảo mật nguy hiểm.
- Với Serve, các tiêu đề nhận dạng Tailscale có thể thỏa mãn xác thực Control UI/WebSocket
  khi `gateway.auth.allowTailscale` là `true` (không cần token/mật khẩu).
  Các điểm cuối HTTP API vẫn yêu cầu token/mật khẩu. Đặt
  `gateway.auth.allowTailscale: false` để yêu cầu thông tin xác thực rõ ràng. Xem
  [Tailscale](/gateway/tailscale) và [Bảo mật](/gateway/security). Luồng không token này giả định rằng máy chủ gateway được tin tưởng.
- `gateway.tailscale.mode: "funnel"` yêu cầu `gateway.auth.mode: "password"` (mật khẩu được chia sẻ).
## Xây dựng giao diện người dùng

Gateway phục vụ các tệp tĩnh từ `dist/control-ui`. Xây dựng chúng bằng:

```bash
pnpm ui:build # auto-installs UI deps on first run
```