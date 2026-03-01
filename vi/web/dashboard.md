---
summary: Truy cập và xác thực bảng điều khiển Gateway (Control UI)
read_when:
  - Thay đổi xác thực bảng điều khiển hoặc chế độ hiển thị
title: Bảng điều khiển
x-i18n:
  source_path: web\dashboard.md
  source_hash: c1243234a86418744cff728c87e5fcd6d1584bab862889d1e94fe3ebad0939cf
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:53.880Z'
---

# Bảng điều khiển (Control UI)

Bảng điều khiển Gateway là Control UI được phục vụ trong trình duyệt tại `/` theo mặc định
(ghi đè bằng `gateway.controlUi.basePath`).

Mở nhanh (Gateway cục bộ):

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (hoặc [http://localhost:18789/](http://localhost:18789/))

Tài liệu tham khảo chính:

- [Control UI](/web/control-ui) để sử dụng và khả năng giao diện.
- [Tailscale](/gateway/tailscale) để tự động hóa Serve/Funnel.
- [Web surfaces](/web) để biết các chế độ liên kết và ghi chú bảo mật.

Xác thực được thực thi tại bắt tay WebSocket thông qua `connect.params.auth`
(token hoặc mật khẩu). Xem `gateway.auth` trong [Cấu hình Gateway](/gateway/configuration).

Ghi chú bảo mật: Control UI là một **bề mặt quản trị** (chat, cấu hình, phê duyệt thực thi).
Không để lộ công khai. Giao diện lưu trữ token trong `localStorage` sau lần tải đầu tiên.
Ưu tiên localhost, Tailscale Serve, hoặc đường hầm SSH.

## Đường dẫn nhanh (được khuyến nghị)

- Sau khi thiết lập ban đầu, CLI tự động mở bảng điều khiển và in một liên kết sạch (không được token hóa).
- Mở lại bất kỳ lúc nào: `openclaw dashboard` (sao chép liên kết, mở trình duyệt nếu có thể, hiển thị gợi ý SSH nếu không có giao diện).
- Nếu giao diện yêu cầu xác thực, dán token từ `gateway.auth.token` (hoặc `OPENCLAW_GATEWAY_TOKEN`) vào cài đặt Control UI.

## Cơ bản về token (cục bộ so với từ xa)

- **Localhost**: mở `http://127.0.0.1:18789/`.
- **Nguồn token**: `gateway.auth.token` (hoặc `OPENCLAW_GATEWAY_TOKEN`); giao diện lưu trữ một bản sao trong localStorage sau khi bạn kết nối.
- **Không phải localhost**: sử dụng Tailscale Serve (không cần token cho Control UI/WebSocket nếu `gateway.auth.allowTailscale: true`, giả định host gateway đáng tin cậy; các API HTTP vẫn cần token/mật khẩu), liên kết tailnet với token, hoặc đường hầm SSH. Xem [Web surfaces](/web).

## Nếu bạn thấy "unauthorized" / 1008

- Đảm bảo gateway có thể truy cập được (cục bộ: `openclaw status`; từ xa: đường hầm SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` rồi mở `http://127.0.0.1:18789/`).
- Lấy token từ host gateway: `openclaw config get gateway.auth.token` (hoặc tạo một token: `openclaw doctor --generate-gateway-token`).
- Trong cài đặt bảng điều khiển, dán token vào trường xác thực, rồi kết nối.