---
summary: >-
  Hướng dẫn thiết lập cho các nhà phát triển làm việc trên ứng dụng OpenClaw
  macOS
read_when:
  - Thiết lập môi trường phát triển macOS
title: Thiết lập Phát triển macOS
x-i18n:
  source_path: platforms\mac\dev-setup.md
  source_hash: 1cb41c449e4a314f6327c63458006aeb4d7723f093f3e3866a76f5ffaa00e2d3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:54.365Z'
---

# Thiết lập Nhà phát triển macOS

Hướng dẫn này bao gồm các bước cần thiết để xây dựng và chạy ứng dụng OpenClaw macOS từ mã nguồn.
## Điều kiện tiên quyết

Trước khi xây dựng ứng dụng, hãy đảm bảo bạn đã cài đặt những điều sau:

1. **Xcode 26.2+**: Bắt buộc để phát triển Swift.
2. **Node.js 22+ & pnpm**: Bắt buộc cho Gateway, CLI và các tập lệnh đóng gói.
## 1. Cài đặt Phụ thuộc

Cài đặt các phụ thuộc toàn dự án:

```bash
pnpm install
```
## 2. Xây dựng và Đóng gói Ứng dụng

Để xây dựng ứng dụng macOS và đóng gói nó thành `dist/OpenClaw.app`, hãy chạy:

```bash
./scripts/package-mac-app.sh
```

If you don't have an Apple Developer ID certificate, the script will automatically use **ad-hoc signing** (`-`).

Để biết thêm về các chế độ chạy dev, cờ ký và khắc phục sự cố Team ID, hãy xem README ứng dụng macOS:
[https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **Lưu ý**: Các ứng dụng được ký ad-hoc có thể kích hoạt các lời nhắc bảo mật. Nếu ứng dụng bị sập ngay lập tức với "Abort trap 6", hãy xem phần [Khắc phục sự cố](#troubleshooting).
## 3. Cài đặt CLI

Ứng dụng macOS mong đợi cài đặt CLI `openclaw` toàn cục để quản lý các tác vụ nền.

**Để cài đặt nó (được khuyến nghị):**

1. Mở ứng dụng OpenClaw.
2. Đi tới tab cài đặt **General**.
3. Nhấp vào **"Install CLI"**.

Ngoài ra, cài đặt nó theo cách thủ công:

```bash
npm install -g openclaw@<version>
```
## Khắc phục sự cố

### Build Fails: Toolchain or SDK Mismatch

Bản dựng ứng dụng macOS yêu cầu macOS SDK mới nhất và bộ công cụ Swift 6.2.

**Các phụ thuộc hệ thống (bắt buộc):**

- **Phiên bản macOS mới nhất có sẵn trong Software Update** (yêu cầu bởi Xcode 26.2 SDKs)
- **Xcode 26.2** (bộ công cụ Swift 6.2)

**Kiểm tra:**

```bash
xcodebuild -version
xcrun swift --version
```

If versions don’t match, update macOS/Xcode and re-run the build.

### App Crashes on Permission Grant

If the app crashes when you try to allow **Speech Recognition** or **Microphone** access, it may be due to a corrupted TCC cache or signature mismatch.

**Fix:**

1. Reset the TCC permissions:

   ```bash
   tccutil reset All ai.openclaw.mac.debug
   ```

2. If that fails, change the `BUNDLE_ID` temporarily in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) to force a "clean slate" from macOS.

### Gateway "Starting..." indefinitely

If the gateway status stays on "Starting...", check if a zombie process is holding the port:

```bash
openclaw gateway status
openclaw gateway stop

# If you’re not using a LaunchAgent (dev mode / manual runs), find the listener:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Nếu một quá trình chạy thủ công đang chiếm cổng, hãy dừng quá trình đó (Ctrl+C). Như một giải pháp cuối cùng, hãy kết thúc PID bạn tìm thấy ở trên.