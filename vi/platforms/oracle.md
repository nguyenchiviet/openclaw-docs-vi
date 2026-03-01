---
summary: OpenClaw trên Oracle Cloud (Always Free ARM)
read_when:
  - Thiết lập OpenClaw trên Oracle Cloud
  - Tìm kiếm hosting VPS chi phí thấp cho OpenClaw
  - Muốn chạy OpenClaw 24/7 trên một máy chủ nhỏ
title: Oracle Cloud
x-i18n:
  source_path: platforms\oracle.md
  source_hash: 8ec927ab5055c915fda464458f85bfb96151967c3b7cd1b1fd2b2f156110fc6d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:33.765Z'
---

# OpenClaw trên Oracle Cloud (OCI)

## Mục tiêu

Chạy một Gateway OpenClaw liên tục trên tầng ARM **Luôn miễn phí** của Oracle Cloud.

Tầng miễn phí của Oracle có thể phù hợp tốt với OpenClaw (đặc biệt nếu bạn đã có tài khoản OCI), nhưng nó có những điểm cân nhắc:

- Kiến trúc ARM (hầu hết mọi thứ hoạt động, nhưng một số tệp nhị phân có thể chỉ dành cho x86)
- Dung lượng và đăng ký có thể khó khăn
## So Sánh Chi Phí (2026)

| Nhà cung cấp | Gói            | Thông số kỹ thuật      | Giá/tháng | Ghi chú                |
| ------------ | --------------- | ---------------------- | -------- | --------------------- |
| Oracle Cloud | Always Free ARM | tối đa 4 OCPU, 24GB RAM | $0       | ARM, dung lượng hạn chế |
| Hetzner      | CX22            | 2 vCPU, 4GB RAM        | ~ $4     | Tùy chọn trả phí rẻ nhất  |
| DigitalOcean | Basic           | 1 vCPU, 1GB RAM        | $6       | Giao diện dễ sử dụng, tài liệu tốt    |
| Vultr        | Cloud Compute   | 1 vCPU, 1GB RAM        | $6       | Nhiều vị trí        |
| Linode       | Nanode          | 1 vCPU, 1GB RAM        | $5       | Hiện là một phần của Akamai    |

---
## Điều kiện tiên quyết

