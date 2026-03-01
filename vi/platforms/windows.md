---
summary: Hỗ trợ Windows (WSL2) + trạng thái ứng dụng đi kèm
read_when:
  - Cài đặt OpenClaw trên Windows
  - Tìm kiếm trạng thái ứng dụng đồng hành Windows
title: Windows (WSL2)
x-i18n:
  source_path: platforms\windows.md
  source_hash: d17df1bd5636502e45697526758648520ab1d7aa04356748695bfbe572005ebd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:51.546Z'
---

# Windows (WSL2)

OpenClaw trên Windows được khuyến nghị **qua WSL2** (Ubuntu được khuyến nghị). CLI + Gateway chạy bên trong Linux, giữ cho runtime nhất quán và làm cho công cụ tương thích hơn nhiều (Node/Bun/pnpm, Linux binaries, Skills). Windows gốc có thể phức tạp hơn. WSL2 cung cấp cho bạn trải nghiệm Linux đầy đủ — một lệnh để cài đặt: `wsl --install`.

Các ứng dụng đi kèm Windows gốc đang được lên kế hoạch.
## Cài đặt (WSL2)

- [Bắt đầu](/start/getting-started) (sử dụng bên trong WSL)
- [Cài đặt & cập nhật](/install/updating)
- Hướng dẫn WSL2 chính thức (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)
## Gateway

- [Sổ tay Gateway](/gateway)
- [Cấu hình](/gateway/configuration)
## Cài đặt dịch vụ Gateway (CLI)

Bên trong WSL2:

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
## Nâng cao: Expose dịch vụ WSL qua LAN (portproxy)

WSL có mạng ảo riêng của nó. Nếu một máy khác cần truy cập một dịch vụ
đang chạy **bên trong WSL** (SSH, máy chủ TTS cục bộ, hoặc Gateway), bạn phải
chuyển tiếp một cổng Windows đến IP WSL hiện tại. IP WSL thay đổi sau khi khởi động lại,
vì vậy bạn có thể cần làm mới quy tắc chuyển tiếp.

Ví dụ (PowerShell **với tư cách Quản trị viên**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Allow the port through Windows Firewall (one-time):

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Refresh the portproxy after WSL restarts:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Notes:

- SSH from another machine targets the **Windows host IP** (example: `ssh user@windows-host -p 2222`).
- Remote nodes must point at a **reachable** Gateway URL (not `127.0.0.1`); use
  `openclaw status --all` to confirm.
- Use `listenaddress=0.0.0.0` for LAN access; `127.0.0.1` giữ nó chỉ cục bộ.
- Nếu bạn muốn điều này tự động, hãy đăng ký một Scheduled Task để chạy bước làm mới
  khi đăng nhập.
## Hướng dẫn cài đặt WSL2 từng bước

### 1) Cài đặt WSL2 + Ubuntu

Mở PowerShell (Admin):

```powershell
wsl --install
# Or pick a distro explicitly:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Reboot if Windows asks.

### 2) Enable systemd (required for gateway install)

In your WSL terminal:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Then from PowerShell:

```powershell
wsl --shutdown
```

Re-open Ubuntu, then verify:

```bash
systemctl --user status
```

### 3) Install OpenClaw (inside WSL)

Follow the Linux Getting Started flow inside WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # auto-installs UI deps on first run
pnpm build
openclaw onboard
```

Hướng dẫn đầy đủ: [Bắt đầu](/start/getting-started)
## Ứng dụng đi kèm Windows

Chúng tôi chưa có ứng dụng đi kèm Windows. Chúng tôi hoan nghênh các đóng góp nếu bạn muốn giúp thực hiện điều này.