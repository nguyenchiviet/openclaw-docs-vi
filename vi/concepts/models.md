---
summary: 'Models CLI: list, set, aliases, fallbacks, scan, status'
read_when:
  - Thêm hoặc sửa đổi các mô hình CLI (models list/set/scan/aliases/fallbacks)
  - Thay đổi hành vi dự phòng mô hình hoặc UX lựa chọn mô hình
  - Cập nhật các probe quét mô hình (tools/images)
title: Models CLI
x-i18n:
  source_path: concepts\models.md
  source_hash: 70c32631615f90f0464cad380aea53d275ae6f1c747b686b25ac553377a78936
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:07.095Z'
---

# CLI Mô hình

Xem [/concepts/model-failover](/concepts/model-failover) để tìm hiểu về xoay vòng hồ sơ xác thực, thời gian chờ và cách nó tương tác với các giải pháp dự phòng.
Tổng quan nhanh về nhà cung cấp + ví dụ: [/concepts/model-providers](/concepts/model-providers).
## Cách lựa chọn mô hình hoạt động

OpenClaw lựa chọn mô hình theo thứ tự này:

1. **Mô hình chính** (`agents.defaults.model.primary` hoặc `agents.defaults.model`).
2. **Dự phòng** trong `agents.defaults.model.fallbacks` (theo thứ tự).
3. **Failover xác thực nhà cung cấp** xảy ra bên trong một nhà cung cấp trước khi chuyển sang
   mô hình tiếp theo.

Liên quan:

- `agents.defaults.models` là danh sách cho phép/danh mục các mô hình OpenClaw có thể sử dụng (cộng với bí danh).
- `agents.defaults.imageModel` được sử dụng **chỉ khi** mô hình chính không thể chấp nhận hình ảnh.
- Các giá trị mặc định cho mỗi agent có thể ghi đè `agents.defaults.model` thông qua `agents.list[].model` cộng với các ràng buộc (xem [/concepts/multi-agent](/concepts/multi-agent)).
## Lựa chọn mô hình nhanh (dựa trên kinh nghiệm)

- **GLM**: tốt hơn một chút cho lập trình/gọi công cụ.
- **MiniMax**: tốt hơn cho viết lách và cảm giác.
## Trình hướng dẫn thiết lập (được khuyến nghị)

Nếu bạn không muốn chỉnh sửa cấu hình thủ công, hãy chạy trình hướng dẫn thiết lập ban đầu:

```bash
openclaw onboard
```

It can set up model + auth for common providers, including **OpenAI Code (Codex)
subscription** (OAuth) and **Anthropic** (API key recommended; `claude
setup-token` cũng được hỗ trợ).
## Các khóa cấu hình (tổng quan)

- `agents.defaults.model.primary` và `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel.primary` và `agents.defaults.imageModel.fallbacks`
- `agents.defaults.models` (danh sách cho phép + bí danh + tham số nhà cung cấp)
- `models.providers` (các nhà cung cấp tùy chỉnh được viết vào `models.json`)

Các tham chiếu mô hình được chuẩn hóa thành chữ thường. Các bí danh nhà cung cấp như `z.ai/*` chuẩn hóa
thành `zai/*`.

