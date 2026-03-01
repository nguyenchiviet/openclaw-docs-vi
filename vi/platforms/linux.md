---
summary: Hỗ trợ Linux + trạng thái ứng dụng đi kèm
read_when:
  - Tìm kiếm trạng thái ứng dụng đồng hành Linux
  - Lập kế hoạch phạm vi nền tảng hoặc đóng góp
title: Ứng dụng Linux
x-i18n:
  source_path: platforms\linux.md
  source_hash: 93b8250cd1267004a3342c8119462d0442af96704f9b3be250d8ee1eeeb7d4cd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:20.242Z'
---

# Ứng dụng Linux

Gateway được hỗ trợ đầy đủ trên Linux. **Node là runtime được khuyến nghị**.
Bun không được khuyến nghị cho Gateway (lỗi WhatsApp/Telegram).

Các ứng dụng đi kèm Linux gốc đang được lên kế hoạch. Chúng tôi hoan nghênh các đóng góp nếu bạn muốn giúp xây dựng một ứng dụng.
## Đường dẫn nhanh cho người mới bắt đầu (VPS)

1. Cài đặt Node 22+
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. Từ laptop của bạn: `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. Mở `http://127.0.0.1:18789/` và dán token của bạn

Hướng dẫn VPS từng bước: [exe.dev](/install/exe-dev)
## Cài đặt

- [Bắt đầu](/start/getting-started)
- [Cài đặt & cập nhật](/install/updating)
- Luồng tùy chọn: [Bun](/install/bun), [Nix](/install/nix), [Docker](/install/docker)
## Gateway

- [Sổ tay Gateway](/gateway)
- [Cấu hình](/gateway/configuration)
## Cài đặt dịch vụ Gateway (CLI)

Sử dụng một trong những cách sau:

```
openclaw onboard --install-daemon
```

Or:

```
openclaw gateway install
```

Or:

```
openclaw configure
```

Select **Gateway service** when prompted.

Repair/migrate:

```
openclaw doctor
```
## Kiểm soát hệ thống (systemd user unit)

OpenClaw cài đặt một dịch vụ **user** systemd theo mặc định. Sử dụng dịch vụ **system** cho các máy chủ được chia sẻ hoặc luôn bật. Ví dụ unit đầy đủ và hướng dẫn nằm trong [Gateway runbook](/gateway).

Thiết lập tối thiểu:

Tạo `~/.config/systemd/user/openclaw-gateway[-<profile>].service`:

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Enable it:

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```