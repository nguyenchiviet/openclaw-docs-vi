---
summary: 'Giao diện điều khiển dựa trên trình duyệt cho Gateway (chat, nodes, config)'
read_when:
  - Bạn muốn vận hành Gateway từ một trình duyệt web
  - Bạn muốn có quyền truy cập Tailnet mà không cần SSH tunnels
title: Giao diện Điều khiển
x-i18n:
  source_path: web\control-ui.md
  source_hash: 6479b3d0afb32083c491a6cb59379d9881193121f73a82dd81fbfb1ee0af7b3d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:34:22.034Z'
---

# Giao diện điều khiển (trình duyệt)

Giao diện điều khiển là một ứng dụng một trang nhỏ **Vite + Lit** được phục vụ bởi Gateway:

- mặc định: `http://<host>:18789/`
- tiền tố tùy chọn: đặt `gateway.controlUi.basePath` (ví dụ: `/openclaw`)

Nó giao tiếp **trực tiếp với WebSocket của Gateway** trên cùng một cổng.
## Bắt đầu nhanh (cục bộ)

Nếu Gateway đang chạy trên cùng một máy tính, hãy mở:

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (hoặc [http://localhost:18789/](http://localhost:18789/))

Nếu trang không tải được, hãy khởi động Gateway trước: `openclaw gateway`.

Xác thực được cung cấp trong quá trình bắt tay WebSocket thông qua:

- `connect.params.auth.token`
- `connect.params.auth.password`
  Bảng điều khiển cài đặt cho phép bạn lưu trữ một token; mật khẩu không được lưu giữ.
  Trình hướng dẫn thiết lập ban đầu tạo một token gateway theo mặc định, vì vậy hãy dán nó ở đây khi kết nối lần đầu.
## Ghép nối thiết bị (kết nối lần đầu)

Khi bạn kết nối với Control UI từ trình duyệt hoặc thiết bị mới, Gateway
yêu cầu **phê duyệt ghép nối một lần** — ngay cả khi bạn đang trên cùng Tailnet
với `gateway.auth.allowTailscale: true`. Đây là một biện pháp bảo mật để ngăn chặn
truy cập trái phép.

**Những gì bạn sẽ thấy:** "disconnected (1008): pairing required"

**Để phê duyệt thiết bị:**

```bash
# List pending requests
openclaw devices list

# Approve by request ID
openclaw devices approve <requestId>
```

Once approved, the device is remembered and won't require re-approval unless
you revoke it with `openclaw devices revoke --device <id> --role <role>`. See
[Devices CLI](/cli/devices) for token rotation and revocation.

**Notes:**

- Local connections (`127.0.0.1`) được tự động phê duyệt.
- Các kết nối từ xa (LAN, Tailnet, v.v.) yêu cầu phê duyệt rõ ràng.
- Mỗi hồ sơ trình duyệt tạo ra một ID thiết bị duy nhất, vì vậy chuyển đổi trình duyệt hoặc
  xóa dữ liệu trình duyệt sẽ yêu cầu ghép nối lại.
## Những gì nó có thể làm (ngày hôm nay)

- Chat với mô hình qua Gateway WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
- Truyền phát các lệnh gọi công cụ + thẻ đầu ra công cụ trực tiếp trong Chat (sự kiện agent)
- Kênh: WhatsApp/Telegram/Discord/Slack + kênh plugin (Mattermost, v.v.) trạng thái + đăng nhập QR + cấu hình theo kênh (`channels.status`, `web.login.*`, `config.patch`)
- Instances: danh sách hiện diện + làm mới (`system-presence`)
- Sessions: danh sách + ghi đè thinking/verbose theo phiên (`sessions.list`, `sessions.patch`)
- Cron jobs: danh sách/thêm/chỉnh sửa/chạy/bật/tắt + lịch sử chạy (`cron.*`)
- Skills: trạng thái, bật/tắt, cài đặt, cập nhật khóa API (`skills.*`)
- Nodes: danh sách + khả năng (`node.list`)
- Phê duyệt thực thi: chỉnh sửa danh sách cho phép Gateway hoặc node + yêu cầu chính sách cho `exec host=gateway/node` (`exec.approvals.*`)
- Cấu hình: xem/chỉnh sửa `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
- Cấu hình: áp dụng + khởi động lại với xác thực (`config.apply`) và đánh thức phiên hoạt động cuối cùng
- Ghi cấu hình bao gồm bảo vệ base-hash để ngăn chặn ghi đè các chỉnh sửa đồng thời
- Lược đồ cấu hình + hiển thị biểu mẫu (`config.schema`, bao gồm lược đồ plugin + kênh); Trình chỉnh sửa JSON thô vẫn có sẵn
- Debug: ảnh chụp trạng thái/sức khỏe/mô hình + nhật ký sự kiện + lệnh gọi RPC thủ công (`status`, `health`, `models.list`)
- Logs: theo dõi trực tiếp nhật ký tệp Gateway với bộ lọc/xuất (`logs.tail`)
- Update: chạy cập nhật gói/git + khởi động lại (`update.run`) với báo cáo khởi động lại

Ghi chú bảng điều khiển Cron jobs:

- Đối với các công việc cô lập, giao hàng mặc định là tóm tắt thông báo. Bạn có thể chuyển sang không có nếu bạn muốn chỉ chạy nội bộ.
- Các trường kênh/mục tiêu xuất hiện khi chọn thông báo.
- Chế độ Webhook sử dụng `delivery.mode = "webhook"` với `delivery.to` được đặt thành URL webhook HTTP(S) hợp lệ.
- Đối với các công việc phiên chính, các chế độ giao hàng webhook và không có sẵn.
- Các điều khiển chỉnh sửa nâng cao bao gồm xóa sau khi chạy, xóa ghi đè agent, các tùy chọn cron chính xác/lệch, ghi đè mô hình/thinking agent và các bộ chuyển đổi giao hàng tốt nhất.
- Xác thực biểu mẫu là nội tuyến với lỗi cấp trường; các giá trị không hợp lệ sẽ vô hiệu hóa nút lưu cho đến khi được sửa.
- Đặt `cron.webhookToken` để gửi mã thông báo người mang riêng, nếu bỏ qua webhook sẽ được gửi mà không có tiêu đề xác thực.
- Fallback không dùng nữa: các công việc kế thừa được lưu trữ với `notify: true` vẫn có thể sử dụng `cron.webhook` cho đến khi được di chuyển.
## Hành vi trò chuyện

- `chat.send` là **không chặn**: nó xác nhận ngay lập tức bằng `{ runId, status: "started" }` và phản hồi được truyền phát qua các sự kiện `chat`.
- Gửi lại với cùng `idempotencyKey` trả về `{ status: "in_flight" }` trong khi chạy, và `{ status: "ok" }` sau khi hoàn thành.
- Các phản hồi `chat.history` có giới hạn kích thước để đảm bảo an toàn giao diện. Khi các mục bảng ghi quá lớn, Gateway có thể cắt ngắn các trường văn bản dài, bỏ qua các khối siêu dữ liệu nặng và thay thế các tin nhắn quá lớn bằng một trình giữ chỗ (`[chat.history omitted: message too large]`).
- `chat.inject` thêm một ghi chú của trợ lý vào bảng ghi phiên và phát sóng một sự kiện `chat` để cập nhật chỉ giao diện (không chạy agent, không gửi kênh).
- Dừng:
  - Nhấp vào **Dừng** (gọi `chat.abort`)
  - Nhập `/stop` (hoặc các cụm từ hủy bỏ độc lập như `stop`, `stop action`, `stop run`, `stop openclaw`, `please stop`) để hủy bỏ ngoài dải
  - `chat.abort` hỗ trợ `{ sessionKey }` (không `runId`) để hủy bỏ tất cả các lần chạy hoạt động cho phiên đó
- Giữ lại một phần khi hủy bỏ:
  - Khi một lần chạy bị hủy bỏ, văn bản trợ lý một phần vẫn có thể được hiển thị trong giao diện
  - Gateway duy trì văn bản trợ lý một phần bị hủy bỏ vào lịch sử bảng ghi khi có đầu ra được lưu đệm
  - Các mục được duy trì bao gồm siêu dữ liệu hủy bỏ để các bộ tiêu thụ bảng ghi có thể phân biệt các phần hủy bỏ một phần với đầu ra hoàn thành bình thường
## Truy cập Tailnet (được khuyến nghị)

### Integrated Tailscale Serve (ưu tiên)

Giữ Gateway trên local loopback và để Tailscale Serve proxy nó với HTTPS:

```bash
openclaw gateway --tailscale serve
```

Open:

- `https://<magicdns>/` (or your configured `gateway.controlUi.basePath`)

By default, Control UI/WebSocket Serve requests can authenticate via Tailscale identity headers
(`tailscale-user-login`) when `gateway.auth.allowTailscale` is `true`. OpenClaw
verifies the identity by resolving the `x-forwarded-for` address with
`tailscale whois` and matching it to the header, and only accepts these when the
request hits loopback with Tailscale’s `x-forwarded-*` headers. Set
`gateway.auth.allowTailscale: false` (or force `gateway.auth.mode: "password"`)
if you want to require a token/password even for Serve traffic.
Tokenless Serve auth assumes the gateway host is trusted. If untrusted local
code may run on that host, require token/password auth.

### Bind to tailnet + token

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Then open:

- `http://<tailscale-ip>:18789/` (or your configured `gateway.controlUi.basePath`)

Paste the token into the UI settings (sent as `connect.params.auth.token`).
## HTTP Không Bảo Mật

Nếu bạn mở bảng điều khiển qua HTTP thuần túy (`http://<lan-ip>` hoặc `http://<tailscale-ip>`),
trình duyệt chạy trong **ngữ cảnh không bảo mật** và chặn WebCrypto. Theo mặc định,
OpenClaw **chặn** các kết nối Control UI mà không có nhận dạng thiết bị.

**Sửa chữa được khuyến nghị:** sử dụng HTTPS (Tailscale Serve) hoặc mở UI cục bộ:

- `https://<magicdns>/` (Serve)
- `http://127.0.0.1:18789/` (trên máy chủ gateway)

**Hành vi chuyển đổi xác thực không bảo mật:**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth` does not bypass Control UI device identity or pairing checks.

**Break-glass only:**

```json5
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth` vô hiệu hóa các kiểm tra nhận dạng thiết bị Control UI và là một
hạ cấp bảo mật nghiêm trọng. Hoàn nguyên nhanh chóng sau khi sử dụng khẩn cấp.

Xem [Tailscale](/gateway/tailscale) để biết hướng dẫn thiết lập HTTPS.
## Xây dựng giao diện người dùng

Gateway phục vụ các tệp tĩnh từ `dist/control-ui`. Xây dựng chúng bằng:

```bash
pnpm ui:build # auto-installs UI deps on first run
```

Optional absolute base (when you want fixed asset URLs):

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

For local development (separate dev server):

```bash
pnpm ui:dev # auto-installs UI deps on first run
```

Then point the UI at your Gateway WS URL (e.g. `ws://127.0.0.1:18789`).
## Gỡ lỗi/kiểm tra: máy chủ dev + Gateway từ xa

Control UI là các tệp tĩnh; mục tiêu WebSocket có thể cấu hình và có thể khác với nguồn HTTP. Điều này rất hữu ích khi bạn muốn máy chủ dev Vite cục bộ nhưng Gateway chạy ở nơi khác.

1. Khởi động máy chủ dev UI: `pnpm ui:dev`
2. Mở một URL như:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Optional one-time auth (if needed):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Notes:

- `gatewayUrl` is stored in localStorage after load and removed from the URL.
- `token` is stored in localStorage; `password` is kept in memory only.
- When `gatewayUrl` is set, the UI does not fall back to config or environment credentials.
  Provide `token` (or `password`) explicitly. Missing explicit credentials is an error.
- Use `wss://` when the Gateway is behind TLS (Tailscale Serve, HTTPS proxy, etc.).
- `gatewayUrl` is only accepted in a top-level window (not embedded) to prevent clickjacking.
- Non-loopback Control UI deployments must set `gateway.controlUi.allowedOrigins`
  explicitly (full origins). This includes remote dev setups.
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` enables
  Host-header origin fallback mode, but it is a dangerous security mode.

Example:

```json5
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

Chi tiết thiết lập truy cập từ xa: [Truy cập từ xa](/gateway/remote).