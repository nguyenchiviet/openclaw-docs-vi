---
summary: Chạy OpenClaw với Ollama (runtime LLM cục bộ)
read_when:
  - Bạn muốn chạy OpenClaw với các mô hình cục bộ thông qua Ollama
  - Bạn cần hướng dẫn thiết lập và cấu hình Ollama
title: Ollama
x-i18n:
  source_path: providers\ollama.md
  source_hash: 40a3887921051d5a202ce47375ab2c1ca8f86ea5f87f2db1ed22f70c4aa7a19b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:38.655Z'
---

# Ollama

Ollama là một runtime LLM cục bộ giúp bạn dễ dàng chạy các mô hình mã nguồn mở trên máy của mình. OpenClaw tích hợp với API gốc của Ollama (`/api/chat`), hỗ trợ truyền phát và gọi công cụ, và có thể **tự động khám phá các mô hình có khả năng công cụ** khi bạn chọn tham gia với `OLLAMA_API_KEY` (hoặc một hồ sơ xác thực) và không xác định một mục `models.providers.ollama` rõ ràng.
## Bắt đầu nhanh

1. Cài đặt Ollama: [https://ollama.ai](https://ollama.ai)

2. Tải xuống một mô hình:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. Enable Ollama for OpenClaw (any value works; Ollama doesn't require a real key):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Use Ollama models:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```
## Khám phá mô hình (nhà cung cấp ngầm định)

Khi bạn đặt `OLLAMA_API_KEY` (hoặc một hồ sơ xác thực) và **không** định nghĩa `models.providers.ollama`, OpenClaw khám phá các mô hình từ phiên bản Ollama cục bộ tại `http://127.0.0.1:11434`:

- Truy vấn `/api/tags` và `/api/show`
- Chỉ giữ lại các mô hình báo cáo khả năng `tools`
- Đánh dấu `reasoning` khi mô hình báo cáo `thinking`
- Đọc `contextWindow` từ `model_info["<arch>.context_length"]` khi có sẵn
- Đặt `maxTokens` thành 10× cửa sổ ngữ cảnh
- Đặt tất cả chi phí thành `0`

Điều này tránh các mục nhập mô hình thủ công trong khi giữ cho danh mục được căn chỉnh với các khả năng của Ollama.

Để xem những mô hình nào có sẵn:

```bash
ollama list
openclaw models list
```

To add a new model, simply pull it with Ollama:

```bash
ollama pull mistral
```

The new model will be automatically discovered and available to use.

If you set `models.providers.ollama` một cách rõ ràng, khám phá tự động sẽ bị bỏ qua và bạn phải định nghĩa các mô hình theo cách thủ công (xem bên dưới).
## Cấu hình

### Thiết lập cơ bản (khám phá ngầm)

Cách đơn giản nhất để bật Ollama là thông qua biến môi trường:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Explicit setup (manual models)

Use explicit config when:

- Ollama runs on another host/port.
- You want to force specific context windows or model lists.
- You want to include models that do not report tool support.

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

If `OLLAMA_API_KEY` is set, you can omit `apiKey` in the provider entry and OpenClaw will fill it for availability checks.

### Custom base URL (explicit config)

If Ollama is running on a different host or port (explicit config disables auto-discovery, so define models manually):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434",
      },
    },
  },
}
```
### Lựa chọn mô hình

Sau khi cấu hình, tất cả các mô hình Ollama của bạn đều có sẵn:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```
## Nâng cao

### Mô hình suy luận

OpenClaw đánh dấu các mô hình có khả năng suy luận khi Ollama báo cáo `thinking` trong `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Model Costs

Ollama is free and runs locally, so all model costs are set to $0.

### Streaming Configuration

OpenClaw's Ollama integration uses the **native Ollama API** (`/api/chat`) by default, which fully supports streaming and tool calling simultaneously. No special configuration is needed.

#### Legacy OpenAI-Compatible Mode

If you need to use the OpenAI-compatible endpoint instead (e.g., behind a proxy that only supports OpenAI format), set `api: "openai-completions"` explicitly:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Note: The OpenAI-compatible endpoint may not support streaming + tool calling simultaneously. You may need to disable streaming with `params: { streaming: false }` in model config.

### Context windows

For auto-discovered models, OpenClaw uses the context window reported by Ollama when available, otherwise it defaults to `8192`. You can override `contextWindow` and `maxTokens` trong cấu hình nhà cung cấp rõ ràng.
## Khắc phục sự cố

### Ollama không được phát hiện

Hãy đảm bảo Ollama đang chạy và bạn đã đặt `OLLAMA_API_KEY` (hoặc một hồ sơ xác thực), và bạn **không** định nghĩa một mục `models.providers.ollama` rõ ràng:

```bash
ollama serve
```

And that the API is accessible:

```bash
curl http://localhost:11434/api/tags
```

### No models available

OpenClaw only auto-discovers models that report tool support. If your model isn't listed, either:

- Pull a tool-capable model, or
- Define the model explicitly in `models.providers.ollama`.

To add models:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Connection refused

Check that Ollama is running on the correct port:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```
## Xem thêm

- [Nhà cung cấp mô hình](/concepts/model-providers) - Tổng quan về tất cả các nhà cung cấp
- [Lựa chọn mô hình](/concepts/models) - Cách chọn mô hình
- [Cấu hình](/gateway/configuration) - Tham chiếu cấu hình đầy đủ