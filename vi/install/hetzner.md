---
summary: >-
  Chạy OpenClaw Gateway 24/7 trên VPS Hetzner giá rẻ (Docker) với trạng thái bền
  vững và các tệp nhị phân được tích hợp sẵn
read_when:
  - >-
    Bạn muốn OpenClaw chạy 24/7 trên một VPS đám mây (không phải trên laptop của
    bạn)
  - 'Bạn muốn một Gateway cấp production, luôn hoạt động trên VPS của riêng mình'
  - >-
    Bạn muốn kiểm soát hoàn toàn tính bền vững, các tệp nhị phân và hành vi khởi
    động lại
  - >-
    Bạn đang chạy OpenClaw trong Docker trên Hetzner hoặc một nhà cung cấp tương
    tự
title: Hetzner
x-i18n:
  source_path: install\hetzner.md
  source_hash: 97339c7ddb8b9007fa51bb5c31b1fe5d2b1e190239216df3bd83ed8859f5dae1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:08:35.295Z'
---

# OpenClaw trên Hetzner (Docker, Hướng dẫn VPS Production)

## Mục tiêu

Chạy một Gateway OpenClaw liên tục trên VPS Hetzner bằng Docker, với trạng thái bền vững, các tệp nhị phân được tích hợp sẵn và hành vi khởi động lại an toàn.

Nếu bạn muốn "OpenClaw 24/7 với khoảng ~$5", đây là thiết lập đáng tin cậy đơn giản nhất.
Giá Hetzner thay đổi; chọn VPS Debian/Ubuntu nhỏ nhất và mở rộng quy mô nếu bạn gặp OOMs.

Nhắc nhở về mô hình bảo mật:

- Các agent được chia sẻ trong công ty là được phép khi mọi người nằm trong cùng một ranh giới tin tưởng và thời gian chạy chỉ dành cho kinh doanh.
- Giữ sự tách biệt nghiêm ngặt: VPS/thời gian chạy chuyên dụng + tài khoản chuyên dụng; không có hồ sơ Apple/Google/trình duyệt/trình quản lý mật khẩu cá nhân trên máy chủ đó.
- Nếu người dùng đối kháng với nhau, hãy chia theo gateway/máy chủ/người dùng OS.

Xem [Bảo mật](/gateway/security) và [Lưu trữ VPS](/vps).
## Chúng ta đang làm gì (nói một cách đơn giản)?

- Thuê một máy chủ Linux nhỏ (Hetzner VPS)
- Cài đặt Docker (runtime ứng dụng cách ly)
- Khởi động Gateway trong Docker
- Lưu trữ `~/.openclaw` + `~/.openclaw/workspace` trên máy chủ (tồn tại qua các lần khởi động lại/xây dựng lại)
- Truy cập Control UI từ laptop của bạn thông qua SSH tunnel

Gateway có thể được truy cập thông qua:

- SSH port forwarding từ laptop của bạn
- Phơi bày cổng trực tiếp nếu bạn tự quản lý tường lửa và token

Hướng dẫn này giả định Ubuntu hoặc Debian trên Hetzner.  
Nếu bạn đang sử dụng VPS Linux khác, hãy ánh xạ các gói tương ứng.
Để tìm hiểu luồng Docker chung, xem [Docker](/install/docker).

---
## Bắt đầu nhanh (các nhà điều hành có kinh nghiệm)

1. Cấp phát Hetzner VPS
2. Cài đặt Docker
3. Clone kho lưu trữ OpenClaw
4. Tạo các thư mục máy chủ lâu dài
5. Cấu hình `.env` và `docker-compose.yml`
6. Tích hợp các tệp nhị phân cần thiết vào hình ảnh
7. `docker compose up -d`
8. Xác minh tính bền vững và quyền truy cập Gateway

---
## Những gì bạn cần

- VPS Hetzner với quyền truy cập root
- Truy cập SSH từ laptop của bạn
- Sự thoải mái cơ bản với SSH + sao chép/dán
- ~20 phút
- Docker và Docker Compose
- Thông tin xác thực mô hình
- Thông tin xác thực nhà cung cấp tùy chọn
  - QR WhatsApp
  - Token bot Telegram
  - Gmail OAuth

---
## 1) Cấp phát VPS

Tạo VPS Ubuntu hoặc Debian trong Hetzner.

Kết nối với tư cách root:

```bash
ssh root@YOUR_VPS_IP
```

Hướng dẫn này giả định VPS là có trạng thái.
Không coi nó là cơ sở hạ tầng có thể loại bỏ.

---
## 2) Cài đặt Docker (trên VPS)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Verify:

```bash
docker --version
docker compose version
```

---
## 3) Clone kho lưu trữ OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Hướng dẫn này giả định rằng bạn sẽ xây dựng một hình ảnh tùy chỉnh để đảm bảo tính bền vững của tệp nhị phân.

---
## 4) Tạo thư mục máy chủ lâu dài

