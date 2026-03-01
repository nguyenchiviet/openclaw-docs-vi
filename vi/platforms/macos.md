---
summary: Ứng dụng đồng hành OpenClaw cho macOS (menu bar + gateway broker)
read_when:
  - Triển khai các tính năng ứng dụng macOS
  - Thay đổi vòng đời gateway hoặc node bridging trên macOS
title: Ứng dụng macOS
x-i18n:
  source_path: platforms\macos.md
  source_hash: 126e63ef853a8acf3c836ae562601912f347927595ad4a4a643fdc4c12d31e2f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:23.246Z'
---

# OpenClaw macOS Companion (thanh menu + gateway broker)

Ứng dụng macOS là **ứng dụng đi kèm thanh menu** cho OpenClaw. Nó sở hữu các quyền,
quản lý/kết nối với Gateway cục bộ (launchd hoặc thủ công), và hiển thị các khả năng macOS
cho agent như một node.
## Chức năng

- Hiển thị thông báo gốc và trạng thái trong thanh menu.
- Quản lý các lời nhắc TCC (Thông báo, Khả năng truy cập, Ghi hình màn hình, Micrô,
  Nhận dạng giọng nói, Tự động hóa/AppleScript).
- Chạy hoặc kết nối với Gateway (cục bộ hoặc từ xa).
- Cung cấp các công cụ chỉ dành cho macOS (Canvas, Camera, Ghi hình màn hình, `system.run`).
- Khởi động dịch vụ máy chủ node cục bộ ở chế độ **remote** (launchd), và dừng nó ở chế độ **local**.
- Tùy chọn lưu trữ **PeekabooBridge** để tự động hóa giao diện người dùng.
- Cài đặt CLI toàn cầu (`openclaw`) thông qua npm/pnpm khi được yêu cầu (không khuyến nghị bun cho runtime Gateway).
## Chế độ cục bộ và chế độ từ xa

- **Cục bộ** (mặc định): ứng dụng kết nối với Gateway đang chạy cục bộ nếu có;
  nếu không, nó bật dịch vụ launchd thông qua `openclaw gateway install`.
- **Từ xa**: ứng dụng kết nối với Gateway qua SSH/Tailscale và không bao giờ khởi động
  một quy trình cục bộ.
  Ứng dụng khởi động **dịch vụ node host** cục bộ để Gateway từ xa có thể truy cập Mac này.
  Ứng dụng không tạo Gateway như một quy trình con.
## Điều khiển Launchd

Ứng dụng quản lý một LaunchAgent cho mỗi người dùng có nhãn `ai.openclaw.gateway`
(hoặc `ai.openclaw.<profile>` khi sử dụng `--profile`/`OPENCLAW_PROFILE`; `com.openclaw.*` cũ vẫn gỡ bỏ).

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Replace the label with `ai.openclaw.<profile>` when running a named profile.

If the LaunchAgent isn’t installed, enable it from the app or run
`openclaw gateway install`.
## Khả năng của node (mac)

Ứng dụng macOS trình bày chính nó như một node. Các lệnh phổ biến:

- Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
- Camera: `camera.snap`, `camera.clip`
- Screen: `screen.record`
- System: `system.run`, `system.notify`

Node báo cáo một bản đồ `permissions` để các agent có thể quyết định những gì được phép.

Node service + app IPC:

- Khi dịch vụ node headless host đang chạy (chế độ remote), nó kết nối với Gateway WS như một node.
- `system.run` thực thi trong ứng dụng macOS (ngữ cảnh UI/TCC) qua một Unix socket cục bộ; các lời nhắc + đầu ra vẫn ở trong ứng dụng.

