---
summary: Thiết lập Hugging Face Inference (xác thực + lựa chọn mô hình)
read_when:
  - Bạn muốn sử dụng Hugging Face Inference với OpenClaw
  - Bạn cần biến môi trường HF token hoặc lựa chọn xác thực CLI
title: Hugging Face (Inference)
x-i18n:
  source_path: providers\huggingface.md
  source_hash: e7ba5cb533f652ba9bb514ab8c5ffbd50170077ac0005555915405c109198a4f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:51.863Z'
---

# Hugging Face (Inference)

[Nhà cung cấp Hugging Face Inference](https://huggingface.co/docs/inference-providers) cung cấp hoàn thành chat tương thích với OpenAI thông qua một API router duy nhất. Bạn có quyền truy cập vào nhiều mô hình (DeepSeek, Llama, và nhiều hơn nữa) với một token. OpenClaw sử dụng **endpoint tương thích với OpenAI** (chỉ hoàn thành chat); để sử dụng text-to-image, embeddings, hoặc speech, hãy sử dụng [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) trực tiếp.

- Nhà cung cấp: `huggingface`
- Xác thực: `HUGGINGFACE_HUB_TOKEN` hoặc `HF_TOKEN` (token chi tiết với **Make calls to Inference Providers**)
- API: Tương thích với OpenAI (`https://router.huggingface.co/v1`)
- Thanh toán: Token HF duy nhất; [giá cả](https://huggingface.co/docs/inference-providers/pricing) tuân theo tỷ giá của nhà cung cấp với một tầng miễn phí.
## Bắt đầu nhanh

1. Tạo một token chi tiết tại [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) với quyền **Make calls to Inference Providers**.
2. Chạy thiết lập ban đầu và chọn **Hugging Face** trong dropdown nhà cung cấp, sau đó nhập khóa API của bạn khi được yêu cầu:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. In the **Default Hugging Face model** dropdown, pick the model you want (the list is loaded from the Inference API when you have a valid token; otherwise a built-in list is shown). Your choice is saved as the default model.
4. You can also set or change the default model later in config:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```
## Ví dụ không tương tác

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

This will set `huggingface/deepseek-ai/DeepSeek-R1` làm mô hình mặc định.
## Ghi chú về môi trường

Nếu Gateway chạy như một daemon (launchd/systemd), hãy đảm bảo `HUGGINGFACE_HUB_TOKEN` hoặc `HF_TOKEN`
có sẵn cho quá trình đó (ví dụ, trong `~/.openclaw/.env` hoặc thông qua
`env.shellEnv`).
## Khám phá mô hình và dropdown thiết lập ban đầu

OpenClaw khám phá các mô hình bằng cách gọi **Inference endpoint trực tiếp**:

```bash
GET https://router.huggingface.co/v1/models
```

(Optional: send `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` or `$HF_TOKEN` for the full list; some endpoints return a subset without auth.) The response is OpenAI-style `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

When you configure a Hugging Face API key (via onboarding, `HUGGINGFACE_HUB_TOKEN`, or `HF_TOKEN`), OpenClaw uses this GET to discover available chat-completion models. During **interactive onboarding**, after you enter your token you see a **Default Hugging Face model** dropdown populated from that list (or the built-in catalog if the request fails). At runtime (e.g. Gateway startup), when a key is present, OpenClaw again calls **GET** `https://router.huggingface.co/v1/models` để làm mới danh mục. Danh sách được hợp nhất với danh mục tích hợp sẵn (cho siêu dữ liệu như cửa sổ ngữ cảnh và chi phí). Nếu yêu cầu không thành công hoặc không có khóa nào được đặt, chỉ danh mục tích hợp sẵn được sử dụng.
## Tên mô hình và các tùy chọn có thể chỉnh sửa

- **Tên từ API:** Tên hiển thị mô hình được **lấy từ GET /v1/models** khi API trả về `name`, `title`, hoặc `display_name`; nếu không, nó được lấy từ id mô hình (ví dụ: `deepseek-ai/DeepSeek-R1` → "DeepSeek R1").
- **Ghi đè tên hiển thị:** Bạn có thể đặt nhãn tùy chỉnh cho mỗi mô hình trong cấu hình để nó xuất hiện theo cách bạn muốn trong CLI và UI:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider / policy selection:** Append a suffix to the **model id** to choose how the router picks the backend:
  - **`:fastest`** — highest throughput (router picks; provider choice is **locked** — no interactive backend picker).
  - **`:cheapest`** — lowest cost per output token (router picks; provider choice is **locked**).
  - **`:provider`** — force a specific backend (e.g. `:sambanova`, `:together`).

  When you select **:cheapest** or **:fastest** (e.g. in the onboarding model dropdown), the provider is locked: the router decides by cost or speed and no optional “prefer specific backend” step is shown. You can add these as separate entries in `models.providers.huggingface.models` or set `model.primary` with the suffix. You can also set your default order in [Inference Provider settings](https://hf.co/settings/inference-providers) (no suffix = use that order).

- **Config merge:** Existing entries in `models.providers.huggingface.models` (e.g. in `models.json`) are kept when config is merged. So any custom `name`, `alias`, hoặc các tùy chọn mô hình bạn đặt ở đó sẽ được giữ lại.
## ID Mô hình và ví dụ cấu hình

Tham chiếu mô hình sử dụng dạng `huggingface/<org>/<model>` (ID kiểu Hub). Danh sách dưới đây từ **GET** `https://router.huggingface.co/v1/models`; danh mục của bạn có thể bao gồm nhiều hơn.

**ID ví dụ (từ điểm cuối suy luận):**

| Mô hình                | Tham chiếu (tiền tố với `huggingface/`)    |
| ---------------------- | ----------------------------------- |
| DeepSeek R1            | `deepseek-ai/DeepSeek-R1`           |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`         |
| Qwen3 8B               | `Qwen/Qwen3-8B`                     |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`          |
| Qwen3 32B              | `Qwen/Qwen3-32B`                    |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`  |
| GPT-OSS 120B           | `openai/gpt-oss-120b`               |
| GLM 4.7                | `zai-org/GLM-4.7`                   |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`              |

Bạn có thể thêm `:fastest`, `:cheapest`, hoặc `:provider` (ví dụ: `:together`, `:sambanova`) vào ID mô hình. Đặt thứ tự mặc định của bạn trong [Cài đặt Nhà cung cấp suy luận](https://hf.co/settings/inference-providers); xem [Nhà cung cấp suy luận](https://huggingface.co/docs/inference-providers) và **GET** `https://router.huggingface.co/v1/models` để xem danh sách đầy đủ.

### Ví dụ cấu hình hoàn chỉnh

**DeepSeek R1 chính với dự phòng Qwen:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen as default, with :cheapest and :fastest variants:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS với bí danh:**
```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Force a specific backend with :provider:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Multiple Qwen and DeepSeek models with policy suffixes:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```