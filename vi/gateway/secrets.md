---
summary: >-
  Quản lý bí mật: hợp đồng SecretRef, hành vi chụp nhanh thời gian chạy và xóa
  an toàn một chiều
read_when:
  - >-
    Cấu hình SecretRefs cho các nhà cung cấp, hồ sơ xác thực, kỹ năng hoặc
    Google Chat
  - >-
    Tải lại/kiểm toán/cấu hình/áp dụng các bí mật hoạt động một cách an toàn
    trong môi trường sản xuất
  - Hiểu về hành vi fail-fast và last-known-good
title: Quản lý Bí mật
x-i18n:
  source_path: gateway\secrets.md
  source_hash: eba9ddd953d4c866ad06a8b691516f4a97f574893e20d8a8f051d980723f1168
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:58.330Z'
---

# Quản lý bí mật

OpenClaw hỗ trợ các tham chiếu bí mật bổ sung để thông tin xác thực không cần phải được lưu trữ dưới dạng văn bản thuần túy trong các tệp cấu hình.

Văn bản thuần túy vẫn hoạt động. Các tham chiếu bí mật là tùy chọn.
## Mục tiêu và mô hình runtime

Các bí mật được phân giải thành một ảnh chụp runtime trong bộ nhớ.

- Phân giải được thực hiện sớm trong quá trình kích hoạt, không phải lười biếng trên các đường dẫn yêu cầu.
- Khởi động thất bại nhanh nếu bất kỳ thông tin xác thực được tham chiếu nào không thể được phân giải.
- Tải lại sử dụng hoán đổi nguyên tử: thành công hoàn toàn hoặc giữ lại phiên bản tốt cuối cùng.
- Các yêu cầu runtime đọc từ ảnh chụp trong bộ nhớ đang hoạt động.

Điều này giữ cho các sự cố của nhà cung cấp bí mật ngoài đường dẫn yêu cầu nóng.
## Tham chiếu thiết lập ban đầu

Khi thiết lập ban đầu chạy ở chế độ tương tác và bạn chọn lưu trữ tham chiếu bí mật, OpenClaw thực hiện kiểm tra preflight nhanh trước khi lưu:

- Tham chiếu Env: xác thực tên biến môi trường và xác nhận giá trị không trống hiển thị trong quá trình thiết lập ban đầu.
- Tham chiếu nhà cung cấp (`file` hoặc `exec`): xác thực nhà cung cấp được chọn, giải quyết `id` được cung cấp, và kiểm tra loại giá trị.

Nếu xác thực không thành công, thiết lập ban đầu sẽ hiển thị lỗi và cho phép bạn thử lại.
## Hợp đồng SecretRef

Sử dụng một hình dạng đối tượng ở mọi nơi:

```json5
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### `source: "env"`

```json5
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

Validation:

- `provider` must match `^[a-z][a-z0-9_-]{0,63}$`
- `id` must match `^[A-Z][A-Z0-9_]{0,127}$`

### `source: "file"`

```json5
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

Validation:

- `provider` must match `^[a-z][a-z0-9_-]{0,63}$`
- `id` must be an absolute JSON pointer (`/...`)
- RFC6901 escaping in segments: `~` => `~0`, `/` => `~1`

### `source: "exec"`

```json5
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

Validation:

- `provider` must match `^[a-z][a-z0-9_-]{0,63}$`
- `id` must match `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`
## Cấu hình nhà cung cấp

Xác định các nhà cung cấp dưới `secrets.providers`:

```json5
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // or "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### Env provider

- Optional allowlist via `allowlist`.
- Missing/empty env values fail resolution.

### File provider

- Reads local file from `path`.
- `mode: "json"` expects JSON object payload and resolves `id` as pointer.
- `mode: "singleValue"` expects ref id `"value"` and returns file contents.
- Path must pass ownership/permission checks.

### Exec provider

- Runs configured absolute binary path, no shell.
- By default, `command` must point to a regular file (not a symlink).
- Set `allowSymlinkCommand: true` to allow symlink command paths (for example Homebrew shims). OpenClaw validates the resolved target path.
- Enable `allowSymlinkCommand` only when required for trusted package-manager paths, and pair it with `trustedDirs` (for example `["/opt/homebrew"]`).
- When `trustedDirs` is set, checks apply to the resolved target path.
- Supports timeout, no-output timeout, output byte limits, env allowlist, and trusted dirs.
- Request payload (stdin):

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```
- Tải trọng phản hồi (stdout):

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "sk-..." } }
```

Optional per-id errors:

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```
## Ví dụ tích hợp Exec

### 1Password CLI

```json5
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### HashiCorp Vault CLI

```json5
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```
### `sops`

```json5
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // required for Homebrew symlinked binaries
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```
## Các trường trong phạm vi (v1)

### `~/.openclaw/openclaw.json`

- `models.providers.<provider>.apiKey`
- `skills.entries.<skillKey>.apiKey`
- `channels.googlechat.serviceAccount`
- `channels.googlechat.serviceAccountRef`
- `channels.googlechat.accounts.<accountId>.serviceAccount`
- `channels.googlechat.accounts.<accountId>.serviceAccountRef`

### `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

