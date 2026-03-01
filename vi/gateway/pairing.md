---
summary: Ghép nối nút do Gateway sở hữu (Tùy chọn B) cho iOS và các nút từ xa khác
read_when:
  - Triển khai phê duyệt ghép nối nút mà không cần giao diện macOS
  - Thêm luồng CLI để phê duyệt các nút từ xa
  - Mở rộng giao thức gateway với quản lý nút
title: Ghép Cặp Sở Hữu Bởi Gateway
x-i18n:
  source_path: gateway\pairing.md
  source_hash: 1f5154292a75ea2c1470324babc99c6c46a5e4e16afb394ed323d28f6168f459
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:58:31.082Z'
---

# Ghép nối do Gateway sở hữu (Tùy chọn B)

Trong ghép nối do Gateway sở hữu, **Gateway** là nguồn sự thật cho phép các node nào được phép tham gia. Các giao diện người dùng (ứng dụng macOS, các client trong tương lai) chỉ là các frontend để phê duyệt hoặc từ chối các yêu cầu đang chờ xử lý.

**Quan trọng:** Các node WS sử dụng **ghép nối thiết bị** (role `node`) trong `connect`. `node.pair.*` là một kho lưu trữ ghép nối riêng biệt và **không** kiểm soát bắt tay WS. Chỉ các client gọi rõ ràng `node.pair.*` mới sử dụng luồng này.
## Khái niệm

- **Yêu cầu chờ xử lý**: một node yêu cầu tham gia; cần phê duyệt.
- **Node được ghép nối**: node được phê duyệt với token xác thực được cấp.
- **Giao thức truyền tải**: điểm cuối WS của Gateway chuyển tiếp các yêu cầu nhưng không quyết định tư cách thành viên. (Hỗ trợ cầu nối TCP cũ đã bị loại bỏ/xóa.)
## Cách ghép nối hoạt động

1. Một node kết nối với Gateway WS và yêu cầu ghép nối.
2. Gateway lưu trữ một **yêu cầu đang chờ xử lý** và phát ra `node.pair.requested`.
3. Bạn phê duyệt hoặc từ chối yêu cầu (CLI hoặc UI).
4. Khi phê duyệt, Gateway cấp một **token mới** (các token được xoay vòng khi ghép nối lại).
5. Node kết nối lại bằng token và bây giờ đã "được ghép nối".

Các yêu cầu đang chờ xử lý hết hạn tự động sau **5 phút**.
## Quy trình làm việc CLI (thân thiện với headless)

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` hiển thị các node được ghép nối/kết nối và khả năng của chúng.
## Bề mặt API (giao thức gateway)

Sự kiện:

- `node.pair.requested` — phát ra khi một yêu cầu chờ xử lý mới được tạo.
- `node.pair.resolved` — phát ra khi một yêu cầu được phê duyệt/từ chối/hết hạn.

Phương thức:

- `node.pair.request` — tạo hoặc tái sử dụng một yêu cầu chờ xử lý.
- `node.pair.list` — liệt kê các node chờ xử lý + được ghép nối.
- `node.pair.approve` — phê duyệt một yêu cầu chờ xử lý (cấp token).
- `node.pair.reject` — từ chối một yêu cầu chờ xử lý.
- `node.pair.verify` — xác minh `{ nodeId, token }`.

Ghi chú:

- `node.pair.request` là idempotent cho mỗi node: các lệnh gọi lặp lại trả về cùng một
  yêu cầu chờ xử lý.
- Phê duyệt **luôn luôn** tạo ra một token mới; không có token nào được trả về từ
  `node.pair.request`.
- Các yêu cầu có thể bao gồm `silent: true` như một gợi ý cho các luồng phê duyệt tự động.
## Phê duyệt tự động (ứng dụng macOS)

Ứng dụng macOS có thể tùy chọn cố gắng thực hiện **phê duyệt im lặng** khi:

- yêu cầu được đánh dấu `silent`, và
- ứng dụng có thể xác minh kết nối SSH đến máy chủ gateway bằng cùng một người dùng.

Nếu phê duyệt im lặng không thành công, nó sẽ quay lại lời nhắc "Phê duyệt/Từ chối" bình thường.
## Lưu trữ (cục bộ, riêng tư)

Trạng thái ghép nối được lưu trữ trong thư mục trạng thái Gateway (mặc định `~/.openclaw`):

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

Nếu bạn ghi đè `OPENCLAW_STATE_DIR`, thư mục `nodes/` sẽ di chuyển cùng với nó.

Ghi chú bảo mật:

- Token là bí mật; coi `paired.json` là nhạy cảm.
- Xoay vòng token yêu cầu phê duyệt lại (hoặc xóa mục node).
## Hành vi giao thức truyền tải

- Giao thức truyền tải là **không trạng thái**; nó không lưu trữ thành viên.
- Nếu Gateway ngoại tuyến hoặc ghép nối bị vô hiệu hóa, các node không thể ghép nối.
- Nếu Gateway ở chế độ từ xa, ghép nối vẫn xảy ra dựa trên kho lưu trữ của Gateway từ xa.