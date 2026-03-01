---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw browser` (hồ sơ, tab, hành động, chuyển
  tiếp extension)
read_when:
  - Bạn sử dụng `openclaw browser` và muốn các ví dụ cho các tác vụ thông thường
  - >-
    Bạn muốn điều khiển trình duyệt đang chạy trên máy khác thông qua một node
    host
  - >-
    Bạn muốn sử dụng relay của tiện ích mở rộng Chrome (kết nối/ngắt kết nối
    thông qua nút thanh công cụ)
title: trình duyệt
x-i18n:
  source_path: cli\browser.md
  source_hash: af35adfd68726fd519c704d046451effd330458c2b8305e713137fb07b2571fd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:31:54.071Z'
---

# `openclaw browser`

Quản lý máy chủ điều khiển trình duyệt của OpenClaw và thực hiện các hành động trình duyệt (tab, ảnh chụp nhanh, ảnh chụp màn hình, điều hướng, nhấp chuột, gõ phím).

Liên quan:

- Công cụ trình duyệt + API: [Công cụ trình duyệt](/tools/browser)
- Tiếp sức tiện ích mở rộng Chrome: [Tiện ích mở rộng Chrome](/tools/chrome-extension)
## Cờ phổ biến

- `--url <gatewayWsUrl>`: URL WebSocket của Gateway (mặc định từ cấu hình).
- `--token <token>`: Token Gateway (nếu cần thiết).
- `--timeout <ms>`: thời gian chờ yêu cầu (ms).
- `--browser-profile <name>`: chọn hồ sơ trình duyệt (mặc định từ cấu hình).
- `--json`: đầu ra có thể đọc bằng máy (nơi được hỗ trợ).
## Bắt đầu nhanh (cục bộ)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```
## Profiles

Profiles là các cấu hình định tuyến trình duyệt có tên. Trong thực tế:

- `openclaw`: khởi chạy/kết nối với một phiên bản Chrome chuyên dụng được quản lý bởi OpenClaw (thư mục dữ liệu người dùng riêng biệt).
- `chrome`: điều khiển các tab Chrome hiện có của bạn thông qua relay extension Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Use a specific profile:

```bash
openclaw browser --browser-profile work tabs
```
## Tabs

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```
## Ảnh chụp nhanh / chụp màn hình / hành động

Ảnh chụp nhanh:

```bash
openclaw browser snapshot
```

Screenshot:

```bash
openclaw browser screenshot
```

Navigate/click/type (ref-based UI automation):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```
## Tiện ích mở rộng Chrome relay (đính kèm qua nút thanh công cụ)

Chế độ này cho phép agent điều khiển một tab Chrome hiện có mà bạn đính kèm thủ công (nó không tự động đính kèm).

Cài đặt tiện ích mở rộng chưa đóng gói vào đường dẫn ổn định:

```bash
openclaw browser extension install
openclaw browser extension path
```

Then Chrome → `chrome://extensions` → bật "Developer mode" → "Load unpacked" → chọn thư mục đã in.

Hướng dẫn đầy đủ: [Tiện ích mở rộng Chrome](/tools/chrome-extension)
## Điều khiển trình duyệt từ xa (proxy node host)

Nếu Gateway chạy trên máy khác với trình duyệt, hãy chạy một **node host** trên máy có Chrome/Brave/Edge/Chromium. Gateway sẽ proxy các hành động trình duyệt đến node đó (không cần máy chủ điều khiển trình duyệt riêng biệt).

Sử dụng `gateway.nodes.browser.mode` để điều khiển định tuyến tự động và `gateway.nodes.browser.node` để ghim một node cụ thể nếu có nhiều node được kết nối.

Bảo mật + thiết lập từ xa: [Công cụ trình duyệt](/tools/browser), [Truy cập từ xa](/gateway/remote), [Tailscale](/gateway/tailscale), [Bảo mật](/gateway/security)