---
summary: >-
  Cách ứng dụng Mac nhúng Gateway WebChat và cách gỡ lỗi


  Nhúng Gateway WebChat trong ứng dụng Mac


  Ứng dụng Mac có thể nhúng Gateway WebChat bằng cách sử dụng WebView để tải
  giao diện web. Các bước chính bao gồm:


  1. **Khởi tạo WebView**: Tạo một WKWebView trong ứng dụng Mac để hiển thị giao
  diện WebChat

  2. **Cấu hình URL**: Trỏ WebView đến URL của Gateway WebChat

  3. **Thiết lập quyền**: Cấp các quyền cần thiết cho WebView (JavaScript,
  cookie, v.v.)

  4. **Xử lý giao tiếp**: Sử dụng message handler để giao tiếp giữa ứng dụng Mac
  và WebChat


  Gỡ lỗi Gateway WebChat trên Mac


  **Sử dụng Safari Developer Tools:**

  - Mở ứng dụng Mac

  - Trong Safari, chọn Develop > [Tên ứng dụng] > [Tên WebView]

  - Sử dụng Web Inspector để kiểm tra HTML, CSS, JavaScript


  **Kiểm tra Console:**

  - Xem các lỗi JavaScript và thông báo console

  - Kiểm tra các yêu cầu mạng (Network tab)


  **Bật chế độ Debug:**

  - Thêm `preferences.setValue(true, forKey:
  "WebKitDeveloperExtrasEnabledPreferenceKey")`

  - Kích hoạt right-click inspect element trên WebView


  **Kiểm tra Logs:**

  - Xem system logs để tìm lỗi liên quan đến WebView

  - Sử dụng Console.app để lọc logs của ứng dụng
read_when:
  - Gỡ lỗi chế độ xem WebChat trên Mac hoặc cổng loopback
title: WebChat
x-i18n:
  source_path: platforms\mac\webchat.md
  source_hash: 6e72893255fa01cafa1252c4d5accf76e87f9b7720158c14442b99b60753363e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:02.245Z'
---

# WebChat (ứng dụng macOS)

Ứng dụng thanh menu macOS nhúng giao diện WebChat dưới dạng chế độ xem SwiftUI gốc. Nó kết nối với Gateway và mặc định sử dụng **phiên chính** cho agent được chọn (với bộ chuyển đổi phiên cho các phiên khác).

- **Chế độ cục bộ**: kết nối trực tiếp đến WebSocket Gateway cục bộ.
- **Chế độ từ xa**: chuyển tiếp cổng điều khiển Gateway qua SSH và sử dụng đường hầm đó làm mặt phẳng dữ liệu.

## Khởi chạy & gỡ lỗi

- Thủ công: Lobster menu → "Open Chat".
- Tự động mở để kiểm tra:

  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```

- Logs: `./scripts/clawlog.sh` (subsystem `ai.openclaw`, category `WebChatSwiftUI`).

## How it’s wired

- Data plane: Gateway WS methods `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` and events `chat`, `agent`, `presence`, `tick`, `health`.
- Session: defaults to the primary session (`main`, or `global` khi phạm vi là global). Giao diện có thể chuyển đổi giữa các phiên.
- Thiết lập ban đầu sử dụng một phiên chuyên dụng để giữ thiết lập lần chạy đầu tiên riêng biệt.

## Bề mặt bảo mật

- Chế độ từ xa chỉ chuyển tiếp cổng điều khiển WebSocket Gateway qua SSH.

## Những hạn chế đã biết

- Giao diện được tối ưu hóa cho các phiên trò chuyện (không phải sandbox trình duyệt đầy đủ).