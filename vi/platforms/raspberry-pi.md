---
summary: OpenClaw trên Raspberry Pi (thiết lập tự lưu trữ với ngân sách hạn chế)
read_when:
  - Thiết lập OpenClaw trên Raspberry Pi
  - Chạy OpenClaw trên các thiết bị ARM
  - Xây dựng một AI cá nhân luôn hoạt động với chi phí thấp
title: Raspberry Pi
x-i18n:
  source_path: platforms\raspberry-pi.md
  source_hash: 90b143a2877a4cea162e04902b89d3b5e0c365331c1c3d62e4ec1c0dded0cf6d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:28.683Z'
---

# OpenClaw trên Raspberry Pi

## Mục tiêu

Chạy một Gateway OpenClaw liên tục, luôn bật trên Raspberry Pi với chi phí **~$35-80** một lần (không có phí hàng tháng).

Hoàn hảo cho:

- Trợ lý AI cá nhân 24/7
- Trung tâm tự động hóa nhà thông minh
- Bot Telegram/WhatsApp tiết kiệm điện năng, luôn khả dụng
## Yêu cầu phần cứng

| Mô hình Pi      | RAM     | Hoạt động? | Ghi chú                            |
| --------------- | ------- | ---------- | ---------------------------------- |
| **Pi 5**        | 4GB/8GB | ✅ Tốt nhất | Nhanh nhất, được khuyến nghị       |
| **Pi 4**        | 4GB     | ✅ Tốt    | Điểm cân bằng cho hầu hết người dùng |
| **Pi 4**        | 2GB     | ✅ OK     | Hoạt động, thêm swap               |
| **Pi 4**        | 1GB     | ⚠️ Chặt   | Có thể hoạt động với swap, cấu hình tối thiểu |
| **Pi 3B+**      | 1GB     | ⚠️ Chậm   | Hoạt động nhưng chậm              |
| **Pi Zero 2 W** | 512MB   | ❌        | Không được khuyến nghị             |

**Thông số tối thiểu:** 1GB RAM, 1 lõi, 500MB đĩa  
**Được khuyến nghị:** 2GB+ RAM, OS 64-bit, Thẻ SD 16GB+ (hoặc USB SSD)
## Những Gì Bạn Cần

- Raspberry Pi 4 hoặc 5 (khuyến nghị 2GB+)
- Thẻ MicroSD (16GB+) hoặc USB SSD (hiệu suất tốt hơn)
- Nguồn điện (khuyến nghị PSU chính thức của Pi)
- Kết nối mạng (Ethernet hoặc WiFi)
- ~30 phút
## 1) Cài đặt hệ điều hành

Sử dụng **Raspberry Pi OS Lite (64-bit)** — không cần desktop cho máy chủ không có màn hình.

