---
summary: >-
  Chạy OpenClaw Gateway 24/7 trên GCP Compute Engine VM (Docker) với trạng thái
  bền vững
read_when:
  - Bạn muốn OpenClaw chạy 24/7 trên GCP
  - 'Bạn muốn một Gateway cấp production, luôn hoạt động trên VM của riêng mình'
  - >-
    Bạn muốn kiểm soát hoàn toàn tính bền vững, các tệp nhị phân và hành vi khởi
    động lại
title: GCP
x-i18n:
  source_path: install\gcp.md
  source_hash: a1a9d24eda1e39722f7728139230efe3a5305d669fa40f4caf371ea77c977e1e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:08:43.303Z'
---

# OpenClaw trên GCP Compute Engine (Docker, Hướng dẫn VPS Production)

## Mục tiêu

Chạy một Gateway OpenClaw liên tục trên VM GCP Compute Engine bằng Docker, với trạng thái bền vững, các tệp nhị phân được tích hợp sẵn và hành vi khởi động lại an toàn.

Nếu bạn muốn "OpenClaw 24/7 với khoảng ~$5-12/tháng", đây là một thiết lập đáng tin cậy trên Google Cloud.
Giá cả thay đổi tùy theo loại máy và khu vực; hãy chọn VM nhỏ nhất phù hợp với khối lượng công việc của bạn và mở rộng quy mô nếu bạn gặp OOMs.
## Chúng ta đang làm gì (nói một cách đơn giản)?

- Tạo một dự án GCP và bật tính năng thanh toán
- Tạo một VM Compute Engine
- Cài đặt Docker (runtime ứng dụng cách ly)
- Khởi động Gateway trong Docker
- Lưu trữ `~/.openclaw` + `~/.openclaw/workspace` trên máy chủ (tồn tại qua các lần khởi động lại/xây dựng lại)
- Truy cập Control UI từ laptop của bạn thông qua SSH tunnel

Gateway có thể được truy cập thông qua:

- SSH port forwarding từ laptop của bạn
- Tiếp xúc cổng trực tiếp nếu bạn tự quản lý tường lửa và token

Hướng dẫn này sử dụng Debian trên GCP Compute Engine.
Ubuntu cũng hoạt động; ánh xạ các gói tương ứng.
Để tìm hiểu luồng Docker chung, xem [Docker](/install/docker).

---
## Bắt đầu nhanh (các nhà điều hành có kinh nghiệm)

1. Tạo dự án GCP + bật Compute Engine API
2. Tạo Compute Engine VM (e2-small, Debian 12, 20GB)
3. SSH vào VM
4. Cài đặt Docker
5. Clone kho lưu trữ OpenClaw
6. Tạo các thư mục máy chủ lâu dài
7. Cấu hình `.env` và `docker-compose.yml`
8. Biên dịch các tệp nhị phân cần thiết, xây dựng và khởi chạy

---
## Những gì bạn cần

- Tài khoản GCP (đủ điều kiện dùng bản miễn phí e2-micro)
- gcloud CLI được cài đặt (hoặc sử dụng Cloud Console)
- Truy cập SSH từ máy tính xách tay của bạn
- Hiểu biết cơ bản về SSH + sao chép/dán
- ~20-30 phút
- Docker và Docker Compose
- Thông tin xác thực mô hình
- Thông tin xác thực nhà cung cấp tùy chọn
  - QR WhatsApp
  - Token bot Telegram
  - Gmail OAuth

---
## 1) Cài đặt gcloud CLI (hoặc sử dụng Console)

**Tùy chọn A: gcloud CLI** (được khuyến nghị cho tự động hóa)

Cài đặt từ [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)

Khởi tạo và xác thực:

```bash
gcloud init
gcloud auth login
```

**Tùy chọn B: Cloud Console**

