---
summary: >-
  Nodes: ghép nối, khả năng, quyền hạn và trợ giúp CLI cho
  canvas/camera/screen/system
read_when:
  - Ghép nối các nút iOS/Android với một gateway
  - Sử dụng node canvas/camera cho ngữ cảnh agent
  - Thêm các lệnh node mới hoặc trợ giúp CLI
title: Nút
x-i18n:
  source_path: nodes\index.md
  source_hash: 044fc1c3ab8d9a47c3bd826d549e1c312db04155634269394166fe6be41fc0a3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:11:07.957Z'
---

# Nodes

Một **node** là một thiết bị đi kèm (macOS/iOS/Android/headless) kết nối với Gateway **WebSocket** (cùng cổng với operators) bằng `role: "node"` và hiển thị một bề mặt lệnh (ví dụ: `canvas.*`, `camera.*`, `system.*`) qua `node.invoke`. Chi tiết giao thức: [Gateway protocol](/gateway/protocol).

Giao thức truyền tải cũ: [Bridge protocol](/gateway/bridge-protocol) (TCP JSONL; không dùng nữa/đã xóa cho các node hiện tại).

macOS cũng có thể chạy ở **chế độ node**: ứng dụng thanh menu kết nối với máy chủ WS của Gateway và hiển thị các lệnh canvas/camera cục bộ của nó dưới dạng một node (vì vậy `openclaw nodes …` hoạt động với Mac này).

Ghi chú:

- Nodes là **thiết bị ngoại vi**, không phải gateway. Chúng không chạy dịch vụ gateway.
- Các tin nhắn Telegram/WhatsApp/v.v. đến **gateway**, không phải đến nodes.
- Sổ tay khắc phục sự cố: [/nodes/troubleshooting](/nodes/troubleshooting)
## Ghép nối + trạng thái

**Các node WS sử dụng ghép nối thiết bị.** Các node trình bày danh tính thiết bị trong `connect`; Gateway
tạo yêu cầu ghép nối thiết bị cho `role: node`. Phê duyệt qua CLI thiết bị (hoặc UI).

CLI nhanh:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Notes:

- `nodes status` marks a node as **paired** when its device pairing role includes `node`.
- `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) is a separate gateway-owned
  node pairing store; it does **not** gate the WS `connect` bắt tay.
## Node host từ xa (system.run)

Sử dụng **node host** khi Gateway của bạn chạy trên một máy và bạn muốn các lệnh
thực thi trên máy khác. Mô hình vẫn nói chuyện với **gateway**; gateway
chuyển tiếp `exec` gọi đến **node host** khi `host=node` được chọn.

### Chạy ở đâu

- **Gateway host**: nhận tin nhắn, chạy mô hình, định tuyến các lệnh công cụ.
- **Node host**: thực thi `system.run`/`system.which` trên máy node.
- **Phê duyệt**: được thực thi trên node host thông qua `~/.openclaw/exec-approvals.json`.

### Khởi động node host (foreground)

Trên máy node:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### Remote gateway via SSH tunnel (loopback bind)

If the Gateway binds to loopback (`gateway.bind=loopback`, default in local mode),
remote node hosts cannot connect directly. Create an SSH tunnel and point the
node host at the local end of the tunnel.

Example (node host -> gateway host):

```bash
# Terminal A (keep running): forward local 18790 -> gateway 127.0.0.1:18789
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Terminal B: export the gateway token and connect through the tunnel
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

Notes:

- The token is `gateway.auth.token` from the gateway config (`~/.openclaw/openclaw.json` on the gateway host).
- `openclaw node run` reads `OPENCLAW_GATEWAY_TOKEN` for auth.

### Start a node host (service)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### Pair + name

