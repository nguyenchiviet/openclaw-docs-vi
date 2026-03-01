---
summary: Panel Canvas được điều khiển bởi Agent nhúng qua WKWebView + custom URL scheme
read_when:
  - Triển khai bảng Canvas trên macOS
  - Thêm các điều khiển agent cho không gian làm việc trực quan
  - Gỡ lỗi tải canvas trong WKWebView
title: Canvas
x-i18n:
  source_path: platforms\mac\canvas.md
  source_hash: b6c71763d693264d943e570a852208cce69fc469976b2a1cdd9e39e2550534c1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:59.470Z'
---

# Canvas (ứng dụng macOS)

Ứng dụng macOS nhúng một **bảng Canvas** được điều khiển bởi agent bằng `WKWebView`. Đây là một không gian làm việc trực quan nhẹ cho HTML/CSS/JS, A2UI và các bề mặt giao diện người dùng tương tác nhỏ.
## Nơi Canvas được lưu trữ

Trạng thái Canvas được lưu trữ trong Application Support:

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Bảng Canvas phục vụ các tệp đó thông qua **lược đồ URL tùy chỉnh**:

- `openclaw-canvas://<session>/<path>`

Ví dụ:

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

Nếu không có `index.html` nào tồn tại ở gốc, ứng dụng sẽ hiển thị **trang scaffold tích hợp sẵn**.
## Hành vi của Panel

- Panel không viền, có thể thay đổi kích thước được neo gần thanh menu (hoặc con trỏ chuột).
- Ghi nhớ kích thước/vị trí cho mỗi phiên.
- Tự động tải lại khi các tệp canvas cục bộ thay đổi.
- Chỉ một panel Canvas hiển thị cùng một lúc (phiên được chuyển đổi khi cần).

Canvas có thể bị vô hiệu hóa từ Settings → **Allow Canvas**. Khi bị vô hiệu hóa, các lệnh canvas node trả về `CANVAS_DISABLED`.
## Bề mặt API của Agent

Canvas được hiển thị qua **Gateway WebSocket**, vì vậy agent có thể:

- hiển thị/ẩn bảng điều khiển
- điều hướng đến một đường dẫn hoặc URL
- đánh giá JavaScript
- chụp ảnh snapshot

Ví dụ CLI:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notes:

- `canvas.navigate` accepts **local canvas paths**, `http(s)` URLs, and `file://` URLs.
- If you pass `"/"`, the Canvas shows the local scaffold or `index.html`.
## A2UI trong Canvas

A2UI được lưu trữ bởi máy chủ canvas của Gateway và được hiển thị bên trong bảng điều khiển Canvas.
Khi Gateway quảng cáo một máy chủ Canvas, ứng dụng macOS sẽ tự động điều hướng đến
trang máy chủ A2UI khi mở lần đầu tiên.

URL máy chủ A2UI mặc định:

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### A2UI commands (v0.8)

Canvas currently accepts **A2UI v0.8** server→client messages:

- `beginRendering`
- `surfaceUpdate`
- `dataModelUpdate`
- `deleteSurface`

`createSurface` (v0.9) is not supported.

CLI example:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Quick smoke:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```
## Kích hoạt chạy agent từ Canvas

Canvas có thể kích hoạt các chạy agent mới thông qua deep links:

- `openclaw://agent?...`

Ví dụ (trong JS):

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

Ứng dụng sẽ yêu cầu xác nhận trừ khi cung cấp một khóa hợp lệ.
## Ghi chú bảo mật

- Canvas scheme chặn directory traversal; các tệp phải nằm dưới session root.
- Nội dung Canvas cục bộ sử dụng một scheme tùy chỉnh (không cần máy chủ local loopback).
- Các URL `http(s)` bên ngoài chỉ được phép khi được điều hướng rõ ràng.