1. Tải xuống [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Chọn hệ điều hành: **Raspberry Pi OS Lite (64-bit)**
3. Nhấp vào biểu tượng bánh răng (⚙️) để cấu hình trước:
   - Đặt tên máy chủ: `gateway-host`
   - Bật SSH
   - Đặt tên người dùng/mật khẩu
   - Cấu hình WiFi (nếu không sử dụng Ethernet)
4. Cài đặt vào thẻ SD / ổ USB
5. Chèn và khởi động Pi
## 2) Kết nối qua SSH

```bash
ssh user@gateway-host
# or use the IP address
ssh user@192.168.x.x
```
## 3) Thiết lập hệ thống

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y git curl build-essential

# Set timezone (important for cron/reminders)
sudo timedatectl set-timezone America/Chicago  # Change to your timezone
```
## 4) Cài đặt Node.js 22 (ARM64)

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # Should show v22.x.x
npm --version
```
## 5) Thêm Swap (Quan trọng cho 2GB trở xuống)

Swap ngăn chặn các sự cố hết bộ nhớ:

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Optimize for low RAM (reduce swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
## 6) Cài đặt OpenClaw

### Option A: Cài đặt tiêu chuẩn (Được khuyến nghị)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Option B: Hackable Install (For tinkering)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

Cài đặt có thể hack cho phép bạn truy cập trực tiếp vào nhật ký và mã — hữu ích để gỡ lỗi các vấn đề cụ thể cho ARM.
## 7) Chạy Thiết lập ban đầu

```bash
openclaw onboard --install-daemon
```

Làm theo trình hướng dẫn:

1. **Chế độ Gateway:** Local
2. **Xác thực:** Khóa API được khuyến nghị (OAuth có thể gặp vấn đề trên Pi headless)
3. **Kênh:** Telegram là lựa chọn dễ nhất để bắt đầu
4. **Daemon:** Có (systemd)
## 8) Xác minh Cài đặt

```bash
# Check status
openclaw status

# Check service
sudo systemctl status openclaw

# View logs
journalctl -u openclaw -f
```
## 9) Truy cập Bảng điều khiển

Vì Pi không có màn hình, hãy sử dụng SSH tunnel:

```bash
# From your laptop/desktop
ssh -L 18789:localhost:18789 user@gateway-host

# Then open in browser
open http://localhost:18789
```

Or use Tailscale for always-on access:

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Update config
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

---
## Tối ưu hóa hiệu suất

### Sử dụng USB SSD (Cải thiện đáng kể)

Thẻ SD chậm và dễ hỏng. USB SSD cải thiện hiệu suất một cách đáng kể:

```bash
# Check if booting from USB
lsblk
```

See [Pi USB boot guide](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) for setup.

### Reduce Memory Usage

```bash
# Disable GPU memory allocation (headless)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

### Monitor Resources

```bash
# Check memory
free -h

# Check CPU temperature
vcgencmd measure_temp

# Live monitoring
htop
```

---
## Ghi chú dành riêng cho ARM

### Tương thích nhị phân

Hầu hết các tính năng của OpenClaw hoạt động trên ARM64, nhưng một số nhị phân bên ngoài có thể cần bản dựng ARM:

| Công cụ            | Trạng thái ARM64 | Ghi chú                             |
| ------------------ | ---------------- | ----------------------------------- |
| Node.js            | ✅               | Hoạt động tốt                      |
| WhatsApp (Baileys) | ✅               | Pure JS, không có vấn đề           |
| Telegram           | ✅               | Pure JS, không có vấn đề           |
| gog (Gmail CLI)    | ⚠️               | Kiểm tra bản phát hành ARM         |
| Chromium (browser) | ✅               | `sudo apt install chromium-browser` |

Nếu một Skill không hoạt động, hãy kiểm tra xem nhị phân của nó có bản dựng ARM hay không. Nhiều công cụ Go/Rust có; một số thì không.

### 32-bit so với 64-bit

**Luôn sử dụng OS 64-bit.** Node.js và nhiều công cụ hiện đại yêu cầu điều này. Kiểm tra bằng:

```bash
uname -m
# Should show: aarch64 (64-bit) not armv7l (32-bit)
```

---
## Thiết lập Mô hình Được Khuyến nghị

Vì Pi chỉ là Gateway (các mô hình chạy trong cloud), hãy sử dụng các mô hình dựa trên API:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**Đừng cố gắng chạy các LLM cục bộ trên Pi** — ngay cả các mô hình nhỏ cũng quá chậm. Hãy để Claude/GPT thực hiện công việc nặng.

---
## Tự động khởi động khi bật máy

Trình hướng dẫn thiết lập ban đầu sẽ cấu hình điều này, nhưng để xác minh:

```bash
# Check service is enabled
sudo systemctl is-enabled openclaw

# Enable if not
sudo systemctl enable openclaw

# Start on boot
sudo systemctl start openclaw
```

---
## Khắc phục sự cố

### Hết bộ nhớ (OOM)

```bash
# Check memory
free -h

# Add more swap (see Step 5)
# Or reduce services running on the Pi
```

### Slow Performance

- Use USB SSD instead of SD card
- Disable unused services: `sudo systemctl disable cups bluetooth avahi-daemon`
- Check CPU throttling: `vcgencmd get_throttled` (should return `0x0`)

### Service Won't Start

```bash
# Check logs
journalctl -u openclaw --no-pager -n 100

# Common fix: rebuild
cd ~/openclaw  # if using hackable install
npm run build
sudo systemctl restart openclaw
```

### ARM Binary Issues

If a skill fails with "exec format error":

1. Check if the binary has an ARM64 build
2. Try building from source
3. Or use a Docker container with ARM support

### WiFi Drops

For headless Pis on WiFi:

```bash
# Disable WiFi power management
sudo iwconfig wlan0 power off

# Make permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

---
## So Sánh Chi Phí

| Thiết lập      | Chi Phí Một Lần | Chi Phí Hàng Tháng | Ghi chú                   |
| -------------- | --------------- | ------------------ | ------------------------- |
| **Pi 4 (2GB)** | ~$45            | $0                 | + điện (~$5/năm)          |
| **Pi 4 (4GB)** | ~$55            | $0                 | Được khuyến nghị          |
| **Pi 5 (4GB)** | ~$60            | $0                 | Hiệu suất tốt nhất        |
| **Pi 5 (8GB)** | ~$80            | $0                 | Quá mức nhưng bền vững    |
| DigitalOcean   | $0              | $6/tháng           | $72/năm                   |
| Hetzner        | $0              | €3.79/tháng        | ~$50/năm                  |

**Hòa vốn:** Một Pi tự trả cho chính nó trong ~6-12 tháng so với VPS đám mây.

---
## Xem thêm

- [Hướng dẫn Linux](/platforms/linux) — thiết lập Linux chung
- [Hướng dẫn DigitalOcean](/platforms/digitalocean) — giải pháp đám mây thay thế
- [Hướng dẫn Hetzner](/install/hetzner) — thiết lập Docker
- [Tailscale](/gateway/tailscale) — truy cập từ xa
- [Nodes](/nodes) — ghép nối máy tính xách tay/điện thoại của bạn với Gateway Pi