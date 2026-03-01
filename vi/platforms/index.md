---
summary: Tổng quan hỗ trợ nền tảng (Gateway + ứng dụng đi kèm)
read_when:
  - Tìm kiếm hỗ trợ hệ điều hành hoặc đường dẫn cài đặt
  - Quyết định nơi chạy Gateway
title: Nền tảng
x-i18n:
  source_path: platforms\index.md
  source_hash: 653f395598b9558cb15b58ab42ed931dba47c70780be1c803d33dd795bad6503
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:15.296Z'
---

# Nền tảng

OpenClaw core được viết bằng TypeScript. **Node là runtime được khuyến nghị**.
Bun không được khuyến nghị cho Gateway (lỗi WhatsApp/Telegram).

Ứng dụng đi kèm tồn tại cho macOS (ứng dụng thanh menu) và các node di động (iOS/Android). Ứng dụng đi kèm cho Windows và
Linux được lên kế hoạch, nhưng Gateway được hỗ trợ đầy đủ ngày hôm nay.
Ứng dụng đi kèm gốc cho Windows cũng được lên kế hoạch; Gateway được khuyến nghị qua WSL2.

## Chọn hệ điều hành của bạn

- macOS: [macOS](/platforms/macos)
- iOS: [iOS](/platforms/ios)
- Android: [Android](/platforms/android)
- Windows: [Windows](/platforms/windows)
- Linux: [Linux](/platforms/linux)

## VPS & hosting

- Hub VPS: [Hosting VPS](/vps)
- Fly.io: [Fly.io](/install/fly)
- Hetzner (Docker): [Hetzner](/install/hetzner)
- GCP (Compute Engine): [GCP](/install/gcp)
- exe.dev (VM + HTTPS proxy): [exe.dev](/install/exe-dev)

## Liên kết phổ biến

- Hướng dẫn cài đặt: [Bắt đầu](/start/getting-started)
- Sổ tay Gateway: [Gateway](/gateway)
- Cấu hình Gateway: [Cấu hình](/gateway/configuration)
- Trạng thái dịch vụ: `openclaw gateway status`

## Cài đặt dịch vụ Gateway (CLI)

Sử dụng một trong những cách sau (tất cả được hỗ trợ):

- Trình hướng dẫn (được khuyến nghị): `openclaw onboard --install-daemon`
- Trực tiếp: `openclaw gateway install`
- Luồng cấu hình: `openclaw configure` → chọn **Dịch vụ Gateway**
- Sửa chữa/di chuyển: `openclaw doctor` (cung cấp tùy chọn cài đặt hoặc sửa chữa dịch vụ)

Mục tiêu dịch vụ phụ thuộc vào hệ điều hành:

- macOS: LaunchAgent (`ai.openclaw.gateway` hoặc `ai.openclaw.<profile>`; cũ `com.openclaw.*`)
- Linux/WSL2: dịch vụ người dùng systemd (`openclaw-gateway[-<profile>].service`)