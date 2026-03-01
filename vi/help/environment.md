---
summary: Nơi OpenClaw tải các biến môi trường và thứ tự ưu tiên
read_when:
  - Bạn cần biết những biến môi trường nào được tải và theo thứ tự nào.
  - Bạn đang gỡ lỗi các khóa API bị thiếu trong Gateway
  - Bạn đang tài liệu hóa xác thực nhà cung cấp hoặc môi trường triển khai
title: Biến Môi Trường
x-i18n:
  source_path: help\environment.md
  source_hash: 50a896f288cc609bc9e2dfe6fb86b27b64d2b597f660bf82dccbba1a7c470d85
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:00:25.000Z'
---

# Biến môi trường

OpenClaw lấy biến môi trường từ nhiều nguồn. Quy tắc là **không bao giờ ghi đè các giá trị hiện có**.
## Thứ tự ưu tiên (cao nhất → thấp nhất)

1. **Biến môi trường của Process** (những gì Gateway process đã có từ shell/daemon cha).
2. **`.env` trong thư mục làm việc hiện tại** (dotenv mặc định; không ghi đè).
3. **`.env` toàn cục** tại `~/.openclaw/.env` (hay còn gọi là `$OPENCLAW_STATE_DIR/.env`; không ghi đè).
4. **Khối `env` Config** trong `~/.openclaw/openclaw.json` (chỉ áp dụng nếu thiếu).
5. **Nhập login-shell tùy chọn** (`env.shellEnv.enabled` hoặc `OPENCLAW_LOAD_SHELL_ENV=1`), chỉ áp dụng cho các khóa dự kiến bị thiếu.

Nếu tệp cấu hình hoàn toàn bị thiếu, bước 4 sẽ bị bỏ qua; nhập shell vẫn chạy nếu được bật.
## Khối cấu hình `env`

Hai cách tương đương để đặt biến môi trường nội tuyến (cả hai đều không ghi đè):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```
## Nhập biến môi trường Shell

`env.shellEnv` chạy shell đăng nhập của bạn và chỉ nhập các khóa dự kiến **bị thiếu**:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Env var equivalents:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`
## Thay thế biến môi trường trong cấu hình

Bạn có thể tham chiếu các biến môi trường trực tiếp trong các giá trị chuỗi cấu hình bằng cú pháp `${VAR_NAME}`:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

Xem [Cấu hình: Thay thế biến môi trường](/gateway/configuration#env-var-substitution-in-config) xem chi tiết.
## Tham chiếu bí mật so với `${ENV}` chuỗi

OpenClaw hỗ trợ hai mẫu được điều khiển bởi env:

- `${VAR}` thay thế chuỗi trong các giá trị cấu hình.
- Các đối tượng SecretRef (`{ source: "env", provider: "default", id: "VAR" }`) cho các trường hỗ trợ tham chiếu bí mật.

Cả hai đều được giải quyết từ env quá trình tại thời điểm kích hoạt. Chi tiết SecretRef được ghi lại trong [Quản lý Bí mật](/gateway/secrets).
## Biến môi trường liên quan đến đường dẫn

| Variable               | Purpose                                                                                                                                                                          |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_HOME`        | Ghi đè thư mục chính được sử dụng để giải quyết tất cả các đường dẫn nội bộ (`~/.openclaw/`, thư mục agent, phiên, thông tin xác thực). Hữu ích khi chạy OpenClaw như một người dùng dịch vụ chuyên dụng. |
| `OPENCLAW_STATE_DIR`   | Ghi đè thư mục trạng thái (mặc định `~/.openclaw`).                                                                                                                            |
| `OPENCLAW_CONFIG_PATH` | Ghi đè đường dẫn tệp cấu hình (mặc định `~/.openclaw/openclaw.json`).                                                                                                                             |
## Ghi nhật ký

| Biến                 | Mục đích                                                                                                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_LOG_LEVEL` | Ghi đè mức độ ghi nhật ký cho cả tệp và bảng điều khiển (ví dụ: `debug`, `trace`). Có ưu tiên cao hơn `logging.level` và `logging.consoleLevel` trong cấu hình. Các giá trị không hợp lệ sẽ bị bỏ qua với cảnh báo. |

### `OPENCLAW_HOME`

Khi được đặt, `OPENCLAW_HOME` thay thế thư mục chính của hệ thống (`$HOME` / `os.homedir()`) cho tất cả độ phân giải đường dẫn nội bộ. Điều này cho phép cách ly hệ thống tệp hoàn toàn cho các tài khoản dịch vụ không giao diện.

**Ưu tiên:** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

**Ví dụ** (macOS LaunchDaemon):

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` can also be set to a tilde path (e.g. `~/svc`), which gets expanded using `$HOME` trước khi sử dụng.
## Liên quan

- [Cấu hình Gateway](/gateway/configuration)
- [Câu hỏi thường gặp: biến môi trường và tải .env](/help/faq#env-vars-and-env-loading)
- [Tổng quan về mô hình](/concepts/models)