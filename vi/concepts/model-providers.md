---
summary: Tổng quan về nhà cung cấp mô hình với các cấu hình ví dụ + luồng CLI
read_when:
  - Bạn cần một tài liệu tham khảo thiết lập mô hình theo từng nhà cung cấp
  - >-
    Bạn muốn các cấu hình ví dụ hoặc lệnh CLI onboarding cho các nhà cung cấp mô
    hình
title: Nhà cung cấp Mô hình
x-i18n:
  source_path: concepts\model-providers.md
  source_hash: 98d4397737dde53a5e6e06e80779d6b82032f1c318cb5bebd93257283bfa08f5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:39:43.994Z'
---

# Nhà cung cấp mô hình

Trang này bao gồm **các nhà cung cấp LLM/mô hình** (không phải các kênh trò chuyện như WhatsApp/Telegram).
Để biết các quy tắc lựa chọn mô hình, xem [/concepts/models](/concepts/models).
## Quy tắc nhanh

- Tham chiếu mô hình sử dụng `provider/model` (ví dụ: `opencode/claude-opus-4-6`).
- Nếu bạn đặt `agents.defaults.models`, nó sẽ trở thành danh sách cho phép.
- Trợ giúp CLI: `openclaw onboard`, `openclaw models list`, `openclaw models set <provider/model>`.
## Xoay vòng khóa API

- Hỗ trợ xoay vòng nhà cung cấp chung cho các nhà cung cấp được chọn.
- Cấu hình nhiều khóa thông qua:
  - `OPENCLAW_LIVE_<PROVIDER>_KEY` (ghi đè trực tiếp duy nhất, ưu tiên cao nhất)
  - `<PROVIDER>_API_KEYS` (danh sách được phân tách bằng dấu phẩy hoặc dấu chấm phẩy)
  - `<PROVIDER>_API_KEY` (khóa chính)
  - `<PROVIDER>_API_KEY_*` (danh sách được đánh số, ví dụ `<PROVIDER>_API_KEY_1`)
- Đối với các nhà cung cấp Google, `GOOGLE_API_KEY` cũng được bao gồm làm dự phòng.
- Thứ tự lựa chọn khóa bảo toàn ưu tiên và loại bỏ các giá trị trùng lặp.
- Các yêu cầu được thử lại với khóa tiếp theo chỉ khi nhận được phản hồi giới hạn tốc độ (ví dụ `429`, `rate_limit`, `quota`, `resource exhausted`).
- Các lỗi không phải giới hạn tốc độ sẽ thất bại ngay lập tức; không có xoay vòng khóa nào được thử.
- Khi tất cả các khóa ứng cử thất bại, lỗi cuối cùng được trả về từ lần thử cuối cùng.
## Các nhà cung cấp tích hợp sẵn (danh mục pi-ai)

OpenClaw được cung cấp kèm danh mục pi‑ai. Các nhà cung cấp này **không** yêu cầu
`models.providers` config; chỉ cần đặt xác thực + chọn một mô hình.

### OpenAI

- Nhà cung cấp: `openai`
- Xác thực: `OPENAI_API_KEY`
- Xoay vòng tùy chọn: `OPENAI_API_KEYS`, `OPENAI_API_KEY_1`, `OPENAI_API_KEY_2`, cộng với `OPENCLAW_LIVE_OPENAI_KEY` (ghi đè duy nhất)
- Mô hình ví dụ: `openai/gpt-5.1-codex`
- CLI: `openclaw onboard --auth-choice openai-api-key`

