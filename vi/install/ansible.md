---
summary: >-
  Cài đặt OpenClaw tự động hóa, được cứng hóa với Ansible, Tailscale VPN và cách
  ly tường lửa
read_when:
  - Bạn muốn triển khai máy chủ tự động với việc tăng cường bảo mật
  - Bạn cần thiết lập cách ly tường lửa với truy cập VPN
  - Bạn đang triển khai đến các máy chủ Debian/Ubuntu từ xa
title: Ansible
x-i18n:
  source_path: install\ansible.md
  source_hash: b1e1e1ea13bff37b22bc58dad4b15a2233c6492771403dff364c738504aa7159
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:07:35.211Z'
---

# Cài đặt Ansible

Cách được khuyến nghị để triển khai OpenClaw tới các máy chủ sản xuất là thông qua **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — một trình cài đặt tự động với kiến trúc ưu tiên bảo mật.
## Bắt đầu nhanh

Cài đặt bằng một lệnh:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Hướng dẫn đầy đủ: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Kho lưu trữ openclaw-ansible là nguồn thông tin chính thức cho triển khai Ansible. Trang này là một tổng quan nhanh.
## Những Gì Bạn Nhận Được

- 🔒 **Bảo mật ưu tiên tường lửa**: UFW + cách ly Docker (chỉ SSH + Tailscale có thể truy cập)
- 🔐 **Tailscale VPN**: Truy cập từ xa an toàn mà không phơi bày dịch vụ công khai
- 🐳 **Docker**: Các container sandbox cách ly, ràng buộc localhost
- 🛡️ **Bảo vệ theo chiều sâu**: Kiến trúc bảo mật 4 lớp
- 🚀 **Thiết lập một lệnh**: Triển khai hoàn chỉnh trong vài phút
- 🔧 **Tích hợp Systemd**: Tự động khởi động khi bật máy với cứng hóa
## Yêu cầu

- **OS**: Debian 11+ hoặc Ubuntu 20.04+
- **Access**: Quyền root hoặc sudo
- **Network**: Kết nối Internet để cài đặt gói
- **Ansible**: 2.14+ (được cài đặt tự động bởi tập lệnh bắt đầu nhanh)
## Những Gì Được Cài Đặt

Playbook Ansible cài đặt và cấu hình:

1. **Tailscale** (mesh VPN để truy cập từ xa an toàn)
2. **UFW firewall** (chỉ các cổng SSH + Tailscale)
3. **Docker CE + Compose V2** (cho các sandbox agent)
4. **Node.js 22.x + pnpm** (các phụ thuộc runtime)
5. **OpenClaw** (chạy trên host, không được container hóa)
6. **Dịch vụ Systemd** (tự động khởi động với bảo mật cứng hóa)

Lưu ý: Gateway chạy **trực tiếp trên host** (không trong Docker), nhưng các sandbox agent sử dụng Docker để cách ly. Xem [Sandboxing](/gateway/sandboxing) để biết chi tiết.
## Thiết lập sau khi cài đặt

Sau khi cài đặt hoàn tất, chuyển sang người dùng openclaw:

```bash
sudo -i -u openclaw
```

The post-install script will guide you through:

1. **Onboarding wizard**: Configure OpenClaw settings
2. **Provider login**: Connect WhatsApp/Telegram/Discord/Signal
3. **Gateway testing**: Verify the installation
4. **Tailscale setup**: Connect to your VPN mesh

### Quick commands

```bash
# Check service status
sudo systemctl status openclaw

# View live logs
sudo journalctl -u openclaw -f

# Restart gateway
sudo systemctl restart openclaw

# Provider login (run as openclaw user)
sudo -i -u openclaw
openclaw channels login
```
## Kiến trúc Bảo mật

### Phòng thủ 4 Lớp

1. **Tường lửa (UFW)**: Chỉ SSH (22) + Tailscale (41641/udp) được phơi bày công khai
2. **VPN (Tailscale)**: Gateway chỉ có thể truy cập qua mạng VPN mesh
3. **Docker Isolation**: Chuỗi iptables DOCKER-USER ngăn chặn phơi bày cổng bên ngoài
4. **Systemd Hardening**: NoNewPrivileges, PrivateTmp, người dùng không có đặc quyền

### Xác minh

Kiểm tra bề mặt tấn công bên ngoài:

```bash
nmap -p- YOUR_SERVER_IP
```

Chỉ nên hiển thị **cổng 22** (SSH) mở. Tất cả các dịch vụ khác (gateway, Docker) đều bị khóa.

### Tính khả dụng của Docker

Docker được cài đặt cho **agent sandboxes** (thực thi công cụ cô lập), không phải để chạy gateway. Gateway chỉ liên kết với localhost và có thể truy cập qua VPN Tailscale.

Xem [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) để cấu hình sandbox.
## Cài đặt Thủ công

Nếu bạn muốn kiểm soát thủ công quá trình tự động hóa:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Or run directly (then manually execute /tmp/openclaw-setup.sh after)
# ansible-playbook playbook.yml --ask-become-pass
```
## Cập nhật OpenClaw

Trình cài đặt Ansible thiết lập OpenClaw để cập nhật thủ công. Xem [Cập nhật](/install/updating) để biết quy trình cập nhật tiêu chuẩn.

Để chạy lại playbook Ansible (ví dụ: để thay đổi cấu hình):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Lưu ý: Điều này là idempotent và an toàn để chạy nhiều lần.
## Khắc phục sự cố

### Tường lửa chặn kết nối của tôi

Nếu bạn bị khóa:

- Đảm bảo bạn có thể truy cập qua Tailscale VPN trước
- Truy cập SSH (cổng 22) luôn được phép
- Gateway **chỉ** có thể truy cập được qua Tailscale theo thiết kế

### Dịch vụ không khởi động

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Test manual start
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker sandbox issues

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Provider login fails

Make sure you're running as the `openclaw` user:

```bash
sudo -i -u openclaw
openclaw channels login
```
## Cấu hình nâng cao

Để biết chi tiết về kiến trúc bảo mật và khắc phục sự cố:

- [Kiến trúc bảo mật](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Chi tiết kỹ thuật](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Hướng dẫn khắc phục sự cố](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)
## Liên quan

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — hướng dẫn triển khai đầy đủ
- [Docker](/install/docker) — thiết lập Gateway được đóng gói
- [Sandboxing](/gateway/sandboxing) — cấu hình sandbox agent
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — cách ly từng agent