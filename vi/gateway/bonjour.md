---
summary: >-
  Bonjour/mDNS discovery + debugging (Gateway beacons, clients, and common
  failure modes)
read_when:
  - Khắc phục sự cố phát hiện Bonjour trên macOS/iOS
  - 'Thay đổi loại dịch vụ mDNS, bản ghi TXT hoặc trải nghiệm khám phá'
title: Xin chào Discovery
x-i18n:
  source_path: gateway\bonjour.md
  source_hash: 2454692d8d590506fe224d92e4aa6ddf2838a2a11e64529da556c7bd3e35a90c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:56:20.390Z'
---

# Khám phá Bonjour / mDNS

OpenClaw sử dụng Bonjour (mDNS / DNS‑SD) như một **tiện ích chỉ dành cho LAN** để khám phá một Gateway đang hoạt động (điểm cuối WebSocket). Đây là phương pháp tốt nhất và **không** thay thế kết nối dựa trên SSH hoặc Tailnet.
## Wide‑area Bonjour (Unicast DNS‑SD) trên Tailscale

Nếu node và gateway ở trên các mạng khác nhau, multicast mDNS sẽ không vượt qua
ranh giới. Bạn có thể giữ nguyên trải nghiệm khám phá thiết bị bằng cách chuyển sang **unicast DNS‑SD**
("Wide‑Area Bonjour") trên Tailscale.

Các bước cấp cao:

1. Chạy máy chủ DNS trên máy chủ gateway (có thể truy cập được qua Tailnet).
2. Xuất bản các bản ghi DNS‑SD cho `_openclaw-gw._tcp` dưới một vùng chuyên dụng
   (ví dụ: `openclaw.internal.`).
3. Cấu hình Tailscale **split DNS** để miền bạn chọn được phân giải qua
   máy chủ DNS đó cho các máy khách (bao gồm iOS).

OpenClaw hỗ trợ bất kỳ miền khám phá nào; `openclaw.internal.` chỉ là một ví dụ.
Các node iOS/Android duyệt cả `local.` và miền wide‑area được cấu hình của bạn.

### Cấu hình Gateway (được khuyến nghị)

```json5
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } }, // enables wide-area DNS-SD publishing
}
```

### One‑time DNS server setup (gateway host)

```bash
openclaw dns setup --apply
```

This installs CoreDNS and configures it to:

- listen on port 53 only on the gateway’s Tailscale interfaces
- serve your chosen domain (example: `openclaw.internal.`) from `~/.openclaw/dns/<domain>.db`

Validate from a tailnet‑connected machine:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Tailscale DNS settings

In the Tailscale admin console:

- Add a nameserver pointing at the gateway’s tailnet IP (UDP/TCP 53).
- Add split DNS so your discovery domain uses that nameserver.

Once clients accept tailnet DNS, iOS nodes can browse
`_openclaw-gw._tcp` in your discovery domain without multicast.

### Gateway listener security (recommended)

The Gateway WS port (default `18789`) mặc định liên kết với local loopback. Để truy cập LAN/tailnet,
liên kết một cách rõ ràng và giữ xác thực được bật.

Đối với các thiết lập chỉ tailnet:
- Đặt `gateway.bind: "tailnet"` trong `~/.openclaw/openclaw.json`.
- Khởi động lại Gateway (hoặc khởi động lại ứng dụng thanh menu macOS).
## Những gì quảng cáo

Chỉ có Gateway quảng cáo `_openclaw-gw._tcp`.
## Loại dịch vụ

- `_openclaw-gw._tcp` — beacon giao thức truyền tải Gateway (được sử dụng bởi các node macOS/iOS/Android).
## Khóa TXT (gợi ý không bí mật)

Gateway quảng cáo các gợi ý nhỏ không bí mật để làm cho các luồng giao diện người dùng thuận tiện:

- `role=gateway`
- `displayName=<friendly name>`
- `lanHost=<hostname>.local`
- `gatewayPort=<port>` (Gateway WS + HTTP)
- `gatewayTls=1` (chỉ khi TLS được bật)
- `gatewayTlsSha256=<sha256>` (chỉ khi TLS được bật và dấu vân tay có sẵn)
- `canvasPort=<port>` (chỉ khi canvas host được bật; hiện tại giống như `gatewayPort`)
- `sshPort=<port>` (mặc định là 22 khi không được ghi đè)
- `transport=gateway`
- `cliPath=<path>` (tùy chọn; đường dẫn tuyệt đối đến điểm vào `openclaw` có thể chạy được)
- `tailnetDns=<magicdns>` (gợi ý tùy chọn khi Tailnet có sẵn)

