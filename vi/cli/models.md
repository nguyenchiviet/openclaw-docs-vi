---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw models` (status/list/set/scan, aliases,
  fallbacks, auth)
read_when:
  - >-
    Bạn muốn thay đổi các mô hình mặc định hoặc xem trạng thái xác thực nhà cung
    cấp
  - Bạn muốn quét các mô hình/nhà cung cấp có sẵn và gỡ lỗi các hồ sơ xác thực
title: mô hình
x-i18n:
  source_path: cli\models.md
  source_hash: 923b6ffc7de382ba25bc6e699f0515607e74877b39f2136ccdba2d99e1b1e9c3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:36:39.872Z'
---

# `openclaw models`

Khám phá mô hình, quét và cấu hình (mô hình mặc định, dự phòng, hồ sơ xác thực).

Liên quan:

- Nhà cung cấp + mô hình: [Models](/providers/models)
- Thiết lập xác thực nhà cung cấp: [Bắt đầu](/start/getting-started)
## Các lệnh phổ biến

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` shows the resolved default/fallbacks plus an auth overview.
When provider usage snapshots are available, the OAuth/token status section includes
provider usage headers.
Add `--probe` to run live auth probes against each configured provider profile.
Probes are real requests (may consume tokens and trigger rate limits).
Use `--agent <id>` to inspect a configured agent’s model/auth state. When omitted,
the command uses `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` if set, otherwise the
configured default agent.

Notes:

- `models set <model-or-alias>` accepts `provider/model` or an alias.
- Model refs are parsed by splitting on the **first** `/`. If the model ID includes `/` (OpenRouter-style), include the provider prefix (example: `openrouter/moonshotai/kimi-k2`).
- If you omit the provider, OpenClaw treats the input as an alias or a model for the **default provider** (only works when there is no `/` in the model ID).

### `models status`

Options:

- `--json`
- `--plain`
- `--check` (exit 1=expired/missing, 2=expiring)
- `--probe` (live probe of configured auth profiles)
- `--probe-provider <name>` (probe one provider)
- `--probe-profile <id>` (repeat or comma-separated profile ids)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`
- `--agent <id>` (configured agent id; overrides `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)
## Bí danh + dự phòng

```bash
openclaw models aliases list
openclaw models fallbacks list
```
## Hồ sơ xác thực

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` runs a provider plugin’s auth flow (OAuth/API key). Use
`openclaw plugins list` to see which providers are installed.

Notes:

- `setup-token` prompts for a setup-token value (generate it with `claude setup-token` on any machine).
- `paste-token` chấp nhận một chuỗi token được tạo ở nơi khác hoặc từ tự động hóa.