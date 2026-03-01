---
summary: >-
  Khám phá nút và các phương thức truyền tải (Bonjour, Tailscale, SSH) để tìm
  gateway
read_when:
  - Triển khai hoặc thay đổi khám phá/quảng cáo Bonjour
  - Điều chỉnh chế độ kết nối từ xa (trực tiếp vs SSH)
  - Thiết kế khám phá nút + ghép nối cho các nút từ xa
title: Khám Phá và Vận Chuyển
x-i18n:
  source_path: gateway\discovery.md
  source_hash: 98e914633522a4cf88e191c04d4f4a8b8d55513715a7dfab2700c9ddd042ea09
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:56:56.969Z'
---

# Khám phá + giao thức truyền tải

OpenClaw có hai vấn đề riêng biệt trông giống nhau ở bề ngoài:

1. **Điều khiển từ xa của Operator**: ứng dụng thanh menu macOS điều khiển một gateway chạy ở nơi khác.
2. **Ghép nối Node**: iOS/Android (và các node trong tương lai) tìm kiếm gateway và ghép nối một cách an toàn.

Mục tiêu thiết kế là giữ tất cả khám phá/quảng cáo mạng trong **Node Gateway** (`openclaw gateway`) và giữ các client (ứng dụng mac, iOS) như những người tiêu dùng.
## Điều khoản

- **Gateway**: một quy trình gateway dài hạn duy nhất sở hữu trạng thái (phiên, ghép nối, sổ đăng ký node) và chạy các kênh. Hầu hết các thiết lập sử dụng một trên mỗi máy chủ; các thiết lập multi-gateway cách ly là có thể.
- **Gateway WS (control plane)**: điểm cuối WebSocket trên `127.0.0.1:18789` theo mặc định; có thể được liên kết với LAN/tailnet thông qua `gateway.bind`.
- **Direct WS transport**: điểm cuối Gateway WS hướng tới LAN/tailnet (không SSH).
- **SSH transport (fallback)**: điều khiển từ xa bằng cách chuyển tiếp `127.0.0.1:18789` qua SSH.
- **Legacy TCP bridge (deprecated/removed)**: giao thức node truyền tải cũ (xem [Bridge protocol](/gateway/bridge-protocol)); không còn được quảng cáo cho khám phá thiết bị.

Chi tiết giao thức:

- [Gateway protocol](/gateway/protocol)
- [Bridge protocol](/gateway/bridge-protocol)
## Tại sao chúng tôi giữ cả "direct" và SSH

- **Direct WS** là trải nghiệm người dùng tốt nhất trên cùng một mạng và trong một tailnet:
  - khám phá tự động trên LAN qua Bonjour
  - mã token ghép nối + ACL được sở hữu bởi gateway
  - không cần quyền truy cập shell; bề mặt giao thức có thể giữ ngắn gọn và có thể kiểm toán
- **SSH** vẫn là giải pháp dự phòng phổ quát:
  - hoạt động ở bất kỳ nơi nào bạn có quyền truy cập SSH (thậm chí trên các mạng không liên quan)
  - vượt qua các vấn đề multicast/mDNS
  - không yêu cầu cổng inbound mới ngoài SSH
## Khám phá thiết bị - đầu vào (cách khách hàng tìm hiểu vị trí của gateway)

### 1) Bonjour / mDNS (chỉ LAN)

Bonjour là best-effort và không vượt qua các mạng. Nó chỉ được sử dụng để thuận tiện cho "cùng LAN".

Hướng mục tiêu:

- **Gateway** quảng cáo điểm cuối WS của nó thông qua Bonjour.
- Khách hàng duyệt và hiển thị danh sách "chọn gateway", sau đó lưu trữ điểm cuối đã chọn.

Khắc phục sự cố và chi tiết beacon: [Bonjour](/gateway/bonjour).

#### Chi tiết beacon dịch vụ

- Loại dịch vụ:
  - `_openclaw-gw._tcp` (beacon vận chuyển gateway)
