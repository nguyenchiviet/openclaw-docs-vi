---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw devices` (ghép nối thiết bị + xoay
  vòng/thu hồi token)
read_when:
  - Bạn đang phê duyệt các yêu cầu ghép nối thiết bị
  - Bạn cần xoay vòng hoặc thu hồi các token thiết bị
title: thiết bị
x-i18n:
  source_path: cli\devices.md
  source_hash: efcc88d20e64556eb06158a8fd5e5c785e9c31421c2a89353d136a6578dc1d1d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:39.063Z'
---

# `openclaw devices`

Quản lý các yêu cầu ghép nối thiết bị và các token có phạm vi thiết bị.
## Lệnh

### `openclaw devices list`

Liệt kê các yêu cầu ghép nối đang chờ xử lý và các thiết bị đã ghép nối.

```
openclaw devices list
openclaw devices list --json
```

### `openclaw devices remove <deviceId>`

Remove one paired device entry.

```
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### `openclaw devices clear --yes [--pending]`

Clear paired devices in bulk.

```
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### `openclaw devices approve [requestId] [--latest]`

Approve a pending device pairing request. If `requestId` is omitted, OpenClaw
automatically approves the most recent pending request.

```
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### `openclaw devices reject <requestId>`

Reject a pending device pairing request.

```
openclaw devices reject <requestId>
```

### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`

Rotate a device token for a specific role (optionally updating scopes).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `openclaw devices revoke --device <id> --role <role>`

Thu hồi mã thông báo thiết bị cho một vai trò cụ thể.
```
openclaw devices revoke --device <deviceId> --role node
```
## Các tùy chọn phổ biến

- `--url <url>`: URL Gateway WebSocket (mặc định là `gateway.remote.url` khi được cấu hình).
- `--token <token>`: Token Gateway (nếu cần thiết).
- `--password <password>`: Mật khẩu Gateway (xác thực bằng mật khẩu).
- `--timeout <ms>`: Hết thời gian chờ RPC.
- `--json`: Đầu ra JSON (được khuyến nghị cho kịch bản).

Lưu ý: khi bạn đặt `--url`, CLI không quay lại thông tin xác thực cấu hình hoặc môi trường.
Chuyển `--token` hoặc `--password` một cách rõ ràng. Thiếu thông tin xác thực rõ ràng là một lỗi.
## Ghi chú

- Xoay vòng token trả về một token mới (nhạy cảm). Hãy coi nó như một bí mật.
- Các lệnh này yêu cầu phạm vi `operator.pairing` (hoặc `operator.admin`).
- `devices clear` được cố ý giới hạn bởi `--yes`.
- Nếu phạm vi ghép nối không khả dụng trên local loopback (và không có `--url` được truyền rõ ràng), list/approve có thể sử dụng fallback ghép nối cục bộ.