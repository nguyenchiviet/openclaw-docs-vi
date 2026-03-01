---
summary: Cách ứng dụng macOS báo cáo trạng thái sức khỏe của gateway/Baileys
read_when:
  - Chỉ báo sức khỏe ứng dụng Mac
title: Kiểm Tra Sức Khỏe
x-i18n:
  source_path: platforms\mac\health.md
  source_hash: 0560e96501ddf53a499f8960cfcf11c2622fcb9056bfd1bcc57876e955cab03d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:48.022Z'
---

# Kiểm tra Sức khỏe trên macOS

Cách xem liệu kênh được liên kết có khỏe mạnh từ ứng dụng thanh menu.

## Thanh menu

- Chấm trạng thái hiện phản ánh sức khỏe Baileys:
  - Xanh lá: được liên kết + socket mở gần đây.
  - Cam: đang kết nối/thử lại.
  - Đỏ: đã đăng xuất hoặc kiểm tra thất bại.
- Dòng thứ hai hiển thị "linked · auth 12m" hoặc hiển thị lý do lỗi.
- Mục menu "Run Health Check" kích hoạt một kiểm tra theo yêu cầu.

## Cài đặt

- Tab Chung có thêm thẻ Sức khỏe hiển thị: tuổi auth được liên kết, đường dẫn/số lượng session-store, thời gian kiểm tra cuối cùng, mã lỗi/trạng thái cuối cùng và các nút cho Chạy Kiểm tra Sức khỏe / Tiết lộ Nhật ký.
- Sử dụng ảnh chụp được lưu trong bộ nhớ cache để giao diện tải ngay lập tức và quay lại một cách duyên dáng khi ngoại tuyến.
- **Tab Kênh** hiển thị trạng thái kênh + các điều khiển cho WhatsApp/Telegram (đăng nhập QR, đăng xuất, kiểm tra, lần ngắt kết nối/lỗi cuối cùng).

## Cách kiểm tra hoạt động

- Ứng dụng chạy `openclaw health --json` qua `ShellExecutor` khoảng ~60s và theo yêu cầu. Kiểm tra tải thông tin xác thực và báo cáo trạng thái mà không gửi tin nhắn.
- Lưu ảnh chụp tốt cuối cùng và lỗi cuối cùng riêng biệt để tránh nhấp nháy; hiển thị dấu thời gian của mỗi cái.

## Khi có nghi ngờ

- Bạn vẫn có thể sử dụng luồng CLI trong [Gateway health](/gateway/health) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) và tail `/tmp/openclaw/openclaw-*.log` cho `web-heartbeat` / `web-reconnect`.