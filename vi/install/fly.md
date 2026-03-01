---
title: Fly.io
description: Deploy OpenClaw on Fly.io
summary: Từng bước triển khai OpenClaw trên Fly.io với lưu trữ bền vững và HTTPS
read_when:
  - Triển khai OpenClaw trên Fly.io
  - 'Thiết lập Fly volumes, các bí mật và cấu hình chạy lần đầu'
x-i18n:
  source_path: install\fly.md
  source_hash: 35fe4d748fb5460dababb0c22e78afe05efb6ef078dee58a22db99125b05ec5d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:16:07.717Z'
---

# Triển khai trên Fly.io

**Mục tiêu:** Gateway OpenClaw chạy trên máy [Fly.io](https://fly.io) với bộ nhớ bền vững, HTTPS tự động và quyền truy cập Discord/kênh.

## Những gì bạn cần

- [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) đã được cài đặt
- Tài khoản Fly.io (gói miễn phí vẫn hoạt động)
- Xác thực mô hình: Khóa API Anthropic (hoặc khóa của nhà cung cấp khác)
- Thông tin xác thực kênh: Mã thông báo bot Discord, mã thông báo Telegram, v.v.
## Lộ trình nhanh cho người mới bắt đầu

1. Sao chép kho lưu trữ → tùy chỉnh `fly.toml`
2. Tạo ứng dụng + ổ đĩa → đặt bí mật
3. Triển khai với `fly deploy`
4. SSH vào để tạo cấu hình hoặc sử dụng Giao diện người dùng điều khiển
## 1) Tạo ứng dụng Fly

```bash
# Clone the repo
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Create a new Fly app (pick your own name)
fly apps create my-openclaw

# Create a persistent volume (1GB is usually enough)
fly volumes create openclaw_data --size 1 --region iad
```

**Tip:** Choose a region close to you. Common options: `lhr` (London), `iad` (Virginia), `sjc` (San Jose).
## 2) Cấu hình fly.toml

Chỉnh sửa `fly.toml` để phù hợp với tên ứng dụng và yêu cầu của bạn.

**Lưu ý bảo mật:** Cấu hình mặc định hiển thị một URL công khai. Để triển khai bảo mật hơn mà không có IP công khai, hãy xem [Triển khai riêng tư](#private-deployment-hardened) hoặc sử dụng `fly.private.toml`.

```toml
app = "my-openclaw"  # Your app name
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```
**Các cài đặt chính:**

| Cài đặt                        | Lý do                                                                       |
| ------------------------------ | --------------------------------------------------------------------------- |
| `--bind lan`                   | Liên kết với `0.0.0.0` để proxy của Fly có thể tiếp cận gateway             |
| `--allow-unconfigured`         | Khởi động mà không có tệp cấu hình (bạn sẽ tạo một tệp sau)                  |
| `internal_port = 3000`         | Phải khớp với `--port 3000` (hoặc `OPENCLAW_GATEWAY_PORT`) cho các kiểm tra sức khỏe của Fly |
| `memory = "2048mb"`            | 512MB quá nhỏ; khuyến nghị 2GB                                              |
| `OPENCLAW_STATE_DIR = "/data"` | Duy trì trạng thái trên ổ đĩa                                               |
## 3) Đặt bí mật

```bash
# Required: Gateway token (for non-loopback binding)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Model provider API keys
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optional: Other providers
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Channel tokens
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Notes:**

- Non-loopback binds (`--bind lan`) require `OPENCLAW_GATEWAY_TOKEN__OC_I19N_0003__openclaw.json` nơi chúng có thể vô tình bị lộ hoặc ghi nhật ký.

## 4) Triển khai

``__OC_I19N_0000__`__OC_I19N_0001__`__OC_I19N_0002__`__OC_I19N_0003__`__OC_I19N_0004__``
## 5) Tạo tệp cấu hình

SSH vào máy để tạo cấu hình phù hợp:

```bash
fly ssh console
```

Create the config directory and file:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```
**Lưu ý:** Với `OPENCLAW_STATE_DIR=/data`, đường dẫn cấu hình là `/data/openclaw.json`.

**Lưu ý:** Mã thông báo Discord có thể đến từ một trong hai nguồn:

- Biến môi trường: `DISCORD_BOT_TOKEN` (khuyên dùng cho các bí mật)
- Tệp cấu hình: `channels.discord.token`

Nếu sử dụng biến môi trường, không cần thêm mã thông báo vào cấu hình. Gateway sẽ tự động đọc `DISCORD_BOT_TOKEN`.

Khởi động lại để áp dụng:

```bash
exit
fly machine restart <machine-id>
```
## 6) Truy cập Gateway

### Giao diện người dùng điều khiển

Mở trong trình duyệt:

```bash
fly open
```

Or visit `https://my-openclaw.fly.dev/`

Paste your gateway token (the one from `OPENCLAW_GATEWAY_TOKEN`) to authenticate.

### Logs

```bash
fly logs              # Live logs
fly logs --no-tail    # Recent logs
```

### SSH Console

```bash
fly ssh console
```
## Khắc phục sự cố

### "Ứng dụng không lắng nghe trên địa chỉ dự kiến"

Gateway đang liên kết với __OC_I19N_0000__ thay vì __OC_I19N_0001__.

**Cách khắc phục:** Thêm __OC_I19N_0002__ vào lệnh xử lý của bạn trong __OC_I19N_0003__.

### Kiểm tra tình trạng không thành công / kết nối bị từ chối

Fly không thể tiếp cận gateway trên cổng đã cấu hình.

**Cách khắc phục:** Đảm bảo __OC_I19N_0004__ khớp với cổng gateway (đặt __OC_I19N_0005__ hoặc __OC_I19N_0006__).

### Sự cố OOM / Bộ nhớ

Container liên tục khởi động lại hoặc bị tắt. Dấu hiệu: __OC_I19N_0007__, __OC_I19N_0008__, hoặc khởi động lại âm thầm.

**Cách khắc phục:** Tăng bộ nhớ trong __OC_I19N_0009__:

``__OC_I19N_0010__`__OC_I19N_0011__`__OC_I19N_0012__``
**Lưu ý:** 512MB là quá nhỏ. 1GB có thể hoạt động nhưng có thể bị hết bộ nhớ (OOM) khi tải nặng hoặc khi ghi nhật ký chi tiết. **Khuyến nghị 2GB.**

### Sự cố khóa Gateway

Gateway từ chối khởi động với lỗi "đã chạy".

Điều này xảy ra khi vùng chứa khởi động lại nhưng tệp khóa PID vẫn còn trên ổ đĩa.

**Cách khắc phục:** Xóa tệp khóa:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

The lock file is at `/data/gateway.*.lock` (not in a subdirectory).

### Config Not Being Read

If using `--allow-unconfigured`, the gateway creates a minimal config. Your custom config at `/data/openclaw.json` should be read on restart.

Verify the config exists:

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### Ghi cấu hình qua SSH
Lệnh `fly ssh console -C` không hỗ trợ chuyển hướng shell. Để ghi một tệp cấu hình:

```bash
# Use echo + tee (pipe from local to remote)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Or use sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Note:** `fly sftp` may fail if the file already exists. Delete first:

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### State Not Persisting

If you lose credentials or sessions after a restart, the state dir is writing to the container filesystem.

**Fix:** Ensure `OPENCLAW_STATE_DIR=/data` is set in `fly.toml` và triển khai lại.
## Cập nhật

Ứng dụng của bạn sẽ được cập nhật tự động khi bạn chạy ``bash
# Pull latest changes
git pull

# Redeploy
fly deploy

# Check health
fly status
fly logs
```

### Updating Machine Command

If you need to change the startup command without a full redeploy:

```bash
# Get machine ID
fly machines list

# Update command
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# Or with memory increase
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Note:** After `fly deploy`, the machine command may reset to what's in `fly.toml`. Nếu bạn đã thực hiện các thay đổi thủ công, hãy áp dụng lại chúng sau khi triển khai.
## Triển khai riêng tư (Tăng cường bảo mật)

Theo mặc định, Fly cấp phát các IP công cộng, giúp Gateway của bạn có thể truy cập tại `https://your-app.fly.dev`. Điều này tiện lợi nhưng có nghĩa là việc triển khai của bạn có thể bị phát hiện bởi các trình quét internet (Shodan, Censys, v.v.).

Để triển khai tăng cường bảo mật mà **không có tiếp xúc công khai**, hãy sử dụng mẫu riêng tư.

### Khi nào nên sử dụng triển khai riêng tư

- Bạn chỉ thực hiện các cuộc gọi/tin nhắn **đi ra** (không có webhook đến)
- Bạn sử dụng các đường hầm **ngrok hoặc Tailscale** cho bất kỳ webhook callback nào
- Bạn truy cập Gateway qua **SSH, proxy hoặc WireGuard** thay vì trình duyệt
- Bạn muốn việc triển khai **ẩn khỏi các trình quét internet**

### Thiết lập

Sử dụng `fly.private.toml` thay vì cấu hình tiêu chuẩn:

```bash
# Deploy with private config
fly deploy -c fly.private.toml
```

Or convert an existing deployment:

```bash
# List current IPs
fly ips list -a my-openclaw

# Release public IPs
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Switch to private config so future deploys don't re-allocate public IPs
# (remove [http_service] or deploy with the private template)
fly deploy -c fly.private.toml

# Allocate private-only IPv6
fly ips allocate-v6 --private -a my-openclaw
```
Sau đó, `fly ips list` chỉ nên hiển thị một IP loại `private`:

```
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### Accessing a private deployment

Since there's no public URL, use one of these methods:

**Option 1: Local proxy (simplest)**

```bash
# Forward local port 3000 to the app
fly proxy 3000:3000 -a my-openclaw

# Then open http://localhost:3000 in browser
```

**Option 2: WireGuard VPN**

```bash
# Create WireGuard config (one-time)
fly wireguard create

# Import to WireGuard client, then access via internal IPv6
# Example: http://[fdaa:x:x:x:x::x]:3000
```
**Tùy chọn 3: Chỉ SSH**

```bash
fly ssh console -a my-openclaw
```

### Webhooks with private deployment

If you need webhook callbacks (Twilio, Telnyx, etc.) without public exposure:

1. **ngrok tunnel** - Run ngrok inside the container or as a sidecar
2. **Tailscale Funnel** - Expose specific paths via Tailscale
3. **Outbound-only** - Some providers (Twilio) work fine for outbound calls without webhooks

Example voice-call config with ngrok:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" },
          "webhookSecurity": {
            "allowedHosts": ["example.ngrok.app"]
          }
        }
      }
    }
  }
}
```
Đường hầm ngrok chạy bên trong container và cung cấp một URL webhook công khai mà không làm lộ ứng dụng Fly. Đặt __OC_I19N_0000__ thành tên máy chủ đường hầm công khai để các tiêu đề máy chủ được chuyển tiếp được chấp nhận.

### Lợi ích bảo mật

| Khía cạnh                          | Công khai         | Riêng tư    |
| --------------------------------- | ----------------- | ----------- |
| Trình quét Internet               | Có thể phát hiện  | Ẩn         |
| Tấn công trực tiếp                | Có thể xảy ra     | Bị chặn     |
| Truy cập giao diện người dùng điều khiển | Trình duyệt        | Proxy/VPN   |
| Phân phối Webhook                 | Trực tiếp         | Qua đường hầm |

## Lưu ý

- Fly.io sử dụng **kiến trúc x86** (không phải ARM)
- Dockerfile tương thích với cả hai kiến trúc
- Để thiết lập ban đầu WhatsApp/Telegram, hãy sử dụng `fly ssh console`
- Dữ liệu bền vững nằm trên ổ đĩa tại `/data`
- Signal yêu cầu Java + signal-cli; hãy sử dụng một image tùy chỉnh và giữ bộ nhớ ở mức 2GB+.
## Chi phí

Với cấu hình khuyến nghị (`shared-cpu-2x`, 2GB RAM):

- ~10-15 USD/tháng tùy thuộc vào mức sử dụng
- Gói miễn phí bao gồm một số hạn mức

Xem [giá của Fly.io](https://fly.io/docs/about/pricing/) để biết chi tiết.
