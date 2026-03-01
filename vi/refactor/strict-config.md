---
summary: Kiểm tra cấu hình nghiêm ngặt + các migration chỉ dành cho doctor
read_when:
  - Thiết kế hoặc triển khai hành vi xác thực cấu hình
  - Làm việc trên các migration cấu hình hoặc quy trình doctor
  - Xử lý các schema cấu hình plugin hoặc gating tải plugin
title: Xác Thực Cấu Hình Nghiêm Ngặt
x-i18n:
  source_path: refactor\strict-config.md
  source_hash: ffce29de7d5a983cfa8e24ba7a23cef3f502d4b7188b55644a0e4c86e5f013a5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:19:39.288Z'
---

# Xác thực cấu hình nghiêm ngặt (migrations chỉ dành cho doctor)

## Mục tiêu

- **Từ chối các khóa cấu hình không xác định ở mọi nơi** (root + nested), ngoại trừ `$schema` metadata ở root.
- **Từ chối cấu hình plugin mà không có schema**; không tải plugin đó.
- **Loại bỏ auto-migration kế thừa khi tải**; migrations chạy qua doctor chỉ.
- **Tự động chạy doctor (dry-run) khi khởi động**; nếu không hợp lệ, chặn các lệnh không phải chẩn đoán.
## Những điều không phải là mục tiêu

- Tương thích ngược khi tải (các khóa cũ không tự động di chuyển).
- Bỏ qua im lặng các khóa không được nhận dạng.
## Quy tắc xác thực nghiêm ngặt

- Cấu hình phải khớp với schema chính xác ở mọi cấp độ.
- Các khóa không xác định là lỗi xác thực (không cho phép passthrough ở root hoặc nested), ngoại trừ root `$schema` khi nó là một chuỗi.
- `plugins.entries.<id>.config` phải được xác thực bởi schema của plugin.
  - Nếu một plugin thiếu schema, **từ chối tải plugin** và hiển thị một lỗi rõ ràng.
- Các khóa `channels.<id>` không xác định là lỗi trừ khi manifest của plugin khai báo id kênh.
- Plugin manifests (`openclaw.plugin.json`) là bắt buộc cho tất cả các plugin.
## Thực thi lược đồ plugin

- Mỗi plugin cung cấp một JSON Schema nghiêm ngặt cho cấu hình của nó (nội tuyến trong manifest).
- Luồng tải plugin:
  1. Giải quyết manifest plugin + lược đồ (`openclaw.plugin.json`).
  2. Xác thực cấu hình dựa trên lược đồ.
  3. Nếu thiếu lược đồ hoặc cấu hình không hợp lệ: chặn tải plugin, ghi lại lỗi.
- Thông báo lỗi bao gồm:
  - ID plugin
  - Lý do (thiếu lược đồ / cấu hình không hợp lệ)
  - Đường dẫn không vượt qua xác thực
- Các plugin bị vô hiệu hóa giữ lại cấu hình của chúng, nhưng Doctor + nhật ký hiển thị cảnh báo.
## Luồng Doctor

- Doctor chạy **mỗi lần** cấu hình được tải (chế độ dry-run theo mặc định).
- Nếu cấu hình không hợp lệ:
  - In tóm tắt + lỗi có thể hành động được.
  - Hướng dẫn: `openclaw doctor --fix`.
- `openclaw doctor --fix`:
  - Áp dụng các bản di chuyển.
  - Xóa các khóa không xác định.
  - Ghi cấu hình được cập nhật.
## Gating lệnh (khi cấu hình không hợp lệ)

Được phép (chỉ chẩn đoán):

- `openclaw doctor`
- `openclaw logs`
- `openclaw health`
- `openclaw help`
- `openclaw status`
- `openclaw gateway status`

Mọi thứ khác phải thất bại cứng với: "Cấu hình không hợp lệ. Chạy `openclaw doctor --fix`."
## Định dạng UX lỗi

- Tiêu đề tóm tắt duy nhất.
- Các phần được nhóm:
  - Các khóa không xác định (đường dẫn đầy đủ)
  - Các khóa cũ / cần di chuyển
  - Lỗi tải plugin (id plugin + lý do + đường dẫn)
## Các điểm triển khai

- `src/config/zod-schema.ts`: loại bỏ passthrough gốc; các đối tượng nghiêm ngặt ở mọi nơi.
- `src/config/zod-schema.providers.ts`: đảm bảo các schema kênh nghiêm ngặt.
- `src/config/validation.ts`: thất bại trên các khóa không xác định; không áp dụng các di chuyển kế thừa.
- `src/config/io.ts`: loại bỏ các di chuyển tự động kế thừa; luôn chạy doctor dry-run.
- `src/config/legacy*.ts`: chuyển cách sử dụng sang doctor chỉ.
- `src/plugins/*`: thêm schema registry + gating.
- CLI command gating trong `src/cli`.
## Kiểm tra

- Từ chối khóa không xác định (gốc + lồng nhau).
- Plugin thiếu schema → tải plugin bị chặn với lỗi rõ ràng.
- Cấu hình không hợp lệ → khởi động gateway bị chặn ngoại trừ các lệnh chẩn đoán.
- Doctor dry-run tự động; `doctor --fix` ghi cấu hình đã sửa.