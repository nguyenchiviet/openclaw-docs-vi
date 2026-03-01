---
summary: Tài liệu tham khảo CLI cho `openclaw node` (máy chủ nút không giao diện)
read_when:
  - Chạy nút máy chủ không giao diện
  - Ghép nối một nút không phải macOS cho system.run
title: nút
x-i18n:
  source_path: cli\node.md
  source_hash: a8b1a57712663e2285c9ecd306fe57d067eb3e6820d7d8aec650b41b022d995a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:36:45.789Z'
---

# `openclaw node`

Chạy một **máy chủ node không giao diện** kết nối với Gateway WebSocket và hiển thị
`system.run` / `system.which` trên máy này.
## Tại sao sử dụng node host?

Sử dụng node host khi bạn muốn các agent **chạy lệnh trên các máy khác** trong mạng của bạn mà không cần cài đặt ứng dụng đi kèm macOS đầy đủ ở đó.

Các trường hợp sử dụng phổ biến:

- Chạy lệnh trên các hộp Linux/Windows từ xa (máy chủ xây dựng, máy phòng thí nghiệm, NAS).
- Giữ exec **sandboxed** trên gateway, nhưng ủy quyền các lần chạy được phê duyệt cho các host khác.
- Cung cấp mục tiêu thực thi nhẹ, không giao diện cho tự động hóa hoặc các node CI.

Thực thi vẫn được bảo vệ bởi **phê duyệt exec** và danh sách cho phép trên mỗi agent trên node host, vì vậy bạn có thể giữ quyền truy cập lệnh được xác định phạm vi và rõ ràng.
## Proxy trình duyệt (không cần cấu hình)

Node tự động quảng cáo một proxy trình duyệt nếu `browser.enabled` không bị vô hiệu hóa trên node. Điều này cho phép agent sử dụng tự động hóa trình duyệt trên node đó mà không cần cấu hình thêm.

Vô hiệu hóa nó trên node nếu cần:

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```
## Chạy (foreground)

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options:

- `--host <host>`: Gateway WebSocket host (default: `127.0.0.1`)
- `--port <port>`: Gateway WebSocket port (default: `18789`)
- `--tls`: Use TLS for the gateway connection
- `--tls-fingerprint <sha256>`: Expected TLS certificate fingerprint (sha256)
- `--node-id <id>`: Override node id (clears pairing token)
- `--display-name <name>``: Ghi đè tên hiển thị của node
## Service (background)

Cài đặt một node host không giao diện như một dịch vụ người dùng.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Options:

- `--host <host>`: Gateway WebSocket host (default: `127.0.0.1`)
- `--port <port>`: Gateway WebSocket port (default: `18789`)
- `--tls`: Use TLS for the gateway connection
- `--tls-fingerprint <sha256>`: Expected TLS certificate fingerprint (sha256)
- `--node-id <id>`: Override node id (clears pairing token)
- `--display-name <name>`: Override the node display name
- `--runtime <runtime>`: Service runtime (`node` or `bun`)
- `--force`: Reinstall/overwrite if already installed

Manage the service:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Use `openclaw node run` for a foreground node host (no service).

Service commands accept `--json` để có kết quả đầu ra có thể đọc được bởi máy.
## Ghép nối

Kết nối đầu tiên tạo một yêu cầu ghép nối node đang chờ xử lý trên Gateway.
Phê duyệt nó qua:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

The node host stores its node id, token, display name, and gateway connection info in
`~/.openclaw/node.json`.
## Phê duyệt thực thi

`system.run` được kiểm soát bởi phê duyệt thực thi cục bộ:

- `~/.openclaw/exec-approvals.json`
- [Phê duyệt thực thi](/tools/exec-approvals)
- `openclaw approvals --node <id|name|ip>` (chỉnh sửa từ Gateway)