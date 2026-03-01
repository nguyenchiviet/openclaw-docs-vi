---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw configure` (các lời nhắc cấu hình tương
  tác)
read_when:
  - >-
    Bạn muốn điều chỉnh thông tin xác thực, thiết bị hoặc các cài đặt mặc định
    của agent một cách tương tác
title: cấu hình
x-i18n:
  source_path: cli\configure.md
  source_hash: a650d7ac9f9b587ab76968dcc7e214f4348446953bf094ed666fdf3123f6908c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:24.252Z'
---

# Thiết lập ban đầu

Lời nhắc tương tác để thiết lập thông tin xác thực, thiết bị và các giá trị mặc định của agent.

Lưu ý: Phần **Mô hình** hiện bao gồm tùy chọn đa lựa chọn cho danh sách cho phép `agents.defaults.models` (những gì hiển thị trong `/model` và bộ chọn mô hình).

Mẹo: `openclaw config` mà không có lệnh con sẽ mở cùng một trình hướng dẫn. Sử dụng `openclaw config get|set|unset` để chỉnh sửa không tương tác.

Liên quan:

- Tham chiếu cấu hình Gateway: [Cấu hình](/gateway/configuration)
- CLI cấu hình: [Cấu hình](/cli/config)

Ghi chú:

- Chọn nơi Gateway chạy sẽ luôn cập nhật `gateway.mode`. Bạn có thể chọn "Tiếp tục" mà không cần các phần khác nếu đó là tất cả những gì bạn cần.
- Các dịch vụ hướng tới kênh (Slack/Discord/Matrix/Microsoft Teams) sẽ nhắc nhập danh sách cho phép kênh/phòng trong quá trình thiết lập. Bạn có thể nhập tên hoặc ID; trình hướng dẫn sẽ phân giải tên thành ID khi có thể.

## Ví dụ

```bash
openclaw configure
openclaw configure --section model --section channels
```