Sơ đồ (SCI):

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```
## Phê duyệt Exec (system.run)

`system.run` được kiểm soát bởi **Phê duyệt Exec** trong ứng dụng macOS (Settings → Exec approvals).
Security + ask + allowlist được lưu trữ cục bộ trên Mac trong:

```
~/.openclaw/exec-approvals.json
```

Example:

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

Notes:

- `allowlist` entries are glob patterns for resolved binary paths.
- Raw shell command text that contains shell control or expansion syntax (`&&`, `||`, `;`, `|`, `` ` ``, `$`, `<`, `>`, `(`, `)`) is treated as an allowlist miss and requires explicit approval (or allowlisting the shell binary).
- Choosing “Always Allow” in the prompt adds that command to the allowlist.
- `system.run` environment overrides are filtered (drops `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`) and then merged with the app’s environment.
- For shell wrappers (`bash|sh|zsh ... -c/-lc`), request-scoped environment overrides are reduced to a small explicit allowlist (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
- For allow-always decisions in allowlist mode, known dispatch wrappers (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) lưu giữ các đường dẫn tệp thực thi bên trong thay vì đường dẫn wrapper. Nếu unwrapping không an toàn, không có mục allowlist nào được lưu giữ tự động.
## Liên kết sâu

Ứng dụng đăng ký lược đồ URL `openclaw://` cho các hành động cục bộ.

### `openclaw://agent`

Kích hoạt yêu cầu Gateway `agent`.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Query parameters:

- `message` (required)
- `sessionKey` (optional)
- `thinking` (optional)
- `deliver` / `to` / `channel` (optional)
- `timeoutSeconds` (optional)
- `key` (optional unattended mode key)

Safety:

- Without `key`, the app prompts for confirmation.
- Without `key`, the app enforces a short message limit for the confirmation prompt and ignores `deliver` / `to` / `channel`.
- With a valid `key`, quá trình chạy không được giám sát (dành cho các tự động hóa cá nhân).
## Thiết lập ban đầu (điển hình)

1. Cài đặt và khởi chạy **OpenClaw.app**.
2. Hoàn thành danh sách kiểm tra quyền (các lời nhắc TCC).
3. Đảm bảo chế độ **Local** đang hoạt động và Gateway đang chạy.
4. Cài đặt CLI nếu bạn muốn truy cập terminal.
## Quy trình xây dựng & phát triển (native)

- `cd apps/macos && swift build`
- `swift run OpenClaw` (hoặc Xcode)
- Đóng gói ứng dụng: `scripts/package-mac-app.sh`
## Gỡ lỗi kết nối Gateway (macOS CLI)

Sử dụng CLI gỡ lỗi để thực hiện cùng một bắt tay WebSocket Gateway và logic khám phá thiết bị mà ứng dụng macOS sử dụng, mà không cần khởi chạy ứng dụng.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Connect options:

- `--url <ws://host:port>`: override config
- `--mode <local|remote>`: resolve from config (default: config or local)
- `--probe`: force a fresh health probe
- `--timeout <ms>`: request timeout (default: `15000`)
- `--json`: structured output for diffing

Discovery options:

- `--include-local`: include gateways that would be filtered as “local”
- `--timeout <ms>`: overall discovery window (default: `2000`)
- `--json`: structured output for diffing

Tip: compare against `openclaw gateway discover --json` to see whether the
macOS app’s discovery pipeline (NWBrowser + tailnet DNS‑SD fallback) differs from
the Node CLI’s `dns-sd` dựa trên khám phá thiết bị.
## Cơ sở hạ tầng kết nối từ xa (SSH tunnels)

Khi ứng dụng macOS chạy ở chế độ **Remote**, nó mở một SSH tunnel để các thành phần UI cục bộ có thể giao tiếp với một Gateway từ xa như thể nó nằm trên localhost.

### Control tunnel (cổng WebSocket của Gateway)

- **Mục đích:** kiểm tra sức khỏe, trạng thái, Web Chat, cấu hình và các cuộc gọi control-plane khác.
- **Cổng cục bộ:** cổng Gateway (mặc định `18789`), luôn ổn định.
- **Cổng từ xa:** cùng cổng Gateway trên máy chủ từ xa.
- **Hành vi:** không có cổng cục bộ ngẫu nhiên; ứng dụng sử dụng lại một tunnel khỏe mạnh hiện có hoặc khởi động lại nếu cần.
- **Hình dạng SSH:** `ssh -N -L <local>:127.0.0.1:<remote>` với BatchMode + ExitOnForwardFailure + các tùy chọn keepalive.
- **Báo cáo IP:** SSH tunnel sử dụng loopback, vì vậy gateway sẽ thấy IP của node là `127.0.0.1`. Sử dụng giao thức truyền tải **Direct (ws/wss)** nếu bạn muốn IP máy khách thực tế xuất hiện (xem [macOS remote access](/platforms/mac/remote)).

Để biết các bước thiết lập, xem [macOS remote access](/platforms/mac/remote). Để biết chi tiết giao thức, xem [Gateway protocol](/gateway/protocol).
## Tài liệu liên quan

- [Sổ tay Gateway](/gateway)
- [Gateway (/platforms/mac/bundled-gateway)
- [Quyền truy cập macOS](/platforms/mac/permissions)
- [Canvas](/platforms/mac/canvas)