Các container Docker là tạm thời.
Tất cả trạng thái lâu dài phải nằm trên máy chủ.

```bash
mkdir -p /root/.openclaw/workspace

# Set ownership to the container user (uid 1000):
chown -R 1000:1000 /root/.openclaw
```

---
## 5) Cấu hình biến môi trường

Tạo `.env` trong thư mục gốc của kho lưu trữ.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Generate strong secrets:

```bash
openssl rand -hex 32
```

**Không commit tệp này.**

---
## 6) Cấu hình Docker Compose

Tạo hoặc cập nhật `docker-compose.yml`.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommended: keep the Gateway loopback-only on the VPS; access via SSH tunnel.
      # To expose it publicly, remove the `127.0.0.1:` prefix and firewall accordingly.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured` is only for bootstrap convenience, it is not a replacement for a proper gateway configuration. Still set auth (`gateway.auth.token` hoặc mật khẩu) và sử dụng cài đặt ràng buộc an toàn cho triển khai của bạn.

---
## 7) Bake required binaries into the image (critical)

Installing binaries inside a running container is a trap.
Anything installed at runtime will be lost on restart.

All external binaries required by skills must be installed at image build time.

The examples below show three common binaries only:

- `gog` for Gmail access
- `goplaces` for Google Places
- `wacli` for WhatsApp

These are examples, not a complete list.
You may install as many binaries as needed using the same pattern.

If you add new skills later that depend on additional binaries, you must:

1. Update the Dockerfile
2. Rebuild the image
3. Restart the containers

**Example Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Add more binaries below using the same pattern

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```
## 8) Xây dựng và khởi chạy

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verify binaries:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Expected output:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

---
## 9) Xác minh Gateway

```bash
docker compose logs -f openclaw-gateway
```

Success:

```
[gateway] listening on ws://0.0.0.0:18789
```

From your laptop:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Open:

`http://127.0.0.1:18789/`

Dán token gateway của bạn.

---
## Dữ liệu lưu trữ ở đâu (nguồn dữ liệu chính)

OpenClaw chạy trong Docker, nhưng Docker không phải là nguồn dữ liệu chính.
Tất cả trạng thái dài hạn phải tồn tại qua các lần khởi động lại, xây dựng lại và khởi động lại hệ thống.

| Thành phần          | Vị trí                            | Cơ chế lưu trữ         | Ghi chú                          |
| ------------------- | --------------------------------- | ---------------------- | -------------------------------- |
| Cấu hình Gateway    | `/home/node/.openclaw/`           | Gắn kết volume máy chủ | Bao gồm `openclaw.json`, token |
| Hồ sơ xác thực mô hình | `/home/node/.openclaw/`           | Gắn kết volume máy chủ | Token OAuth, khóa API            |
| Cấu hình Skill      | `/home/node/.openclaw/skills/`    | Gắn kết volume máy chủ | Trạng thái cấp Skill             |
| Không gian làm việc agent | `/home/node/.openclaw/workspace/` | Gắn kết volume máy chủ | Mã và tạo tác agent              |
| Phiên WhatsApp      | `/home/node/.openclaw/`           | Gắn kết volume máy chủ | Bảo toàn đăng nhập QR            |
| Keyring Gmail       | `/home/node/.openclaw/`           | Volume máy chủ + mật khẩu | Yêu cầu `GOG_KEYRING_PASSWORD`  |
| Tệp nhị phân bên ngoài | `/usr/local/bin/`                 | Hình ảnh Docker        | Phải được tích hợp vào lúc xây dựng |
| Node runtime        | Hệ thống tệp container            | Hình ảnh Docker        | Được xây dựng lại mỗi lần xây dựng hình ảnh |
| Gói OS              | Hệ thống tệp container            | Hình ảnh Docker        | Không cài đặt lúc chạy           |
| Container Docker    | Tạm thời                         | Có thể khởi động lại   | An toàn để xóa                   |

---
## Cơ sở hạ tầng dưới dạng mã (Terraform)

Đối với các nhóm ưa thích quy trình infrastructure-as-code, một thiết lập Terraform được duy trì bởi cộng đồng cung cấp:

- Cấu hình Terraform mô-đun với quản lý trạng thái từ xa
- Cấp phát tự động thông qua cloud-init
- Các tập lệnh triển khai (bootstrap, deploy, backup/restore)
- Cứng hóa bảo mật (tường lửa, UFW, chỉ truy cập SSH)
- Cấu hình SSH tunnel để truy cập gateway

**Kho lưu trữ:**

- Cơ sở hạ tầng: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
- Cấu hình Docker: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

Phương pháp này bổ sung cho thiết lập Docker ở trên với các triển khai có thể tái tạo, cơ sở hạ tầng được kiểm soát phiên bản và khôi phục thảm họa tự động.

> **Lưu ý:** Được duy trì bởi cộng đồng. Để báo cáo vấn đề hoặc đóng góp, xem các liên kết kho lưu trữ ở trên.