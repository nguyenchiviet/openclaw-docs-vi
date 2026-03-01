---
summary: Loopback WebChat static host và Gateway WS sử dụng cho chat UI
read_when:
  - Gỡ lỗi hoặc cấu hình quyền truy cập WebChat
title: WebChat
x-i18n:
  source_path: web\webchat.md
  source_hash: 18739c0332e9a78e78d51275b3f5f55e267c9b11316a79bf38ac95b7d3f3bdd1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:34:45.820Z'
---

# WebChat (Gateway WebSocket UI)

Trạng thái: Giao diện chat SwiftUI trên macOS/iOS nói chuyện trực tiếp với Gateway WebSocket.

## Nó là gì

- Giao diện chat gốc cho gateway (không có trình duyệt nhúng và không có máy chủ tĩnh cục bộ).
- Sử dụng các phiên và quy tắc định tuyến giống như các kênh khác.
- Định tuyến xác định: các câu trả lời luôn quay lại WebChat.

## Bắt đầu nhanh

1. Khởi động gateway.
2. Mở giao diện WebChat (ứng dụng macOS/iOS) hoặc tab chat Control UI.
3. Đảm bảo xác thực gateway được cấu hình (bắt buộc theo mặc định, ngay cả trên local loopback).

## Cách hoạt động (hành vi)

- Giao diện kết nối với Gateway WebSocket và sử dụng `chat.history`, `chat.send`, và `chat.inject`.
- `chat.history` bị giới hạn để đảm bảo ổn định: Gateway có thể cắt ngắn các trường văn bản dài, bỏ qua siêu dữ liệu nặng và thay thế các mục quá lớn bằng `[chat.history omitted: message too large]`.
- `chat.inject` thêm một ghi chú trợ lý trực tiếp vào bảng ghi và phát nó đến giao diện (không chạy agent).
- Các lần chạy bị hủy có thể giữ đầu ra trợ lý một phần hiển thị trong giao diện.
- Gateway duy trì văn bản trợ lý một phần bị hủy vào lịch sử bảng ghi khi có đầu ra được lưu đệm, và đánh dấu các mục đó bằng siêu dữ liệu hủy.
- Lịch sử luôn được tìm nạp từ gateway (không có giám sát tệp cục bộ).
- Nếu gateway không thể truy cập được, WebChat ở chế độ chỉ đọc.

## Bảng công cụ agent Control UI

- Bảng Tools của Control UI `/agents` tìm nạp danh mục thời gian chạy qua `tools.catalog` và gắn nhãn cho mỗi công cụ là `core` hoặc `plugin:<id>` (cộng với `optional` cho các công cụ plugin tùy chọn).
- Nếu `tools.catalog` không khả dụng, bảng sẽ quay lại danh sách tĩnh được tích hợp sẵn.
- Bảng chỉnh sửa cấu hình hồ sơ và ghi đè, nhưng quyền truy cập thời gian chạy hiệu quả vẫn tuân theo ưu tiên chính sách (`allow`/`deny`, ghi đè cho mỗi agent và nhà cung cấp/kênh).

## Sử dụng từ xa

- Chế độ từ xa đường hầm Gateway WebSocket qua SSH/Tailscale.
- Bạn không cần chạy máy chủ WebChat riêng biệt.

## Tham chiếu cấu hình (WebChat)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn kênh:

- Không có khối `webchat.*` chuyên dụng. WebChat sử dụng điểm cuối gateway + cài đặt xác thực bên dưới.

Các tùy chọn toàn cục liên quan:

- `gateway.port`, `gateway.bind`: máy chủ/cổng WebSocket.
- `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: xác thực WebSocket (token/mật khẩu).
- `gateway.auth.mode: "trusted-proxy"`: xác thực reverse-proxy cho khách hàng trình duyệt (xem [Xác thực Proxy đáng tin cậy](/gateway/trusted-proxy-auth)).
- `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: mục tiêu gateway từ xa.
- `session.*`: lưu trữ phiên và mặc định khóa chính.