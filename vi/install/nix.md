---
summary: Cài đặt OpenClaw một cách khai báo với Nix
read_when:
  - Bạn muốn các bản cài đặt có thể tái tạo lại và có khả năng rollback
  - Bạn đã sử dụng Nix/NixOS/Home Manager
  - Bạn muốn mọi thứ được ghim và quản lý một cách khai báo
title: |-
  I need more context to provide an accurate translation. "Nix" could mean:

  1. **Nix** (the package manager/OS) - should remain as "Nix"
  2. "nix" (verb - to reject/cancel) - "từ chối" or "hủy bỏ"

  Could you provide the full sentence or context you'd like translated?
x-i18n:
  source_path: install\nix.md
  source_hash: 70098e84aed7f91a86ba504562fd2ad7e0eace1af26d8f8b5a4be71d9920c027
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:09:02.444Z'
---

# Cài đặt Nix

Cách được khuyến nghị để chạy OpenClaw với Nix là thông qua **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — một mô-đun Home Manager đầy đủ tính năng.
## Bắt đầu nhanh

Dán đoạn này vào AI agent của bạn (Claude, Cursor, v.v.):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Hướng dẫn đầy đủ: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Kho lưu trữ nix-openclaw là nguồn thông tin chính thức cho cài đặt Nix. Trang này chỉ là một tổng quan nhanh.
## Những gì bạn nhận được

- Gateway + ứng dụng macOS + công cụ (whisper, spotify, cameras) — tất cả được ghim
- Dịch vụ Launchd tồn tại sau khi khởi động lại
- Hệ thống plugin với cấu hình khai báo
- Khôi phục tức thì: `home-manager switch --rollback`

---
## Hành vi Runtime Nix Mode

Khi `OPENCLAW_NIX_MODE=1` được đặt (tự động với nix-openclaw):

OpenClaw hỗ trợ **Nix mode** giúp cấu hình trở nên xác định và vô hiệu hóa các luồng cài đặt tự động.
Bật nó bằng cách xuất:

```bash
OPENCLAW_NIX_MODE=1
```

On macOS, the GUI app does not automatically inherit shell env vars. You can
also enable Nix mode via defaults:

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### Config + state paths

OpenClaw reads JSON5 config from `OPENCLAW_CONFIG_PATH` and stores mutable data in `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (default precedence: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (default: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (default: `$OPENCLAW_STATE_DIR/openclaw.json`)

Khi chạy dưới Nix, hãy đặt các biến này một cách rõ ràng vào các vị trí được quản lý bởi Nix để trạng thái runtime và cấu hình
không nằm trong kho lưu trữ bất biến.

### Hành vi runtime trong Nix mode

- Các luồng cài đặt tự động và tự thay đổi bị vô hiệu hóa
- Các phụ thuộc bị thiếu hiển thị các thông báo khắc phục sự cố cụ thể cho Nix
- Giao diện hiển thị một biểu ngữ Nix mode chỉ đọc khi có
## Ghi chú về đóng gói (macOS)

Quy trình đóng gói macOS mong đợi một mẫu Info.plist ổn định tại:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) sao chép mẫu này vào gói ứng dụng và vá các trường động
(ID gói, phiên bản/bản dựng, Git SHA, khóa Sparkle). Điều này giữ cho plist có tính xác định cho đóng gói SwiftPM
và bản dựng Nix (không dựa vào bộ công cụ Xcode đầy đủ).
## Liên quan

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — hướng dẫn thiết lập đầy đủ
- [Wizard](/start/wizard) — thiết lập CLI không dùng Nix
- [Docker](/install/docker) — thiết lập trong container