---
summary: 'Các script trong kho lưu trữ: mục đích, phạm vi và lưu ý an toàn'
read_when:
  - Chạy các script từ repo
  - Thêm hoặc thay đổi các script trong ./scripts
title: Kịch bản
x-i18n:
  source_path: help\scripts.md
  source_hash: efd220df28f20b338fbc4f5e6152c8abeade4b56f76496476e7e99928a8dedbe
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:05:49.040Z'
---

# Scripts

Thư mục `scripts/` chứa các script trợ giúp cho quy trình làm việc cục bộ và các tác vụ vận hành.
Sử dụng những script này khi một tác vụ rõ ràng liên kết với một script; nếu không, hãy ưu tiên CLI.

## Quy ước

- Scripts là **tùy chọn** trừ khi được tham chiếu trong tài liệu hoặc danh sách kiểm tra phát hành.
- Ưu tiên các giao diện CLI khi chúng tồn tại (ví dụ: giám sát xác thực sử dụng `openclaw models status --check`).
- Giả định rằng scripts là dành riêng cho máy chủ; hãy đọc chúng trước khi chạy trên một máy mới.

## Scripts giám sát xác thực

Scripts giám sát xác thực được tài liệu hóa tại đây:
[/automation/auth-monitoring](/automation/auth-monitoring)

## Khi thêm scripts

- Giữ scripts tập trung và có tài liệu.
- Thêm một mục ngắn trong tài liệu liên quan (hoặc tạo một tài liệu nếu thiếu).