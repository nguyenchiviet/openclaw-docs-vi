---
summary: 'Xác thực mô hình: OAuth, khóa API và token thiết lập'
read_when:
  - Gỡ lỗi xác thực mô hình hoặc hết hạn OAuth
  - Ghi chép xác thực hoặc lưu trữ thông tin xác thực
title: Xác thực
x-i18n:
  source_path: gateway\authentication.md
  source_hash: 55b4e66fe1c212e7f74bbe89171b949a821a87f581a25b905847b52c7df5d6e0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:55:20.170Z'
---

# Xác thực

OpenClaw hỗ trợ OAuth và khóa API cho các nhà cung cấp mô hình. Đối với tài khoản Anthropic, chúng tôi khuyên bạn nên sử dụng **khóa API**. Để truy cập đăng ký Claude, hãy sử dụng token dài hạn được tạo bởi `claude setup-token`.

Xem [/concepts/oauth](/concepts/oauth) để biết toàn bộ luồng OAuth và bố cục lưu trữ.
Đối với xác thực dựa trên SecretRef (`env`/`file`/`exec` nhà cung cấp), xem [Quản lý Bí mật](/gateway/secrets).
## Thiết lập Anthropic được khuyến nghị (API key)

Nếu bạn sử dụng Anthropic trực tiếp, hãy sử dụng API key.

1. Tạo API key trong Anthropic Console.
2. Đặt nó trên **gateway host** (máy chạy `openclaw gateway`).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. If the Gateway runs under systemd/launchd, prefer putting the key in
   `~/.openclaw/.env` so the daemon can read it:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Then restart the daemon (or restart your Gateway process) and re-check:

```bash
openclaw models status
openclaw doctor
```

If you’d rather not manage env vars yourself, the onboarding wizard can store
API keys for daemon use: `openclaw onboard`.

See [Help](/help) for details on env inheritance (`env.shellEnv`,
`~/.openclaw/.env`, systemd/launchd).
## Anthropic: setup-token (xác thực theo subscription)

Đối với Anthropic, đường dẫn được khuyến nghị là **API key**. Nếu bạn đang sử dụng subscription Claude, luồng setup-token cũng được hỗ trợ. Chạy nó trên **host gateway**:

```bash
claude setup-token
```

Then paste it into OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

If the token was created on another machine, paste it manually:

```bash
openclaw models auth paste-token --provider anthropic
```

If you see an Anthropic error like:

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…use an Anthropic API key instead.

Manual token entry (any provider; writes `auth-profiles.json` + updates config):

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Auth profile refs are also supported for static credentials:

- `api_key` credentials can use `keyRef: { source, provider, id }`
- `token` credentials can use `tokenRef: { source, provider, id }`

Automation-friendly check (exit `1` when expired/missing, `2` when expiring):

```bash
openclaw models status --check
```

Optional ops scripts (systemd/Termux) are documented here:
[/automation/auth-monitoring](/automation/auth-monitoring)

> `claude setup-token` yêu cầu một TTY tương tác.
## Kiểm tra trạng thái xác thực mô hình

```bash
openclaw models status
openclaw doctor
```
## Hành vi xoay vòng khóa API (Gateway)

Một số nhà cung cấp hỗ trợ thử lại yêu cầu với các khóa thay thế khi lệnh gọi API
đạt giới hạn tốc độ của nhà cung cấp.

- Thứ tự ưu tiên:
  - `OPENCLAW_LIVE_<PROVIDER>_KEY` (ghi đè đơn lẻ)
  - `<PROVIDER>_API_KEYS`
  - `<PROVIDER>_API_KEY`
  - `<PROVIDER>_API_KEY_*`
- Các nhà cung cấp Google cũng bao gồm `GOOGLE_API_KEY` như một lựa chọn dự phòng bổ sung.
- Danh sách khóa giống nhau được loại bỏ trùng lặp trước khi sử dụng.
- OpenClaw thử lại với khóa tiếp theo chỉ cho các lỗi giới hạn tốc độ (ví dụ
  `429`, `rate_limit`, `quota`, `resource exhausted`).
- Các lỗi không phải giới hạn tốc độ không được thử lại với các khóa thay thế.
- Nếu tất cả các khóa đều thất bại, lỗi cuối cùng từ lần thử cuối cùng sẽ được trả về.
## Kiểm soát thông tin xác thực nào được sử dụng

### Mỗi phiên (lệnh chat)

Sử dụng `/model <alias-or-id>@<profileId>` để ghim thông tin xác thực nhà cung cấp cụ thể cho phiên hiện tại (ví dụ về ID hồ sơ: `anthropic:default`, `anthropic:work`).

Sử dụng `/model` (hoặc `/model list`) để có bộ chọn nhỏ gọn; sử dụng `/model status` để xem đầy đủ (ứng cử viên + hồ sơ xác thực tiếp theo, cộng với chi tiết điểm cuối nhà cung cấp khi được cấu hình).

### Mỗi agent (ghi đè CLI)

Đặt ghi đè thứ tự hồ sơ xác thực rõ ràng cho một agent (được lưu trữ trong `auth-profiles.json` của agent đó):

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Use `--agent <id>` để nhắm mục tiêu một agent cụ thể; bỏ qua nó để sử dụng agent mặc định được cấu hình.
## Khắc phục sự cố

### "Không tìm thấy thông tin xác thực"

Nếu hồ sơ token Anthropic bị thiếu, hãy chạy `claude setup-token` trên
**máy chủ gateway**, sau đó kiểm tra lại:

```bash
openclaw models status
```

### Token expiring/expired

Run `openclaw models status` to confirm which profile is expiring. If the profile
is missing, rerun `claude setup-token` và dán token lại.
## Yêu cầu

- Đăng ký Claude Max hoặc Pro (cho `claude setup-token`)
- Claude Code CLI được cài đặt (lệnh `claude` có sẵn)