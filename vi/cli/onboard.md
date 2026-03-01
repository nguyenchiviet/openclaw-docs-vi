---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw onboard` (trình hướng dẫn onboarding
  tương tác)
read_when:
  - >-
    Bạn muốn hướng dẫn thiết lập cho Gateway, workspace, auth, channels, và
    Skills
title: Bắt đầu sử dụng
x-i18n:
  source_path: cli\onboard.md
  source_hash: 5b80b5e671bde4435fa88a0e8eb93f82466d047c6594cc5377034f488c70eee9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:08.591Z'
---

# `openclaw onboard`

Trình hướng dẫn thiết lập ban đầu tương tác (thiết lập Gateway cục bộ hoặc từ xa).
## Hướng dẫn liên quan

- Trung tâm thiết lập ban đầu CLI: [Trình hướng dẫn thiết lập ban đầu (/start/wizard)
- Tổng quan thiết lập ban đầu: [Tổng quan thiết lập ban đầu](/start/onboarding-overview)
- Tài liệu tham khảo thiết lập ban đầu CLI: [Tài liệu tham khảo thiết lập ban đầu CLI](/start/wizard-cli-reference)
- Tự động hóa CLI: [Tự động hóa CLI](/start/wizard-cli-automation)
- Thiết lập ban đầu macOS: [Thiết lập ban đầu (/start/onboarding)
## Ví dụ

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Non-interactive custom provider:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` is optional in non-interactive mode. If omitted, onboarding checks `CUSTOM_API_KEY`.

Store provider keys as refs instead of plaintext:

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

With `--secret-input-mode ref`, onboarding writes env-backed refs instead of plaintext key values.
For auth-profile backed providers this writes `keyRef` entries; for custom providers this writes `models.providers.<id>.apiKey` as an env ref (for example `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`).

Non-interactive `ref` mode contract:

- Set the provider env var in the onboarding process environment (for example `OPENAI_API_KEY`).
- Do not pass inline key flags (for example `--openai-api-key`) unless that env var is also set.
- If an inline key flag is passed without the required env var, onboarding fails fast with guidance.

Interactive onboarding behavior with reference mode:

- Choose **Use secret reference** when prompted.
- Then choose either:
  - Environment variable
  - Configured secret provider (`file` or `exec`)
- Onboarding performs a fast preflight validation before saving the ref.
  - If validation fails, onboarding shows the error and lets you retry.

Non-interactive Z.AI endpoint choices:

Note: `--auth-choice zai-api-key` now auto-detects the best Z.AI endpoint for your key (prefers the general API with `zai/glm-5`).
If you specifically want the GLM Coding Plan endpoints, pick `zai-coding-global` or `zai-coding-cn`.

```bash
# Promptless endpoint selection
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Other Z.AI endpoint choices:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```
Ví dụ Mistral không tương tác:

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

Flow notes:

- `quickstart`: minimal prompts, auto-generates a gateway token.
- `manual`: full prompts for port/bind/auth (alias of `advanced`).
- Local onboarding DM scope behavior: [CLI Onboarding Reference](/start/wizard-cli-reference#outputs-and-internals).
- Fastest first chat: `openclaw dashboard` (Giao diện điều khiển, không cần thiết lập kênh).
- Nhà cung cấp tùy chỉnh: kết nối bất kỳ điểm cuối tương thích OpenAI hoặc Anthropic nào,
  bao gồm các nhà cung cấp được lưu trữ không có trong danh sách. Sử dụng Unknown để tự động phát hiện.
## Các lệnh tiếp theo thường gặp

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` does not imply non-interactive mode. Use `--non-interactive` cho các tập lệnh.
</Note>