On the gateway host:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Tùy chọn đặt tên:
- `--display-name` trên `openclaw node run` / `openclaw node install` (lưu trữ trong `~/.openclaw/node.json` trên node).
- `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (ghi đè gateway).

### Danh sách cho phép các lệnh

Phê duyệt exec là **cho mỗi node host**. Thêm các mục danh sách cho phép từ gateway:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Approvals live on the node host at `~/.openclaw/exec-approvals.json`.

### Point exec at the node

Configure defaults (gateway config):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Or per session:

```
/exec host=node security=allowlist node=<id-or-name>
```

Once set, any `exec` call with `host=node` chạy trên node host (tuân theo danh sách cho phép/phê duyệt của node).

Liên quan:

- [Node host CLI](/cli/node)
- [Exec tool](/tools/exec)
- [Exec approvals](/tools/exec-approvals)
## Gọi lệnh

Mức thấp (RPC thô):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Các trình trợ giúp mức cao hơn tồn tại cho các quy trình công việc phổ biến "cung cấp cho agent một tệp đính kèm MEDIA".
## Ảnh chụp màn hình (ảnh chụp canvas)

Nếu node đang hiển thị Canvas (WebView), `canvas.snapshot` trả về `{ format, base64 }`.

Trợ giúp CLI (ghi vào tệp tạm thời và in `MEDIA:<path>`):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### Canvas controls

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notes:

- `canvas present` accepts URLs or local file paths (`--target`), plus optional `--x/--y/--width/--height` for positioning.
- `canvas eval` accepts inline JS (`--js`) or a positional arg.

### A2UI (Canvas)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Ghi chú:

- Chỉ hỗ trợ A2UI v0.8 JSONL (v0.9/createSurface bị từ chối).
## Ảnh + video (node camera)

Ảnh (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # default: both facings (2 MEDIA lines)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Video clips (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notes:

- The node must be **foregrounded** for `canvas.*` and `camera.*` (background calls return `NODE_BACKGROUND_UNAVAILABLE`).
- Clip duration is clamped (currently `<= 60s`) to avoid oversized base64 payloads.
- Android will prompt for `CAMERA`/`RECORD_AUDIO` permissions when possible; denied permissions fail with `*_PERMISSION_REQUIRED__OC_I18N_0012__`.
## Ghi hình màn hình (nodes)

Nodes expose `screen.record` (mp4). Ví dụ:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notes:

- `screen.record` requires the node app to be foregrounded.
- Android will show the system screen-capture prompt before recording.
- Screen recordings are clamped to `<= 60s`.
- `--no-audio` disables microphone capture (supported on iOS/Android; macOS uses system capture audio).
- Use `--screen <index>` để chọn màn hình khi có nhiều màn hình khả dụng.
## Vị trí (nodes)

Nodes hiển thị `location.get` khi Location được bật trong cài đặt.

CLI helper:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Ghi chú:

- Location **tắt theo mặc định**.
- "Always" yêu cầu quyền hệ thống; tìm nạp nền là tốt nhất có thể.
- Phản hồi bao gồm lat/lon, độ chính xác (mét) và dấu thời gian.
## SMS (Android nodes)

Android nodes có thể expose `sms.send` khi người dùng cấp quyền **SMS** và thiết bị hỗ trợ điện thoại.

Low-level invoke:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notes:

- The permission prompt must be accepted on the Android device before the capability is advertised.
- Wi-Fi-only devices without telephony will not advertise `sms.send`.
## Các lệnh hệ thống (node host / mac node)

Node macOS hiển thị `system.run`, `system.notify`, và `system.execApprovals.get/set`.
Node host không giao diện hiển thị `system.run`, `system.which`, và `system.execApprovals.get/set`.

Ví dụ:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notes:

- `system.run` returns stdout/stderr/exit code in the payload.
- `system.notify` respects notification permission state on the macOS app.
- `system.run` supports `--cwd`, `--env KEY=VAL`, `--command-timeout`, and `--needs-screen-recording`.
- For shell wrappers (`bash|sh|zsh ... -c/-lc`), request-scoped `--env` values are reduced to an explicit allowlist (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
- For allow-always decisions in allowlist mode, known dispatch wrappers (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) persist inner executable paths instead of wrapper paths. If unwrapping is not safe, no allowlist entry is persisted automatically.
- On Windows node hosts in allowlist mode, shell-wrapper runs via `cmd.exe /c` require approval (allowlist entry alone does not auto-allow the wrapper form).
- `system.notify` supports `--priority <passive|active|timeSensitive>` and `--delivery <system|overlay|auto>`.
- Node hosts ignore `PATH` overrides and strip dangerous startup/shell keys (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`). If you need extra PATH entries, configure the node host service environment (or install tools in standard locations) instead of passing `PATH` via `--env`.
- On macOS node mode, `system.run` is gated by exec approvals in the macOS app (Settings → Exec approvals).
  Ask/allowlist/full behave the same as the headless node host; denied prompts return `SYSTEM_RUN_DENIED`.
- On headless node host, `system.run` is gated by exec approvals (`~/.openclaw/exec-approvals.json`).
## Liên kết node Exec

Khi có nhiều node khả dụng, bạn có thể liên kết exec với một node cụ thể.
Điều này đặt node mặc định cho `exec host=node` (và có thể được ghi đè cho từng agent).

Mặc định toàn cục:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Per-agent override:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Unset to allow any node:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```
## Bản đồ quyền hạn

Các node có thể bao gồm một bản đồ `permissions` trong `node.list` / `node.describe`, được khóa theo tên quyền hạn (ví dụ: `screenRecording`, `accessibility`) với các giá trị boolean (`true` = được cấp).
## Nút máy chủ không giao diện (đa nền tảng)

OpenClaw có thể chạy một **nút máy chủ không giao diện** (không có UI) kết nối với Gateway
WebSocket và hiển thị `system.run` / `system.which`. Điều này hữu ích trên Linux/Windows
hoặc để chạy một nút tối thiểu cùng với một máy chủ.

Khởi động nó:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notes:

- Pairing is still required (the Gateway will show a node approval prompt).
- The node host stores its node id, token, display name, and gateway connection info in `~/.openclaw/node.json`.
- Exec approvals are enforced locally via `~/.openclaw/exec-approvals.json`
  (see [Exec approvals](/tools/exec-approvals)).
- On macOS, the headless node host executes `system.run` locally by default. Set
  `OPENCLAW_NODE_EXEC_HOST=app` to route `system.run` through the companion app exec host; add
  `OPENCLAW_NODE_EXEC_FALLBACK=0` to require the app host and fail closed if it is unavailable.
- Add `--tls` / `--tls-fingerprint` khi Gateway WS sử dụng TLS.
## Chế độ node Mac

- Ứng dụng thanh menu macOS kết nối với máy chủ WS Gateway dưới dạng một node (vì vậy `openclaw nodes …` hoạt động với Mac này).
- Ở chế độ từ xa, ứng dụng mở một đường hầm SSH cho cổng Gateway và kết nối với `localhost`.