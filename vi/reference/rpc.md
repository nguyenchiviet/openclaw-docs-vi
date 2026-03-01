---
summary: >-
  Bộ điều hợp RPC cho các CLI bên ngoài (signal-cli, legacy imsg) và các mẫu
  gateway
read_when:
  - Thêm hoặc thay đổi các tích hợp CLI bên ngoài
  - 'Gỡ lỗi các bộ điều hợp RPC (signal-cli, imsg)'
title: Bộ Điều Hợp RPC
x-i18n:
  source_path: reference\rpc.md
  source_hash: 06dc6b97184cc704ba4ec4a9af90502f4316bcf717c3f4925676806d8b184c57
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:19:49.224Z'
---

# Bộ điều hợp RPC

OpenClaw tích hợp các CLI bên ngoài thông qua JSON-RPC. Hiện tại sử dụng hai mẫu.

## Mẫu A: HTTP daemon (signal-cli)

- `signal-cli` chạy như một daemon với JSON-RPC qua HTTP.
- Luồng sự kiện là SSE (`/api/v1/events`).
- Kiểm tra sức khỏe: `/api/v1/check`.
- OpenClaw sở hữu vòng đời khi `channels.signal.autoStart=true`.

Xem [Signal](/channels/signal) để biết cách thiết lập và các endpoint.

## Mẫu B: stdio child process (cũ: imsg)

> **Lưu ý:** Đối với các thiết lập iMessage mới, hãy sử dụng [BlueBubbles](/channels/bluebubbles) thay thế.

- OpenClaw sinh ra `imsg rpc` như một child process (tích hợp iMessage cũ).
- JSON-RPC được phân tách theo dòng qua stdin/stdout (một đối tượng JSON trên mỗi dòng).
- Không có cổng TCP, không cần daemon.

Các phương thức cốt lõi được sử dụng:

- `watch.subscribe` → thông báo (`method: "message"`)
- `watch.unsubscribe`
- `send`
- `chats.list` (kiểm tra/chẩn đoán)

Xem [iMessage](/channels/imessage) để biết cách thiết lập cũ và địa chỉ (`chat_id` được ưu tiên).

## Hướng dẫn bộ điều hợp

- Gateway sở hữu quy trình (bắt đầu/dừng gắn với vòng đời nhà cung cấp).
- Giữ các RPC client có khả năng phục hồi: timeout, khởi động lại khi thoát.
- Ưu tiên các ID ổn định (ví dụ: `chat_id`) hơn các chuỗi hiển thị.