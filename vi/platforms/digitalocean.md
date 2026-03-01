---
summary: OpenClaw trên DigitalOcean (tùy chọn VPS trả phí đơn giản)
read_when:
  - Thiết lập OpenClaw trên DigitalOcean
  - Tìm kiếm hosting VPS giá rẻ cho OpenClaw
title: DigitalOcean
x-i18n:
  source_path: platforms\digitalocean.md
  source_hash: a927c4d61f30b94db1c624ccebfc950f7050ab1b425efc60542f0bc4c629af8b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:37.367Z'
---

# OpenClaw trên DigitalOcean

## Mục tiêu

Chạy một Gateway OpenClaw liên tục trên DigitalOcean với giá **$6/tháng** (hoặc $4/tháng với giá dành riêng).

Nếu bạn muốn tùy chọn $0/tháng và không phiền với ARM + thiết lập dành riêng cho nhà cung cấp, hãy xem [hướng dẫn Oracle Cloud](/platforms/oracle).
## So sánh Chi phí (2026)

| Nhà cung cấp | Gói            | Thông số kỹ thuật      | Giá/tháng   | Ghi chú                               |
| ------------ | --------------- | ---------------------- | ----------- | ------------------------------------- |
| Oracle Cloud | Always Free ARM | tối đa 4 OCPU, 24GB RAM | $0          | ARM, dung lượng hạn chế / vấn đề đăng ký |
| Hetzner      | CX22            | 2 vCPU, 4GB RAM        | €3.79 (~$4) | Tùy chọn trả phí rẻ nhất              |
| DigitalOcean | Basic           | 1 vCPU, 1GB RAM        | $6          | Giao diện dễ sử dụng, tài liệu tốt    |
| Vultr        | Cloud Compute   | 1 vCPU, 1GB RAM        | $6          | Nhiều vị trí                          |
| Linode       | Nanode          | 1 vCPU, 1GB RAM        | $5          | Hiện là một phần của Akamai           |

**Chọn nhà cung cấp:**

- DigitalOcean: giao diện đơn giản nhất + thiết lập dễ dự đoán (hướng dẫn này)
- Hetzner: tỷ lệ giá/hiệu suất tốt (xem [hướng dẫn Hetzner](/install/hetzner))
- Oracle Cloud: có thể $0/tháng, nhưng khó sử dụng hơn và chỉ ARM (xem [hướng dẫn Oracle](/platforms/oracle))

---
## Điều kiện tiên quyết

