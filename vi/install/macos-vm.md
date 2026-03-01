---
summary: >-
  Chạy OpenClaw trong một VM macOS được cách ly (cục bộ hoặc được lưu trữ) khi
  bạn cần cách ly hoặc iMessage
read_when:
  - Bạn muốn OpenClaw được cách ly khỏi môi trường macOS chính của mình
  - Bạn muốn tích hợp iMessage (BlueBubbles) trong một sandbox
  - Bạn muốn một môi trường macOS có thể đặt lại được mà bạn có thể sao chép
  - Bạn muốn so sánh các tùy chọn macOS VM cục bộ so với được lưu trữ
title: Máy ảo macOS
x-i18n:
  source_path: install\macos-vm.md
  source_hash: 4d1c85a5e4945f9f0796038cd5960edecb71ec4dffb6f9686be50adb75180716
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:09:19.739Z'
---

# OpenClaw trên macOS VMs (Sandboxing)

## Khuyến nghị mặc định (hầu hết người dùng)

- **VPS Linux nhỏ** cho Gateway luôn hoạt động và chi phí thấp. Xem [VPS hosting](/vps).
- **Phần cứng chuyên dụng** (Mac mini hoặc Linux box) nếu bạn muốn kiểm soát toàn bộ và **IP dân cư** cho tự động hóa trình duyệt. Nhiều trang web chặn IP trung tâm dữ liệu, vì vậy duyệt web cục bộ thường hoạt động tốt hơn.
- **Kết hợp:** giữ Gateway trên VPS rẻ tiền và kết nối Mac của bạn như một **node** khi bạn cần tự động hóa trình duyệt/UI. Xem [Nodes](/nodes) và [Gateway remote](/gateway/remote).

Sử dụng macOS VM khi bạn cụ thể cần các khả năng chỉ dành cho macOS (iMessage/BlueBubbles) hoặc muốn cách ly nghiêm ngặt khỏi Mac hàng ngày của bạn.
## Tùy chọn macOS VM

### Local VM trên Mac Apple Silicon của bạn (Lume)

