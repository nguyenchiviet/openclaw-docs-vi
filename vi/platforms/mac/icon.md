---
summary: Biểu tượng thanh menu và hoạt ảnh trạng thái cho OpenClaw trên macOS
read_when:
  - Thay đổi hành vi biểu tượng thanh menu
title: Biểu tượng thanh menu
x-i18n:
  source_path: platforms\mac\icon.md
  source_hash: a67a6e6bbdc2b611ba365d3be3dd83f9e24025d02366bc35ffcce9f0b121872b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:51.395Z'
---

# Trạng thái biểu tượng thanh menu

Tác giả: steipete · Cập nhật: 2025-12-06 · Phạm vi: ứng dụng macOS (`apps/macos`)

- **Chờ:** Hoạt ảnh biểu tượng bình thường (nhấp nháy, lắc nhẹ thỉnh thoảng).
- **Tạm dừng:** Mục trạng thái sử dụng `appearsDisabled`; không có chuyển động.
- **Kích hoạt bằng giọng nói (tai to):** Bộ phát hiện thức dậy bằng giọng nói gọi `AppState.triggerVoiceEars(ttl: nil)` khi nghe thấy từ khóa thức dậy, giữ `earBoostActive=true` trong khi lời nói được ghi lại. Tai phóng to (1,9x), có lỗ tai tròn để dễ đọc, sau đó rơi xuống qua `stopVoiceEars()` sau 1 giây im lặng. Chỉ được kích hoạt từ đường ống giọng nói trong ứng dụng.
- **Đang hoạt động (agent chạy):** `AppState.isWorking=true` điều khiển chuyển động vi mô "chạy chân/đuôi": lắc chân nhanh hơn và độ lệch nhẹ trong khi công việc đang diễn ra. Hiện được chuyển đổi xung quanh các lần chạy agent WebChat; thêm cùng một chuyển đổi xung quanh các tác vụ dài khác khi bạn kết nối chúng.

Điểm kết nối

- Thức dậy bằng giọng nói: gọi `AppState.triggerVoiceEars(ttl: nil)` trên runtime/tester khi kích hoạt và `stopVoiceEars()` sau 1 giây im lặng để khớp với cửa sổ ghi lại.
- Hoạt động của agent: đặt `AppStateStore.shared.setWorking(true/false)` xung quanh các khoảng thời gian công việc (đã được thực hiện trong lệnh gọi agent WebChat). Giữ các khoảng thời gian ngắn và đặt lại trong các khối `defer` để tránh hoạt ảnh bị kẹt.

Hình dạng và kích thước

- Biểu tượng cơ sở được vẽ trong `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
- Tỷ lệ tai mặc định là `1.0`; tăng cường giọng nói đặt `earScale=1.9` và chuyển đổi `earHoles=true` mà không thay đổi khung tổng thể (hình ảnh mẫu 18×18 pt được hiển thị thành kho lưu trữ Retina 36×36 px).
- Chạy sử dụng lắc chân lên tới ~1,0 với một chuyển động ngang nhỏ; nó được thêm vào bất kỳ lắc chân chờ nào hiện có.

Ghi chú về hành vi

- Không có chuyển đổi CLI/broker bên ngoài cho tai/đang hoạt động; giữ nó bên trong các tín hiệu của chính ứng dụng để tránh vỗ cánh vô tình.
- Giữ TTL ngắn (&lt;10s) để biểu tượng quay lại đường cơ sở nhanh chóng nếu một công việc bị treo.