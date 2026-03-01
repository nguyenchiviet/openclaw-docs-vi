---
summary: >-
  Plugin Zalo Personal: Đăng nhập QR + nhắn tin qua zca-cli (cài đặt plugin +
  cấu hình kênh + CLI + công cụ)
read_when:
  - Bạn muốn hỗ trợ Zalo Personal (không chính thức) trong OpenClaw
  - Bạn đang cấu hình hoặc phát triển plugin zalouser
title: Plugin Zalo Cá nhân
x-i18n:
  source_path: plugins\zalouser.md
  source_hash: b29b788b023cd50720e24fe6719f02e9f86c8bca9c73b3638fb53c2316718672
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:18.800Z'
---

# Zalo Personal (plugin)

Hỗ trợ Zalo Personal cho OpenClaw thông qua một plugin, sử dụng `zca-cli` để tự động hóa một tài khoản người dùng Zalo thông thường.

> **Cảnh báo:** Tự động hóa không chính thức có thể dẫn đến tạm khóa/cấm tài khoản. Sử dụng tại rủi ro của bạn.
## Đặt tên

Channel id là `zalouser` để làm rõ ràng rằng điều này tự động hóa một **tài khoản người dùng Zalo cá nhân** (không chính thức). Chúng tôi giữ `zalo` dành riêng cho một tích hợp API Zalo chính thức tiềm năng trong tương lai.
## Nơi nó chạy

Plugin này chạy **bên trong quy trình Gateway**.

Nếu bạn sử dụng Gateway từ xa, hãy cài đặt/cấu hình nó trên **máy chạy Gateway**, sau đó khởi động lại Gateway.
## Cài đặt

### Tùy chọn A: cài đặt từ npm

```bash
openclaw plugins install @openclaw/zalouser
```

Restart the Gateway afterwards.

### Option B: install from a local folder (dev)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Khởi động lại Gateway sau đó.
## Điều kiện tiên quyết: zca-cli

Máy Gateway phải có `zca` trên `PATH`:

```bash
zca --version
```
## Cấu hình

Cấu hình kênh nằm dưới `channels.zalouser` (không phải `plugins.entries.*`):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```
## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```
## Agent tool

Tool name: `zalouser`

Actions: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`