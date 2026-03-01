---
summary: Manifest plugin + yêu cầu JSON schema (xác thực cấu hình nghiêm ngặt)
read_when:
  - Bạn đang xây dựng một plugin OpenClaw
  - Bạn cần gửi một schema cấu hình plugin hoặc gỡ lỗi các lỗi xác thực plugin
title: Bản kê khai Plugin
x-i18n:
  source_path: plugins\manifest.md
  source_hash: 234c7c0e77f22f5cd3c7fa0c06d442ce2c543b45cdeb35229d19f2f805dafcd2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:50.014Z'
---

# Tệp kê khai plugin (openclaw.plugin.json)

Mỗi plugin **phải** có một tệp `openclaw.plugin.json` trong **thư mục gốc của plugin**.
OpenClaw sử dụng tệp kê khai này để xác thực cấu hình **mà không thực thi mã plugin**.
Các tệp kê khai bị thiếu hoặc không hợp lệ được coi là lỗi plugin và chặn xác thực cấu hình.

Xem hướng dẫn hệ thống plugin đầy đủ: [Plugins](/tools/plugin).
## Các trường bắt buộc

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Required keys:

- `id` (string): canonical plugin id.
- `configSchema` (object): JSON Schema for plugin config (inline).

Optional keys:

- `kind` (string): plugin kind (example: `"memory"`).
- `channels` (array): channel ids registered by this plugin (example: `["matrix"]`).
- `providers` (array): provider ids registered by this plugin.
- `skills` (array): skill directories to load (relative to the plugin root).
- `name` (string): display name for the plugin.
- `description` (string): short plugin summary.
- `uiHints` (object): config field labels/placeholders/sensitive flags for UI rendering.
- `version` (string): phiên bản plugin (thông tin).
## Yêu cầu JSON Schema

- **Mỗi plugin phải có một JSON Schema**, ngay cả khi nó không chấp nhận cấu hình.
- Một schema trống là chấp nhận được (ví dụ: `{ "type": "object", "additionalProperties": false }`).
- Các schema được xác thực khi đọc/ghi cấu hình, không phải khi chạy.
## Hành vi xác thực

- Các khóa `channels.*` không xác định là **lỗi**, trừ khi id kênh được khai báo bởi
  một plugin manifest.
- `plugins.entries.<id>`, `plugins.allow`, `plugins.deny`, và `plugins.slots.*`
  phải tham chiếu đến các id plugin **có thể khám phá**. Các id không xác định là **lỗi**.
- Nếu một plugin được cài đặt nhưng có manifest hoặc schema bị hỏng hoặc bị thiếu,
  xác thực sẽ thất bại và Doctor sẽ báo cáo lỗi plugin.
- Nếu cấu hình plugin tồn tại nhưng plugin bị **vô hiệu hóa**, cấu hình sẽ được giữ lại và
  một **cảnh báo** sẽ được hiển thị trong Doctor + logs.
## Ghi chú

- Manifest là **bắt buộc cho tất cả các plugin**, bao gồm cả các tải từ hệ thống tệp cục bộ.
- Runtime vẫn tải mô-đun plugin riêng biệt; manifest chỉ dùng cho
  khám phá + xác thực.
- Nếu plugin của bạn phụ thuộc vào các mô-đun gốc, hãy ghi lại các bước xây dựng và bất kỳ
  yêu cầu allowlist của trình quản lý gói nào (ví dụ: pnpm `allow-build-scripts`
  - `pnpm rebuild <package>`).