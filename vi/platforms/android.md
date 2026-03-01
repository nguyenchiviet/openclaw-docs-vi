---
summary: 'Ứng dụng Android (node): sổ tay kết nối + Canvas/Chat/Camera'
read_when:
  - Ghép nối hoặc kết nối lại nút Android
  - Gỡ lỗi khám phá Gateway Android hoặc xác thực
  - Xác minh tính nhất quán của lịch sử trò chuyện trên các ứng dụng khách
title: Ứng dụng Android
x-i18n:
  source_path: platforms\android.md
  source_hash: cd12aada854dd0d5c1eae05601d8eac409303fae0640063d9a1a3d0d15889876
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:11:32.609Z'
---

# Ứng dụng Android (Node)

## Hỗ trợ snapshot

- Vai trò: ứng dụng node đi kèm (Android không lưu trữ Gateway).
- Gateway bắt buộc: có (chạy trên macOS, Linux hoặc Windows qua WSL2).
- Cài đặt: [Bắt đầu](/start/getting-started) + [Ghép nối](/gateway/pairing).
- Gateway: [Runbook](/gateway) + [Cấu hình](/gateway/configuration).
  - Giao thức: [Gateway protocol](/gateway/protocol) (nodes + control plane).
## Kiểm soát hệ thống

Kiểm soát hệ thống (launchd/systemd) nằm trên máy chủ Gateway. Xem [Gateway](/gateway).
## Sổ tay kết nối

Android node app ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android kết nối trực tiếp đến Gateway WebSocket (mặc định `ws://<host>:18789`) và sử dụng ghép nối do Gateway sở hữu.

### Điều kiện tiên quyết

- Bạn có thể chạy Gateway trên máy "master".
- Thiết bị/trình giả lập Android có thể truy cập WebSocket gateway:
  - Cùng LAN với mDNS/NSD, **hoặc**
  - Cùng Tailscale tailnet sử dụng Wide-Area Bonjour / unicast DNS-SD (xem bên dưới), **hoặc**
  - Host/port gateway thủ công (dự phòng)
- Bạn có thể chạy CLI (`openclaw`) trên máy gateway (hoặc qua SSH).

### 1) Khởi động Gateway

```bash
openclaw gateway --port 18789 --verbose
```

Confirm in logs you see something like:

- `listening on ws://0.0.0.0:18789`

For tailnet-only setups (recommended for Vienna ⇄ London), bind the gateway to the tailnet IP:

- Set `gateway.bind: "tailnet"` in `~/.openclaw/openclaw.json` on the gateway host.
- Restart the Gateway / macOS menubar app.

### 2) Verify discovery (optional)

From the gateway machine:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

More debugging notes: [Bonjour](/gateway/bonjour).

#### Tailnet (Vienna ⇄ London) discovery via unicast DNS-SD

Android NSD/mDNS discovery won’t cross networks. If your Android node and the gateway are on different networks but connected via Tailscale, use Wide-Area Bonjour / unicast DNS-SD instead:

1. Set up a DNS-SD zone (example `openclaw.internal.`) on the gateway host and publish `_openclaw-gw._tcp` records.
2. Cấu hình Tailscale split DNS cho miền đã chọn của bạn trỏ đến máy chủ DNS đó.

Chi tiết và ví dụ cấu hình CoreDNS: [Bonjour](/gateway/bonjour).

### 3) Kết nối từ Android

Trong ứng dụng Android:

- Ứng dụng giữ kết nối gateway của nó hoạt động thông qua **dịch vụ foreground** (thông báo liên tục).
- Mở **Cài đặt**.
- Dưới **Discovered Gateways**, chọn gateway của bạn và nhấn **Kết nối**.
- Nếu mDNS bị chặn, hãy sử dụng **Advanced → Manual Gateway** (host + port) và **Connect (Manual)**.

Sau lần ghép nối thành công đầu tiên, Android tự động kết nối lại khi khởi động:
- Điểm cuối thủ công (nếu được bật), nếu không
- Gateway được khám phá cuối cùng (cố gắng tốt nhất).

### 4) Phê duyệt ghép nối (CLI)

Trên máy gateway:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Pairing details: [Gateway pairing](/gateway/pairing).

### 5) Verify the node is connected

- Via nodes status:

  ```bash
  openclaw nodes status
  ```

- Via Gateway:

  ```bash
  openclaw gateway call node.list --params "{}"
  ```

### 6) Chat + history

The Android node’s Chat sheet uses the gateway’s **primary session key** (`main`), so history and replies are shared with WebChat and other clients:

- History: `chat.history`
- Send: `chat.send`
- Push updates (best-effort): `chat.subscribe` → `event:"chat"`

### 7) Canvas + camera

#### Gateway Canvas Host (recommended for web content)

If you want the node to show real HTML/CSS/JS that the agent can edit on disk, point the node at the Gateway canvas host.

Note: nodes load canvas from the Gateway HTTP server (same port as `gateway.port`, default `18789`).

1. Create `~/.openclaw/workspace/canvas/index.html` on the gateway host.

2. Navigate the node to it (LAN):

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

Tailnet (optional): if both devices are on Tailscale, use a MagicDNS name or tailnet IP instead of `.local`, e.g. `http://<gateway-magicdns>:18789/__openclaw__/canvas/`.

This server injects a live-reload client into HTML and reloads on file changes.
The A2UI host lives at `http://<gateway-host>:18789/__openclaw__/a2ui/`.

Canvas commands (foreground only):

- `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (use `{"url":""}` or `{"url":"/"}` to return to the default scaffold). `canvas.snapshot` returns `{ format, base64 }` (default `format="jpeg"`).
- A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` bí danh cũ)

Các lệnh camera (chỉ ở chế độ nền trước; được kiểm soát quyền):

- `camera.snap` (jpg)
- `camera.clip` (mp4)

Xem [Camera node](/nodes/camera) để biết các tham số và trợ giúp CLI.