- Tài khoản Oracle Cloud ([đăng ký](https://www.oracle.com/cloud/free/)) — xem [hướng dẫn đăng ký cộng đồng](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) nếu bạn gặp sự cố
- Tài khoản Tailscale (miễn phí tại [tailscale.com](https://tailscale.com))
- ~30 phút
## 1) Tạo một Instance OCI

1. Đăng nhập vào [Oracle Cloud Console](https://cloud.oracle.com/)
2. Điều hướng đến **Compute → Instances → Create Instance**
3. Cấu hình:
   - **Name:** `openclaw`
   - **Image:** Ubuntu 24.04 (aarch64)
   - **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
   - **OCPUs:** 2 (hoặc tối đa 4)
   - **Memory:** 12 GB (hoặc tối đa 24 GB)
   - **Boot volume:** 50 GB (tối đa 200 GB miễn phí)
   - **SSH key:** Thêm khóa công khai của bạn
4. Nhấp vào **Create**
5. Ghi chú địa chỉ IP công khai

**Mẹo:** Nếu tạo instance thất bại với thông báo "Out of capacity", hãy thử một availability domain khác hoặc thử lại sau. Dung lượng free tier có hạn.
## 2) Kết nối và Cập nhật

```bash
# Connect via public IP
ssh ubuntu@YOUR_PUBLIC_IP

# Update system
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Note:** `build-essential` là bắt buộc để biên dịch ARM của một số phụ thuộc.
## 3) Cấu hình Người dùng và Tên máy chủ

```bash
# Set hostname
sudo hostnamectl set-hostname openclaw

# Set password for ubuntu user
sudo passwd ubuntu

# Enable lingering (keeps user services running after logout)
sudo loginctl enable-linger ubuntu
```
## 4) Cài đặt Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

This enables Tailscale SSH, so you can connect via `ssh openclaw` from any device on your tailnet — no public IP needed.

Verify:

```bash
tailscale status
```

**From now on, connect via Tailscale:** `ssh ubuntu@openclaw` (hoặc sử dụng IP Tailscale).
## 5) Cài đặt OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

When prompted "How do you want to hatch your bot?", select **"Do this later"**.

> Note: If you hit ARM-native build issues, start with system packages (e.g. `sudo apt install -y build-essential`) trước khi sử dụng Homebrew.
## 6) Cấu hình Gateway (local loopback + xác thực token) và bật Tailscale Serve

Sử dụng xác thực token làm mặc định. Nó có thể dự đoán được và tránh cần bất kỳ cờ Control UI "xác thực không an toàn" nào.

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Expose over Tailscale Serve (HTTPS + tailnet access)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```
## 7) Xác minh

```bash
# Check version
openclaw --version

# Check daemon status
systemctl --user status openclaw-gateway

# Check Tailscale Serve
tailscale serve status

# Test local response
curl http://localhost:18789
```
## 8) Khóa bảo mật VCN

Bây giờ mọi thứ đã hoạt động, hãy khóa VCN để chặn tất cả lưu lượng truy cập ngoại trừ Tailscale. Virtual Cloud Network của OCI hoạt động như một tường lửa ở cạnh mạng — lưu lượng truy cập bị chặn trước khi đến instance của bạn.

1. Đi tới **Networking → Virtual Cloud Networks** trong OCI Console
2. Nhấp vào VCN của bạn → **Security Lists** → Default Security List
3. **Xóa** tất cả các quy tắc ingress ngoại trừ:
   - `0.0.0.0/0 UDP 41641` (Tailscale)
4. Giữ các quy tắc egress mặc định (cho phép tất cả lưu lượng đi)

Điều này chặn SSH trên cổng 22, HTTP, HTTPS và mọi thứ khác ở cạnh mạng. Từ bây giờ, bạn chỉ có thể kết nối qua Tailscale.

---
## Truy cập Giao diện Điều khiển

Từ bất kỳ thiết bị nào trên mạng Tailscale của bạn:

```
https://openclaw.<tailnet-name>.ts.net/
```

Replace `<tailnet-name>` with your tailnet name (visible in `tailscale status`).

Không cần SSH tunnel. Tailscale cung cấp:

- Mã hóa HTTPS (chứng chỉ tự động)
- Xác thực thông qua danh tính Tailscale
- Truy cập từ bất kỳ thiết bị nào trên tailnet của bạn (laptop, điện thoại, v.v.)

---
## Bảo mật: VCN + Tailscale (đường cơ sở được khuyến nghị)

Với VCN bị khóa (chỉ mở UDP 41641) và Gateway được liên kết với local loopback, bạn sẽ có bảo vệ phòng chống sâu sắc mạnh mẽ: lưu lượng công khai bị chặn tại cạnh mạng, và quyền truy cập quản trị diễn ra qua tailnet của bạn.

Thiết lập này thường loại bỏ _nhu cầu_ về các quy tắc tường lửa dựa trên máy chủ bổ sung chỉ để ngăn chặn tấn công brute force SSH trên toàn Internet — nhưng bạn vẫn nên giữ hệ điều hành được cập nhật, chạy `openclaw security audit`, và xác minh rằng bạn không vô tình lắng nghe trên các giao diện công khai.

### Những Gì Đã Được Bảo Vệ

| Bước truyền thống  | Cần thiết?  | Lý do                                                                        |
| ------------------ | ----------- | ---------------------------------------------------------------------------- |
| Tường lửa UFW      | Không       | VCN chặn trước khi lưu lượng đến instance                                    |
| fail2ban           | Không       | Không có brute force nếu cổng 22 bị chặn tại VCN                             |
| Cứng hóa sshd      | Không       | Tailscale SSH không sử dụng sshd                                             |
| Vô hiệu hóa đăng nhập root | Không | Tailscale sử dụng danh tính Tailscale, không phải người dùng hệ thống        |
| Xác thực chỉ khóa SSH | Không    | Tailscale xác thực qua tailnet của bạn                                       |
| Cứng hóa IPv6      | Thường không | Tùy thuộc vào cài đặt VCN/subnet của bạn; xác minh những gì thực sự được gán/tiếp xúc |

### Vẫn Được Khuyến Nghị

- **Quyền thông tin xác thực:** `chmod 700 ~/.openclaw`
- **Kiểm toán bảo mật:** `openclaw security audit`
- **Cập nhật hệ thống:** `sudo apt update && sudo apt upgrade` thường xuyên
- **Giám sát Tailscale:** Xem lại các thiết bị trong [bảng điều khiển quản trị Tailscale](https://login.tailscale.com/admin)

### Xác Minh Tư Thế Bảo Mật

```bash
# Confirm no public ports listening
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Verify Tailscale SSH is active
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# Optional: disable sshd entirely
sudo systemctl disable --now ssh
```

---
## Dự phòng: SSH Tunnel

Nếu Tailscale Serve không hoạt động, hãy sử dụng SSH tunnel:

```bash
# From your local machine (via Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Then open `http://localhost:18789`.

---
## Khắc phục sự cố

### Tạo instance thất bại ("Hết dung lượng")

Các instance ARM miễn phí rất phổ biến. Hãy thử:

- Miền khả dụng khác
- Thử lại vào giờ không cao điểm (sáng sớm)
- Sử dụng bộ lọc "Always Free" khi chọn shape

### Tailscale không kết nối

```bash
# Check status
sudo tailscale status

# Re-authenticate
sudo tailscale up --ssh --hostname=openclaw --reset
```

### Gateway won't start

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### Can't reach Control UI

```bash
# Verify Tailscale Serve is running
tailscale serve status

# Check gateway is listening
curl http://localhost:18789

# Restart if needed
systemctl --user restart openclaw-gateway
```

### ARM binary issues

Some tools may not have ARM builds. Check:

```bash
uname -m  # Should show aarch64
```

Most npm packages work fine. For binaries, look for `linux-arm64` or `aarch64` releases.

---
## Tính bền vững

Tất cả trạng thái được lưu trữ trong:

- `~/.openclaw/` — cấu hình, thông tin xác thực, dữ liệu phiên
- `~/.openclaw/workspace/` — không gian làm việc (SOUL.md, bộ nhớ, tạo phẩm)

Sao lưu định kỳ:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

---
## Xem thêm

- [Truy cập từ xa Gateway](/gateway/remote) — các mẫu truy cập từ xa khác
- [Tích hợp Tailscale](/gateway/tailscale) — tài liệu Tailscale đầy đủ
- [Cấu hình Gateway](/gateway/configuration) — tất cả các tùy chọn cấu hình
- [Hướng dẫn DigitalOcean](/platforms/digitalocean) — nếu bạn muốn dùng dịch vụ trả phí + đăng ký dễ dàng hơn
- [Hướng dẫn Hetzner](/install/hetzner) — giải pháp thay thế dựa trên Docker