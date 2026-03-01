---
title: Xác Minh Chính Thức (Các Mô Hình Bảo Mật)
summary: >-
  Các mô hình bảo mật được kiểm tra bằng máy cho các đường dẫn rủi ro cao nhất
  của OpenClaw.
read_when:
  - Xem xét các đảm bảo hoặc giới hạn của mô hình bảo mật chính thức
  - Kiểm tra mô hình bảo mật TLA+/TLC - Tái tạo hoặc cập nhật
permalink: /security/formal-verification/
x-i18n:
  source_path: security\formal-verification.md
  source_hash: b576c7437f598eba41c3e8c644eac801113437c8292ddddf25df67bac504a576
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:23:09.050Z'
---

# Xác minh Hình thức (Mô hình Bảo mật)

Trang này theo dõi **các mô hình bảo mật hình thức** của OpenClaw (TLA+/TLC ngày nay; thêm nữa khi cần).

> Lưu ý: một số liên kết cũ hơn có thể tham chiếu đến tên dự án trước đó.

**Mục tiêu (hướng phát triển):** cung cấp một lập luận được kiểm tra bằng máy rằng OpenClaw thực thi chính sách bảo mật dự định của nó (ủy quyền, cách ly phiên, kiểm soát công cụ và an toàn cấu hình sai), dưới các giả định rõ ràng.

**Đây là gì (ngày nay):** một **bộ kiểm tra hồi quy bảo mật** có thể thực thi, được điều khiển bởi kẻ tấn công:

- Mỗi yêu cầu có một kiểm tra mô hình có thể chạy được trên một không gian trạng thái hữu hạn.
- Nhiều yêu cầu có một **mô hình âm** được ghép nối tạo ra một dấu vết phản ví dụ cho một lớp lỗi thực tế.

**Đây không phải là gì (chưa):** một bằng chứng rằng "OpenClaw an toàn trong mọi khía cạnh" hoặc rằng triển khai TypeScript đầy đủ là chính xác.
## Nơi các mô hình được lưu trữ

Các mô hình được duy trì trong một kho lưu trữ riêng: [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).
## Những lưu ý quan trọng

- Đây là **các mô hình**, không phải là triển khai TypeScript đầy đủ. Sự khác biệt giữa mô hình và mã là có thể xảy ra.
- Kết quả bị giới hạn bởi không gian trạng thái được khám phá bởi TLC; "xanh" không ngụ ý bảo mật vượt quá các giả định và giới hạn được mô hình hóa.
- Một số yêu cầu dựa trên các giả định môi trường rõ ràng (ví dụ: triển khai chính xác, đầu vào cấu hình chính xác).
## Tái tạo kết quả

Hiện tại, kết quả được tái tạo bằng cách sao chép kho mô hình cục bộ và chạy TLC (xem bên dưới). Một lần lặp trong tương lai có thể cung cấp:

- Các mô hình chạy CI với các tạo phẩm công khai (dấu vết phản ví dụ, nhật ký chạy)
- một quy trình "chạy mô hình này" được lưu trữ cho các kiểm tra nhỏ, có giới hạn

Bắt đầu:

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ required (TLC runs on the JVM).
# The repo vendors a pinned `tla2tools.jar` (TLA+ tools) and provides `bin/tlc` + Make targets.

make <target>
```

### Gateway exposure and open gateway misconfiguration

**Claim:** binding beyond loopback without auth can make remote compromise possible / increases exposure; token/password blocks unauth attackers (per the model assumptions).

- Green runs:
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Red (expected):
  - `make gateway-exposure-v2-negative`

See also: `docs/gateway-exposure-matrix.md` in the models repo.

### Nodes.run pipeline (highest-risk capability)

**Claim:** `nodes.run` requires (a) node command allowlist plus declared commands and (b) live approval when configured; approvals are tokenized to prevent replay (in the model).

- Green runs:
  - `make nodes-pipeline`
  - `make approvals-token`
- Red (expected):
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### Pairing store (DM gating)

**Claim:** pairing requests respect TTL and pending-request caps.

- Green runs:
  - `make pairing`
  - `make pairing-cap`
- Red (expected):
  - `make pairing-negative`
  - `make pairing-cap-negative`

### Ingress gating (mentions + control-command bypass)

**Claim:** in group contexts requiring mention, an unauthorized “control command” cannot bypass mention gating.

- Green:
  - `make ingress-gating`
- Đỏ (dự kiến):
- `make ingress-gating-negative`

### Cách ly định tuyến/khóa phiên

**Yêu cầu:** Tin nhắn riêng từ các peer khác biệt không sụp đổ vào cùng một phiên trừ khi được liên kết/cấu hình rõ ràng.

- Xanh lá cây:
  - `make routing-isolation`
- Đỏ (dự kiến):
  - `make routing-isolation-negative`
## v1++: các mô hình bị giới hạn bổ sung (đồng thời, thử lại, độ chính xác theo dõi)

Đây là các mô hình tiếp theo giúp tăng độ chính xác xung quanh các chế độ lỗi trong thế giới thực (cập nhật không nguyên tử, thử lại và fan-out tin nhắn).

### Đồng thời lưu trữ ghép nối / tính idempotency

**Yêu cầu:** một kho lưu trữ ghép nối phải thực thi `MaxPending` và tính idempotency ngay cả dưới các xen kẽ (tức là "check-then-write" phải là nguyên tử / bị khóa; làm mới không nên tạo bản sao).

Ý nghĩa:

- Dưới các yêu cầu đồng thời, bạn không thể vượt quá `MaxPending` cho một kênh.
- Các yêu cầu/làm mới lặp lại cho cùng một `(channel, sender)` không nên tạo các hàng đang chờ xử lý trực tiếp trùng lặp.

- Chạy xanh:
  - `make pairing-race` (kiểm tra giới hạn nguyên tử/bị khóa)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Đỏ (dự kiến):
  - `make pairing-race-negative` (cuộc đua giới hạn begin/commit không nguyên tử)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### Tương quan theo dõi Ingress / tính idempotency

**Yêu cầu:** việc tiếp nhận phải bảo tồn tương quan theo dõi trên fan-out và phải là idempotent dưới các lần thử lại của nhà cung cấp.

Ý nghĩa:

- Khi một sự kiện bên ngoài trở thành nhiều tin nhắn bên trong, mọi phần đều giữ cùng một danh tính theo dõi/sự kiện.
- Các lần thử lại không dẫn đến xử lý kép.
- Nếu ID sự kiện nhà cung cấp bị thiếu, dedup sẽ quay lại một khóa an toàn (ví dụ: ID theo dõi) để tránh bỏ các sự kiện riêng biệt.

- Xanh:
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Đỏ (dự kiến):
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### Ưu tiên dmScope định tuyến + identityLinks

**Yêu cầu:** định tuyến phải giữ các phiên tin nhắn riêng bị cô lập theo mặc định, và chỉ thu gọn các phiên khi được cấu hình rõ ràng (ưu tiên kênh + liên kết danh tính).

Ý nghĩa:

- Ghi đè dmScope dành riêng cho kênh phải thắng so với các giá trị mặc định toàn cầu.
- identityLinks chỉ nên thu gọn trong các nhóm được liên kết rõ ràng, không phải trên các ngang hàng không liên quan.

- Xanh:
  - `make routing-precedence`
  - `make routing-identitylinks`
- Đỏ (dự kiến):
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`
I'm ready to translate. Please provide the Markdown section you'd like me to translate from English to Vietnamese.