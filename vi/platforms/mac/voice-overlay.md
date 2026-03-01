---
summary: Vòng đời voice overlay khi wake-word và push-to-talk trùng lặp
read_when:
  - Điều chỉnh hành vi phủ tiếng nói
title: Lớp phủ Giọng nói
x-i18n:
  source_path: platforms\mac\voice-overlay.md
  source_hash: 1efcc26ec05d2f421cb2cf462077d002381995b338d00db77d5fdba9b8d938b6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:39.900Z'
---

# Vòng đời Overlay Giọng nói (macOS)

Đối tượng: những người đóng góp ứng dụng macOS. Mục tiêu: giữ cho overlay giọng nói có thể dự đoán được khi từ khóa唤醒 và push-to-talk trùng lặp.

## Ý định hiện tại

- Nếu overlay đã hiển thị từ từ khóa唤醒 và người dùng nhấn phím tắt, phiên hotkey _áp dụng_ văn bản hiện có thay vì đặt lại nó. Overlay vẫn hiển thị trong khi hotkey được giữ. Khi người dùng thả ra: gửi nếu có văn bản được cắt, nếu không thì đóng.
- Từ khóa唤醒 một mình vẫn tự động gửi khi im lặng; push-to-talk gửi ngay khi thả ra.

## Đã triển khai (9 tháng 12, 2025)

- Các phiên overlay hiện mang một token cho mỗi lần ghi âm (từ khóa唤醒 hoặc push-to-talk). Các bản cập nhật partial/final/send/dismiss/level bị loại bỏ khi token không khớp, tránh các callback cũ.
- Push-to-talk áp dụng bất kỳ văn bản overlay hiển thị nào làm tiền tố (vì vậy nhấn hotkey trong khi overlay wake lên sẽ giữ văn bản và thêm bài phát biểu mới). Nó chờ tối đa 1,5 giây để có bản ghi chép cuối cùng trước khi quay lại văn bản hiện tại.
- Nhật ký chime/overlay được phát ra tại `info` trong các danh mục `voicewake.overlay`, `voicewake.ptt` và `voicewake.chime` (bắt đầu phiên, partial, final, send, dismiss, lý do chime).

## Các bước tiếp theo

1. **VoiceSessionCoordinator (actor)**
   - Sở hữu chính xác một `VoiceSession` tại một thời điểm.
   - API (dựa trên token): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Loại bỏ các callback mang token cũ (ngăn chặn các trình nhận dạng cũ từ việc mở lại overlay).
2. **VoiceSession (model)**
   - Các trường: `token`, `source` (wakeWord|pushToTalk), văn bản committed/volatile, cờ chime, bộ hẹn giờ (auto-send, idle), `overlayMode` (display|editing|sending), thời hạn cooldown.
3. **Liên kết overlay**
   - `VoiceSessionPublisher` (`ObservableObject`) phản ánh phiên hoạt động vào SwiftUI.
   - `VoiceWakeOverlayView` chỉ kết xuất thông qua publisher; nó không bao giờ thay đổi các singleton toàn cục trực tiếp.
   - Các hành động người dùng overlay (`sendNow`, `dismiss`, `edit`) gọi lại vào coordinator với token phiên.
4. **Đường dẫn gửi thống nhất**
   - Trên `endCapture`: nếu văn bản được cắt trống → đóng; nếu không `performSend(session:)` (phát chime gửi một lần, chuyển tiếp, đóng).
   - Push-to-talk: không có độ trễ; từ khóa唤醒: độ trễ tùy chọn cho auto-send.
   - Áp dụng một cooldown ngắn cho thời gian chạy wake sau khi push-to-talk kết thúc để từ khóa唤醒 không kích hoạt lại ngay lập tức.
5. **Ghi nhật ký**
   - Coordinator phát ra nhật ký `.info` trong hệ thống con `ai.openclaw`, danh mục `voicewake.overlay` và `voicewake.chime`.
   - Các sự kiện chính: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

## Danh sách kiểm tra gỡ lỗi

- Phát trực tiếp nhật ký trong khi tái tạo overlay bị dính:

  ```bash
  sudo log stream --predicate 'subsystem == "ai.openclaw" AND category CONTAINS "voicewake"' --level info --style compact
  ```

- Verify only one active session token; stale callbacks should be dropped by the coordinator.
- Ensure push-to-talk release always calls `endCapture` with the active token; if text is empty, expect `dismiss` without chime or send.

## Migration steps (suggested)

1. Add `VoiceSessionCoordinator`, `VoiceSession`, and `VoiceSessionPublisher`.
2. Refactor `VoiceWakeRuntime` to create/update/end sessions instead of touching `VoiceWakeOverlayController` directly.
3. Refactor `VoicePushToTalk` to adopt existing sessions and call `endCapture` on release; apply runtime cooldown.
4. Wire `VoiceWakeOverlayController` đến publisher; loại bỏ các lệnh gọi trực tiếp từ runtime/PTT.
5. Thêm các bài kiểm tra tích hợp cho việc áp dụng phiên, cooldown và loại bỏ văn bản trống.