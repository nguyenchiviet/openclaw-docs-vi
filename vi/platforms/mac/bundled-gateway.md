---
summary: Gateway runtime trên macOS (dịch vụ launchd bên ngoài)
read_when:
  - Đóng gói OpenClaw.app
  - Gỡ lỗi dịch vụ launchd Gateway trên macOS
  - Cài đặt gateway CLI cho macOS
title: Gateway trên macOS
x-i18n:
  source_path: platforms\mac\bundled-gateway.md
  source_hash: c1ba561b24f093a6bf0a5cc1258a443464cdaa7cdfae656ec1629a94442bf46d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:21.049Z'
---

# Gateway trên macOS (external launchd)

OpenClaw.app không còn đóng gói Node/Bun hoặc runtime Gateway. Ứng dụng macOS
mong đợi một cài đặt `openclaw` CLI **bên ngoài**, không sinh ra Gateway như một
tiến trình con, và quản lý một dịch vụ launchd cho mỗi người dùng để giữ Gateway
chạy (hoặc kết nối với một Gateway cục bộ hiện có nếu đã có một Gateway đang chạy).
## Cài đặt CLI (bắt buộc cho chế độ cục bộ)

Bạn cần Node 22+ trên Mac, sau đó cài đặt `openclaw` toàn cục:

```bash
npm install -g openclaw@<version>
```

Nút **Cài đặt CLI** của ứng dụng macOS chạy cùng một quy trình thông qua npm/pnpm (bun không được khuyến nghị cho runtime Gateway).
## Launchd (Gateway as LaunchAgent)

Label:

- `ai.openclaw.gateway` (hoặc `ai.openclaw.<profile>`; `com.openclaw.*` cũ có thể vẫn tồn tại)

Vị trí Plist (cho mỗi người dùng):

- `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
  (hoặc `~/Library/LaunchAgents/ai.openclaw.<profile>.plist`)

Trình quản lý:

- Ứng dụng macOS sở hữu cài đặt/cập nhật LaunchAgent ở chế độ Local.
- CLI cũng có thể cài đặt nó: `openclaw gateway install`.

Hành vi:

- "OpenClaw Active" bật/tắt LaunchAgent.
- Thoát ứng dụng **không** dừng gateway (launchd giữ nó hoạt động).
- Nếu Gateway đã chạy trên cổng được cấu hình, ứng dụng sẽ kết nối với nó thay vì khởi động một cái mới.

Ghi nhật ký:

- launchd stdout/err: `/tmp/openclaw/openclaw-gateway.log`
## Tương thích phiên bản

Ứng dụng macOS kiểm tra phiên bản Gateway so với phiên bản của nó. Nếu chúng không tương thích, hãy cập nhật CLI toàn cục để phù hợp với phiên bản ứng dụng.
## Kiểm tra cơ bản

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Then:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```