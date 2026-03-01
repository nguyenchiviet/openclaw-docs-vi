---
summary: 'Gỡ cài đặt OpenClaw hoàn toàn (CLI, service, state, workspace)'
read_when:
  - Bạn muốn xóa OpenClaw khỏi một máy
  - Dịch vụ gateway vẫn đang chạy sau khi gỡ cài đặt
title: Gỡ cài đặt
x-i18n:
  source_path: install\uninstall.md
  source_hash: 34c7d3e4ad17333439048dfda739fc27db47e7f9e4212fe17db0e4eb3d3ab258
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:09:35.007Z'
---

# Gỡ cài đặt

Hai cách:

- **Cách dễ dàng** nếu `openclaw` vẫn được cài đặt.
- **Xóa dịch vụ thủ công** nếu CLI đã bị xóa nhưng dịch vụ vẫn đang chạy.
## Đường dẫn dễ dàng (CLI vẫn được cài đặt)

Khuyến nghị: sử dụng trình gỡ cài đặt tích hợp:

```bash
openclaw uninstall
```

Non-interactive (automation / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Manual steps (same result):

1. Stop the gateway service:

```bash
openclaw gateway stop
```

2. Uninstall the gateway service (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3. Delete state + config:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

If you set `OPENCLAW_CONFIG_PATH` to a custom location outside the state dir, delete that file too.

4. Delete your workspace (optional, removes agent files):

```bash
rm -rf ~/.openclaw/workspace
```

5. Remove the CLI install (pick the one you used):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. If you installed the macOS app:

```bash
rm -rf /Applications/OpenClaw.app
```

Notes:

- If you used profiles (`--profile` / `OPENCLAW_PROFILE`), repeat step 3 for each state dir (defaults are `~/.openclaw-<profile>`).
- Ở chế độ remote, thư mục state nằm trên **máy chủ gateway**, vì vậy hãy chạy các bước 1-4 ở đó.

## Xóa dịch vụ thủ công (CLI chưa được cài đặt)

Sử dụng điều này nếu dịch vụ gateway tiếp tục chạy nhưng `openclaw` bị thiếu.

### macOS (launchd)

Nhãn mặc định là `ai.openclaw.gateway` (hoặc `ai.openclaw.<profile>`; `com.openclaw.*` cũ có thể vẫn tồn tại):

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

If you used a profile, replace the label and plist name with `ai.openclaw.<profile>`. Remove any legacy `com.openclaw.*` plists if present.

### Linux (systemd user unit)

Default unit name is `openclaw-gateway.service` (or `openclaw-gateway-<profile>.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Scheduled Task)

Default task name is `OpenClaw Gateway` (or `OpenClaw Gateway (<profile>)`).
The task script lives under your state dir.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

If you used a profile, delete the matching task name and `~\.openclaw-<profile>\gateway.cmd`.
## Cài đặt bình thường so với kiểm tra mã nguồn

### Cài đặt bình thường (install.sh / npm / pnpm / bun)

Nếu bạn đã sử dụng `https://openclaw.ai/install.sh` hoặc `install.ps1`, CLI đã được cài đặt với `npm install -g openclaw@latest`.
Gỡ cài đặt nó bằng `npm rm -g openclaw` (hoặc `pnpm remove -g` / `bun remove -g` nếu bạn cài đặt theo cách đó).

### Kiểm tra mã nguồn (git clone)

Nếu bạn chạy từ một kiểm tra repo (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1. Gỡ cài đặt dịch vụ Gateway **trước khi** xóa repo (sử dụng đường dẫn dễ dàng ở trên hoặc gỡ cài đặt dịch vụ thủ công).
2. Xóa thư mục repo.
3. Xóa trạng thái + không gian làm việc như được hiển thị ở trên.