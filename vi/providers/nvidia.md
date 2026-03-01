---
summary: Sử dụng API tương thích OpenAI của NVIDIA trong OpenClaw
read_when:
  - Bạn muốn sử dụng các mô hình NVIDIA trong OpenClaw
  - Bạn cần thiết lập NVIDIA_API_KEY
title: NVIDIA
x-i18n:
  source_path: providers\nvidia.md
  source_hash: 81e7a1b6cd6821b68db9c71b864d36023b1ccfad1641bf88e2bc2957782edf8b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:25.315Z'
---

# NVIDIA

NVIDIA cung cấp một API tương thích với OpenAI tại `https://integrate.api.nvidia.com/v1` cho các mô hình Nemotron và NeMo. Xác thực bằng khóa API từ [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Thiết lập CLI

Xuất khóa một lần, sau đó chạy thiết lập ban đầu và đặt một mô hình NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

If you still pass `--token`, remember it lands in shell history and `ps` output; prefer the env var when possible.

## Config snippet

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Model IDs

- `nvidia/llama-3.1-nemotron-70b-instruct` (default)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Notes

- OpenAI-compatible `/v1` endpoint; use an API key from NVIDIA NGC.
- Provider auto-enables when `NVIDIA_API_KEY` được đặt; sử dụng các giá trị mặc định tĩnh (cửa sổ ngữ cảnh 131.072 token, tối đa 4.096 token).