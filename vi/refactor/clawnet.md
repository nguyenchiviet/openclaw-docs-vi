---
summary: >-
  Clawnet tái cấu trúc: thống nhất giao thức mạng, vai trò, xác thực, phê duyệt,
  danh tính
read_when:
  - >-
    Lập kế hoạch cho một giao thức mạng thống nhất cho các nút và máy khách toán
    tử
  - >-
    Sắp xếp lại phê duyệt, ghép nối, TLS và trạng thái hiện diện trên các thiết
    bị
title: Cấu trúc lại Clawnet
x-i18n:
  source_path: refactor\clawnet.md
  source_hash: 719b219c3b326479658fe6101c80d5273fc56eb3baf50be8535e0d1d2bb7987f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:19:14.699Z'
---

# Clawnet refactor (protocol + auth unification)

## Xin chào

Xin chào Peter — hướng đi tuyệt vời; điều này mở khóa UX đơn giản hơn + bảo mật mạnh hơn.
## Mục đích

Tài liệu duy nhất, chặt chẽ cho:

- Trạng thái hiện tại: giao thức, luồng, ranh giới tin cậy.
- Điểm đau: phê duyệt, định tuyến đa bước, trùng lặp giao diện.
- Trạng thái mới được đề xuất: một giao thức, vai trò có phạm vi, xác thực/ghép nối thống nhất, ghim TLS.
- Mô hình nhận dạng: ID ổn định + slug dễ thương.
- Kế hoạch di chuyển, rủi ro, câu hỏi mở.
## Mục tiêu (từ cuộc thảo luận)

- Một giao thức cho tất cả các client (ứng dụng Mac, CLI, iOS, Android, node không giao diện).
- Mọi người tham gia mạng được xác thực + ghép nối.
- Rõ ràng về vai trò: node vs operator.
- Phê duyệt trung tâm được định tuyến đến nơi người dùng ở.
- Mã hóa TLS + ghim tùy chọn cho tất cả lưu lượng từ xa.
- Giảm thiểu trùng lặp mã.
- Một máy duy nhất nên xuất hiện một lần (không có mục nhập trùng lặp UI/node).
## Mục tiêu không đạt (rõ ràng)

- Loại bỏ tách biệt khả năng (vẫn cần quyền tối thiểu).
- Tiếp xúc toàn bộ mặt phẳng điều khiển gateway mà không kiểm tra phạm vi.
- Làm cho xác thực phụ thuộc vào nhãn của con người (các slug vẫn không phải bảo mật).
# Trạng thái hiện tại (hiện trạng)

## Hai giao thức

### 1) Gateway WebSocket (mặt phẳng điều khiển)

- Bề mặt API đầy đủ: cấu hình, kênh, mô hình, phiên, chạy agent, nhật ký, node, v.v.
- Liên kết mặc định: local loopback. Truy cập từ xa qua SSH/Tailscale.
- Xác thực: token/mật khẩu qua `connect`.
- Không có ghim TLS (dựa vào local loopback/tunnel).
- Mã:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

### 2) Bridge (giao thức truyền tải node)

