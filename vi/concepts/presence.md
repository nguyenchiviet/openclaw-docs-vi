---
summary: 'Cách các mục nhập presence của OpenClaw được tạo ra, hợp nhất và hiển thị'
read_when:
  - Gỡ lỗi tab Instances
  - Điều tra các hàng instance trùng lặp hoặc cũ
  - Thay đổi gateway WS connect hoặc system-event beacons
title: Sự hiện diện
x-i18n:
  source_path: concepts\presence.md
  source_hash: c752c76a880878fed673d656db88beb5dbdeefff2491985127ad791521f97d00
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:01.411Z'
---

# Sự hiện diện

"Sự hiện diện" của OpenClaw là một chế độ xem nhẹ, tối ưu hết sức của:

- **Gateway** chính nó, và
- **các client kết nối với Gateway** (ứng dụng mac, WebChat, CLI, v.v.)

Sự hiện diện được sử dụng chủ yếu để hiển thị tab **Instances** của ứng dụng macOS và cung cấp khả năng hiển thị nhanh cho người điều hành.
## Các trường Presence (những gì được hiển thị)

Các mục Presence là các đối tượng có cấu trúc với các trường như:

- `instanceId` (tùy chọn nhưng được khuyến nghị mạnh mẽ): định danh máy khách ổn định (thường là `connect.client.instanceId`)
- `host`: tên máy chủ thân thiện với con người
- `ip`: địa chỉ IP nỗ lực tốt nhất
- `version`: chuỗi phiên bản máy khách
- `deviceFamily` / `modelIdentifier`: gợi ý phần cứng
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: "giây kể từ lần nhập liệu cuối cùng của người dùng" (nếu biết)
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: dấu thời gian cập nhật cuối cùng (ms kể từ epoch)
## Nhà sản xuất (nơi presence đến từ)

Các mục presence được tạo ra từ nhiều nguồn và **được hợp nhất**.

### 1) Mục self của Gateway

Gateway luôn tạo một mục "self" khi khởi động để các UI hiển thị máy chủ gateway
ngay cả trước khi bất kỳ client nào kết nối.

### 2) WebSocket connect

Mỗi client WS bắt đầu với một yêu cầu `connect`. Khi bắt tay thành công,
Gateway sẽ upsert một mục presence cho kết nối đó.

#### Tại sao các lệnh CLI một lần không hiển thị

CLI thường kết nối cho các lệnh một lần ngắn. Để tránh spam danh sách
Instances, `client.mode === "cli"` **không** được chuyển thành mục presence.

### 3) Các beacon `system-event`

Các client có thể gửi các beacon định kỳ phong phú hơn thông qua phương thức `system-event`. Ứng dụng mac
sử dụng điều này để báo cáo tên máy chủ, IP và `lastInputSeconds`.

### 4) Node connects (role: node)

Khi một node kết nối qua Gateway WebSocket với `role: node`, Gateway
sẽ upsert một mục presence cho node đó (cùng luồng như các client WS khác).
## Quy tắc hợp nhất + loại bỏ trùng lặp (tại sao `instanceId` quan trọng)

Các mục nhập hiện diện được lưu trữ trong một bản đồ trong bộ nhớ duy nhất:

- Các mục nhập được khóa bằng một **khóa hiện diện**.
- Khóa tốt nhất là một `instanceId` ổn định (từ `connect.client.instanceId`) tồn tại qua các lần khởi động lại.
- Các khóa không phân biệt chữ hoa chữ thường.

Nếu một client kết nối lại mà không có `instanceId` ổn định, nó có thể xuất hiện dưới dạng một
**hàng trùng lặp**.
## TTL và kích thước giới hạn

Presence được thiết kế để tồn tại tạm thời:

- **TTL:** các mục cũ hơn 5 phút sẽ bị xóa
- **Số mục tối đa:** 200 (mục cũ nhất bị loại bỏ trước)

Điều này giữ cho danh sách luôn mới và tránh tăng trưởng bộ nhớ không giới hạn.
## Cảnh báo về kết nối từ xa/tunnel (IP loopback)

Khi một client kết nối qua SSH tunnel / local port forward, Gateway có thể
thấy địa chỉ từ xa là `127.0.0.1`. Để tránh ghi đè một IP client được báo cáo tốt,
các địa chỉ từ xa loopback bị bỏ qua.
## Người tiêu dùng

### Tab Instances trên macOS

Ứng dụng macOS hiển thị kết quả đầu ra của `system-presence` và áp dụng một chỉ báo trạng thái nhỏ (Hoạt động/Không hoạt động/Cũ) dựa trên thời gian của bản cập nhật cuối cùng.
## Mẹo gỡ lỗi

- Để xem danh sách thô, gọi `system-presence` đối với Gateway.
- Nếu bạn thấy các mục trùng lặp:
  - xác nhận rằng các client gửi một `client.instanceId` ổn định trong bắt tay
  - xác nhận rằng các beacon định kỳ sử dụng cùng một `instanceId`
  - kiểm tra xem mục dẫn xuất từ kết nối có thiếu `instanceId` hay không (các mục trùng lặp là dự kiến)