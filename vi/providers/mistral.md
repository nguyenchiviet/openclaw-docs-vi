---
summary: Sử dụng các mô hình Mistral và dịch vụ phiên âm Voxtral với OpenClaw
read_when:
  - Bạn muốn sử dụng các mô hình Mistral trong OpenClaw
  - Bạn cần onboarding khóa API Mistral và tham chiếu mô hình
title: Mistral
x-i18n:
  source_path: providers\mistral.md
  source_hash: 4f3efe060cbaeb14e20439ade040e57d27e7d98fb9dd06e657f6a69ae808f24f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:59.444Z'
---

# Mistral

OpenClaw hỗ trợ Mistral cho cả định tuyến mô hình văn bản/hình ảnh (`mistral/...`) và
phiên âm âm thanh qua Voxtral trong hiểu biết phương tiện.
Mistral cũng có thể được sử dụng cho nhúng bộ nhớ (`memorySearch.provider = "mistral"`).

## Thiết lập CLI

```bash
openclaw onboard --auth-choice mistral-api-key
# or non-interactive
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## Config snippet (LLM provider)

```json5
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## Config snippet (audio transcription with Voxtral)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## Notes

- Mistral auth uses `MISTRAL_API_KEY`.
- Provider base URL defaults to `https://api.mistral.ai/v1`.
- Onboarding default model is `mistral/mistral-large-latest`.
- Media-understanding default audio model for Mistral is `voxtral-mini-latest`.
- Media transcription path uses `/v1/audio/transcriptions`.
- Memory embeddings path uses `/v1/embeddings` (default model: `mistral-embed`).