- Bề mặt danh sách cho phép hẹp, nhận dạng node + ghép nối.
- JSONL qua TCP; TLS tùy chọn + ghim dấu vân tay chứng chỉ.
- TLS quảng cáo dấu vân tay trong TXT khám phá.
- Mã:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`
## Các client mặt phẳng điều khiển ngày nay

- CLI → Gateway WS qua `callGateway` (`src/gateway/call.ts`).
- macOS app UI → Gateway WS (`GatewayConnection`).
- Web Control UI → Gateway WS.
- ACP → Gateway WS.
- Điều khiển trình duyệt sử dụng máy chủ điều khiển HTTP riêng của nó.
## Các node ngày hôm nay

- Ứng dụng macOS ở chế độ node kết nối với cầu nối Gateway (`MacNodeBridgeSession`).
- Các ứng dụng iOS/Android kết nối với cầu nối Gateway.
- Ghép cặp + token cho mỗi node được lưu trữ trên gateway.
## Luồng phê duyệt hiện tại (exec)

- Agent sử dụng `system.run` qua Gateway.
- Gateway gọi node qua bridge.
- Node runtime quyết định phê duyệt.
- Lời nhắc UI được hiển thị bởi ứng dụng mac (khi node == ứng dụng mac).
- Node trả về `invoke-res` cho Gateway.
- Multi‑hop, UI gắn với máy chủ node.
## Hiện diện + danh tính ngày hôm nay

- Các mục hiện diện Gateway từ các client WS.
- Các mục hiện diện Node từ bridge.
- Ứng dụng mac có thể hiển thị hai mục cho cùng một máy (UI + node).
- Danh tính Node được lưu trữ trong pairing store; danh tính UI riêng biệt.

---
# Vấn đề / Điểm yếu

- Hai ngăn xếp giao thức cần duy trì (WS + Bridge).
- Phê duyệt trên các nút từ xa: lời nhắc xuất hiện trên máy chủ nút, không phải nơi người dùng ở.
- TLS pinning chỉ tồn tại cho bridge; WS phụ thuộc vào SSH/Tailscale.
- Nhân đôi danh tính: cùng một máy hiển thị dưới dạng nhiều phiên bản.
- Vai trò không rõ ràng: khả năng UI + node + CLI không được phân tách rõ ràng.

---
# Trạng thái mới được đề xuất (Clawnet)

## Một giao thức, hai vai trò

Giao thức WS duy nhất với vai trò + phạm vi.

- **Vai trò: node** (máy chủ khả năng)
- **Vai trò: operator** (mặt phẳng điều khiển)
- **Phạm vi** tùy chọn cho operator:
  - `operator.read` (trạng thái + xem)
  - `operator.write` (chạy agent, gửi)
  - `operator.admin` (cấu hình, kênh, mô hình)

### Hành vi vai trò

**Node**

- Có thể đăng ký khả năng (`caps`, `commands`, quyền).
- Có thể nhận lệnh `invoke` (`system.run`, `camera.*`, `canvas.*`, `screen.record`, v.v.).
- Có thể gửi sự kiện: `voice.transcript`, `agent.request`, `chat.subscribe`.
- Không thể gọi API mặt phẳng điều khiển cấu hình/mô hình/kênh/phiên/điều khiển agent.

**Operator**

- API mặt phẳng điều khiển đầy đủ, được kiểm soát bởi phạm vi.
- Nhận tất cả các phê duyệt.
- Không thực thi trực tiếp các hành động hệ điều hành; định tuyến đến các node.

### Quy tắc chính

Vai trò là theo kết nối, không phải theo thiết bị. Một thiết bị có thể mở cả hai vai trò, riêng biệt.

---
# Xác thực thống nhất + ghép nối

## Danh tính máy khách

Mỗi máy khách cung cấp:

- `deviceId` (ổn định, được lấy từ khóa thiết bị).
- `displayName` (tên con người).
- `role` + `scope` + `caps` + `commands`.
## Quy trình ghép nối (thống nhất)

- Client kết nối mà không xác thực.
- Gateway tạo một **yêu cầu ghép nối** cho `deviceId` đó.
- Operator nhận lời nhắc; phê duyệt/từ chối.
- Gateway cấp thông tin xác thực được ràng buộc với:
  - khóa công khai của thiết bị
  - vai trò (các)
  - phạm vi (các)
  - khả năng/lệnh
- Client lưu trữ token, kết nối lại với xác thực.
## Xác thực liên kết thiết bị (tránh phát lại bearer token)

Được ưu tiên: cặp khóa thiết bị.

- Thiết bị tạo cặp khóa một lần.
- `deviceId = fingerprint(publicKey)`.
- Gateway gửi nonce; thiết bị ký; Gateway xác minh.
- Token được cấp cho một khóa công khai (proof‑of‑possession), không phải một chuỗi.

Các lựa chọn thay thế:

- mTLS (chứng chỉ máy khách): mạnh nhất, phức tạp hơn về vận hành.
- Bearer token có thời hạn ngắn chỉ như một giai đoạn tạm thời (xoay vòng + thu hồi sớm).
## Phê duyệt im lặng (Heuristic SSH)

Định nghĩa nó một cách chính xác để tránh một điểm yếu. Ưu tiên một trong những cách sau:

- **Chỉ cục bộ**: tự động ghép nối khi client kết nối qua local loopback/Unix socket.
- **Thử thách qua SSH**: gateway phát hành nonce; client chứng minh SSH bằng cách tìm nạp nó.
- **Cửa sổ hiện diện vật lý**: sau khi phê duyệt cục bộ trên UI máy chủ gateway, cho phép tự động ghép nối trong một khoảng thời gian ngắn (ví dụ: 10 phút).

Luôn ghi nhật ký + ghi lại các phê duyệt tự động.

---
# TLS ở mọi nơi (dev + prod)

## Tái sử dụng TLS bridge hiện có

Sử dụng TLS runtime hiện tại + fingerprint pinning:

- `src/infra/bridge/server/tls.ts`
- logic xác minh fingerprint trong `src/node-host/bridge-client.ts`
## Áp dụng cho WS

- Máy chủ WS hỗ trợ TLS với cùng cert/key + fingerprint.
- Các client WS có thể ghim fingerprint (tùy chọn).
- Discovery quảng cáo TLS + fingerprint cho tất cả các endpoint.
  - Discovery chỉ là gợi ý định vị; không bao giờ là điểm tin cậy.
## Tại sao

- Giảm sự phụ thuộc vào SSH/Tailscale để bảo mật thông tin.
- Làm cho các kết nối di động từ xa an toàn theo mặc định.

---
# Thiết kế lại phê duyệt (tập trung)

## Hiện tại

Phê duyệt xảy ra trên node host (mac app node runtime). Lời nhắc xuất hiện nơi node chạy.
## Đề xuất

Phê duyệt được **lưu trữ trên gateway**, giao diện người dùng được cung cấp cho các client của nhà điều hành.

### Luồng mới

1. Gateway nhận `system.run` intent (agent).
2. Gateway tạo bản ghi phê duyệt: `approval.requested`.
3. Giao diện người dùng của nhà điều hành hiển thị lời nhắc.
4. Quyết định phê duyệt được gửi đến gateway: `approval.resolve`.
5. Gateway gọi lệnh node nếu được phê duyệt.
6. Node thực thi, trả về `invoke-res`.

### Ngữ nghĩa phê duyệt (tăng cường bảo mật)

- Phát sóng đến tất cả các nhà điều hành; chỉ giao diện người dùng hoạt động hiển thị modal (những giao diện khác nhận được toast).
- Giải quyết đầu tiên thắng; gateway từ chối các giải quyết tiếp theo vì đã được giải quyết.
- Hết thời gian chờ mặc định: từ chối sau N giây (ví dụ: 60 giây), ghi lại lý do.
- Giải quyết yêu cầu `operator.approvals` scope.
## Lợi ích

- Lời nhắc xuất hiện nơi người dùng ở (mac/phone).
- Phê duyệt nhất quán cho các node từ xa.
- Node runtime vẫn ở chế độ headless; không phụ thuộc vào UI.

---
# Ví dụ về rõ ràng vai trò

## Ứng dụng iPhone

- **Vai trò Node** cho: mic, camera, voice chat, vị trí, push‑to‑talk.
- **operator.read** tùy chọn cho chế độ xem trạng thái và chat.
- **operator.write/admin** tùy chọn chỉ khi được bật rõ ràng.
## Ứng dụng macOS

- Vai trò Operator theo mặc định (điều khiển giao diện).
- Vai trò Node khi "Mac node" được bật (system.run, screen, camera).
- Cùng deviceId cho cả hai kết nối → mục giao diện được hợp nhất.
## CLI

- Vai trò Operator luôn luôn.
- Phạm vi được xác định bởi lệnh con:
  - `status`, `logs` → đọc
  - `agent`, `message` → ghi
  - `config`, `channels` → quản trị
  - phê duyệt + ghép nối → `operator.approvals` / `operator.pairing`

---
# Danh tính + slug

## ID ổn định

Bắt buộc để xác thực; không bao giờ thay đổi.
Ưu tiên:

- Dấu vân tay cặp khóa (hash khóa công khai).
## Cute slug (lobster‑themed)

Nhãn con người duy nhất.

- Ví dụ: `scarlet-claw`, `saltwave`, `mantis-pinch`.
- Được lưu trữ trong sổ đăng ký gateway, có thể chỉnh sửa.
- Xử lý va chạm: `-2`, `-3`.
## Nhóm giao diện người dùng

Cùng `deviceId` trên các vai trò → hàng "Instance" duy nhất:

- Badge: `operator`, `node`.
- Hiển thị khả năng + lần cuối cùng được nhìn thấy.

---
# Chiến lược di chuyển

## Giai đoạn 0: Tài liệu + căn chỉnh

- Công bố tài liệu này.
- Kiểm kê tất cả các lệnh gọi giao thức + quy trình phê duyệt.
## Giai đoạn 1: Thêm vai trò/phạm vi vào WS

- Mở rộng `connect` params với `role`, `scope`, `deviceId`.
- Thêm gating danh sách cho phép cho vai trò node.
## Giai đoạn 2: Tương thích cầu nối

- Giữ cầu nối chạy.
- Thêm hỗ trợ nút WS song song.
- Gating các tính năng phía sau cờ cấu hình.
## Giai đoạn 3: Phê duyệt tập trung

- Thêm yêu cầu phê duyệt + giải quyết sự kiện trong WS.
- Cập nhật giao diện ứng dụng Mac để nhắc nhở + phản hồi.
- Node runtime dừng nhắc nhở giao diện người dùng.
## Giai đoạn 4: Thống nhất TLS

- Thêm cấu hình TLS cho WS bằng cách sử dụng bridge TLS runtime.
- Thêm pinning cho các client.
## Giai đoạn 5: Loại bỏ bridge

- Di chuyển iOS/Android/mac node sang WS.
- Giữ bridge làm giải pháp dự phòng; xóa khi ổn định.
## Phase 6: Xác thực liên kết thiết bị

- Yêu cầu danh tính dựa trên khóa cho tất cả các kết nối không cục bộ.
- Thêm giao diện người dùng cho thu hồi + xoay vòng.

---
# Ghi chú bảo mật

- Vai trò/danh sách cho phép được thực thi tại ranh giới Gateway.
- Không có client nào nhận được API "đầy đủ" mà không có phạm vi operator.
- Ghép nối được yêu cầu cho _tất cả_ kết nối.
- TLS + pinning giảm rủi ro MITM cho thiết bị di động.
- Phê duyệt im lặng SSH là một tiện lợi; vẫn được ghi lại + có thể thu hồi.
- Khám phá thiết bị không bao giờ là một điểm tin cậy.
- Các yêu cầu khả năng được xác minh dựa trên danh sách cho phép của máy chủ theo nền tảng/loại.
# Truyền phát + tải trọng lớn (phương tiện node)

WS control plane hoạt động tốt cho các tin nhắn nhỏ, nhưng các node cũng thực hiện:

- clip camera
- ghi lại màn hình
- luồng âm thanh

Các tùy chọn:

1. WS binary frames + chunking + quy tắc backpressure.
2. Endpoint truyền phát riêng biệt (vẫn TLS + xác thực).
3. Giữ bridge lâu hơn cho các lệnh nặng phương tiện, di chuyển cuối cùng.

Chọn một trước khi triển khai để tránh sai lệch.
# Chính sách khả năng + lệnh

- Các khả năng/lệnh được báo cáo bởi node được coi là **yêu cầu**.
- Gateway thực thi danh sách cho phép theo từng nền tảng.
- Bất kỳ lệnh mới nào đều yêu cầu phê duyệt của nhà điều hành hoặc thay đổi danh sách cho phép rõ ràng.
- Kiểm tra các thay đổi bằng dấu thời gian.
# Kiểm toán + giới hạn tốc độ

- Ghi nhật ký: yêu cầu ghép nối, phê duyệt/từ chối, cấp phát/xoay vòng/thu hồi token.
- Giới hạn tốc độ spam ghép nối và lời nhắc phê duyệt.
# Vệ sinh giao thức

- Phiên bản giao thức rõ ràng + mã lỗi.
- Quy tắc kết nối lại + chính sách nhịp tim.
- TTL hiện diện và ngữ nghĩa lần cuối cùng nhìn thấy.

---
# Các câu hỏi mở

1. Một thiết bị chạy cả hai vai trò: mô hình token
   - Khuyến nghị các token riêng biệt cho mỗi vai trò (node vs operator).
   - Cùng deviceId; các scope khác nhau; thu hồi rõ ràng hơn.

2. Độ chi tiết của scope operator
   - read/write/admin + approvals + pairing (tối thiểu khả thi).
   - Xem xét các scope theo tính năng sau này.

3. UX xoay vòng token + thu hồi
   - Tự động xoay vòng khi thay đổi vai trò.
   - UI để thu hồi theo deviceId + vai trò.

4. Khám phá thiết bị
   - Mở rộng TXT Bonjour hiện tại để bao gồm WS TLS fingerprint + role hints.
   - Coi như các gợi ý định vị chỉ.

5. Phê duyệt xuyên mạng
   - Phát tới tất cả các client operator; UI hoạt động hiển thị modal.
   - Phản hồi đầu tiên thắng; gateway thực thi tính nguyên tử.
# Tóm tắt (TL;DR)

- Hiện tại: mặt phẳng điều khiển WS + vận chuyển nút Bridge.
- Vấn đề: phê duyệt + trùng lặp + hai ngăn xếp.
- Đề xuất: một giao thức WS với các vai trò + phạm vi rõ ràng, ghép nối thống nhất + ghim TLS, phê duyệt do gateway lưu trữ, ID thiết bị ổn định + slug dễ thương.
- Kết quả: UX đơn giản hơn, bảo mật mạnh hơn, ít trùng lặp hơn, định tuyến di động tốt hơn.