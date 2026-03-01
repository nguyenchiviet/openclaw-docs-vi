---
summary: >-
  Chế độ kích hoạt bằng giọng nói và push-to-talk cùng chi tiết định tuyến trong
  ứng dụng Mac
read_when:
  - Đang làm việc trên các đường dẫn voice wake hoặc PTT
title: Kích hoạt bằng giọng nói
x-i18n:
  source_path: platforms\mac\voicewake.md
  source_hash: f6440bb89f349ba5c1c9aacffe95e568681beb9899ca736dedfe2f4a366cb5e4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:49.742Z'
---

# Giọng nói Kích hoạt & Bấm để nói

## Chế độ

- **Chế độ từ kích hoạt** (mặc định): Trình nhận dạng giọng nói luôn bật chờ các token kích hoạt (`swabbleTriggerWords`). Khi khớp, nó bắt đầu ghi âm, hiển thị lớp phủ với văn bản một phần và tự động gửi sau khi im lặng.
- **Bấm để nói (Giữ Right Option)**: giữ phím Right Option để ghi âm ngay lập tức—không cần kích hoạt. Lớp phủ xuất hiện khi giữ; thả sẽ hoàn tất và chuyển tiếp sau một khoảng thời gian ngắn để bạn có thể chỉnh sửa văn bản.
## Hành vi thời gian chạy (từ khóa唤醒)

- Speech recognizer nằm trong `VoiceWakeRuntime`.
- Trigger chỉ kích hoạt khi có **tạm dừng có ý nghĩa** giữa từ khóa唤醒 và từ tiếp theo (~0,55s khoảng cách). Overlay/chime có thể bắt đầu trên tạm dừng ngay cả trước khi lệnh bắt đầu.
- Cửa sổ im lặng: 2,0s khi lời nói đang chảy, 5,0s nếu chỉ nghe thấy trigger.
- Dừng cứng: 120s để ngăn chặn các phiên chạy trốn.
- Debounce giữa các phiên: 350ms.
- Overlay được điều khiển qua `VoiceWakeOverlayController` với tô màu committed/volatile.
- Sau khi gửi, recognizer khởi động lại sạch sẽ để lắng nghe trigger tiếp theo.
## Bất biến vòng đời

- Nếu Voice Wake được bật và quyền được cấp, trình nhận dạng từ khóa đánh thức phải đang lắng nghe (ngoại trừ trong quá trình chụp push-to-talk rõ ràng).
- Khả năng hiển thị lớp phủ (bao gồm cả việc loại bỏ thủ công qua nút X) không bao giờ được phép ngăn trình nhận dạng tiếp tục hoạt động.
## Chế độ lỗi overlay dính (trước đây)

Trước đây, nếu overlay bị dính ở trạng thái hiển thị và bạn đóng nó theo cách thủ công, Voice Wake có thể xuất hiện "chết" vì nỗ lực khởi động lại của runtime có thể bị chặn bởi khả năng hiển thị overlay và không có lần khởi động lại nào được lên lịch sau đó.

Tăng cường độ bền vững:

- Khởi động lại wake runtime không còn bị chặn bởi khả năng hiển thị overlay.
- Hoàn thành loại bỏ overlay kích hoạt `VoiceWakeRuntime.refresh(...)` thông qua `VoiceSessionCoordinator`, vì vậy loại bỏ X thủ công luôn tiếp tục lắng nghe.
## Các tính năng cụ thể của Push-to-talk

- Phát hiện phím tắt sử dụng một `.flagsChanged` toàn cầu để theo dõi **Option phải** (`keyCode 61` + `.option`). Chúng tôi chỉ quan sát các sự kiện (không chặn).
- Pipeline ghi âm nằm trong `VoicePushToTalk`: bắt đầu Speech ngay lập tức, truyền phát các phần từng phần đến overlay, và gọi `VoiceWakeForwarder` khi phát hành.
- Khi push-to-talk bắt đầu, chúng tôi tạm dừng runtime wake-word để tránh xung đột khi ghi âm; nó tự động khởi động lại sau khi phát hành.
- Quyền hạn: yêu cầu Microphone + Speech; để xem các sự kiện cần phê duyệt Accessibility/Input Monitoring.
- Bàn phím bên ngoài: một số có thể không hiển thị Option phải như mong đợi—cung cấp phím tắt dự phòng nếu người dùng báo cáo lỗi.
## Cài đặt dành cho người dùng

- **Voice Wake** toggle: bật chế độ runtime wake-word.
- **Hold Cmd+Fn to talk**: bật monitor push-to-talk. Bị vô hiệu hóa trên macOS < 26.
- Bộ chọn ngôn ngữ & micrô, đồng hồ mức trực tiếp, bảng từ kích hoạt, bộ kiểm tra (chỉ cục bộ; không chuyển tiếp).
- Bộ chọn micrô lưu giữ lựa chọn cuối cùng nếu một thiết bị ngắt kết nối, hiển thị gợi ý ngắt kết nối và tạm thời quay lại mặc định hệ thống cho đến khi nó trở lại.
- **Sounds**: âm thanh khi phát hiện kích hoạt và khi gửi; mặc định là âm thanh hệ thống "Glass" của macOS. Bạn có thể chọn bất kỳ tệp nào có thể tải `NSSound` (ví dụ: MP3/WAV/AIFF) cho mỗi sự kiện hoặc chọn **No Sound**.
## Hành vi chuyển tiếp

- Khi Voice Wake được bật, các bản ghi âm được chuyển tiếp đến gateway/agent hoạt động (cùng chế độ cục bộ hoặc từ xa được sử dụng bởi phần còn lại của ứng dụng mac).
- Các câu trả lời được gửi đến **nhà cung cấp chính được sử dụng lần cuối** (WhatsApp/Telegram/Discord/WebChat). Nếu gửi không thành công, lỗi sẽ được ghi lại và lần chạy vẫn có thể nhìn thấy được thông qua nhật ký WebChat/phiên.
## Chuyển tiếp tải trọng

- `VoiceWakeForwarder.prefixedTranscript(_:)` thêm gợi ý máy trước khi gửi. Được chia sẻ giữa các đường dẫn wake-word và push-to-talk.
## Xác minh nhanh

- Bật push-to-talk, giữ Cmd+Fn, nói, thả ra: overlay sẽ hiển thị các phần sau đó gửi.
- Trong khi giữ, tai thanh menu sẽ vẫn phóng to (sử dụng `triggerVoiceEars(ttl:nil)`); chúng sẽ thu nhỏ sau khi thả ra.