Ghi chú bảo mật:

- Các bản ghi TXT Bonjour/mDNS **không được xác thực**. Các client không được coi TXT là định tuyến có thẩm quyền.
- Các client nên định tuyến bằng cách sử dụng điểm cuối dịch vụ đã giải quyết (SRV + A/AAAA). Coi `lanHost`, `tailnetDns`, `gatewayPort` và `gatewayTlsSha256` chỉ là gợi ý.
- Ghim TLS không bao giờ được phép cho phép `gatewayTlsSha256` được quảng cáo ghi đè lên một chân được lưu trữ trước đó.
- Các node iOS/Android nên coi các kết nối trực tiếp dựa trên khám phá là **chỉ TLS** và yêu cầu xác nhận rõ ràng của người dùng trước khi tin tưởng một dấu vân tay lần đầu tiên.
## Gỡ lỗi trên macOS

Các công cụ tích hợp hữu ích:

- Duyệt các instance:

  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```

- Resolve one instance (replace `<instance>`):

  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

Nếu duyệt hoạt động nhưng phân giải thất bại, bạn thường gặp phải vấn đề về chính sách LAN hoặc bộ phân giải mDNS.
## Gỡ lỗi trong nhật ký Gateway

Gateway ghi một tệp nhật ký lăn (được in khi khởi động là
`gateway log file: ...`). Tìm các dòng `bonjour:`, đặc biệt:

- `bonjour: advertise failed ...`
- `bonjour: ... name conflict resolved` / `hostname conflict resolved`
- `bonjour: watchdog detected non-announced service ...`
## Gỡ lỗi trên node iOS

Node iOS sử dụng `NWBrowser` để khám phá `_openclaw-gw._tcp`.

Để ghi lại nhật ký:

- Settings → Gateway → Advanced → **Discovery Debug Logs**
- Settings → Gateway → Advanced → **Discovery Logs** → reproduce → **Copy**

Nhật ký bao gồm các chuyển đổi trạng thái trình duyệt và thay đổi tập kết quả.
## Các chế độ lỗi phổ biến

- **Bonjour không hoạt động trên các mạng khác nhau**: sử dụng Tailnet hoặc SSH.
- **Multicast bị chặn**: một số mạng Wi‑Fi vô hiệu hóa mDNS.
- **Sleep / interface churn**: macOS có thể tạm thời loại bỏ kết quả mDNS; hãy thử lại.
- **Browse hoạt động nhưng resolve thất bại**: giữ tên máy đơn giản (tránh emoji hoặc dấu câu), sau đó khởi động lại Gateway. Tên instance dịch vụ được lấy từ tên máy chủ, vì vậy các tên quá phức tạp có thể làm nhầm lẫn một số resolver.
## Tên instance được escape (`\032`)

Bonjour/DNS‑SD thường escape các byte trong tên instance dịch vụ dưới dạng các chuỗi `\DDD`
thập phân (ví dụ: khoảng trắng trở thành `\032`).

- Đây là điều bình thường ở cấp độ giao thức.
- Giao diện người dùng nên giải mã để hiển thị (iOS sử dụng `BonjourEscapes.decode`).
## Vô hiệu hóa / cấu hình

- `OPENCLAW_DISABLE_BONJOUR=1` vô hiệu hóa quảng cáo (cũ: `OPENCLAW_DISABLE_BONJOUR`).
- `gateway.bind` trong `~/.openclaw/openclaw.json` kiểm soát chế độ liên kết Gateway.
- `OPENCLAW_SSH_PORT` ghi đè cổng SSH được quảng cáo trong TXT (cũ: `OPENCLAW_SSH_PORT`).
- `OPENCLAW_TAILNET_DNS` xuất bản gợi ý MagicDNS trong TXT (cũ: `OPENCLAW_TAILNET_DNS`).
- `OPENCLAW_CLI_PATH` ghi đè đường dẫn CLI được quảng cáo (cũ: `OPENCLAW_CLI_PATH`).
## Tài liệu liên quan

- Chính sách khám phá thiết bị và lựa chọn giao thức truyền tải: [Khám phá thiết bị](/gateway/discovery)
- Ghép nối node + phê duyệt: [Ghép nối Gateway](/gateway/pairing)