- `profiles.<profileId>.keyRef` cho `type: "api_key"`
- `profiles.<profileId>.tokenRef` cho `type: "token"`

Các thay đổi lưu trữ thông tin xác thực OAuth nằm ngoài phạm vi.
## Hành vi bắt buộc và thứ tự ưu tiên

- Trường không có ref: không thay đổi.
- Trường có ref: bắt buộc tại thời điểm kích hoạt.
- Nếu cả plaintext và ref đều tồn tại, ref sẽ chiếm ưu tiên tại thời chạy và plaintext bị bỏ qua.

Mã cảnh báo:

- `SECRETS_REF_OVERRIDES_PLAINTEXT`
## Kích hoạt bí mật

Kích hoạt bí mật được thử trên:

- Khởi động (kiểm tra sơ bộ cộng với kích hoạt cuối cùng)
- Đường dẫn hot-apply tải lại cấu hình
- Đường dẫn kiểm tra khởi động lại tải lại cấu hình
- Tải lại thủ công qua `secrets.reload`

Hợp đồng kích hoạt:

- Thành công hoán đổi snapshot một cách nguyên tử.
- Lỗi khởi động hủy bỏ khởi động gateway.
- Lỗi tải lại lúc chạy giữ lại snapshot tốt nhất được biết.
## Tín hiệu operator bị suy giảm và phục hồi

Khi kích hoạt lúc tải lại không thành công sau trạng thái lành mạnh, OpenClaw vào trạng thái secrets bị suy giảm.

Mã sự kiện hệ thống một lần và mã nhật ký:

- `SECRETS_RELOADER_DEGRADED`
- `SECRETS_RELOADER_RECOVERED`

Hành vi:

- Suy giảm: runtime giữ lại ảnh chụp cuối cùng được biết là tốt.
- Phục hồi: phát ra một lần sau khi kích hoạt thành công.
- Các lỗi lặp lại trong khi đã bị suy giảm ghi nhật ký cảnh báo nhưng không gửi spam sự kiện.
- Lỗi khởi động fail-fast không phát ra sự kiện suy giảm vì chưa có ảnh chụp runtime nào.
## Kiểm tra và cấu hình quy trình làm việc

Sử dụng luồng toán tử mặc định này:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

Migration completeness:

- Include `skills.entries.<skillKey>.apiKey` targets when those skills use API keys.
- If `audit --check` still reports plaintext findings after a partial migration, migrate the remaining reported paths and rerun audit.

### `secrets audit`

Findings include:

- plaintext values at rest (`openclaw.json`, `auth-profiles.json`, `.env`)
- unresolved refs
- precedence shadowing (`auth-profiles` taking priority over config refs)
- legacy residues (`auth.json`, OAuth out-of-scope reminders)

### `secrets configure`

Interactive helper that:

- configures `secrets.providers` first (`env`/`file`/`exec`, add/edit/remove)
- lets you select secret-bearing fields in `openclaw.json`
- captures SecretRef details (`source`, `provider`, `id`)
- runs preflight resolution
- can apply immediately

Helpful modes:

- `openclaw secrets configure --providers-only`
- `openclaw secrets configure --skip-provider-setup`

`configure` apply defaults to:

- scrub matching static creds from `auth-profiles.json` for targeted providers
- scrub legacy static `api_key` entries from `auth.json`
- scrub matching known secret lines from `<config-dir>/.env`

### `secrets apply`

Apply a saved plan:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

Để biết chi tiết hợp đồng target/path nghiêm ngặt và các quy tắc từ chối chính xác, xem:

- [Hợp đồng Kế hoạch Secrets Apply](/gateway/secrets-plan-contract)
## Chính sách an toàn một chiều

OpenClaw cố ý **không** ghi các bản sao lưu rollback chứa các giá trị bí mật dạng văn bản thuần túy trước khi di chuyển.

Mô hình an toàn:

- preflight phải thành công trước khi ở chế độ ghi
- kích hoạt runtime được xác thực trước khi commit
- áp dụng cập nhật tệp bằng cách thay thế tệp nguyên tử và khôi phục trong bộ nhớ theo nỗ lực tốt nhất khi xảy ra lỗi
## `auth.json` ghi chú tương thích

Đối với thông tin xác thực tĩnh, thời gian chạy OpenClaw không còn phụ thuộc vào `auth.json` dạng văn bản.

- Nguồn thông tin xác thực của Runtime được phân giải từ ảnh chụp nhanh trong bộ nhớ.
- Các mục `auth.json` tĩnh `api_key` kế thừa được loại bỏ khi được phát hiện.
- Hành vi tương thích kế thừa liên quan đến OAuth vẫn được giữ riêng biệt.
## Tài liệu liên quan

- Lệnh CLI: [secrets](/cli/secrets)
- Chi tiết hợp đồng Plan: [Secrets Apply Plan Contract](/gateway/secrets-plan-contract)
- Thiết lập xác thực: [Authentication](/gateway/authentication)
- Tư thế bảo mật: [Security](/gateway/security)
- Ưu tiên môi trường: [Environment Variables](/help/environment)