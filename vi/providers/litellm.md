---
summary: >-
  Chạy OpenClaw thông qua LiteLLM Proxy để truy cập mô hình thống nhất và theo
  dõi chi phí
read_when:
  - Bạn muốn định tuyến OpenClaw thông qua một proxy LiteLLM
  - >-
    Bạn cần theo dõi chi phí, ghi nhật ký hoặc định tuyến mô hình thông qua
    LiteLLM
x-i18n:
  source_path: providers\litellm.md
  source_hash: 269529671c60864972441606c730b5ca327546a45d3b264dbd03204c4401936f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:09.459Z'
---

# LiteLLM

[LiteLLM](https://litellm.ai) là một gateway LLM mã nguồn mở cung cấp API thống nhất cho hơn 100 nhà cung cấp mô hình. Định tuyến OpenClaw qua LiteLLM để có được theo dõi chi phí tập trung, ghi nhật ký và tính linh hoạt để chuyển đổi backend mà không cần thay đổi cấu hình OpenClaw của bạn.
## Tại sao sử dụng LiteLLM với OpenClaw?

- **Theo dõi chi phí** — Xem chính xác OpenClaw chi tiêu bao nhiêu trên tất cả các mô hình
- **Định tuyến mô hình** — Chuyển đổi giữa Claude, GPT-4, Gemini, Bedrock mà không cần thay đổi cấu hình
- **Khóa ảo** — Tạo khóa có giới hạn chi tiêu cho OpenClaw
- **Ghi nhật ký** — Nhật ký yêu cầu/phản hồi đầy đủ để gỡ lỗi
- **Dự phòng** — Chuyển đổi tự động nếu nhà cung cấp chính của bạn bị sự cố
## Bắt đầu nhanh

### Thông qua thiết lập ban đầu

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manual setup

1. Start LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Point OpenClaw to LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Xong. OpenClaw hiện định tuyến qua LiteLLM.
## Cấu hình

### Biến môi trường

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Config file

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```
## Khóa ảo

Tạo một khóa chuyên dụng cho OpenClaw với giới hạn chi tiêu:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Use the generated key as `LITELLM_API_KEY`.
## Định tuyến mô hình

LiteLLM có thể định tuyến các yêu cầu mô hình đến các backend khác nhau. Cấu hình trong `config.yaml` LiteLLM của bạn:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw keeps requesting `claude-opus-4-6` — LiteLLM xử lý định tuyến.
## Xem mức sử dụng

Kiểm tra bảng điều khiển hoặc API của LiteLLM:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```
## Ghi chú

- LiteLLM chạy trên `http://localhost:4000` theo mặc định
- OpenClaw kết nối thông qua điểm cuối `/v1/chat/completions` tương thích với OpenAI
- Tất cả các tính năng của OpenClaw hoạt động thông qua LiteLLM — không có giới hạn
## Xem thêm

- [Tài liệu LiteLLM](https://docs.litellm.ai)
- [Nhà cung cấp Mô hình](/concepts/model-providers)