---
summary: 'Chạy nhiều OpenClaw Gateways trên một máy chủ (cách ly, cổng, và hồ sơ)'
read_when:
  - Chạy nhiều hơn một Gateway trên cùng một máy
  - Bạn cần config/state/ports riêng biệt cho mỗi Gateway
title: Nhiều Gateways
x-i18n:
  source_path: gateway\multiple-gateways.md
  source_hash: 493bd45bc6939ae7328afcd0351ca7cb4c93c17b819e8ad0fdf0f0312bf9b639
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:58:26.169Z'
---

# Nhiều Gateway (cùng một máy chủ)

Hầu hết các thiết lập nên sử dụng một Gateway vì một Gateway duy nhất có thể xử lý nhiều kết nối nhắn tin và agent. Nếu bạn cần cách ly mạnh hơn hoặc dự phòng (ví dụ: một bot cứu hộ), hãy chạy các Gateway riêng biệt với các hồ sơ/cổng cách ly.
## Danh sách kiểm tra cách ly (bắt buộc)

- `OPENCLAW_CONFIG_PATH` — tệp cấu hình cho mỗi instance
- `OPENCLAW_STATE_DIR` — phiên, thông tin xác thực, bộ nhớ cache cho mỗi instance
- `agents.defaults.workspace` — thư mục gốc workspace cho mỗi instance
- `gateway.port` (hoặc `--port`) — duy nhất cho mỗi instance
- Các cổng dẫn xuất (trình duyệt/canvas) không được trùng lặp

Nếu những cái này được chia sẻ, bạn sẽ gặp phải các cuộc đua cấu hình và xung đột cổng.
## Khuyến nghị: profiles (`--profile`)

Profiles tự động xác định phạm vi `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` và thêm hậu tố vào tên dịch vụ.

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Per-profile services:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```
## Hướng dẫn Rescue-bot

Chạy một Gateway thứ hai trên cùng một máy chủ với:

- profile/config riêng
- thư mục state riêng
- workspace riêng
- base port riêng (cộng với các port dẫn xuất)

Điều này giữ cho rescue bot cách ly khỏi bot chính để nó có thể gỡ lỗi hoặc áp dụng các thay đổi cấu hình nếu bot chính bị down.

Khoảng cách port: để lại ít nhất 20 port giữa các base port để các port browser/canvas/CDP dẫn xuất không bao giờ xung đột.

### Cách cài đặt (rescue bot)

```bash
# Main bot (existing or fresh, without --profile param)
# Runs on port 18789 + Chrome CDC/Canvas/... Ports
openclaw onboard
openclaw gateway install

# Rescue bot (isolated profile + ports)
openclaw --profile rescue onboard
# Notes:
# - workspace name will be postfixed with -rescue per default
# - Port should be at least 18789 + 20 Ports,
#   better choose completely different base port, like 19789,
# - rest of the onboarding is the same as normal

# To install the service (if not happened automatically during onboarding)
openclaw --profile rescue gateway install
```
## Ánh xạ cổng (dẫn xuất)

Cổng cơ sở = `gateway.port` (hoặc `OPENCLAW_GATEWAY_PORT` / `--port`).

- cổng dịch vụ điều khiển trình duyệt = cơ sở + 2 (chỉ local loopback)
- canvas host được phục vụ trên máy chủ HTTP Gateway (cùng cổng với `gateway.port`)
- Các cổng CDP hồ sơ trình duyệt tự động cấp phát từ `browser.controlPort + 9 .. + 108`

Nếu bạn ghi đè bất kỳ cái nào trong cấu hình hoặc biến môi trường, bạn phải giữ chúng duy nhất cho mỗi instance.
## Ghi chú về Browser/CDP (lỗi thường gặp)

- **Không** ghim `browser.cdpUrl` vào các giá trị giống nhau trên nhiều instance.
- Mỗi instance cần cổng điều khiển trình duyệt riêng và phạm vi CDP (được lấy từ cổng gateway của nó).
- Nếu bạn cần các cổng CDP rõ ràng, hãy đặt `browser.profiles.<name>.cdpPort` cho mỗi instance.
- Chrome từ xa: sử dụng `browser.profiles.<name>.cdpUrl` (cho mỗi hồ sơ, cho mỗi instance).
## Ví dụ biến môi trường thủ công

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```
## Kiểm tra nhanh

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```