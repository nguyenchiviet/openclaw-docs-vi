---
summary: Sử dụng các mô hình tập trung vào quyền riêng tư của Venice AI trong OpenClaw
read_when:
  - Bạn muốn suy luận tập trung vào quyền riêng tư trong OpenClaw
  - Bạn muốn hướng dẫn cài đặt Venice AI
title: Venice AI
x-i18n:
  source_path: providers\venice.md
  source_hash: 28dc85ad42067ca5426218aa539602595e66c1a3ac19cc4d2ecce5f43529699e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:17:40.444Z'
---

# Venice AI (Venice highlight)

**Venice** là thiết lập Venice nổi bật của chúng tôi cho suy luận ưu tiên quyền riêng tư với quyền truy cập ẩn danh tùy chọn vào các mô hình độc quyền.

Venice AI cung cấp suy luận AI tập trung vào quyền riêng tư với hỗ trợ cho các mô hình không bị kiểm duyệt và quyền truy cập vào các mô hình độc quyền chính thông qua proxy ẩn danh của họ. Tất cả suy luận là riêng tư theo mặc định—không huấn luyện trên dữ liệu của bạn, không ghi nhật ký.
## Tại sao Venice trong OpenClaw

- **Suy luận riêng tư** cho các mô hình mã nguồn mở (không ghi nhật ký).
- **Các mô hình không bị kiểm duyệt** khi bạn cần chúng.
- **Truy cập ẩn danh** vào các mô hình độc quyền (Opus/GPT/Gemini) khi chất lượng quan trọng.
- Các điểm cuối tương thích OpenAI `/v1`.
## Chế độ Bảo mật

Venice cung cấp hai mức độ bảo mật — hiểu rõ điều này là chìa khóa để chọn mô hình của bạn:

| Chế độ           | Mô tả                                                                                                          | Mô hình                                         |
| -------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Riêng tư**    | Hoàn toàn riêng tư. Các lời nhắc/phản hồi **không bao giờ được lưu trữ hoặc ghi nhật ký**. Tạm thời.                                          | Llama, Qwen, DeepSeek, Venice Uncensored, v.v. |
| **Ẩn danh** | Được chuyển qua Venice với siêu dữ liệu bị loại bỏ. Nhà cung cấp cơ bản (OpenAI, Anthropic) nhìn thấy các yêu cầu ẩn danh. | Claude, GPT, Gemini, Grok, Kimi, MiniMax       |
## Tính năng

- **Tập trung vào quyền riêng tư**: Chọn giữa chế độ "private" (hoàn toàn riêng tư) và "anonymized" (được proxy)
- **Mô hình không bị kiểm duyệt**: Truy cập các mô hình không có hạn chế nội dung
- **Truy cập mô hình chính**: Sử dụng Claude, GPT-5.2, Gemini, Grok thông qua proxy ẩn danh của Venice
- **API tương thích OpenAI**: Các điểm cuối `/v1` tiêu chuẩn để tích hợp dễ dàng
- **Truyền phát**: ✅ Được hỗ trợ trên tất cả các mô hình
- **Gọi hàm**: ✅ Được hỗ trợ trên các mô hình được chọn (kiểm tra khả năng của mô hình)
- **Tầm nhìn**: ✅ Được hỗ trợ trên các mô hình có khả năng tầm nhìn
- **Không có giới hạn tốc độ cứng**: Điều tiết sử dụng công bằng có thể áp dụng cho mức sử dụng cực đoan
## Thiết lập

### 1. Lấy API Key