Tất cả các bước có thể được thực hiện thông qua giao diện web tại [https://console.cloud.google.com](https://console.cloud.google.com)

---
## 2) Tạo một dự án GCP

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Enable billing at [https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) (required for Compute Engine).

Enable the Compute Engine API:

```bash
gcloud services enable compute.googleapis.com
```

**Bảng điều khiển:**

1. Đi tới IAM & Admin > Create Project
2. Đặt tên và tạo
3. Bật thanh toán cho dự án
4. Điều hướng tới APIs & Services > Enable APIs > tìm kiếm "Compute Engine API" > Enable

---
## 3) Tạo VM

**Loại máy:**

| Loại      | Thông số kỹ thuật        | Chi phí            | Ghi chú                                      |
| --------- | ------------------------ | ------------------ | -------------------------------------------- |
| e2-medium | 2 vCPU, 4GB RAM          | ~$25/tháng         | Đáng tin cậy nhất cho Docker builds cục bộ  |
| e2-small  | 2 vCPU, 2GB RAM          | ~$12/tháng         | Mức tối thiểu được khuyến nghị cho Docker build |
| e2-micro  | 2 vCPU (chia sẻ), 1GB RAM | Đủ điều kiện dùng miễn phí | Thường gặp lỗi OOM với Docker build (exit 137) |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1. Go to Compute Engine > VM instances > Create instance
2. Name: `openclaw-gateway`
3. Region: `us-central1`, Zone: `us-central1-a`
4. Machine type: `e2-small`
5. Boot disk: Debian 12, 20GB
6. Tạo

---
## 4) SSH vào VM

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:**

Nhấp vào nút "SSH" bên cạnh VM của bạn trong bảng điều khiển Compute Engine.

Lưu ý: Việc lan truyền khóa SSH có thể mất 1-2 phút sau khi tạo VM. Nếu kết nối bị từ chối, hãy chờ và thử lại.

---
## 5) Cài đặt Docker (trên VM)

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Log out and back in for the group change to take effect:

```bash
exit
```

Then SSH back in:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Verify:

```bash
docker --version
docker compose version
```

---
## 6) Clone kho lưu trữ OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Hướng dẫn này giả định rằng bạn sẽ xây dựng một hình ảnh tùy chỉnh để đảm bảo tính bền vững của tệp nhị phân.

---
## 7) Tạo thư mục máy chủ lâu dài

Các container Docker là tạm thời.
Tất cả trạng thái lâu dài phải nằm trên máy chủ.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

---
## 8) Cấu hình biến môi trường

Tạo `.env` trong thư mục gốc của kho lưu trữ.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Generate strong secrets:

```bash
openssl rand -hex 32
```

**Không commit tệp này.**

---
## 9) Cấu hình Docker Compose

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
      # Recommended: keep the Gateway loopback-only on the VM; access via SSH tunnel.
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
      ]
```

---
## 10) Bake required binaries into the image (critical)

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
---

## 11) Xây dựng và khởi chạy

```bash
docker compose build
docker compose up -d openclaw-gateway
```

If build fails with `Killed` / `exit code 137` during `pnpm install --frozen-lockfile`, the VM is out of memory. Use `e2-small` minimum, or `e2-medium` for more reliable first builds.

When binding to LAN (`OPENCLAW_GATEWAY_BIND=lan`), configure a trusted browser origin before continuing:

```bash
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]' --strict-json
```

If you changed the gateway port, replace `18789` with your configured port.

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
## 12) Xác minh Gateway

```bash
docker compose logs -f openclaw-gateway
```

Success:

```
[gateway] listening on ws://0.0.0.0:18789
```

---
## 13) Truy cập từ máy tính xách tay của bạn

Tạo một SSH tunnel để chuyển tiếp cổng Gateway:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Open in your browser:

`http://127.0.0.1:18789/`

Fetch a fresh tokenized dashboard link:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
```

Paste the token from that URL.

If Control UI shows `unauthorized` or `disconnected (1008): pairing required`, approve the browser device:

```bash
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

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
## Cập nhật

Để cập nhật OpenClaw trên VM:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

---
## Khắc phục sự cố

**Kết nối SSH bị từ chối**

Việc lan truyền khóa SSH có thể mất 1-2 phút sau khi tạo VM. Hãy chờ và thử lại.

**Vấn đề đăng nhập OS Login**

Kiểm tra hồ sơ OS Login của bạn:

```bash
gcloud compute os-login describe-profile
```

Ensure your account has the required IAM permissions (Compute OS Login or Compute OS Admin Login).

**Out of memory (OOM)**

If Docker build fails with `Killed` and `exit code 137`, the VM was OOM-killed. Upgrade to e2-small (minimum) or e2-medium (recommended for reliable local builds):

```bash
# Stop the VM first
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Change machine type
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# Start the VM
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

---
## Tài khoản dịch vụ (thực tiễn bảo mật tốt nhất)

Để sử dụng cá nhân, tài khoản người dùng mặc định của bạn hoạt động tốt.

Để tự động hóa hoặc các đường ống CI/CD, hãy tạo một tài khoản dịch vụ chuyên dụng với quyền tối thiểu:

1. Tạo một tài khoản dịch vụ:

   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Grant Compute Instance Admin role (or narrower custom role):

   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

Tránh sử dụng vai trò Owner cho tự động hóa. Sử dụng nguyên tắc quyền tối thiểu.

Xem [https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles) để biết chi tiết về vai trò IAM.

---
## Bước tiếp theo

- Thiết lập các kênh nhắn tin: [Channels](/channels)
- Ghép nối các thiết bị cục bộ làm node: [Nodes](/nodes)
- Cấu hình Gateway: [Gateway configuration](/gateway/configuration)