Chạy OpenClaw trong một sandbox macOS VM trên Mac Apple Silicon hiện có của bạn bằng cách sử dụng [Lume](https://cua.ai/docs/lume).

Điều này cung cấp cho bạn:

- Môi trường macOS đầy đủ trong isolation (máy chủ của bạn vẫn sạch sẽ)
- Hỗ trợ iMessage thông qua BlueBubbles (không thể trên Linux/Windows)
- Reset tức thì bằng cách sao chép VMs
- Không có chi phí phần cứng bổ sung hoặc đám mây

### Nhà cung cấp Mac được lưu trữ (cloud)

Nếu bạn muốn macOS trong cloud, các nhà cung cấp Mac được lưu trữ cũng hoạt động:

- [MacStadium](https://www.macstadium.com/) (Macs được lưu trữ)
- Các nhà cung cấp Mac được lưu trữ khác cũng hoạt động; hãy làm theo tài liệu VM + SSH của họ

Khi bạn có quyền truy cập SSH vào một macOS VM, tiếp tục ở bước 6 dưới đây.

---
## Bắt đầu nhanh (Lume, người dùng có kinh nghiệm)

1. Cài đặt Lume
2. `lume create openclaw --os macos --ipsw latest`
3. Hoàn thành Trình hướng dẫn Thiết lập, bật Đăng nhập từ xa (SSH)
4. `lume run openclaw --no-display`
5. SSH vào, cài đặt OpenClaw, cấu hình các kênh
6. Xong

---
## Những gì bạn cần (Lume)

- Mac Apple Silicon (M1/M2/M3/M4)
- macOS Sequoia hoặc mới hơn trên máy chủ
- ~60 GB dung lượng đĩa trống cho mỗi VM
- ~20 phút

---
## 1) Cài đặt Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

If `~/.local/bin` isn't in your PATH:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Verify:

```bash
lume --version
```

Tài liệu: [Cài đặt Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

---
## 2) Tạo macOS VM

```bash
lume create openclaw --os macos --ipsw latest
```

Điều này tải xuống macOS và tạo VM. Một cửa sổ VNC sẽ tự động mở ra.

Lưu ý: Quá trình tải xuống có thể mất một lúc tùy thuộc vào kết nối của bạn.

---
## 3) Trình hướng dẫn thiết lập hoàn chỉnh

Trong cửa sổ VNC:

1. Chọn ngôn ngữ và khu vực
2. Bỏ qua Apple ID (hoặc đăng nhập nếu bạn muốn iMessage sau này)
3. Tạo tài khoản người dùng (ghi nhớ tên người dùng và mật khẩu)
4. Bỏ qua tất cả các tính năng tùy chọn

Sau khi thiết lập hoàn tất, bật SSH:

1. Mở System Settings → General → Sharing
2. Bật "Remote Login"

---
## 4) Lấy địa chỉ IP của VM

```bash
lume get openclaw
```

Look for the IP address (usually `192.168.64.x`).

---
## 5) SSH vào VM

```bash
ssh youruser@192.168.64.X
```

Replace `youruser` với tài khoản bạn đã tạo, và IP với địa chỉ IP của VM của bạn.

---
## 6) Cài đặt OpenClaw

Bên trong VM:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Làm theo các lời nhắc thiết lập ban đầu để cấu hình nhà cung cấp mô hình của bạn (Anthropic, OpenAI, v.v.).

---
## 7) Cấu hình kênh

Chỉnh sửa tệp cấu hình:

```bash
nano ~/.openclaw/openclaw.json
```

Add your channels:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Then login to WhatsApp (scan QR):

```bash
openclaw channels login
```

---
## 8) Chạy VM ở chế độ không có giao diện

Dừng VM và khởi động lại mà không có màn hình:

```bash
lume stop openclaw
lume run openclaw --no-display
```

The VM runs in the background. OpenClaw's daemon keeps the gateway running.

To check status:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

---
## Bonus: Tích hợp iMessage

Đây là tính năng nổi bật khi chạy trên macOS. Sử dụng [BlueBubbles](https://bluebubbles.app) để thêm iMessage vào OpenClaw.

Bên trong VM:

1. Tải BlueBubbles từ bluebubbles.app
2. Đăng nhập bằng Apple ID của bạn
3. Bật Web API và đặt mật khẩu
4. Trỏ webhook BlueBubbles đến gateway của bạn (ví dụ: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

Thêm vào cấu hình OpenClaw của bạn:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Khởi động lại gateway. Bây giờ agent của bạn có thể gửi và nhận iMessages.

Chi tiết thiết lập đầy đủ: [Kênh BlueBubbles](/channels/bluebubbles)

---
## Lưu một hình ảnh vàng

Trước khi tùy chỉnh thêm, hãy chụp ảnh trạng thái sạch của bạn:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Reset anytime:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

---
## Chạy 24/7

Giữ VM chạy bằng cách:

- Giữ Mac của bạn được cắm vào nguồn
- Vô hiệu hóa chế độ ngủ trong System Settings → Energy Saver
- Sử dụng `caffeinate` nếu cần

Để luôn bật thực sự, hãy cân nhắc sử dụng Mac mini chuyên dụng hoặc VPS nhỏ. Xem [Lưu trữ VPS](/vps).

---
## Khắc phục sự cố

| Vấn đề                  | Giải pháp                                                                           |
| ------------------------ | ---------------------------------------------------------------------------------- |
| Không thể SSH vào VM        | Kiểm tra "Remote Login" được bật trong System Settings của VM                            |
| IP của VM không hiển thị        | Chờ VM khởi động hoàn toàn, chạy `lume get openclaw` lại                           |
| Lume command không tìm thấy   | Thêm `~/.local/bin` vào PATH của bạn                                                    |
| WhatsApp QR không quét được | Đảm bảo bạn đã đăng nhập vào VM (không phải host) khi chạy `openclaw channels login` |

---
## Tài liệu liên quan

- [Lưu trữ VPS](/vps)
- [Nodes](/nodes)
- [Gateway từ xa](/gateway/remote)
- [Kênh BlueBubbles](/channels/bluebubbles)
- [Bắt đầu nhanh Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
- [Tham chiếu CLI Lume](https://cua.ai/docs/lume/reference/cli-reference)
- [Thiết lập VM không giám sát](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (nâng cao)
- [Docker Sandboxing](/install/docker) (phương pháp cách ly thay thế)