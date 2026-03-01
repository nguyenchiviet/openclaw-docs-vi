---
summary: Quy trình thiết lập nâng cao và phát triển cho OpenClaw
read_when:
  - Thiết lập một máy tính mới
  - >-
    Bạn muốn có "phiên bản mới nhất và tốt nhất" mà không làm hỏng cài đặt cá
    nhân của mình
title: Thiết lập
x-i18n:
  source_path: start\setup.md
  source_hash: 6a29664648c63f9091a971179f0196b54f9b0f6da71d1e68008b8a30216b5458
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:25:22.470Z'
---

# Thiết lập

<Note>
Nếu bạn đang thiết lập lần đầu tiên, hãy bắt đầu với [Bắt đầu](/start/getting-started).
Để biết chi tiết về trình hướng dẫn, xem [Trình hướng dẫn thiết lập ban đầu](/start/wizard).
</Note>

Cập nhật lần cuối: 2026-01-01
## TL;DR

- **Tùy chỉnh nằm ngoài repo:** `~/.openclaw/workspace` (workspace) + `~/.openclaw/openclaw.json` (config).
- **Quy trình ổn định:** cài đặt ứng dụng macOS; để nó chạy Gateway đi kèm.
- **Quy trình phát triển:** chạy Gateway tự mình qua `pnpm gateway:watch`, sau đó để ứng dụng macOS kết nối ở chế độ Local.
## Điều kiện tiên quyết (từ nguồn)

- Node `>=22`
- `pnpm`
- Docker (tùy chọn; chỉ dành cho thiết lập được đóng gói/e2e — xem [Docker](/install/docker))
## Chiến lược tùy chỉnh (để cập nhật không gây hại)

Nếu bạn muốn "100% tùy chỉnh cho tôi" _và_ cập nhật dễ dàng, hãy giữ tùy chỉnh của bạn trong:

- **Config:** `~/.openclaw/openclaw.json` (JSON/JSON5-ish)
- **Workspace:** `~/.openclaw/workspace` (skills, prompts, memories; tạo nó thành một kho git riêng tư)

Bootstrap một lần:

```bash
openclaw setup
```

From inside this repo, use the local CLI entry:

```bash
openclaw setup
```

If you don’t have a global install yet, run it via `pnpm openclaw setup`.
## Chạy Gateway từ kho lưu trữ này

Sau `pnpm build`, bạn có thể chạy CLI được đóng gói trực tiếp:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```
## Quy trình ổn định (ứng dụng macOS trước tiên)

1. Cài đặt + khởi chạy **OpenClaw.app** (thanh menu).
2. Hoàn thành danh sách kiểm tra thiết lập ban đầu/quyền (lời nhắc TCC).
3. Đảm bảo Gateway là **Local** và đang chạy (ứng dụng quản lý nó).
4. Liên kết các bề mặt (ví dụ: WhatsApp):

```bash
openclaw channels login
```

5. Sanity check:

```bash
openclaw health
```

If onboarding is not available in your build:

- Run `openclaw setup`, then `openclaw channels login`, then start the Gateway manually (`openclaw gateway`).
## Quy trình phát triển tối tân (Gateway trong terminal)

Mục tiêu: làm việc trên Gateway TypeScript, nhận hot reload, giữ giao diện ứng dụng macOS được kết nối.

### 0) (Tùy chọn) Chạy ứng dụng macOS từ nguồn cũng vậy

Nếu bạn cũng muốn ứng dụng macOS ở phiên bản tối tân:

```bash
./scripts/restart-mac.sh
```

### 1) Start the dev Gateway

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` runs the gateway in watch mode and reloads on TypeScript changes.

### 2) Point the macOS app at your running Gateway

In **OpenClaw.app**:

- Connection Mode: **Local**
  The app will attach to the running gateway on the configured port.

### 3) Verify

- In-app Gateway status should read **“Using existing gateway …”**
- Or via CLI:

```bash
openclaw health
```

### Common footguns

- **Wrong port:** Gateway WS defaults to `ws://127.0.0.1:18789`; keep app + CLI on the same port.
- **Where state lives:**
  - Credentials: `~/.openclaw/credentials/`
  - Sessions: `~/.openclaw/agents/<agentId>/sessions/`
  - Logs: `/tmp/openclaw/`
## Bản đồ lưu trữ thông tin xác thực

Sử dụng điều này khi gỡ lỗi xác thực hoặc quyết định những gì cần sao lưu:

- **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- **Telegram bot token**: config/env hoặc `channels.telegram.tokenFile`
- **Discord bot token**: config/env (tệp token chưa được hỗ trợ)
- **Slack tokens**: config/env (`channels.slack.*`)
- **Danh sách cho phép ghép nối**:
  - `~/.openclaw/credentials/<channel>-allowFrom.json` (tài khoản mặc định)
  - `~/.openclaw/credentials/<channel>-<accountId>-allowFrom.json` (tài khoản không mặc định)
- **Hồ sơ xác thực mô hình**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **Tải trọng bí mật được hỗ trợ bởi tệp (tùy chọn)**: `~/.openclaw/secrets.json`
- **Nhập OAuth cũ**: `~/.openclaw/credentials/oauth.json`
  Chi tiết hơn: [Bảo mật](/gateway/security#credential-storage-map).
## Cập nhật (mà không làm hỏng thiết lập của bạn)

- Giữ `~/.openclaw/workspace` và `~/.openclaw/` là "những thứ của bạn"; đừng đặt các prompt/cấu hình cá nhân vào repo `openclaw`.
- Cập nhật nguồn: `git pull` + `pnpm install` (khi lockfile thay đổi) + tiếp tục sử dụng `pnpm gateway:watch`.
## Linux (dịch vụ người dùng systemd)

Các cài đặt Linux sử dụng dịch vụ người dùng systemd **user**. Theo mặc định, systemd dừng các dịch vụ người dùng khi đăng xuất/không hoạt động, điều này sẽ dừng Gateway. Thiết lập ban đầu cố gắng bật lingering cho bạn (có thể yêu cầu sudo). Nếu nó vẫn tắt, hãy chạy:

```bash
sudo loginctl enable-linger $USER
```

Đối với các máy chủ luôn bật hoặc đa người dùng, hãy cân nhắc sử dụng dịch vụ **system** thay vì dịch vụ người dùng (không cần lingering). Xem [Gateway runbook](/gateway) để biết các ghi chú về systemd.
## Tài liệu liên quan

- [Sổ tay vận hành Gateway](/gateway) (cờ, giám sát, cổng)
- [Cấu hình Gateway](/gateway/configuration) (lược đồ cấu hình + ví dụ)
- [Discord](/channels/discord) và [Telegram](/channels/telegram) (thẻ trả lời + cài đặt replyToMode)
- [Thiết lập trợ lý OpenClaw](/start/openclaw)
- [Ứng dụng macOS](/platforms/macos) (vòng đời gateway)