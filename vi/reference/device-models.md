---
summary: >-
  Cách OpenClaw cung cấp các định danh mô hình thiết bị Apple dưới dạng tên thân
  thiện trong ứng dụng macOS.
read_when:
  - Cập nhật ánh xạ định danh mô hình thiết bị hoặc các tệp NOTICE/license
  - Thay đổi cách Instances UI hiển thị tên thiết bị
title: Cơ sở Dữ liệu Mô hình Thiết bị
x-i18n:
  source_path: reference\device-models.md
  source_hash: 1d99c2538a0d8fdd80fa468fa402f63479ef2522e83745a0a46527a86238aeb2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:19:48.659Z'
---

# Cơ sở dữ liệu mô hình thiết bị (tên thân thiện)

Ứng dụng đi kèm macOS hiển thị tên mô hình thiết bị Apple thân thiện trong UI **Instances** bằng cách ánh xạ các định danh mô hình Apple (ví dụ: `iPad16,6`, `Mac16,6`) thành các tên có thể đọc được.

Ánh xạ được đưa vào dưới dạng JSON tại:

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## Nguồn dữ liệu

Hiện tại chúng tôi đưa vào ánh xạ từ kho lưu trữ được cấp phép MIT:

- `kyle-seongwoo-jun/apple-device-identifiers`

Để giữ cho các bản dựng có tính xác định, các tệp JSON được ghim vào các commit thượng nguồn cụ thể (được ghi lại trong `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

## Cập nhật cơ sở dữ liệu

1. Chọn các commit thượng nguồn bạn muốn ghim (một cho iOS, một cho macOS).
2. Cập nhật các hash commit trong `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3. Tải lại các tệp JSON, được ghim vào các commit đó:

```bash
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<commit sha for mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. Ensure `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` still matches upstream (replace it if the upstream license changes).
5. Verify the macOS app builds cleanly (no warnings):

```bash
swift build --package-path apps/macos
```