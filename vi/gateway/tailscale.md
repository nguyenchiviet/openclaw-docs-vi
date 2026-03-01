---
summary: Tích hợp Tailscale Serve/Funnel cho bảng điều khiển Gateway
read_when:
  - Hiển thị Gateway Control UI bên ngoài localhost
  - Tự động hóa quyền truy cập tailnet hoặc bảng điều khiển công khai
title: Tailscale
x-i18n:
  source_path: gateway\tailscale.md
  source_hash: 30c91f2ca5ab6aef19fbb166c170673020ce1c4e308a6abd6724774a5e595e9c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:43.680Z'
---

# Tailscale (Bảng điều khiển Gateway)

OpenClaw có thể tự động cấu hình Tailscale **Serve** (tailnet) hoặc **Funnel** (công khai) cho bảng điều khiển Gateway và cổng WebSocket. Điều này giữ cho Gateway được ràng buộc với local loopback trong khi Tailscale cung cấp HTTPS, định tuyến và (đối với Serve) các tiêu đề xác thực danh tính.
## Chế độ

- `serve`: Chỉ Tailnet Phục vụ qua `tailscale serve`. Gateway ở lại `127.0.0.1`.
- `funnel`: HTTPS công khai qua `tailscale funnel`. OpenClaw yêu cầu mật khẩu được chia sẻ.
- `off`: Mặc định (không tự động hóa Tailscale).
## Xác thực

Đặt `gateway.auth.mode` để kiểm soát bắt tay:

- `token` (mặc định khi `OPENCLAW_GATEWAY_TOKEN` được đặt)
- `password` (bí mật dùng chung qua `OPENCLAW_GATEWAY_PASSWORD` hoặc cấu hình)

Khi `tailscale.mode = "serve"` và `gateway.auth.allowTailscale` là `true`,
xác thực Control UI/WebSocket có thể sử dụng tiêu đề nhận dạng Tailscale
(`tailscale-user-login`) mà không cần cung cấp token/mật khẩu. OpenClaw xác minh
danh tính bằng cách phân giải địa chỉ `x-forwarded-for` qua daemon Tailscale cục bộ
(`tailscale whois`) và so khớp nó với tiêu đề trước khi chấp nhận. OpenClaw chỉ
coi một yêu cầu là Serve khi nó đến từ local loopback với các tiêu đề
`x-forwarded-for`, `x-forwarded-proto` và `x-forwarded-host` của Tailscale.
Các điểm cuối HTTP API (ví dụ `/v1/*`, `/tools/invoke` và `/api/channels/*`)
vẫn yêu cầu xác thực token/mật khẩu.
Luồng không có token này giả định rằng máy chủ gateway được tin tưởng. Nếu mã cục bộ
không đáng tin cậy có thể chạy trên cùng một máy chủ, hãy vô hiệu hóa `gateway.auth.allowTailscale`
và yêu cầu xác thực token/mật khẩu thay thế.
Để yêu cầu thông tin xác thực rõ ràng, hãy đặt `gateway.auth.allowTailscale: false` hoặc
buộc `gateway.auth.mode: "password"`.
## Ví dụ cấu hình

### Chỉ Tailnet (Serve)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Open: `https://<magicdns>/` (or your configured `gateway.controlUi.basePath`)

### Tailnet-only (bind to Tailnet IP)

Use this when you want the Gateway to listen directly on the Tailnet IP (no Serve/Funnel).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

Connect from another Tailnet device:

- Control UI: `http://<tailscale-ip>:18789/`
- WebSocket: `ws://<tailscale-ip>:18789`

Note: loopback (`http://127.0.0.1:18789`) will **not** work in this mode.

### Public internet (Funnel + shared password)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

Prefer `OPENCLAW_GATEWAY_PASSWORD` thay vì lưu mật khẩu vào đĩa.
## Ví dụ CLI

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```
## Ghi chú

- Tailscale Serve/Funnel yêu cầu `tailscale` CLI được cài đặt và đăng nhập.
- `tailscale.mode: "funnel"` từ chối khởi động trừ khi chế độ xác thực là `password` để tránh tiếp xúc công khai.
- Đặt `gateway.tailscale.resetOnExit` nếu bạn muốn OpenClaw hoàn tác `tailscale serve`
  hoặc cấu hình `tailscale funnel` khi tắt.
- `gateway.bind: "tailnet"` là một liên kết Tailnet trực tiếp (không HTTPS, không Serve/Funnel).
- `gateway.bind: "auto"` ưa thích local loopback; sử dụng `tailnet` nếu bạn chỉ muốn Tailnet.
- Serve/Funnel chỉ tiếp xúc **Gateway control UI + WS**. Các node kết nối qua
  cùng một điểm cuối Gateway WS, vì vậy Serve có thể hoạt động để truy cập node.
## Điều khiển trình duyệt (Gateway từ xa + trình duyệt cục bộ)

Nếu bạn chạy Gateway trên một máy nhưng muốn điều khiển trình duyệt trên một máy khác,
hãy chạy một **node host** trên máy trình duyệt và giữ cả hai trên cùng một tailnet.
Gateway sẽ chuyển tiếp các hành động trình duyệt đến node; không cần máy chủ điều khiển riêng biệt hoặc Serve URL.

Tránh sử dụng Funnel để điều khiển trình duyệt; coi việc ghép nối node như truy cập của nhà điều hành.
## Điều kiện tiên quyết và giới hạn của Tailscale

- Serve yêu cầu HTTPS được bật cho tailnet của bạn; CLI sẽ nhắc nếu nó bị thiếu.
- Serve chèn các tiêu đề nhận dạng Tailscale; Funnel thì không.
- Funnel yêu cầu Tailscale v1.38.3+, MagicDNS, HTTPS được bật và một thuộc tính nút funnel.
- Funnel chỉ hỗ trợ các cổng `443`, `8443` và `10000` qua TLS.
- Funnel trên macOS yêu cầu biến thể ứng dụng Tailscale mã nguồn mở.
## Tìm hiểu thêm

- Tailscale Serve overview: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
- `tailscale serve` command: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
- Tailscale Funnel overview: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
- `tailscale funnel` command: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)