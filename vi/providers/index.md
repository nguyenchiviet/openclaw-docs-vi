---
summary: Các nhà cung cấp mô hình (LLMs) được hỗ trợ bởi OpenClaw
read_when:
  - Bạn muốn chọn một nhà cung cấp mô hình
  - Bạn cần một cái nhìn tổng quan nhanh về các backend LLM được hỗ trợ
title: Nhà cung cấp Mô hình
x-i18n:
  source_path: providers\index.md
  source_hash: 1f2638d7f2d4f19a4feff46013e9bd980eb4f3b89995453c9ef33144001d460a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:07.572Z'
---

# Nhà cung cấp Mô hình

OpenClaw có thể sử dụng nhiều nhà cung cấp LLM. Chọn một nhà cung cấp, xác thực, sau đó đặt mô hình mặc định là `provider/model`.

Đang tìm tài liệu kênh trò chuyện (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/v.v.)? Xem [Kênh](/channels).
## Highlight: Venice (Venice AI)

Venice là thiết lập Venice AI được khuyến nghị của chúng tôi để suy luận ưu tiên quyền riêng tư với tùy chọn sử dụng Opus cho các tác vụ khó.

- Mặc định: `venice/llama-3.3-70b`
- Tốt nhất: `venice/claude-opus-45` (Opus vẫn là mô hình mạnh nhất)

Xem [Venice AI](/providers/venice).
## Bắt đầu nhanh

1. Xác thực với nhà cung cấp (thường qua `openclaw onboard`).
2. Đặt mô hình mặc định:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```
## Tài liệu nhà cung cấp

- [OpenAI (/providers/openai)
- [Anthropic (/providers/anthropic)
- [Qwen (/providers/qwen)
- [OpenRouter](/providers/openrouter)
- [LiteLLM (/providers/litellm)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Together AI](/providers/together)
- [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
- [Moonshot AI (/providers/moonshot)
- [Mistral](/providers/mistral)
- [OpenCode Zen](/providers/opencode)
- [Amazon Bedrock](/providers/bedrock)
- [Z.AI](/providers/zai)
- [Xiaomi](/providers/xiaomi)
- [GLM models](/providers/glm)
- [MiniMax](/providers/minimax)
- [Venice (/providers/venice)
- [Hugging Face (/providers/huggingface)
- [Ollama (/providers/ollama)
- [vLLM (/providers/vllm)
- [Qianfan](/providers/qianfan)
- [NVIDIA](/providers/nvidia)
## Nhà cung cấp dịch vụ chuyển đổi giọng nói thành văn bản

- [Deepgram (/providers/deepgram)
## Công cụ cộng đồng

- [Claude Max API Proxy](/providers/claude-max-api-proxy) - Sử dụng Claude Max/Pro subscription làm điểm cuối API tương thích với OpenAI

Để xem danh mục nhà cung cấp đầy đủ (xAI, Groq, Mistral, v.v.) và cấu hình nâng cao,
xem [Model providers](/concepts/model-providers).