Các ví dụ cấu hình nhà cung cấp (bao gồm OpenCode Zen) nằm trong
[/gateway/configuration](/gateway/configuration#opencode-zen-multi-model-proxy).
## "Mô hình không được phép" (và lý do tại sao các câu trả lời dừng lại)

Nếu `agents.defaults.models` được đặt, nó sẽ trở thành **danh sách cho phép** cho `/model` và các ghi đè phiên. Khi người dùng chọn một mô hình không có trong danh sách cho phép đó, OpenClaw trả về:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

This happens **before** a normal reply is generated, so the message can feel
like it “didn’t respond.” The fix is to either:

- Add the model to `agents.defaults.models`, or
- Clear the allowlist (remove `agents.defaults.models`), or
- Pick a model from `/model list`.

Example allowlist config:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```
## Chuyển đổi mô hình trong trò chuyện (`/model`)

Bạn có thể chuyển đổi mô hình cho phiên hiện tại mà không cần khởi động lại:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Notes:

- `/model` (and `/model list`) is a compact, numbered picker (model family + available providers).
- On Discord, `/model` and `/models` open an interactive picker with provider and model dropdowns plus a Submit step.
- `/model <#>` selects from that picker.
- `/model status` is the detailed view (auth candidates and, when configured, provider endpoint `baseUrl` + `api` mode).
- Model refs are parsed by splitting on the **first** `/`. Use `provider/model` when typing `/model <ref>`.
- If the model ID itself contains `/` (OpenRouter-style), you must include the provider prefix (example: `/model openrouter/moonshotai/kimi-k2`).
- If you omit the provider, OpenClaw treats the input as an alias or a model for the **default provider** (only works when there is no `/` trong ID mô hình).

Hành vi/cấu hình lệnh đầy đủ: [Lệnh gạch chéo](/tools/slash-commands).
## Lệnh CLI

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (no subcommand) is a shortcut for `models status`.

### `models list`

Shows configured models by default. Useful flags:

- `--all`: full catalog
- `--local`: local providers only
- `--provider <name>`: filter by provider
- `--plain`: one model per line
- `--json`: machine‑readable output

### `models status`

Shows the resolved primary model, fallbacks, image model, and an auth overview
of configured providers. It also surfaces OAuth expiry status for profiles found
in the auth store (warns within 24h by default). `--plain` prints only the
resolved primary model.
OAuth status is always shown (and included in `--json` output). If a configured
provider has no credentials, `models status` prints a **Missing auth** section.
JSON includes `auth.oauth` (warn window + profiles) and `auth.providers`
(effective auth per provider).
Use `--check` for automation (exit `1` when missing/expired, `2` when expiring).

Preferred Anthropic auth is the Claude Code CLI setup-token (run anywhere; paste on the gateway host if needed):

```bash
claude setup-token
openclaw models status
```
## Quét (Các mô hình miễn phí của OpenRouter)

`openclaw models scan` kiểm tra **danh mục mô hình miễn phí** của OpenRouter và có thể
tùy chọn kiểm tra các mô hình để hỗ trợ công cụ và hình ảnh.

Các cờ chính:

- `--no-probe`: bỏ qua các kiểm tra trực tiếp (chỉ siêu dữ liệu)
- `--min-params <b>`: kích thước tham số tối thiểu (tỷ)
- `--max-age-days <days>`: bỏ qua các mô hình cũ hơn
- `--provider <name>`: bộ lọc tiền tố nhà cung cấp
- `--max-candidates <n>`: kích thước danh sách dự phòng
- `--set-default`: đặt `agents.defaults.model.primary` thành lựa chọn đầu tiên
- `--set-image`: đặt `agents.defaults.imageModel.primary` thành lựa chọn hình ảnh đầu tiên

Kiểm tra yêu cầu khóa API OpenRouter (từ hồ sơ xác thực hoặc
`OPENROUTER_API_KEY`). Nếu không có khóa, hãy sử dụng `--no-probe` để chỉ liệt kê các ứng cử viên.

Kết quả quét được xếp hạng theo:

1. Hỗ trợ hình ảnh
2. Độ trễ công cụ
3. Kích thước ngữ cảnh
4. Số lượng tham số

Đầu vào

- Danh sách `/models` của OpenRouter (bộ lọc `:free`)
- Yêu cầu khóa API OpenRouter từ hồ sơ xác thực hoặc `OPENROUTER_API_KEY` (xem [/environment](/help/environment))
- Bộ lọc tùy chọn: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
- Kiểm soát kiểm tra: `--timeout`, `--concurrency`

Khi chạy trong TTY, bạn có thể chọn dự phòng một cách tương tác. Ở chế độ không tương tác,
hãy chuyển `--yes` để chấp nhận các giá trị mặc định.
## Bản ghi mô hình (`models.json`)

Các nhà cung cấp tùy chỉnh trong `models.providers` được ghi vào `models.json` dưới thư mục
agent (mặc định `~/.openclaw/agents/<agentId>/models.json`). Tệp này
được hợp nhất theo mặc định trừ khi `models.mode` được đặt thành `replace`.

Thứ tự ưu tiên chế độ hợp nhất cho các ID nhà cung cấp phù hợp:

- `apiKey`/`baseUrl` không trống đã có trong agent `models.json` sẽ thắng.
- Agent `apiKey`/`baseUrl` trống hoặc bị thiếu sẽ quay lại cấu hình `models.providers`.
- Các trường nhà cung cấp khác được làm mới từ dữ liệu danh mục cấu hình và chuẩn hóa.