```json5
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

- Provider: `anthropic`
- Auth: `ANTHROPIC_API_KEY` or `claude setup-token`
- Optional rotation: `ANTHROPIC_API_KEYS`, `ANTHROPIC_API_KEY_1`, `ANTHROPIC_API_KEY_2`, plus `OPENCLAW_LIVE_ANTHROPIC_KEY` (single override)
- Example model: `anthropic/claude-opus-4-6`
- CLI: `openclaw onboard --auth-choice token` (paste setup-token) or `openclaw models auth paste-token --provider anthropic`

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

- Provider: `openai-codex`
- Auth: OAuth (ChatGPT)
- Example model: `openai-codex/gpt-5.3-codex`
- CLI: `openclaw onboard --auth-choice openai-codex` or `openclaw models auth login --provider openai-codex`
- Default transport is `auto` (WebSocket-first, SSE fallback)
- Override per model via `agents.defaults.models["openai-codex/<model>"].params.transport` (`"sse"`, `"websocket"`, or `"auto"`)

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

- Provider: `opencode`
- Auth: `OPENCODE_API_KEY` (or `OPENCODE_ZEN_API_KEY`)
- Example model: `opencode/claude-opus-4-6`
- CLI: `openclaw onboard --auth-choice opencode-zen`

```json5
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```
### Google Gemini (API key)

- Nhà cung cấp: `google`
- Xác thực: `GEMINI_API_KEY`
- Xoay vòng tùy chọn: `GEMINI_API_KEYS`, `GEMINI_API_KEY_1`, `GEMINI_API_KEY_2`, `GOOGLE_API_KEY` fallback, và `OPENCLAW_LIVE_GEMINI_KEY` (ghi đè duy nhất)
- Mô hình ví dụ: `google/gemini-3-pro-preview`
- CLI: `openclaw onboard --auth-choice gemini-api-key`

### Google Vertex, Antigravity, và Gemini CLI

- Nhà cung cấp: `google-vertex`, `google-antigravity`, `google-gemini-cli`
- Xác thực: Vertex sử dụng gcloud ADC; Antigravity/Gemini CLI sử dụng các luồng xác thực tương ứng của chúng
- Cảnh báo: Antigravity và Gemini CLI OAuth trong OpenClaw là các tích hợp không chính thức. Một số người dùng đã báo cáo các hạn chế tài khoản Google sau khi sử dụng các ứng dụng khách của bên thứ ba. Hãy xem lại các điều khoản của Google và sử dụng một tài khoản không quan trọng nếu bạn chọn tiếp tục.
- Antigravity OAuth được cung cấp dưới dạng plugin đi kèm (`google-antigravity-auth`, bị vô hiệu hóa theo mặc định).
  - Bật: `openclaw plugins enable google-antigravity-auth`
  - Đăng nhập: `openclaw models auth login --provider google-antigravity --set-default`
- Gemini CLI OAuth được cung cấp dưới dạng plugin đi kèm (`google-gemini-cli-auth`, bị vô hiệu hóa theo mặc định).
  - Bật: `openclaw plugins enable google-gemini-cli-auth`
  - Đăng nhập: `openclaw models auth login --provider google-gemini-cli --set-default`
  - Lưu ý: bạn **không** dán ID ứng dụng khách hoặc bí mật vào `openclaw.json`. Luồng đăng nhập CLI lưu trữ
    các token trong các hồ sơ xác thực trên máy chủ gateway.

### Z.AI (GLM)

- Nhà cung cấp: `zai`
- Xác thực: `ZAI_API_KEY`
- Mô hình ví dụ: `zai/glm-4.7`
- CLI: `openclaw onboard --auth-choice zai-api-key`
  - Bí danh: `z.ai/*` và `z-ai/*` chuẩn hóa thành `zai/*`

### Vercel AI Gateway

- Nhà cung cấp: `vercel-ai-gateway`
- Xác thực: `AI_GATEWAY_API_KEY`
- Mô hình ví dụ: `vercel-ai-gateway/anthropic/claude-opus-4.6`
- CLI: `openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

- Nhà cung cấp: `kilocode`
- Xác thực: `KILOCODE_API_KEY`
- Mô hình ví dụ: `kilocode/anthropic/claude-opus-4.6`
- CLI: `openclaw onboard --kilocode-api-key <key>`
- URL cơ sở: `https://api.kilo.ai/api/gateway/`
- Danh mục được xây dựng sẵn mở rộng bao gồm GLM-5 Free, MiniMax M2.5 Free, GPT-5.2, Gemini 3 Pro Preview, Gemini 3 Flash Preview, Grok Code Fast 1, và Kimi K2.5.

Xem [/providers/kilocode](/providers/kilocode) để biết chi tiết thiết lập.

### Các nhà cung cấp được xây dựng sẵn khác

- OpenRouter: `openrouter` (`OPENROUTER_API_KEY`)
- Mô hình ví dụ: `openrouter/anthropic/claude-sonnet-4-5`
- Kilo Gateway: `kilocode` (`KILOCODE_API_KEY`)
- Mô hình ví dụ: `kilocode/anthropic/claude-opus-4.6`
- xAI: `xai` (`XAI_API_KEY`)
- Mistral: `mistral` (`MISTRAL_API_KEY`)
- Mô hình ví dụ: `mistral/mistral-large-latest`
- CLI: `openclaw onboard --auth-choice mistral-api-key`
- Groq: `groq` (`GROQ_API_KEY`)
- Cerebras: `cerebras` (`CEREBRAS_API_KEY`)
  - Các mô hình GLM trên Cerebras sử dụng các id `zai-glm-4.7` và `zai-glm-4.6`.
  - URL cơ sở tương thích với OpenAI: `https://api.cerebras.ai/v1`.
- GitHub Copilot: `github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
- Hugging Face Inference: `huggingface` (`HUGGINGFACE_HUB_TOKEN` hoặc `HF_TOKEN`) — bộ định tuyến tương thích với OpenAI; mô hình ví dụ: `huggingface/deepseek-ai/DeepSeek-R1`; CLI: `openclaw onboard --auth-choice huggingface-api-key`. Xem [Hugging Face (/providers/huggingface).
## Nhà cung cấp qua `models.providers` (URL tùy chỉnh/cơ sở)

Sử dụng `models.providers` (hoặc `models.json`) để thêm nhà cung cấp **tùy chỉnh** hoặc các proxy tương thích với OpenAI/Anthropic.

### Moonshot AI (Kimi)

Moonshot sử dụng các endpoint tương thích với OpenAI, vì vậy hãy cấu hình nó như một nhà cung cấp tùy chỉnh:

- Nhà cung cấp: `moonshot`
- Xác thực: `MOONSHOT_API_KEY`
- Mô hình ví dụ: `moonshot/kimi-k2.5`

ID mô hình Kimi K2:

{/_moonshot-kimi-k2-model-refs:start_/ && null}

- `moonshot/kimi-k2.5`
- `moonshot/kimi-k2-0905-preview`
- `moonshot/kimi-k2-turbo-preview`
- `moonshot/kimi-k2-thinking`
- `moonshot/kimi-k2-thinking-turbo`
  {/_moonshot-kimi-k2-model-refs:end_/ && null}

```json5
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }],
      },
    },
  },
}
```

### Kimi Coding

Kimi Coding uses Moonshot AI's Anthropic-compatible endpoint:

- Provider: `kimi-coding`
- Auth: `KIMI_API_KEY`
- Example model: `kimi-coding/k2p5`

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```
### Qwen OAuth (tầng miễn phí)

Qwen cung cấp quyền truy cập OAuth vào Qwen Coder + Vision thông qua luồng mã thiết bị.
Bật plugin đi kèm, sau đó đăng nhập:

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

Model refs:

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

See [/providers/qwen](/providers/qwen) for setup details and notes.

### Volcano Engine (Doubao)

Volcano Engine (火山引擎) provides access to Doubao and other models in China.

- Provider: `volcengine` (coding: `volcengine-plan`)
- Auth: `VOLCANO_ENGINE_API_KEY`
- Example model: `volcengine/doubao-seed-1-8-251228`
- CLI: `openclaw onboard --auth-choice volcengine-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

Available models:

- `volcengine/doubao-seed-1-8-251228` (Doubao Seed 1.8)
- `volcengine/doubao-seed-code-preview-251028`
- `volcengine/kimi-k2-5-260127` (Kimi K2.5)
- `volcengine/glm-4-7-251222` (GLM 4.7)
- `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

Coding models (`volcengine-plan`):

- `volcengine-plan/ark-code-latest`
- `volcengine-plan/doubao-seed-code`
- `volcengine-plan/kimi-k2.5`
- `volcengine-plan/kimi-k2-thinking`
- `volcengine-plan/glm-4.7`

### BytePlus (International)

BytePlus ARK provides access to the same models as Volcano Engine for international users.

- Provider: `byteplus` (coding: `byteplus-plan`)
- Auth: `BYTEPLUS_API_KEY`
- Example model: `byteplus/seed-1-8-251228`
- CLI: `openclaw onboard --auth-choice byteplus-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```
Các mô hình có sẵn:

- `byteplus/seed-1-8-251228` (Seed 1.8)
- `byteplus/kimi-k2-5-260127` (Kimi K2.5)
- `byteplus/glm-4-7-251222` (GLM 4.7)

Các mô hình lập trình (`byteplus-plan`):

- `byteplus-plan/ark-code-latest`
- `byteplus-plan/doubao-seed-code`
- `byteplus-plan/kimi-k2.5`
- `byteplus-plan/kimi-k2-thinking`
- `byteplus-plan/glm-4.7`

### Synthetic

Synthetic cung cấp các mô hình tương thích với Anthropic phía sau nhà cung cấp `synthetic`:

- Nhà cung cấp: `synthetic`
- Xác thực: `SYNTHETIC_API_KEY`
- Mô hình ví dụ: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`
- CLI: `openclaw onboard --auth-choice synthetic-api-key`

```json5
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" } },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.1", name: "MiniMax M2.1" }],
      },
    },
  },
}
```

### MiniMax

MiniMax is configured via `models.providers` because it uses custom endpoints:

- MiniMax (Anthropic‑compatible): `--auth-choice minimax-api`
- Auth: `MINIMAX_API_KEY`

See [/providers/minimax](/providers/minimax) for setup details, model options, and config snippets.

### Ollama

Ollama is a local LLM runtime that provides an OpenAI-compatible API:

- Provider: `ollama`
- Auth: None required (local server)
- Example model: `ollama/llama3.3`
- Cài đặt: [https://ollama.ai](https://ollama.ai)
```bash
# Install Ollama, then pull a model:
ollama pull llama3.3
```

```json5
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

Ollama is automatically detected when running locally at `http://127.0.0.1:11434/v1`. See [/providers/ollama](/providers/ollama) for model recommendations and custom configuration.

### vLLM

vLLM is a local (or self-hosted) OpenAI-compatible server:

- Provider: `vllm`
- Auth: Optional (depends on your server)
- Default base URL: `http://127.0.0.1:8000/v1`

To opt in to auto-discovery locally (any value works if your server doesn’t enforce auth):

```bash
export VLLM_API_KEY="vllm-local"
```

Then set a model (replace with one of the IDs returned by `/v1/models`):

```json5
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

See [/providers/vllm](/providers/vllm) for details.

### Local proxies (LM Studio, vLLM, LiteLLM, etc.)

Example (OpenAI‑compatible):

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
Ghi chú:

- Đối với các nhà cung cấp tùy chỉnh, `reasoning`, `input`, `cost`, `contextWindow`, và `maxTokens` là tùy chọn.
  Khi bị bỏ qua, OpenClaw sẽ mặc định sử dụng:
  - `reasoning: false`
  - `input: ["text"]`
  - `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
  - `contextWindow: 200000`
  - `maxTokens: 8192`
- Khuyến nghị: đặt các giá trị rõ ràng phù hợp với giới hạn proxy/mô hình của bạn.
## Ví dụ CLI

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

Xem thêm: [/gateway/configuration](/gateway/configuration) để xem các ví dụ cấu hình đầy đủ.