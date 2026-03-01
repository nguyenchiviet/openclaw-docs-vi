---
summary: >-
  Chụp ảnh từ camera (iOS node + ứng dụng macOS) để agent sử dụng: ảnh (jpg) và
  video clip ngắn (mp4)
read_when:
  - Thêm hoặc sửa đổi chức năng chụp ảnh camera trên các nút iOS hoặc macOS
  - Mở rộng quy trình tệp tạm thời MEDIA có thể truy cập bởi agent
title: Chụp Ảnh Máy Ảnh
x-i18n:
  source_path: nodes\camera.md
  source_hash: d028f41eebe3058fbace2186f8703a6b90d39f642884c2c493eb2d10c39072aa
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:16.140Z'
---

# Chụp ảnh từ camera (agent)

OpenClaw hỗ trợ **chụp ảnh từ camera** cho các quy trình làm việc của agent:

- **node iOS** (được ghép nối qua Gateway): chụp **ảnh** (`jpg`) hoặc **video clip ngắn** (`mp4`, có tùy chọn âm thanh) qua `node.invoke`.
- **node Android** (được ghép nối qua Gateway): chụp **ảnh** (`jpg`) hoặc **video clip ngắn** (`mp4`, có tùy chọn âm thanh) qua `node.invoke`.
- **ứng dụng macOS** (node qua Gateway): chụp **ảnh** (`jpg`) hoặc **video clip ngắn** (`mp4`, có tùy chọn âm thanh) qua `node.invoke`.

Tất cả quyền truy cập camera đều được kiểm soát bởi **cài đặt do người dùng kiểm soát**.
## Nút iOS

### Cài đặt người dùng (bật theo mặc định)

- Tab Cài đặt iOS → **Camera** → **Cho phép Camera** (`camera.enabled`)
  - Mặc định: **bật** (khóa bị thiếu được coi là bật).
  - Khi tắt: các lệnh `camera.*` trả về `CAMERA_DISABLED`.

### Lệnh (qua Gateway `node.invoke`)

- `camera.list`
  - Tải trọng phản hồi:
    - `devices`: mảng của `{ id, name, position, deviceType }`

- `camera.snap`
  - Tham số:
    - `facing`: `front|back` (mặc định: `front`)
    - `maxWidth`: số (tùy chọn; mặc định `1600` trên nút iOS)
    - `quality`: `0..1` (tùy chọn; mặc định `0.9`)
    - `format`: hiện tại `jpg`
    - `delayMs`: số (tùy chọn; mặc định `0`)
    - `deviceId`: chuỗi (tùy chọn; từ `camera.list`)
  - Tải trọng phản hồi:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Bảo vệ tải trọng: các ảnh được nén lại để giữ tải trọng base64 dưới 5 MB.

- `camera.clip`
  - Tham số:
    - `facing`: `front|back` (mặc định: `front`)
    - `durationMs`: số (mặc định `3000`, giới hạn tối đa `60000`)
    - `includeAudio`: boolean (mặc định `true`)
    - `format`: hiện tại `mp4`
    - `deviceId`: chuỗi (tùy chọn; từ `camera.list`)
  - Tải trọng phản hồi:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

### Yêu cầu chạy ở nền trước

Giống như `canvas.*`, nút iOS chỉ cho phép các lệnh `camera.*` chạy ở **nền trước**. Các lệnh gọi ở nền sau trả về `NODE_BACKGROUND_UNAVAILABLE`.

### Trợ giúp CLI (tệp tạm thời + MEDIA)

Cách dễ nhất để lấy tệp đính kèm là thông qua trợ giúp CLI, trợ giúp này ghi các phương tiện được giải mã vào tệp tạm thời và in `MEDIA:<path>`.

Ví dụ:

```bash
openclaw nodes camera snap --node <id>               # default: both front + back (2 MEDIA lines)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Ghi chú:
- `nodes camera snap` mặc định là **cả hai** hướng để cung cấp cho agent cả hai chế độ xem.
- Các tệp đầu ra là tạm thời (trong thư mục tạm của hệ điều hành) trừ khi bạn xây dựng trình bao bọc của riêng mình.

## Android node

### Cài đặt người dùng Android (bật theo mặc định)

- Bảng Cài đặt Android → **Camera** → **Cho phép Camera** (`camera.enabled`)
  - Mặc định: **bật** (khóa bị thiếu được coi là được bật).
  - Khi tắt: các lệnh `camera.*` trả về `CAMERA_DISABLED`.

### Quyền

- Android yêu cầu quyền thời gian chạy:
  - `CAMERA` cho cả `camera.snap` và `camera.clip`.
  - `RECORD_AUDIO` cho `camera.clip` khi `includeAudio=true`.

Nếu quyền bị thiếu, ứng dụng sẽ nhắc khi có thể; nếu bị từ chối, các yêu cầu `camera.*` sẽ thất bại với lỗi `*_PERMISSION_REQUIRED`.

### Yêu cầu chạy ở chế độ nền trước của Android

Giống như `canvas.*`, Android node chỉ cho phép các lệnh `camera.*` ở **chế độ nền trước**. Các lệnh gọi ở chế độ nền trả về `NODE_BACKGROUND_UNAVAILABLE`.

### Lệnh Android (qua Gateway `node.invoke`)

- `camera.list`
  - Tải trọng phản hồi:
    - `devices`: mảng của `{ id, name, position, deviceType }`

### Bảo vệ tải trọng

Các ảnh được nén lại để giữ tải trọng base64 dưới 5 MB.
## Ứng dụng macOS

### Cài đặt người dùng (tắt theo mặc định)

Ứng dụng đi kèm macOS hiển thị một hộp kiểm:

- **Settings → General → Allow Camera** (`openclaw.cameraEnabled`)
  - Mặc định: **tắt**
  - Khi tắt: các yêu cầu camera trả về "Camera disabled by user".

### CLI helper (node invoke)

Sử dụng CLI `openclaw` chính để gọi các lệnh camera trên node macOS.

Ví dụ:

```bash
openclaw nodes camera list --node <id>            # list camera ids
openclaw nodes camera snap --node <id>            # prints MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # prints MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # prints MEDIA:<path> (legacy flag)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Notes:

- `openclaw nodes camera snap` defaults to `maxWidth=1600` unless overridden.
- On macOS, `camera.snap` waits `delayMs` (mặc định 2000ms) sau khi khởi động/tiếp xúc ổn định trước khi chụp.
- Các tải trọng ảnh được nén lại để giữ base64 dưới 5 MB.
## An toàn + giới hạn thực tế

- Truy cập camera và microphone kích hoạt các lời nhắc cấp phép thông thường của hệ điều hành (và yêu cầu chuỗi sử dụng trong Info.plist).
- Các clip video bị giới hạn (hiện tại `<= 60s`) để tránh payload node quá lớn (chi phí base64 + giới hạn tin nhắn).
## Video màn hình macOS (cấp độ hệ điều hành)

Đối với video _màn hình_ (không phải camera), hãy sử dụng ứng dụng đi kèm macOS:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # prints MEDIA:<path>
```

Ghi chú:

- Yêu cầu quyền **Ghi hình màn hình** của macOS (TCC).