1. Đăng ký tại [venice.ai](https://venice.ai)
2. Đi tới **Cài đặt → API Keys → Tạo khóa mới**
3. Sao chép API key của bạn (định dạng: `vapi_xxxxxxxxxxxx`)

### 2. Cấu hình OpenClaw

**Tùy chọn A: Biến môi trường**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Option B: Interactive Setup (Recommended)**

```bash
openclaw onboard --auth-choice venice-api-key
```

This will:

1. Prompt for your API key (or use existing `VENICE_API_KEY`)
2. Show all available Venice models
3. Let you pick your default model
4. Configure the provider automatically

**Option C: Non-interactive**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3. Verify Setup

```bash
openclaw agent --model venice/llama-3.3-70b --message "Hello, are you working?"
```
## Lựa chọn Mô hình

Sau khi thiết lập, OpenClaw hiển thị tất cả các mô hình Venice có sẵn. Chọn dựa trên nhu cầu của bạn:

- **Mặc định (lựa chọn của chúng tôi)**: `venice/llama-3.3-70b` để suy luận riêng tư, hiệu suất cân bằng.
- **Chất lượng tổng thể tốt nhất**: `venice/claude-opus-45` cho các công việc khó (Opus vẫn là mô hình mạnh nhất).
- **Quyền riêng tư**: Chọn các mô hình "private" để suy luận hoàn toàn riêng tư.
- **Khả năng**: Chọn các mô hình "anonymized" để truy cập Claude, GPT, Gemini thông qua proxy của Venice.

Thay đổi mô hình mặc định của bạn bất kỳ lúc nào:

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

List all available models:

```bash
openclaw models list | grep venice
```
## Cấu hình qua `openclaw configure`

1. Chạy `openclaw configure`
2. Chọn **Model/auth**
3. Chọn **Venice AI**
## Mô Hình Nào Tôi Nên Sử Dụng?

| Trường Hợp Sử Dụng                     | Mô Hình Được Đề Xuất                | Lý Do                                       |
| ---------------------------- | -------------------------------- | ----------------------------------------- |
| **Trò chuyện chung**             | `llama-3.3-70b`                  | Tốt cho mọi mục đích, hoàn toàn riêng tư            |
| **Chất lượng tốt nhất**     | `claude-opus-45`                 | Opus vẫn là mô hình mạnh nhất cho các tác vụ khó |
| **Quyền riêng tư + chất lượng Claude** | `claude-opus-45`                 | Lý luận tốt nhất thông qua proxy ẩn danh       |
| **Lập trình**                   | `qwen3-coder-480b-a35b-instruct` | Tối ưu hóa cho mã, ngữ cảnh 262k              |
| **Tác vụ thị giác**             | `qwen3-vl-235b-a22b`             | Mô hình thị giác riêng tư tốt nhất                 |
| **Không kiểm duyệt**               | `venice-uncensored`              | Không có hạn chế nội dung                   |
| **Nhanh + rẻ**             | `qwen3-4b`                       | Nhẹ, vẫn có khả năng                |
| **Lý luận phức tạp**        | `deepseek-v3.2`                  | Lý luận mạnh, riêng tư                 |
## Các Mô Hình Có Sẵn (Tổng cộng 25)

### Mô Hình Riêng Tư (15) — Hoàn toàn Riêng Tư, Không Ghi Nhật Ký

| ID Mô Hình                       | Tên                     | Ngữ cảnh (token) | Tính năng                |
| -------------------------------- | ----------------------- | ---------------- | ----------------------- |
| `llama-3.3-70b`                  | Llama 3.3 70B           | 131k             | Tổng quát                |
| `llama-3.2-3b`                   | Llama 3.2 3B            | 131k             | Nhanh, nhẹ              |
| `hermes-3-llama-3.1-405b`        | Hermes 3 Llama 3.1 405B | 131k             | Tác vụ phức tạp         |
| `qwen3-235b-a22b-thinking-2507`  | Qwen3 235B Thinking     | 131k             | Suy luận                |
| `qwen3-235b-a22b-instruct-2507`  | Qwen3 235B Instruct     | 131k             | Tổng quát               |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B        | 262k             | Mã                      |
| `qwen3-next-80b`                 | Qwen3 Next 80B          | 262k             | Tổng quát               |
| `qwen3-vl-235b-a22b`             | Qwen3 VL 235B           | 262k             | Thị giác                |
| `qwen3-4b`                       | Venice Small (Qwen3 4B) | 32k              | Nhanh, suy luận         |
| `deepseek-v3.2`                  | DeepSeek V3.2           | 163k             | Suy luận                |
| `venice-uncensored`              | Venice Uncensored       | 32k              | Không kiểm duyệt        |
| `mistral-31-24b`                 | Venice Medium (Mistral) | 131k             | Thị giác                |
| `google-gemma-3-27b-it`          | Gemma 3 27B Instruct    | 202k             | Thị giác                |
| `openai-gpt-oss-120b`            | OpenAI GPT OSS 120B     | 131k             | Tổng quát               |
| `zai-org-glm-4.7`                | GLM 4.7                 | 202k             | Suy luận, đa ngôn ngữ   |

### Mô Hình Ẩn danh (10) — Thông qua Venice Proxy

| ID Mô Hình               | Mô Hình Gốc       | Ngữ cảnh (token) | Tính năng             |
| ------------------------ | ----------------- | ---------------- | --------------------- |
| `claude-opus-45`         | Claude Opus 4.5   | 202k             | Suy luận, thị giác    |
| `claude-sonnet-45`       | Claude Sonnet 4.5 | 202k             | Suy luận, thị giác    |
| `openai-gpt-52`          | GPT-5.2           | 262k             | Suy luận              |
| `openai-gpt-52-codex`    | GPT-5.2 Codex     | 262k             | Suy luận, thị giác    |
| `gemini-3-pro-preview`   | Gemini 3 Pro      | 202k             | Suy luận, thị giác    |
| `gemini-3-flash-preview` | Gemini 3 Flash    | 262k             | Suy luận, thị giác    |
| `grok-41-fast`           | Grok 4.1 Fast     | 262k             | Suy luận, thị giác    |
| `grok-code-fast-1`       | Grok Code Fast 1  | 262k             | Suy luận, mã          |
| `kimi-k2-thinking`       | Kimi K2 Thinking  | 262k             | Suy luận              |
| `minimax-m21`            | MiniMax M2.1      | 202k             | Suy luận              |
## Khám phá Mô hình

OpenClaw tự động khám phá các mô hình từ Venice API khi `VENICE_API_KEY` được đặt. Nếu API không thể truy cập được, nó sẽ quay lại một danh mục tĩnh.

Điểm cuối `/models` là công khai (không cần xác thực để liệt kê), nhưng suy luận yêu cầu một khóa API hợp lệ.
## Truyền phát & Hỗ trợ Công cụ

| Tính năng             | Hỗ trợ                                                  |
| -------------------- | ------------------------------------------------------- |
| **Truyền phát**       | ✅ Tất cả các mô hình                                   |
| **Gọi hàm**          | ✅ Hầu hết các mô hình (kiểm tra `supportsFunctionCalling` trong API) |
| **Thị giác/Hình ảnh** | ✅ Các mô hình được đánh dấu bằng tính năng "Vision"    |
| **Chế độ JSON**      | ✅ Được hỗ trợ thông qua `response_format`               |
## Giá cả

Venice sử dụng hệ thống dựa trên tín chỉ. Kiểm tra [venice.ai/pricing](https://venice.ai/pricing) để xem giá hiện tại:

- **Mô hình riêng tư**: Thường có chi phí thấp hơn
- **Mô hình ẩn danh**: Tương tự như giá API trực tiếp + phí Venice nhỏ
## So sánh: Venice vs Direct API

| Khía cạnh     | Venice (Ẩn danh)              | Direct API          |
| ------------ | ----------------------------- | ------------------- |
| **Quyền riêng tư**  | Metadata bị loại bỏ, ẩn danh | Tài khoản của bạn được liên kết |
| **Độ trễ**  | +10-50ms (proxy)              | Trực tiếp              |
| **Tính năng** | Hỗ trợ hầu hết các tính năng       | Đầy đủ tính năng       |
| **Thanh toán**  | Tín dụng Venice                | Thanh toán nhà cung cấp    |
## Ví dụ Sử dụng

```bash
# Use default private model
openclaw agent --model venice/llama-3.3-70b --message "Quick health check"

# Use Claude via Venice (anonymized)
openclaw agent --model venice/claude-opus-45 --message "Summarize this task"

# Use uncensored model
openclaw agent --model venice/venice-uncensored --message "Draft options"

# Use vision model with image
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# Use coding model
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```
## Khắc phục sự cố

### Khóa API không được nhận dạng

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Ensure the key starts with `vapi_`.

### Model not available

The Venice model catalog updates dynamically. Run `openclaw models list` to see currently available models. Some models may be temporarily offline.

### Connection issues

Venice API is at `https://api.venice.ai/api/v1`. Đảm bảo mạng của bạn cho phép kết nối HTTPS.
## Ví dụ tệp cấu hình

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
## Liên kết

- [Venice AI](https://venice.ai)
- [Tài liệu API](https://docs.venice.ai)
- [Giá cả](https://venice.ai/pricing)
- [Trạng thái](https://status.venice.ai)