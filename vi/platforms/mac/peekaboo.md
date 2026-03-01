---
summary: Tích hợp PeekabooBridge cho tự động hóa giao diện macOS
read_when:
  - Lưu trữ PeekabooBridge trong OpenClaw.app
  - Tích hợp Peekaboo thông qua Swift Package Manager
  - Thay đổi giao thức/đường dẫn PeekabooBridge
title: Cầu Peekaboo
x-i18n:
  source_path: platforms\mac\peekaboo.md
  source_hash: b5b9ddb9a7c59e153a1d5d23c33616bb1542b5c7dadedc3af340aeee9ba03487
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:10.488Z'
---

# Peekaboo Bridge (tự động hóa giao diện macOS)

OpenClaw có thể lưu trữ **PeekabooBridge** như một broker tự động hóa giao diện cục bộ, nhận thức về quyền. Điều này cho phép `peekaboo` CLI điều khiển tự động hóa giao diện trong khi tái sử dụng các quyền TCC của ứng dụng macOS.

## Đây là gì (và không phải là gì)

- **Host**: OpenClaw.app có thể hoạt động như một host PeekabooBridge.
- **Client**: sử dụng `peekaboo` CLI (không có bề mặt `openclaw ui ...` riêng biệt).
- **UI**: các lớp phủ trực quan ở lại trong Peekaboo.app; OpenClaw là một host broker mỏng.

## Bật bridge

Trong ứng dụng macOS:

- Settings → **Enable Peekaboo Bridge**

Khi được bật, OpenClaw khởi động một máy chủ UNIX socket cục bộ. Nếu bị vô hiệu hóa, host sẽ dừng lại và `peekaboo` sẽ quay lại các host khác có sẵn.

## Thứ tự khám phá client

Các client Peekaboo thường cố gắng các host theo thứ tự này:

1. Peekaboo.app (UX đầy đủ)
2. Claude.app (nếu được cài đặt)
3. OpenClaw.app (broker mỏng)

Sử dụng `peekaboo bridge status --verbose` để xem host nào đang hoạt động và đường dẫn socket nào đang được sử dụng. Bạn có thể ghi đè bằng:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## Security & permissions

- The bridge validates **caller code signatures**; an allowlist of TeamIDs is
  enforced (Peekaboo host TeamID + OpenClaw app TeamID).
- Requests time out after ~10 seconds.
- If required permissions are missing, the bridge returns a clear error message
  rather than launching System Settings.

## Snapshot behavior (automation)

Snapshots are stored in memory and expire automatically after a short window.
If you need longer retention, re‑capture from the client.

## Troubleshooting

- If `peekaboo` reports “bridge client is not authorized”, ensure the client is
  properly signed or run the host with `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  chỉ ở chế độ **debug**.
- Nếu không tìm thấy host nào, hãy mở một trong các ứng dụng host (Peekaboo.app hoặc OpenClaw.app) và xác nhận rằng các quyền đã được cấp.