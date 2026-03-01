---
summary: 'Ứng dụng iOS node: kết nối với Gateway, ghép nối, canvas và khắc phục sự cố'
read_when:
  - Ghép nối hoặc kết nối lại nút iOS
  - Chạy ứng dụng iOS từ mã nguồn
  - Gỡ lỗi khám phá gateway hoặc lệnh canvas
title: Ứng dụng iOS
x-i18n:
  source_path: platforms\ios.md
  source_hash: 1f81383ab4867a683c1deb213e23d6189fbe6445a4b8994e55938ad0b47bdc01
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:27.006Z'
---

# Ứng dụng iOS (Node)

Tính khả dụng: bản xem trước nội bộ. Ứng dụng iOS chưa được phân phối công khai.
## Nó làm gì

- Kết nối với Gateway qua WebSocket (LAN hoặc tailnet).
- Hiển thị các khả năng của node: Canvas, Chụp ảnh màn hình, Chụp camera, Vị trí, Chế độ nói chuyện, Đánh thức bằng giọng nói.
- Nhận các lệnh `node.invoke` và báo cáo các sự kiện trạng thái của node.
## Yêu cầu

- Gateway chạy trên một thiết bị khác (macOS, Linux, hoặc Windows qua WSL2).
- Đường dẫn mạng:
  - Cùng LAN qua Bonjour, **hoặc**
  - Tailnet qua unicast DNS-SD (ví dụ miền: `openclaw.internal.`), **hoặc**
  - Host/port thủ công (dự phòng).
## Bắt đầu nhanh (ghép cặp + kết nối)

1. Khởi động Gateway:

```bash
openclaw gateway --port 18789
```

2. In the iOS app, open Settings and pick a discovered gateway (or enable Manual Host and enter host/port).

3. Approve the pairing request on the gateway host:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. Verify connection:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```
## Đường dẫn khám phá thiết bị

### Bonjour (LAN)

Gateway quảng cáo `_openclaw-gw._tcp` trên `local.`. Ứng dụng iOS liệt kê những cái này tự động.

### Tailnet (cross-network)

Nếu mDNS bị chặn, hãy sử dụng vùng DNS-SD unicast (chọn một miền; ví dụ: `openclaw.internal.`) và Tailscale split DNS.
Xem [Bonjour](/gateway/bonjour) để xem ví dụ CoreDNS.

### Host/port thủ công

Trong Cài đặt, bật **Manual Host** và nhập host + port của gateway (mặc định `18789`).
## Canvas + A2UI

Node iOS hiển thị một canvas WKWebView. Sử dụng `node.invoke` để điều khiển nó:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

Notes:

- The Gateway canvas host serves `/__openclaw__/canvas/` and `/__openclaw__/a2ui/`.
- It is served from the Gateway HTTP server (same port as `gateway.port`, default `18789`).
- The iOS node auto-navigates to A2UI on connect when a canvas host URL is advertised.
- Return to the built-in scaffold with `canvas.navigate` and `{"url":""}`.

### Canvas eval / snapshot

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```
## Chế độ kích hoạt bằng giọng nói + nói chuyện

- Chế độ kích hoạt bằng giọng nói và nói chuyện có sẵn trong Cài đặt.
- iOS có thể tạm dừng âm thanh nền; hãy coi các tính năng giọng nói là nỗ lực tốt nhất khi ứng dụng không hoạt động.
## Lỗi thường gặp

- `NODE_BACKGROUND_UNAVAILABLE`: đưa ứng dụng iOS vào tiền cảnh (các lệnh canvas/camera/screen yêu cầu điều này).
- `A2UI_HOST_NOT_CONFIGURED`: Gateway không quảng cáo URL canvas host; kiểm tra `canvasHost` trong [Cấu hình Gateway](/gateway/configuration).
- Lời nhắc ghép nối không bao giờ xuất hiện: chạy `openclaw nodes pending` và phê duyệt thủ công.
- Kết nối lại không thành công sau khi cài đặt lại: mã thông báo ghép nối Keychain đã bị xóa; hãy ghép nối lại node.
## Tài liệu liên quan

- [Ghép nối](/gateway/pairing)
- [Khám phá thiết bị](/gateway/discovery)
- [Bonjour](/gateway/bonjour)