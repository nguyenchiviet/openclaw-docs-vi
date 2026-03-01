---
summary: Chạy OpenClaw Gateway trên exe.dev (VM + HTTPS proxy) để truy cập từ xa
read_when:
  - Bạn muốn một máy chủ Linux luôn bật với giá rẻ cho Gateway
  - Bạn muốn truy cập điều khiển UI từ xa mà không cần chạy VPS của riêng mình
title: exe.dev
x-i18n:
  source_path: install\exe-dev.md
  source_hash: 3c90f57e37145333429328477a3e12306586aa53283127daec75e065dbb85e39
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:07:27.547Z'
---

# exe.dev

Mục tiêu: OpenClaw Gateway chạy trên một VM exe.dev, có thể truy cập từ laptop của bạn qua: `https://<vm-name>.exe.xyz`

Trang này giả định image **exeuntu** mặc định của exe.dev. Nếu bạn chọn một distro khác, hãy ánh xạ các gói tương ứng.
## Đường dẫn nhanh cho người mới bắt đầu

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. Điền khóa xác thực/token của bạn khi cần
3. Nhấp vào "Agent" bên cạnh VM của bạn và chờ...
4. ???
5. Lợi nhuận
## Những gì bạn cần

- Tài khoản exe.dev
- `ssh exe.dev` truy cập vào các máy ảo [exe.dev](https://exe.dev) (tùy chọn)
## Cài đặt tự động với Shelley

Shelley, agent của [exe.dev](https://exe.dev), có thể cài đặt OpenClaw ngay lập tức với prompt của chúng tôi.
Prompt được sử dụng như sau:

```
Set up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by "openclaw devices list" and "openclaw devices approve <request id>". Make sure the dashboard shows that OpenClaw's health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final "reachable" should be <vm-name>.exe.xyz, without port specification.
```
## Cài đặt thủ công

## 1) Tạo máy ảo

Từ thiết bị của bạn:

```bash
ssh exe.dev new
```

Then connect:

```bash
ssh <vm-name>.exe.xyz
```

Tip: keep this VM **stateful**. OpenClaw stores state under `~/.openclaw/` and `~/.openclaw/workspace/`.
## 2) Cài đặt các điều kiện tiên quyết (trên VM)

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```
## 3) Cài đặt OpenClaw

Chạy tập lệnh cài đặt OpenClaw:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```
## 4) Thiết lập nginx để proxy OpenClaw tới cổng 8000

Chỉnh sửa `/etc/nginx/sites-enabled/default` với

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout settings for long-lived connections
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```
## 5) Truy cập OpenClaw và cấp quyền

Truy cập `https://<vm-name>.exe.xyz/` (xem kết quả Control UI từ thiết lập ban đầu). Nếu nó yêu cầu xác thực, dán token từ `gateway.auth.token` trên VM (lấy bằng `openclaw config get gateway.auth.token`, hoặc tạo một token mới bằng `openclaw doctor --generate-gateway-token`). Phê duyệt các thiết bị bằng `openclaw devices list` và `openclaw devices approve <requestId>`. Khi không chắc chắn, hãy sử dụng Shelley từ trình duyệt của bạn!
## Truy cập từ xa

Truy cập từ xa được xử lý bởi xác thực của [exe.dev](https://exe.dev). Theo
mặc định, lưu lượng HTTP từ cổng 8000 được chuyển tiếp đến `https://<vm-name>.exe.xyz`
với xác thực email.
## Cập nhật

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Hướng dẫn: [Cập nhật](/install/updating)