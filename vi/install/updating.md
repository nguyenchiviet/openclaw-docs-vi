---
summary: >-
  Cập nhật OpenClaw một cách an toàn (cài đặt toàn cục hoặc từ mã nguồn), cùng
  với chiến lược quay lại phiên bản cũ
read_when:
  - Cập nhật OpenClaw
  - Có gì đó bị hỏng sau một bản cập nhật
title: Đang cập nhật
x-i18n:
  source_path: install\updating.md
  source_hash: 38548cf2dc64ff94880f9641f5cdbbc4e8fcad541b45631719097ed241b53da6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:09:49.587Z'
---

# Cập nhật

OpenClaw đang phát triển nhanh chóng (trước phiên bản "1.0"). Hãy coi việc cập nhật như triển khai cơ sở hạ tầng: cập nhật → chạy kiểm tra → khởi động lại (hoặc sử dụng `openclaw update`, tự động khởi động lại) → xác minh.
## Khuyến nghị: chạy lại trình cài đặt trang web (nâng cấp tại chỗ)

**Đường dẫn** cập nhật được ưu tiên là chạy lại trình cài đặt từ trang web. Nó
phát hiện các bản cài đặt hiện có, nâng cấp tại chỗ và chạy `openclaw doctor` khi
cần thiết.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Notes:

- Add `--no-onboard` if you don’t want the onboarding wizard to run again.
- For **source installs**, use:

  ```bash
  curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
  ```

  The installer will `git pull --rebase` **only** if the repo is clean.

- For **global installs**, the script uses `npm install -g openclaw@latest` under the hood.
- Legacy note: `clawdbot` vẫn có sẵn như một lớp tương thích.
## Trước khi cập nhật

- Biết cách bạn đã cài đặt: **toàn cục** (npm/pnpm) vs **từ mã nguồn** (git clone).
- Biết Gateway của bạn đang chạy như thế nào: **terminal foreground** vs **dịch vụ được giám sát** (launchd/systemd).
- Chụp ảnh tùy chỉnh của bạn:
  - Cấu hình: `~/.openclaw/openclaw.json`
  - Thông tin xác thực: `~/.openclaw/credentials/`
  - Không gian làm việc: `~/.openclaw/workspace`
## Cập nhật (cài đặt toàn cục)

Cài đặt toàn cục (chọn một):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

We do **not** recommend Bun for the Gateway runtime (WhatsApp/Telegram bugs).

To switch update channels (git + npm installs):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Use `--tag <dist-tag|version>` for a one-off install tag/version.

See [Development channels](/install/development-channels) for channel semantics and release notes.

Note: on npm installs, the gateway logs an update hint on startup (checks the current channel tag). Disable via `update.checkOnStart: false`.

### Core auto-updater (optional)

Auto-updater is **off by default** and is a core Gateway feature (not a plugin).

```json
{
  "update": {
    "channel": "stable",
    "auto": {
      "enabled": true,
      "stableDelayHours": 6,
      "stableJitterHours": 12,
      "betaCheckIntervalHours": 1
    }
  }
}
```

Behavior:

- `stable`: when a new version is seen, OpenClaw waits `stableDelayHours` and then applies a deterministic per-install jitter in `stableJitterHours` (spread rollout).
- `beta`: checks on `betaCheckIntervalHours` cadence (default: hourly) and applies when an update is available.
- `dev`: no automatic apply; use manual `openclaw update`.

Use `openclaw update --dry-run` to preview update actions before enabling automation.

