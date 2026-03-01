---
summary: Gateway singleton guard sử dụng WebSocket listener bind
read_when:
  - Chạy hoặc gỡ lỗi quy trình Gateway
  - Điều tra việc thực thi một phiên bản duy nhất
title: Khóa Gateway
x-i18n:
  source_path: gateway\gateway-lock.md
  source_hash: 15fdfa066d1925da8b4632073a876709f77ca8d40e6828c174a30d953ba4f8e9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:56:31.603Z'
---

# Khóa Gateway

Cập nhật lần cuối: 2025-12-11

## Lý do

- Đảm bảo chỉ một instance gateway chạy trên mỗi cổng cơ sở trên cùng một máy chủ; các gateway bổ sung phải sử dụng các profile cách ly và cổng duy nhất.
- Tồn tại sau các sự cố/SIGKILL mà không để lại các tệp khóa cũ.
- Thất bại nhanh chóng với thông báo lỗi rõ ràng khi cổng điều khiển đã bị chiếm dụng.

## Cơ chế

- Gateway liên kết trình nghe WebSocket (mặc định `ws://127.0.0.1:18789`) ngay lập tức khi khởi động bằng cách sử dụng một trình nghe TCP độc quyền.
- Nếu liên kết không thành công với `EADDRINUSE`, khởi động sẽ ném `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Hệ điều hành phát hành trình nghe tự động khi thoát khỏi bất kỳ quy trình nào, bao gồm các sự cố và SIGKILL—không cần tệp khóa riêng biệt hoặc bước dọn dẹp.
- Khi tắt, gateway đóng máy chủ WebSocket và máy chủ HTTP cơ bản để giải phóng cổng nhanh chóng.

## Bề mặt lỗi

- Nếu một quy trình khác giữ cổng, khởi động sẽ ném `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Các lỗi liên kết khác xuất hiện dưới dạng `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`.

## Ghi chú hoạt động

- Nếu cổng bị chiếm dụng bởi _một quy trình khác_, lỗi là như nhau; giải phóng cổng hoặc chọn cổng khác bằng `openclaw gateway --port <port>`.
- Ứng dụng macOS vẫn duy trì bảo vệ PID nhẹ riêng của nó trước khi tạo gateway; khóa thời gian chạy được thực thi bằng liên kết WebSocket.