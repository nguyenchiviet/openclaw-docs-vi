---
summary: >-
  Sửa các vấn đề khởi động Chrome/Brave/Edge/Chromium CDP trên Linux cho điều
  khiển trình duyệt OpenClaw
read_when: 'Browser control fails on Linux, especially with snap Chromium'
title: Khắc phục sự cố Trình duyệt
x-i18n:
  source_path: tools\browser-linux-troubleshooting.md
  source_hash: bac2301022511a0bf8ebe1309606cc03e8a979ff74866c894f89d280ca3e514e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:27:49.043Z'
---

# Khắc phục sự cố Trình duyệt (Linux)

## Vấn đề: "Failed to start Chrome CDP on port 18800"

Máy chủ điều khiển trình duyệt của OpenClaw không thể khởi chạy Chrome/Brave/Edge/Chromium với lỗi:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### Root Cause

On Ubuntu (and many Linux distros), the default Chromium installation is a **snap package**. Snap's AppArmor confinement interferes with how OpenClaw spawns and monitors the browser process.

The `apt install chromium` command installs a stub package that redirects to snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

This is NOT a real browser — it's just a wrapper.

### Solution 1: Install Google Chrome (Recommended)

Install the official Google Chrome `.deb` package, which is not sandboxed by snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # if there are dependency errors
```

Then update your OpenClaw config (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Solution 2: Use Snap Chromium with Attach-Only Mode

If you must use snap Chromium, configure OpenClaw to attach to a manually-started browser:

1. Update config:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```
2. Khởi động Chromium thủ công:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Optionally create a systemd user service to auto-start Chrome:

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Enable with: `systemctl --user enable --now openclaw-browser.service`

### Verifying the Browser Works

Check status:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Test browsing:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Config Reference

| Option                   | Description                                                          | Default                                                     |
| ------------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| `browser.enabled`        | Enable browser control                                               | `true`                                                      |
| `browser.executablePath` | Path to a Chromium-based browser binary (Chrome/Brave/Edge/Chromium) | auto-detected (prefers default browser when Chromium-based) |
| `browser.headless`       | Run without GUI                                                      | `false`                                                     |
| `browser.noSandbox`      | Add `--no-sandbox` flag (needed for some Linux setups)               | `false`                                                     |
| `browser.attachOnly`     | Don't launch browser, only attach to existing                        | `false`                                                     |
| `browser.cdpPort`        | Chrome DevTools Protocol port                                        | `18800`                                                     |

### Problem: "Chrome extension relay is running, but no tab is connected"

You’re using the `chrome` profile (extension relay). Nó mong đợi tiện ích mở rộng trình duyệt OpenClaw được gắn vào một tab đang hoạt động.
Các tùy chọn sửa:

1. **Sử dụng trình duyệt được quản lý:** `openclaw browser start --browser-profile openclaw`
   (hoặc đặt `browser.defaultProfile: "openclaw"`).
2. **Sử dụng phần mở rộng relay:** cài đặt phần mở rộng, mở một tab và nhấp vào biểu tượng phần mở rộng OpenClaw để đính kèm nó.

Ghi chú:

- Hồ sơ `chrome` sử dụng **trình duyệt Chromium mặc định của hệ thống** khi có thể.
- Hồ sơ `openclaw` cục bộ tự động gán `cdpPort`/`cdpUrl`; chỉ đặt những cái đó cho CDP từ xa.