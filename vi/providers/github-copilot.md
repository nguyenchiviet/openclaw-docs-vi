---
summary: Đăng nhập vào GitHub Copilot từ OpenClaw bằng cách sử dụng device flow
read_when:
  - Bạn muốn sử dụng GitHub Copilot làm nhà cung cấp mô hình
  - Bạn cần luồng `openclaw models auth login-github-copilot`
title: GitHub Copilot
x-i18n:
  source_path: providers\github-copilot.md
  source_hash: 503e0496d92c921e2f7111b1b4ba16374f5b781643bfbc6cb69cea97d9395c25
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:45.733Z'
---

# GitHub Copilot

## GitHub Copilot là gì?

GitHub Copilot là trợ lý mã hóa AI của GitHub. Nó cung cấp quyền truy cập vào các mô hình Copilot cho tài khoản và gói GitHub của bạn. OpenClaw có thể sử dụng Copilot như một nhà cung cấp mô hình theo hai cách khác nhau.
## Hai cách sử dụng Copilot trong OpenClaw

### 1) Nhà cung cấp GitHub Copilot tích hợp sẵn (`github-copilot`)

Sử dụng luồng đăng nhập thiết bị gốc để lấy token GitHub, sau đó trao đổi nó lấy
token API Copilot khi OpenClaw chạy. Đây là đường dẫn **mặc định** và đơn giản nhất
vì nó không yêu cầu VS Code.

### 2) Plugin Copilot Proxy (`copilot-proxy`)

Sử dụng tiện ích mở rộng **Copilot Proxy** VS Code làm cầu nối cục bộ. OpenClaw giao tiếp với
điểm cuối `/v1` của proxy và sử dụng danh sách mô hình bạn cấu hình ở đó. Chọn
tùy chọn này khi bạn đã chạy Copilot Proxy trong VS Code hoặc cần định tuyến qua nó.
Bạn phải bật plugin và giữ tiện ích mở rộng VS Code chạy.

Sử dụng GitHub Copilot làm nhà cung cấp mô hình (`github-copilot`). Lệnh đăng nhập chạy
luồng thiết bị GitHub, lưu hồ sơ xác thực và cập nhật cấu hình của bạn để sử dụng hồ sơ
đó.
## Thiết lập CLI

```bash
openclaw models auth login-github-copilot
```

You'll be prompted to visit a URL and enter a one-time code. Keep the terminal
open until it completes.

### Optional flags

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```
## Đặt mô hình mặc định

```bash
openclaw models set github-copilot/gpt-4o
```

### Config snippet

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```
## Ghi chú

- Yêu cầu TTY tương tác; chạy trực tiếp trong terminal.
- Tính khả dụng của mô hình Copilot phụ thuộc vào gói của bạn; nếu một mô hình bị từ chối, hãy thử
  một ID khác (ví dụ `github-copilot/gpt-4.1`).
- Đăng nhập lưu trữ token GitHub trong kho lưu trữ hồ sơ xác thực và trao đổi nó để lấy token
  API Copilot khi OpenClaw chạy.