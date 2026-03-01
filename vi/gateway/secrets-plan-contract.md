---
summary: >-
  Hợp đồng cho các kế hoạch `secrets apply`: các đường dẫn đích được phép, xác
  thực và hành vi auth-profile chỉ tham chiếu
read_when:
  - Tạo hoặc xem xét các tệp kế hoạch `openclaw secrets apply`
  - Gỡ lỗi lỗi `Invalid plan target path`
  - >-
    Hiểu cách `keyRef` và `tokenRef` ảnh hưởng đến khám phá nhà cung cấp ngầm
    định
title: Hợp đồng Áp dụng Kế hoạch Bí mật
x-i18n:
  source_path: gateway\secrets-plan-contract.md
  source_hash: 87ba69b3780abeaba34dba522d33c8a767e9866f84fb2eb80a15ecaa24006e2b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:38.487Z'
---

# Hợp đồng kế hoạch áp dụng bí mật

Trang này định nghĩa hợp đồng nghiêm ngặt được thực thi bởi `openclaw secrets apply`.

Nếu mục tiêu không phù hợp với các quy tắc này, áp dụng sẽ thất bại trước khi thay đổi cấu hình.
## Hình dạng tệp kế hoạch

`openclaw secrets apply --from <plan.json>` mong đợi một mảng `targets` các mục tiêu kế hoạch:

```json5
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```
## Các loại đích và đường dẫn được phép

| `target.type`                        | Hình dạng `target.path` được phép                               | Quy tắc khớp id tùy chọn                              |
| ------------------------------------ | --------------------------------------------------------- | --------------------------------------------------- |
| `models.providers.apiKey`            | `models.providers.<providerId>.apiKey`                    | `providerId` phải khớp với `<providerId>` khi có |
| `skills.entries.apiKey`              | `skills.entries.<skillKey>.apiKey`                        | n/a                                                 |
| `channels.googlechat.serviceAccount` | `channels.googlechat.serviceAccount`                      | `accountId` phải để trống/bỏ qua                   |
| `channels.googlechat.serviceAccount` | `channels.googlechat.accounts.<accountId>.serviceAccount` | `accountId` phải khớp với `<accountId>` khi có   |
## Quy tắc xác thực đường dẫn

Mỗi mục tiêu được xác thực với tất cả những điều sau:

- `type` phải là một trong các loại mục tiêu được phép ở trên.
- `path` phải là một đường dẫn dấu chấm không trống.
- `pathSegments` có thể được bỏ qua. Nếu được cung cấp, nó phải chuẩn hóa thành chính xác cùng một đường dẫn như `path`.
- Các phân đoạn bị cấm bị từ chối: `__proto__`, `prototype`, `constructor`.
- Đường dẫn chuẩn hóa phải khớp với một trong các hình dạng đường dẫn được phép cho loại mục tiêu.
- Nếu `providerId` / `accountId` được đặt, nó phải khớp với id được mã hóa trong đường dẫn.
## Hành vi khi thất bại

Nếu một mục tiêu không vượt qua xác thực, apply sẽ thoát với lỗi như:

```text
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

Không có phần thay đổi nào được cam kết cho đường dẫn mục tiêu không hợp lệ đó.
## Hồ sơ xác thực chỉ tham chiếu và khám phá nhà cung cấp ngầm

Khám phá nhà cung cấp ngầm cũng xem xét các hồ sơ xác thực lưu trữ tham chiếu thay vì thông tin xác thực văn bản thuần:

- `type: "api_key"` hồ sơ có thể sử dụng `keyRef` (ví dụ: tham chiếu được hỗ trợ bởi env).
- `type: "token"` hồ sơ có thể sử dụng `tokenRef`.

Hành vi:

- Đối với các nhà cung cấp khóa API (ví dụ: `volcengine`, `byteplus`), hồ sơ chỉ tham chiếu vẫn có thể kích hoạt các mục nhà cung cấp ngầm.
- Đối với `github-copilot`, nếu hồ sơ không có token văn bản thuần, khám phá sẽ cố gắng phân giải biến môi trường `tokenRef` trước khi trao đổi token.
## Kiểm tra Operator

```bash
# Validate plan without writes
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# Then apply for real
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

If apply fails with an invalid target path message, regenerate the plan with `openclaw secrets configure` hoặc sửa đường dẫn đích thành một trong các hình dạng được phép ở trên.
## Tài liệu liên quan

- [Quản lý Bí mật](/gateway/secrets)
- [CLI `secrets`](/cli/secrets)
- [Tham chiếu Cấu hình](/gateway/configuration-reference)