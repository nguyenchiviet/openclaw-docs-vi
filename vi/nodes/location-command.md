---
summary: 'Lệnh Location cho các node (location.get), chế độ quyền, và hành vi chạy nền'
read_when:
  - Hỗ trợ nút vị trí hoặc giao diện người dùng quyền truy cập
  - Thiết kế các luồng vị trí nền + đẩy thông báo
title: Lệnh Vị Trí
x-i18n:
  source_path: nodes\location-command.md
  source_hash: 23124096256384d2b28157352b072309c61c970a20e009aac5ce4a8250dc3764
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:53.446Z'
---

# Lệnh vị trí (nodes)

## TL;DR

- `location.get` là một lệnh node (thông qua `node.invoke`).
- Tắt theo mặc định.
- Cài đặt sử dụng bộ chọn: Tắt / Khi đang sử dụng / Luôn luôn.
- Bật tắt riêng: Vị trí chính xác.
## Tại sao lại là bộ chọn (không chỉ là công tắc)

Quyền hạn của hệ điều hành có nhiều cấp độ. Chúng tôi có thể hiển thị bộ chọn trong ứng dụng, nhưng hệ điều hành vẫn quyết định việc cấp quyền thực tế.

- iOS/macOS: người dùng có thể chọn **While Using** (Khi đang sử dụng) hoặc **Always** (Luôn luôn) trong các lời nhắc hệ thống/Cài đặt. Ứng dụng có thể yêu cầu nâng cấp, nhưng hệ điều hành có thể yêu cầu Cài đặt.
- Android: vị trí nền là một quyền riêng biệt; trên Android 10+ nó thường yêu cầu một quy trình Cài đặt.
- Vị trí chính xác là một cấp quyền riêng biệt (iOS 14+ "Precise", Android "fine" so với "coarse").

Bộ chọn trong giao diện người dùng điều khiển chế độ được yêu cầu của chúng tôi; cấp quyền thực tế nằm trong cài đặt hệ điều hành.
## Mô hình Cài đặt

Mỗi thiết bị node:

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

Hành vi giao diện người dùng:

- Chọn `whileUsing` yêu cầu quyền foreground.
- Chọn `always` trước tiên đảm bảo `whileUsing`, sau đó yêu cầu background (hoặc gửi người dùng đến Cài đặt nếu cần).
- Nếu hệ điều hành từ chối mức được yêu cầu, hãy quay lại mức cao nhất được cấp và hiển thị trạng thái.
## Ánh xạ quyền (node.permissions)

Tùy chọn. Node macOS báo cáo `location` thông qua bản đồ quyền; iOS/Android có thể bỏ qua nó.
## Command: `location.get`

Được gọi thông qua `node.invoke`.

Params (gợi ý):

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Response payload:

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Errors (stable codes):

- `LOCATION_DISABLED`: selector is off.
- `LOCATION_PERMISSION_REQUIRED`: permission missing for requested mode.
- `LOCATION_BACKGROUND_UNAVAILABLE`: app is backgrounded but only While Using allowed.
- `LOCATION_TIMEOUT`: no fix in time.
- `LOCATION_UNAVAILABLE`: lỗi hệ thống / không có nhà cung cấp.
## Hành vi chạy nền (tương lai)

Mục tiêu: mô hình có thể yêu cầu vị trí ngay cả khi node chạy nền, nhưng chỉ khi:

- Người dùng chọn **Luôn luôn**.
- OS cấp quyền vị trí nền.
- Ứng dụng được phép chạy nền cho vị trí (chế độ nền iOS / dịch vụ foreground Android hoặc cấp quyền đặc biệt).

Luồng được kích hoạt bằng push (tương lai):

1. Gateway gửi push đến node (silent push hoặc dữ liệu FCM).
2. Node thức dậy ngắn gọn và yêu cầu vị trí từ thiết bị.
3. Node chuyển tiếp payload đến Gateway.

Ghi chú:

- iOS: Yêu cầu quyền Luôn luôn + chế độ vị trí nền. Silent push có thể bị giới hạn; dự kiến các lỗi không liên tục.
- Android: vị trí nền có thể yêu cầu dịch vụ foreground; nếu không, dự kiến bị từ chối.
## Tích hợp Mô hình/Công cụ

- Bề mặt công cụ: `nodes` tool adds `location_get` action (node required).
- CLI: `openclaw nodes location get --node <id>`.
- Hướng dẫn agent: chỉ gọi khi người dùng bật vị trí và hiểu phạm vi.
## Sao chép UX (được đề xuất)

- Off: "Chia sẻ vị trí bị tắt."
- While Using: "Chỉ khi OpenClaw đang mở."
- Always: "Cho phép vị trí nền. Yêu cầu quyền hệ thống."
- Precise: "Sử dụng vị trí GPS chính xác. Tắt để chia sẻ vị trí gần đúng."