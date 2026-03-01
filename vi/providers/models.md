---
summary: Các nhà cung cấp mô hình (LLMs) được hỗ trợ bởi OpenClaw
read_when:
  - Bạn muốn chọn một nhà cung cấp mô hình
  - Bạn muốn các ví dụ thiết lập nhanh cho xác thực LLM + lựa chọn mô hình
title: Hướng dẫn Bắt đầu Nhanh với Nhà cung cấp Mô hình
x-i18n:
  source_path: providers\models.md
  source_hash: 297ccf273b92e5ead84d7464784d3dd43fdcd9e1812af132acb19713dbdc05ea
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:27.403Z'
---

# Nhà cung cấp Mô hình

OpenClaw có thể sử dụng nhiều nhà cung cấp LLM. Chọn một, xác thực, sau đó đặt mô hình mặc định là `provider/model`.

## Nổi bật: Venice (Venice AI)

Venice là thiết lập Venice AI được khuyến nghị của chúng tôi để suy luận ưu tiên quyền riêng tư với tùy chọn sử dụng Opus cho các tác vụ khó nhất.

- Mặc định: `venice/llama-3.3-70b`
- Tốt nhất: `venice/claude-opus-45` (Opus vẫn là mô hình mạnh nhất)

Xem [Venice AI](/providers/venice).

## Bắt đầu nhanh (hai bước)

1. Xác thực với nhà cung cấp (thường qua `openclaw onboard`).
2. Đặt mô hình mặc định:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Nhà cung cấp được hỗ trợ (bộ khởi động)

- [OpenAI (/providers/openai)
- [Anthropic (/providers/anthropic)
- [OpenRouter](/providers/openrouter)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
- [Moonshot AI (/providers/moonshot)
- [Mistral](/providers/mistral)
- [Synthetic](/providers/synthetic)
- [OpenCode Zen](/providers/opencode)
- [Z.AI](/providers/zai)
- [GLM models](/providers/glm)
- [MiniMax](/providers/minimax)
- [Venice (/providers/venice)
- [Amazon Bedrock](/providers/bedrock)
- [Qianfan](/providers/qianfan)

Để xem danh mục nhà cung cấp đầy đủ (xAI, Groq, Mistral, v.v.) và cấu hình nâng cao, xem [Nhà cung cấp mô hình](/concepts/model-providers).