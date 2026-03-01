---
summary: Cài đặt Skills trên macOS và trạng thái được hỗ trợ bởi gateway
read_when:
  - Cập nhật giao diện cài đặt Skills trên macOS
  - Thay đổi gating hoặc hành vi cài đặt Skills
title: Skills
x-i18n:
  source_path: platforms\mac\skills.md
  source_hash: ecd5286bbe49eed89319686c4f7d6da55ef7b0d3952656ba98ef5e769f3fbf79
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:30.546Z'
---

# Skills (macOS)

Ứng dụng macOS hiển thị các Skills OpenClaw thông qua gateway; nó không phân tích các Skills cục bộ.

## Nguồn dữ liệu

- `skills.status` (gateway) trả về tất cả các Skills cùng với tính đủ điều kiện và các yêu cầu bị thiếu
  (bao gồm các khối danh sách cho phép đối với các Skills được đóng gói).
- Các yêu cầu được lấy từ `metadata.openclaw.requires` trong mỗi `SKILL.md`.

## Hành động cài đặt

- `metadata.openclaw.install` xác định các tùy chọn cài đặt (brew/node/go/uv).
- Ứng dụng gọi `skills.install` để chạy các trình cài đặt trên máy chủ gateway.
- Gateway chỉ hiển thị một trình cài đặt ưu tiên khi có nhiều trình được cung cấp
  (brew khi có sẵn, nếu không thì trình quản lý node từ `skills.install`, mặc định là npm).

## Biến môi trường/Khóa API

- Ứng dụng lưu trữ các khóa trong `~/.openclaw/openclaw.json` dưới `skills.entries.<skillKey>`.
- `skills.update` vá `enabled`, `apiKey`, và `env`.

## Chế độ từ xa

- Cài đặt + cập nhật cấu hình xảy ra trên máy chủ gateway (không phải Mac cục bộ).