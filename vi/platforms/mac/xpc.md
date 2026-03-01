---
summary: >-
  Kiến trúc IPC macOS cho ứng dụng OpenClaw, vận chuyển nút gateway, và
  PeekabooBridge
read_when:
  - Chỉnh sửa hợp đồng IPC hoặc ứng dụng thanh menu IPC
title: macOS IPC
x-i18n:
  source_path: platforms\mac\xpc.md
  source_hash: d0211c334a4a59b71afb29dd7b024778172e529fa618985632d3d11d795ced92
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:59.643Z'
---

# Kiến trúc IPC macOS của OpenClaw

**Mô hình hiện tại:** một Unix socket cục bộ kết nối **dịch vụ node host** với **ứng dụng macOS** để phê duyệt exec + `system.run`. Một `openclaw-mac` CLI gỡ lỗi tồn tại để kiểm tra khám phá/kết nối; các hành động agent vẫn chảy qua Gateway WebSocket và `node.invoke`. Tự động hóa UI sử dụng PeekabooBridge.

## Mục tiêu

- Một phiên bản ứng dụng GUI duy nhất sở hữu tất cả công việc liên quan đến TCC (thông báo, ghi hình màn hình, mic, lời nói, AppleScript).
- Một bề mặt nhỏ cho tự động hóa: Gateway + lệnh node, cộng với PeekabooBridge để tự động hóa UI.
- Quyền dự đoán được: luôn cùng một bundle ID được ký, được khởi chạy bởi launchd, vì vậy các cấp phép TCC vẫn giữ nguyên.

## Cách hoạt động

### Gateway + node transport

- Ứng dụng chạy Gateway (chế độ cục bộ) và kết nối với nó dưới dạng một node.
- Các hành động agent được thực hiện thông qua `node.invoke` (ví dụ: `system.run`, `system.notify`, `canvas.*`).

### Node service + app IPC

- Một dịch vụ node host không có giao diện kết nối với Gateway WebSocket.
- `system.run` các yêu cầu được chuyển tiếp đến ứng dụng macOS qua một Unix socket cục bộ.
- Ứng dụng thực hiện exec trong ngữ cảnh UI, nhắc nếu cần, và trả về kết quả.

Sơ đồ (SCI):

```
Agent -> Gateway -> Node Service (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Mac App (UI + TCC + system.run)
```

### PeekabooBridge (UI automation)

- UI automation uses a separate UNIX socket named `bridge.sock` and the PeekabooBridge JSON protocol.
- Host preference order (client-side): Peekaboo.app → Claude.app → OpenClaw.app → local execution.
- Security: bridge hosts require an allowed TeamID; DEBUG-only same-UID escape hatch is guarded by `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (Peekaboo convention).
- See: [PeekabooBridge usage](/platforms/mac/peekaboo) for details.

## Operational flows

- Restart/rebuild: `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  - Kills existing instances
  - Swift build + package
  - Writes/bootstraps/kickstarts the LaunchAgent
- Single instance: app exits early if another instance with the same bundle ID is running.

## Hardening notes

- Prefer requiring a TeamID match for all privileged surfaces.
- PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (DEBUG-only) may allow same-UID callers for local development.
- All communication remains local-only; no network sockets are exposed.
- TCC prompts originate only from the GUI app bundle; keep the signed bundle ID stable across rebuilds.
- IPC hardening: socket mode `0600`, token, peer-UID checks, HMAC challenge/response, short TTL.