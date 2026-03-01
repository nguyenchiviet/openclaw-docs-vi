---
summary: 'Giao thức Bridge (các nút cũ): TCP JSONL, ghép cặp, RPC có phạm vi'
read_when:
  - Xây dựng hoặc gỡ lỗi các node client (chế độ node iOS/Android/macOS)
  - Điều tra các lỗi xác thực pairing hoặc bridge
  - Kiểm toán bề mặt nút được phơi bày bởi gateway
title: Giao thức Bridge
x-i18n:
  source_path: gateway\bridge-protocol.md
  source_hash: 2923c683c77336b92b62dc6d66fa3ff5d4b4fac56c4ec2dc2cca8d8bbcc5d834
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:56:10.702Z'
---

# Bridge protocol (giao thức truyền tải node cũ)

Bridge protocol là một giao thức truyền tải node **cũ** (TCP JSONL). Các client node mới
nên sử dụng giao thức Gateway WebSocket thống nhất thay thế.

Nếu bạn đang xây dựng một operator hoặc node client, hãy sử dụng
[Gateway protocol](/gateway/protocol).

**Lưu ý:** Các bản dựng OpenClaw hiện tại không còn cung cấp TCP bridge listener; tài liệu này được giữ lại cho mục đích tham khảo lịch sử.
Các khóa cấu hình `bridge.*` cũ không còn là một phần của lược đồ cấu hình.
## Tại sao chúng tôi có cả hai

- **Ranh giới bảo mật**: cầu nối chỉ tiếp xúc một danh sách cho phép nhỏ thay vì toàn bộ bề mặt API của gateway.
- **Ghép nối + nhận dạng node**: việc chấp nhận node được sở hữu bởi gateway và liên kết với một token dành riêng cho từng node.
- **Trải nghiệm khám phá**: các node có thể khám phá gateway thông qua Bonjour trên LAN hoặc kết nối trực tiếp qua tailnet.
- **Loopback WS**: toàn bộ mặt phẳng điều khiển WS vẫn ở cục bộ trừ khi được đường hầm qua SSH.
## Giao thức truyền tải

- TCP, một đối tượng JSON trên mỗi dòng (JSONL).
- TLS tùy chọn (khi `bridge.tls.enabled` là true).
- Cổng listener mặc định cũ là `18790` (các bản dựng hiện tại không khởi động một cầu nối TCP).

Khi TLS được bật, các bản ghi khám phá TXT bao gồm `bridgeTls=1` cộng với
`bridgeTlsSha256` như một gợi ý không bí mật. Lưu ý rằng các bản ghi TXT Bonjour/mDNS
không được xác thực; các máy khách không được coi dấu vân tay được quảng cáo là một
ghim có thẩm quyền mà không có ý định người dùng rõ ràng hoặc xác minh ngoài băng khác.
## Bắt tay + ghép nối

1. Client gửi `hello` với siêu dữ liệu node + token (nếu đã ghép nối).
2. Nếu chưa ghép nối, gateway trả lời `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. Client gửi `pair-request`.
4. Gateway chờ phê duyệt, sau đó gửi `pair-ok` và `hello-ok`.

`hello-ok` trả về `serverName` và có thể bao gồm `canvasHostUrl`.
## Frames

Client → Gateway:

- `req` / `res`: RPC gateway có phạm vi (chat, sessions, config, health, voicewake, skills.bins)
- `event`: tín hiệu node (voice transcript, agent request, chat subscribe, exec lifecycle)

Gateway → Client:

- `invoke` / `invoke-res`: lệnh node (`canvas.*`, `camera.*`, `screen.record`,
  `location.get`, `sms.send`)
- `event`: cập nhật chat cho các phiên đã đăng ký
- `ping` / `pong`: keepalive

Thực thi danh sách cho phép cũ nằm trong `src/gateway/server-bridge.ts` (đã xóa).
## Sự kiện vòng đời Exec

Các node có thể phát ra sự kiện `exec.finished` hoặc `exec.denied` để hiển thị hoạt động system.run.
Những sự kiện này được ánh xạ tới các sự kiện hệ thống trong Gateway. (Các node cũ có thể vẫn phát ra `exec.started`.)

Các trường payload (tất cả tùy chọn trừ khi được ghi chú):

- `sessionKey` (bắt buộc): phiên agent để nhận sự kiện hệ thống.
- `runId`: id exec duy nhất để nhóm.
- `command`: chuỗi lệnh thô hoặc được định dạng.
- `exitCode`, `timedOut`, `success`, `output`: chi tiết hoàn thành (chỉ khi hoàn thành).
- `reason`: lý do từ chối (chỉ khi bị từ chối).
## Sử dụng Tailnet

- Liên kết bridge với địa chỉ IP tailnet: `bridge.bind: "tailnet"` trong
  `~/.openclaw/openclaw.json`.
- Các client kết nối qua tên MagicDNS hoặc địa chỉ IP tailnet.
- Bonjour **không** hoạt động trên các mạng khác nhau; sử dụng host/port thủ công hoặc DNS‑SD toàn vùng
  khi cần thiết.
## Phiên bản

Bridge hiện đang ở **phiên bản 1 ngầm định** (không có thương lượng min/max). Khả năng tương thích ngược được mong đợi; hãy thêm trường phiên bản giao thức bridge trước bất kỳ thay đổi phá vỡ nào.