Then:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```
Ghi chú:

- Nếu Gateway của bạn chạy như một dịch vụ, `openclaw gateway restart` được ưu tiên hơn việc kết thúc các PID.
- Nếu bạn đang ghim vào một phiên bản cụ thể, xem "Rollback / pinning" bên dưới.
## Cập nhật (`openclaw update`)

Đối với **cài đặt từ nguồn** (git checkout), ưu tiên:

```bash
openclaw update
```

It runs a safe-ish update flow:

- Requires a clean worktree.
- Switches to the selected channel (tag or branch).
- Fetches + rebases against the configured upstream (dev channel).
- Installs deps, builds, builds the Control UI, and runs `openclaw doctor`.
- Restarts the gateway by default (use `--no-restart` to skip).

If you installed via **npm/pnpm** (no git metadata), `openclaw update` sẽ cố gắng cập nhật thông qua trình quản lý gói của bạn. Nếu nó không thể phát hiện cài đặt, hãy sử dụng "Cập nhật (cài đặt toàn cầu)" thay thế.
## Cập nhật (Control UI / RPC)

Control UI có **Cập nhật & Khởi động lại** (RPC: `update.run`). Nó:

1. Chạy cùng một luồng cập nhật nguồn như `openclaw update` (chỉ git checkout).
2. Ghi một sentinel khởi động lại với một báo cáo có cấu trúc (đuôi stdout/stderr).
3. Khởi động lại gateway và ping phiên hoạt động cuối cùng với báo cáo.

Nếu rebase thất bại, gateway sẽ hủy bỏ và khởi động lại mà không áp dụng bản cập nhật.
## Cập nhật (từ nguồn)

Từ kho lưu trữ được kiểm tra:

Được ưu tiên:

```bash
openclaw update
```

Manual (equivalent-ish):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps on first run
openclaw doctor
openclaw health
```

Notes:

- `pnpm build` matters when you run the packaged `openclaw` binary ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) or use Node to run `dist/`.
- If you run from a repo checkout without a global install, use `pnpm openclaw ...` for CLI commands.
- If you run directly from TypeScript (`pnpm openclaw ...`), a rebuild is usually unnecessary, but **config migrations still apply** → run doctor.
- Switching between global and git installs is easy: install the other flavor, then run `openclaw doctor` để điểm vào dịch vụ Gateway được viết lại thành cài đặt hiện tại.
## Luôn chạy: `openclaw doctor`

Doctor là lệnh "cập nhật an toàn". Nó cố tình đơn giản: sửa chữa + di chuyển + cảnh báo.

Lưu ý: nếu bạn đang sử dụng **cài đặt từ mã nguồn** (git checkout), `openclaw doctor` sẽ đề xuất chạy `openclaw update` trước.

Những điều điển hình mà nó thực hiện:

- Di chuyển các khóa cấu hình không dùng nữa / vị trí tệp cấu hình cũ.
- Kiểm tra các chính sách tin nhắn riêng và cảnh báo về các cài đặt "mở" rủi ro.
- Kiểm tra tình trạng Gateway và có thể đề xuất khởi động lại.
- Phát hiện và di chuyển các dịch vụ gateway cũ hơn (launchd/systemd; legacy schtasks) sang các dịch vụ OpenClaw hiện tại.
- Trên Linux, đảm bảo systemd user lingering (để Gateway tồn tại sau khi đăng xuất).

Chi tiết: [Doctor](/gateway/doctor)
## Khởi động / dừng / khởi động lại Gateway

CLI (hoạt động bất kể hệ điều hành):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

If you’re supervised:

- macOS launchd (app-bundled LaunchAgent): `launchctl kickstart -k gui/$UID/ai.openclaw.gateway` (use `ai.openclaw.<profile>`; legacy `com.openclaw.*` still works)
- Linux systemd user service: `systemctl --user restart openclaw-gateway[-<profile>].service`
- Windows (WSL2): `systemctl --user restart openclaw-gateway[-<profile>].service`
  - `launchctl`/`systemctl` only work if the service is installed; otherwise run `openclaw gateway install`.

Runbook + nhãn dịch vụ chính xác: [Gateway runbook](/gateway)
## Quay lại / ghim phiên bản (khi có sự cố)

### Ghim (cài đặt toàn cục)

Cài đặt một phiên bản đã biết là hoạt động tốt (thay thế `<version>` bằng phiên bản cuối cùng hoạt động):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Tip: to see the current published version, run `npm view openclaw version`.

Then restart + re-run doctor:

```bash
openclaw doctor
openclaw gateway restart
```

### Pin (source) by date

Pick a commit from a date (example: “state of main as of 2026-01-01”):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Then reinstall deps + restart:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

If you want to go back to latest later:

```bash
git checkout main
git pull
```
## Nếu bạn gặp vấn đề

- Chạy `openclaw doctor` lại và đọc kỹ kết quả (nó thường cho bạn biết cách khắc phục).
- Kiểm tra: [Khắc phục sự cố](/gateway/troubleshooting)
- Hỏi trên Discord: [https://discord.gg/clawd](https://discord.gg/clawd)