- Tài khoản DigitalOcean ([đăng ký với $200 tín dụng miễn phí](https://m.do.co/c/signup))
- Cặp khóa SSH (hoặc sẵn sàng sử dụng xác thực mật khẩu)
- ~20 phút
## 1) Tạo một Droplet

<Warning>
Sử dụng một hình ảnh cơ sở sạch (Ubuntu 24.04 LTS). Tránh các hình ảnh Marketplace 1-click của bên thứ ba trừ khi bạn đã xem xét các tập lệnh khởi động và cài đặt tường lửa mặc định của chúng.
</Warning>

1. Đăng nhập vào [DigitalOcean](https://cloud.digitalocean.com/)
2. Nhấp vào **Create → Droplets**
3. Chọn:
   - **Region:** Gần nhất với bạn (hoặc người dùng của bạn)
   - **Image:** Ubuntu 24.04 LTS
   - **Size:** Basic → Regular → **$6/mo** (1 vCPU, 1GB RAM, 25GB SSD)
   - **Authentication:** SSH key (được khuyến nghị) hoặc mật khẩu
4. Nhấp vào **Create Droplet**
5. Ghi chú địa chỉ IP
## 2) Kết nối qua SSH

```bash
ssh root@YOUR_DROPLET_IP
```
## 3) Cài đặt OpenClaw

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verify
openclaw --version
```
## 4) Chạy Thiết lập ban đầu

```bash
openclaw onboard --install-daemon
```

Trình hướng dẫn sẽ hướng dẫn bạn qua:

- Xác thực mô hình (khóa API hoặc OAuth)
- Thiết lập kênh (Telegram, WhatsApp, Discord, v.v.)
- Token Gateway (được tạo tự động)
- Cài đặt Daemon (systemd)
## 5) Xác minh Gateway

```bash
# Check status
openclaw status

# Check service
systemctl --user status openclaw-gateway.service

# View logs
journalctl --user -u openclaw-gateway.service -f
```
## 6) Truy cập Bảng điều khiển

Gateway liên kết với local loopback theo mặc định. Để truy cập Control UI:

**Tùy chọn A: SSH Tunnel (được khuyến nghị)**

```bash
# From your local machine
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Then open: http://localhost:18789
```

**Option B: Tailscale Serve (HTTPS, loopback-only)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Open: `https://<magicdns>/`

Notes:

- Serve keeps the Gateway loopback-only and authenticates Control UI/WebSocket traffic via Tailscale identity headers (tokenless auth assumes trusted gateway host; HTTP APIs still require token/password).
- To require token/password instead, set `gateway.auth.allowTailscale: false` or use `gateway.auth.mode: "password"`.

**Option C: Tailnet bind (no Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Open: `http://<tailscale-ip>:18789` (yêu cầu token).
## 7) Kết nối các kênh của bạn

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Scan QR code
```

Xem [Kênh](/channels) để biết các nhà cung cấp khác.

---
## Tối ưu hóa cho RAM 1GB

Droplet $6 chỉ có 1GB RAM. Để mọi thứ chạy mượt mà:

### Thêm swap (được khuyến nghị)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Use a lighter model

If you're hitting OOMs, consider:

- Using API-based models (Claude, GPT) instead of local models
- Setting `agents.defaults.model.primary` to a smaller model

### Monitor memory

```bash
free -h
htop
```

---
## Tính bền vững

Tất cả trạng thái nằm trong:

- `~/.openclaw/` — cấu hình, thông tin xác thực, dữ liệu phiên
- `~/.openclaw/workspace/` — không gian làm việc (SOUL.md, bộ nhớ, v.v.)

Những thứ này tồn tại qua các lần khởi động lại. Sao lưu chúng định kỳ:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

---
## Giải pháp thay thế Oracle Cloud miễn phí

Oracle Cloud cung cấp các instance ARM **Luôn miễn phí** mạnh mẽ hơn đáng kể so với bất kỳ tùy chọn trả phí nào ở đây — với giá $0/tháng.

| Những gì bạn nhận được | Thông số kỹ thuật      |
| ---------------------- | ---------------------- |
| **4 OCPUs**            | ARM Ampere A1          |
| **24GB RAM**           | Đủ và hơn thế nữa      |
| **200GB bộ nhớ**       | Khối lưu trữ           |
| **Miễn phí mãi mãi**   | Không tính phí thẻ tín dụng |

**Lưu ý:**

- Đăng ký có thể gặp khó khăn (thử lại nếu thất bại)
- Kiến trúc ARM — hầu hết mọi thứ hoạt động, nhưng một số tệp nhị phân cần bản dựng ARM

Để xem hướng dẫn thiết lập đầy đủ, hãy xem [Oracle Cloud](/platforms/oracle). Để biết các mẹo đăng ký và khắc phục sự cố quá trình đăng ký, hãy xem [hướng dẫn cộng đồng](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) này.

---
## Khắc phục sự cố

### Gateway không khởi động

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### Port already in use

```bash
lsof -i :18789
kill <PID>
```

### Out of memory

```bash
# Check memory
free -h

# Add more swap
# Or upgrade to $12/mo droplet (2GB RAM)
```

---
## Xem thêm

- [Hướng dẫn Hetzner](/install/hetzner) — rẻ hơn, mạnh hơn
- [Cài đặt Docker](/install/docker) — thiết lập được đóng gói
- [Tailscale](/gateway/tailscale) — truy cập từ xa an toàn
- [Cấu hình](/gateway/configuration) — tham chiếu cấu hình đầy đủ