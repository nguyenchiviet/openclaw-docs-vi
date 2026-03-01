---
summary: Chế độ exec nâng cao và các chỉ thị /elevated
read_when:
  - >-
    Điều chỉnh các cài đặt mặc định chế độ nâng cao, danh sách cho phép hoặc
    hành vi lệnh slash
title: Chế độ Nâng cao
x-i18n:
  source_path: tools\elevated.md
  source_hash: d3fcd557c3c792209eaa09773e215e70d9406e7bdea52f2ed80ca270c82ae750
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:29:26.680Z'
---

# Chế độ Nâng cao (/elevated directives)

## Chức năng

- `/elevated on` chạy trên máy chủ Gateway và giữ lại phê duyệt exec (giống như `/elevated ask`).
- `/elevated full` chạy trên máy chủ Gateway **và** tự động phê duyệt exec (bỏ qua phê duyệt exec).
- `/elevated ask` chạy trên máy chủ Gateway nhưng giữ lại phê duyệt exec (giống như `/elevated on`).
- `on`/`ask` **không** bắt buộc `exec.security=full`; chính sách bảo mật/hỏi được cấu hình vẫn áp dụng.
- Chỉ thay đổi hành vi khi agent **sandboxed** (nếu không, exec đã chạy trên máy chủ).
- Dạng directive: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- Chỉ `on|off|ask|full` được chấp nhận; bất cứ điều gì khác sẽ trả về gợi ý và không thay đổi trạng thái.

## Điều khiển (và điều không điều khiển)

- **Cổng khả dụng**: `tools.elevated` là đường cơ sở toàn cầu. `agents.list[].tools.elevated` có thể hạn chế thêm nâng cao cho mỗi agent (cả hai phải cho phép).
- **Trạng thái mỗi phiên**: `/elevated on|off|ask|full` đặt mức nâng cao cho khóa phiên hiện tại.
- **Directive nội tuyến**: `/elevated on|ask|full` bên trong một tin nhắn áp dụng cho tin nhắn đó chỉ.
- **Nhóm**: Trong trò chuyện nhóm, các directive nâng cao chỉ được tôn trọng khi agent được đề cập. Các tin nhắn chỉ lệnh bỏ qua yêu cầu đề cập được coi là đã được đề cập.
- **Thực thi trên máy chủ**: nâng cao bắt buộc `exec` lên máy chủ Gateway; `full` cũng đặt `security=full`.
- **Phê duyệt**: `full` bỏ qua phê duyệt exec; `on`/`ask` tôn trọng chúng khi các quy tắc danh sách cho phép/hỏi yêu cầu.
- **Agent không sandboxed**: không hoạt động cho vị trí; chỉ ảnh hưởng đến gating, logging và trạng thái.
- **Chính sách công cụ vẫn áp dụng**: nếu `exec` bị từ chối bởi chính sách công cụ, nâng cao không thể được sử dụng.
- **Tách biệt với `/exec`**: `/exec` điều chỉnh mặc định mỗi phiên cho những người gửi được phép và không yêu cầu nâng cao.

## Thứ tự giải quyết

1. Directive nội tuyến trên tin nhắn (áp dụng chỉ cho tin nhắn đó).
2. Ghi đè phiên (được đặt bằng cách gửi tin nhắn chỉ directive).
3. Mặc định toàn cầu (`agents.defaults.elevatedDefault` trong cấu hình).

## Đặt mặc định phiên

- Gửi một tin nhắn chỉ là directive (cho phép khoảng trắng), ví dụ `/elevated full`.
- Tin nhắn xác nhận được gửi (`Elevated mode set to full...` / `Elevated mode disabled.`).
- Nếu quyền truy cập nâng cao bị vô hiệu hóa hoặc người gửi không có trong danh sách cho phép được phê duyệt, directive sẽ trả lời bằng lỗi có thể hành động và không thay đổi trạng thái phiên.
- Gửi `/elevated` (hoặc `/elevated:`) không có đối số để xem mức nâng cao hiện tại.

## Khả dụng + danh sách cho phép

- Cổng tính năng: `tools.elevated.enabled` (mặc định có thể tắt qua cấu hình ngay cả khi mã hỗ trợ).
- Danh sách cho phép người gửi: `tools.elevated.allowFrom` với danh sách cho phép mỗi nhà cung cấp (ví dụ `discord`, `whatsapp`).
- Các mục danh sách cho phép không có tiền tố chỉ khớp với các giá trị danh tính có phạm vi người gửi (`SenderId`, `SenderE164`, `From`); các trường định tuyến người nhận không bao giờ được sử dụng để ủy quyền nâng cao.
- Siêu dữ liệu người gửi có thể thay đổi yêu cầu tiền tố rõ ràng:
  - `name:<value>` khớp với `SenderName`
  - `username:<value>` khớp với `SenderUsername`
  - `tag:<value>` khớp với `SenderTag`
  - `id:<value>`, `from:<value>`, `e164:<value>` có sẵn để nhắm mục tiêu danh tính rõ ràng
- Cổng mỗi agent: `agents.list[].tools.elevated.enabled` (tùy chọn; chỉ có thể hạn chế thêm).
- Danh sách cho phép mỗi agent: `agents.list[].tools.elevated.allowFrom` (tùy chọn; khi được đặt, người gửi phải khớp với **cả** danh sách cho phép toàn cầu + mỗi agent).
- Fallback Discord: nếu `tools.elevated.allowFrom.discord` bị bỏ qua, danh sách `channels.discord.allowFrom` được sử dụng làm fallback (cũ: `channels.discord.dm.allowFrom`). Đặt `tools.elevated.allowFrom.discord` (thậm chí `[]`) để ghi đè. Danh sách cho phép mỗi agent **không** sử dụng fallback.
- Tất cả các cổng phải vượt qua; nếu không, nâng cao được coi là không khả dụng.

## Logging + trạng thái

- Các lệnh gọi exec nâng cao được ghi nhật ký ở mức thông tin.
- Trạng thái phiên bao gồm chế độ nâng cao (ví dụ `elevated=ask`, `elevated=full`).