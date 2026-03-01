---
summary: 'Khắc phục sự cố ghép nối nút, yêu cầu chạy nền trước, quyền hạn và lỗi công cụ'
read_when:
  - >-
    Node được kết nối nhưng các công cụ camera/canvas/screen/exec không hoạt
    động
  - Bạn cần mô hình tư duy về ghép nối nút so với phê duyệt
title: Khắc phục sự cố Node
x-i18n:
  source_path: nodes\troubleshooting.md
  source_hash: d5c053beb8b9ce9b63085ac2bb00f83ce3f046b78f9ee85c225c650742991adc
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:56.900Z'
---

# Khắc phục sự cố node

Sử dụng trang này khi một node hiển thị trong trạng thái nhưng các công cụ node không hoạt động.
## Bậc lệnh

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then run node specific checks:

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Healthy signals:

- Node is connected and paired for role `node`.
- `nodes describe` bao gồm khả năng bạn đang gọi.
- Phê duyệt thực thi hiển thị chế độ/danh sách cho phép dự kiến.
## Yêu cầu chạy ở nền trước

`canvas.*`, `camera.*`, và `screen.*` chỉ chạy ở nền trước trên các node iOS/Android.

Kiểm tra nhanh và sửa lỗi:

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

If you see `NODE_BACKGROUND_UNAVAILABLE`, đưa ứng dụng node vào nền trước và thử lại.
## Ma trận quyền hạn

| Khả năng                   | iOS                                     | Android                                      | Ứng dụng node macOS                | Mã lỗi điển hình           |
| ---------------------------- | --------------------------------------- | -------------------------------------------- | ----------------------------- | ------------------------------ |
| `camera.snap`, `camera.clip` | Camera (+ mic cho âm thanh clip)           | Camera (+ mic cho âm thanh clip)                | Camera (+ mic cho âm thanh clip) | `*_PERMISSION_REQUIRED`        |
| `screen.record`              | Ghi hình màn hình (+ mic tùy chọn)       | Nhắc nhở chụp màn hình (+ mic tùy chọn)       | Ghi hình màn hình              | `*_PERMISSION_REQUIRED`        |
| `location.get`               | Trong khi sử dụng hoặc Luôn luôn (tùy thuộc vào chế độ) | Vị trí nền trước/nền dựa trên chế độ | Quyền vị trí           | `LOCATION_PERMISSION_REQUIRED` |
| `system.run`                 | n/a (đường dẫn node host)                    | n/a (đường dẫn node host)                         | Yêu cầu phê duyệt Exec       | `SYSTEM_RUN_DENIED`            |
## Ghép nối so với phê duyệt

Đây là những cổng khác nhau:

1. **Ghép nối thiết bị**: node này có thể kết nối với Gateway không?
2. **Phê duyệt thực thi**: node này có thể chạy một lệnh shell cụ thể không?

Kiểm tra nhanh:

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

If pairing is missing, approve the node device first.
If pairing is fine but `system.run` không thành công, hãy sửa phê duyệt thực thi/danh sách cho phép.
## Mã lỗi node phổ biến

- `NODE_BACKGROUND_UNAVAILABLE` → ứng dụng đang chạy nền; đưa nó lên tiền cảnh.
- `CAMERA_DISABLED` → tắt camera bị vô hiệu hóa trong cài đặt node.
- `*_PERMISSION_REQUIRED` → quyền hệ điều hành bị thiếu/từ chối.
- `LOCATION_DISABLED` → chế độ vị trí bị tắt.
- `LOCATION_PERMISSION_REQUIRED` → chế độ vị trí được yêu cầu không được cấp.
- `LOCATION_BACKGROUND_UNAVAILABLE` → ứng dụng đang chạy nền nhưng chỉ có quyền Khi đang sử dụng.
- `SYSTEM_RUN_DENIED: approval required` → yêu cầu exec cần phê duyệt rõ ràng.
- `SYSTEM_RUN_DENIED: allowlist miss` → lệnh bị chặn bởi chế độ danh sách cho phép.
  Trên các máy chủ node Windows, các dạng shell-wrapper như `cmd.exe /c ...` được coi là không khớp danh sách cho phép trong
  chế độ danh sách cho phép trừ khi được phê duyệt qua luồng yêu cầu.
## Vòng lặp phục hồi nhanh

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

Nếu vẫn gặp vấn đề:

- Phê duyệt lại ghép nối thiết bị.
- Mở lại ứng dụng node (foreground).
- Cấp lại quyền hệ điều hành.
- Tạo lại/điều chỉnh chính sách phê duyệt exec.

Liên quan:

- [/nodes/index](/nodes/index)
- [/nodes/camera](/nodes/camera)
- [/nodes/location-command](/nodes/location-command)
- [/tools/exec-approvals](/tools/exec-approvals)
- [/gateway/pairing](/gateway/pairing)