- Khóa TXT (không bí mật):
  - `role=gateway`
  - `lanHost=<hostname>.local`
  - `sshPort=22` (hoặc bất kỳ cái nào được quảng cáo)
  - `gatewayPort=18789` (Gateway WS + HTTP)
  - `gatewayTls=1` (chỉ khi TLS được bật)
  - `gatewayTlsSha256=<sha256>` (chỉ khi TLS được bật và dấu vân tay có sẵn)
  - `canvasPort=<port>` (cổng máy chủ canvas; hiện tại giống như `gatewayPort` khi máy chủ canvas được bật)
  - `cliPath=<path>` (tùy chọn; đường dẫn tuyệt đối đến điểm vào `openclaw` có thể chạy được hoặc nhị phân)
  - `tailnetDns=<magicdns>` (gợi ý tùy chọn; tự động phát hiện khi Tailscale có sẵn)

Ghi chú bảo mật:

- Bản ghi TXT Bonjour/mDNS **không được xác thực**. Khách hàng phải coi các giá trị TXT chỉ là gợi ý UX.
- Định tuyến (máy chủ/cổng) nên ưu tiên **điểm cuối dịch vụ được phân giải** (SRV + A/AAAA) hơn `lanHost`, `tailnetDns` hoặc `gatewayPort` được cung cấp bởi TXT.
- Ghim TLS không bao giờ được phép cho phép `gatewayTlsSha256` được quảng cáo ghi đè lên một chân được lưu trữ trước đó.
- Các nút iOS/Android nên coi các kết nối trực tiếp dựa trên khám phá thiết bị là **chỉ TLS** và yêu cầu xác nhận rõ ràng "tin tưởng dấu vân tay này" trước khi lưu trữ chân lần đầu tiên (xác minh ngoài băng).

Vô hiệu hóa/ghi đè:

- `OPENCLAW_DISABLE_BONJOUR=1` vô hiệu hóa quảng cáo.
- `gateway.bind` trong `~/.openclaw/openclaw.json` kiểm soát chế độ liên kết Gateway.
- `OPENCLAW_SSH_PORT` ghi đè cổng SSH được quảng cáo trong TXT (mặc định là 22).
- `OPENCLAW_TAILNET_DNS` xuất bản gợi ý `tailnetDns` (MagicDNS).
- `OPENCLAW_CLI_PATH` ghi đè đường dẫn CLI được quảng cáo.

### 2) Tailnet (xuyên mạng)

Đối với các thiết lập kiểu London/Vienna, Bonjour sẽ không giúp được. Mục tiêu "trực tiếp" được khuyến nghị là:

- Tên MagicDNS Tailscale (ưu tiên) hoặc IP tailnet ổn định.

Nếu gateway có thể phát hiện nó đang chạy dưới Tailscale, nó sẽ xuất bản `tailnetDns` làm gợi ý tùy chọn cho khách hàng (bao gồm cả beacon khu vực rộng).

### 3) Mục tiêu SSH / Thủ công

Khi không có tuyến đường trực tiếp (hoặc trực tiếp bị vô hiệu hóa), khách hàng luôn có thể kết nối qua SSH bằng cách chuyển tiếp cổng gateway local loopback.

Xem [Truy cập từ xa](/gateway/remote).
## Lựa chọn giao thức truyền tải (chính sách máy khách)

Hành vi máy khách được khuyến nghị:

1. Nếu một điểm cuối trực tiếp được ghép nối được cấu hình và có thể truy cập được, hãy sử dụng nó.
2. Nếu không, nếu Bonjour tìm thấy một gateway trên LAN, hãy cung cấp lựa chọn "Sử dụng gateway này" chỉ cần một lần nhấn và lưu nó làm điểm cuối trực tiếp.
3. Nếu không, nếu DNS/IP tailnet được cấu hình, hãy thử trực tiếp.
4. Nếu không, hãy quay lại SSH.
## Ghép nối + xác thực (giao thức truyền tải trực tiếp)

Gateway là nguồn sự thật cho việc chấp nhận node/client.

- Các yêu cầu ghép nối được tạo/phê duyệt/từ chối trong gateway (xem [Gateway pairing](/gateway/pairing)).
- Gateway thực thi:
  - xác thực (token / cặp khóa)
  - scopes/ACLs (gateway không phải là proxy thô cho mọi phương thức)
  - giới hạn tốc độ
## Trách nhiệm theo thành phần

- **Gateway**: quảng cáo các beacon khám phá, quản lý quyết định ghép nối và lưu trữ điểm cuối WS.
- **ứng dụng macOS**: giúp bạn chọn gateway, hiển thị các lời nhắc ghép nối và chỉ sử dụng SSH như một giải pháp dự phòng.
- **các node iOS/Android**: duyệt Bonjour để thuận tiện và kết nối đến Gateway WS được ghép nối.