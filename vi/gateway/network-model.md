---
summary: 'Cách Gateway, các node, và canvas host kết nối với nhau.'
read_when:
  - Bạn muốn có cái nhìn tổng quát về mô hình mạng của Gateway
title: Mô hình mạng
x-i18n:
  source_path: gateway\network-model.md
  source_hash: 9a93c40a18143b0fbe7e5b36480e238678820344de32e5292cd74742ee9605a7
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:58:16.838Z'
---

Hầu hết các hoạt động chảy qua Gateway (`openclaw gateway`), một quy trình chạy lâu dài duy nhất sở hữu các kết nối kênh và mặt phẳng điều khiển WebSocket.

## Các quy tắc cốt lõi

- Một Gateway trên mỗi máy chủ được khuyến nghị. Đó là quy trình duy nhất được phép sở hữu phiên WhatsApp Web. Đối với các bot cứu hộ hoặc cách ly nghiêm ngặt, hãy chạy nhiều gateway với các hồ sơ và cổng cách ly. Xem [Multiple gateways](/gateway/multiple-gateways).
- Local loopback trước tiên: Gateway WS mặc định là `ws://127.0.0.1:18789`. Trình hướng dẫn tạo một mã thông báo gateway theo mặc định, ngay cả đối với local loopback. Để truy cập tailnet, hãy chạy `openclaw gateway --bind tailnet --token ...` vì các mã thông báo được yêu cầu cho các ràng buộc không phải local loopback.
- Các node kết nối với Gateway WS qua LAN, tailnet hoặc SSH khi cần. Cầu TCP kế thừa đã bị loại bỏ.
- Canvas host được phục vụ bởi máy chủ HTTP Gateway trên **cùng cổng** với Gateway (mặc định `18789`):
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    Khi `gateway.auth` được cấu hình và Gateway ràng buộc vượt quá local loopback, các tuyến này được bảo vệ bởi xác thực Gateway. Các client node sử dụng các URL khả năng có phạm vi node được liên kết với phiên WS hoạt động của chúng. Xem [Gateway configuration](/gateway/configuration) (`canvasHost`, `gateway`).
- Sử dụng từ xa thường là SSH tunnel hoặc tailnet VPN. Xem [Remote access](/gateway/remote) và [Discovery](/gateway/discovery).