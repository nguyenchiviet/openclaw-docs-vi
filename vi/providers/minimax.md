---
summary: Sử dụng MiniMax M2.1 trong OpenClaw
read_when:
  - Bạn muốn các mô hình MiniMax trong OpenClaw
  - Bạn cần hướng dẫn thiết lập MiniMax
title: MiniMax
x-i18n:
  source_path: providers\minimax.md
  source_hash: 291cdecbe68e1cb10d87510a1e6ca26f5af07d46309ca7203c62a4acef8a0501
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:17.819Z'
---

# MiniMax

MiniMax là một công ty AI xây dựng họ mô hình **M2/M2.1**. Phiên bản tập trung vào mã hóa hiện tại là **MiniMax M2.1** (23 tháng 12, 2025), được xây dựng cho các tác vụ phức tạp trong thế giới thực.

Nguồn: [Ghi chú phát hành MiniMax M2.1](https://www.minimax.io/news/minimax-m21)
## Tổng quan mô hình (M2.1)

MiniMax nổi bật những cải tiến này trong M2.1:

- **Lập trình đa ngôn ngữ** mạnh hơn (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
- **Phát triển web/ứng dụng** tốt hơn và chất lượng đầu ra thẩm mỹ (bao gồm cả di động gốc).
- Xử lý **hướng dẫn tổng hợp** được cải thiện cho các quy trình kiểu văn phòng, dựa trên
  tư duy xen kẽ và thực thi ràng buộc tích hợp.
- **Phản hồi ngắn gọn hơn** với mức sử dụng token thấp hơn và vòng lặp lặp lại nhanh hơn.
- Khả năng tương thích **framework công cụ/agent** mạnh hơn và quản lý ngữ cảnh (Claude Code,
  Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
- Đầu ra **đối thoại và viết kỹ thuật** chất lượng cao hơn.
## MiniMax M2.1 vs MiniMax M2.1 Lightning

- **Tốc độ:** Lightning là biến thể "nhanh" trong tài liệu định giá của MiniMax.
- **Chi phí:** Định giá cho thấy chi phí đầu vào giống nhau, nhưng Lightning có chi phí đầu ra cao hơn.
- **Định tuyến kế hoạch mã hóa:** Back-end Lightning không trực tiếp khả dụng trên kế hoạch mã hóa MiniMax. MiniMax tự động định tuyến hầu hết các yêu cầu đến Lightning, nhưng quay lại back-end M2.1 thông thường trong các đợt tăng lưu lượng.
## Chọn một thiết lập

### MiniMax OAuth (Coding Plan) — được khuyến nghị

**Tốt nhất cho:** thiết lập nhanh với MiniMax Coding Plan thông qua OAuth, không cần khóa API.

Bật plugin OAuth được đi kèm và xác thực:

```bash
openclaw plugins enable minimax-portal-auth  # skip if already loaded.
openclaw gateway restart  # restart if gateway is already running
openclaw onboard --auth-choice minimax-portal
```

You will be prompted to select an endpoint:

- **Global** - International users (`api.minimax.io`)
- **CN** - Users in China (`api.minimaxi.com`)

See [MiniMax OAuth plugin README](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth) for details.

### MiniMax M2.1 (API key)

**Best for:** hosted MiniMax with Anthropic-compatible API.

Configure via CLI:

- Run `openclaw configure`
- Select **Model/auth**
- Choose **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.1 làm dự phòng (Opus chính)
**Tốt nhất cho:** giữ Opus 4.6 làm mô hình chính, chuyển sang MiniMax M2.1 nếu thất bại.

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
    },
  },
}
```

### Optional: Local via LM Studio (manual)

**Best for:** local inference with LM Studio.
We have seen strong results with MiniMax M2.1 on powerful hardware (e.g. a
desktop/server) using LM Studio's local server.

Configure manually via `openclaw.json`:

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```
## Cấu hình qua `openclaw configure`

Sử dụng trình hướng dẫn cấu hình tương tác để thiết lập MiniMax mà không cần chỉnh sửa JSON:

1. Chạy `openclaw configure`.
2. Chọn **Model/auth**.
3. Chọn **MiniMax M2.1**.
4. Chọn mô hình mặc định của bạn khi được nhắc.
## Tùy chọn cấu hình

- `models.providers.minimax.baseUrl`: ưu tiên `https://api.minimax.io/anthropic` (tương thích với Anthropic); `https://api.minimax.io/v1` là tùy chọn cho các payload tương thích với OpenAI.
- `models.providers.minimax.api`: ưu tiên `anthropic-messages`; `openai-completions` là tùy chọn cho các payload tương thích với OpenAI.
- `models.providers.minimax.apiKey`: Khóa API MiniMax (`MINIMAX_API_KEY`).
- `models.providers.minimax.models`: định nghĩa `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
- `agents.defaults.models`: các mô hình bí danh mà bạn muốn trong danh sách cho phép.
- `models.mode`: giữ `merge` nếu bạn muốn thêm MiniMax cùng với các mô hình tích hợp.
## Ghi chú

- Model refs là `minimax/<model>`.
- Coding Plan usage API: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (yêu cầu khóa coding plan).
- Cập nhật giá trị định giá trong `models.json` nếu bạn cần theo dõi chi phí chính xác.
- Liên kết giới thiệu cho MiniMax Coding Plan (giảm 10%): [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
- Xem [/concepts/model-providers](/concepts/model-providers) để biết quy tắc nhà cung cấp.
- Sử dụng `openclaw models list` và `openclaw models set minimax/MiniMax-M2.1` để chuyển đổi.
## Khắc phục sự cố

### "Unknown model: minimax/MiniMax-M2.1"

Điều này thường có nghĩa là **nhà cung cấp MiniMax chưa được cấu hình** (không có mục nhà cung cấp và không tìm thấy hồ sơ xác thực MiniMax/khóa biến môi trường). Một bản sửa lỗi cho phát hiện này có trong **2026.1.12** (chưa phát hành tại thời điểm viết). Sửa bằng cách:

- Nâng cấp lên **2026.1.12** (hoặc chạy từ nguồn `main`), sau đó khởi động lại Gateway.
- Chạy `openclaw configure` và chọn **MiniMax M2.1**, hoặc
- Thêm khối `models.providers.minimax` theo cách thủ công, hoặc
- Đặt `MINIMAX_API_KEY` (hoặc hồ sơ xác thực MiniMax) để nhà cung cấp có thể được tiêm.

Đảm bảo ID mô hình **phân biệt chữ hoa chữ thường**:

- `minimax/MiniMax-M2.1`
- `minimax/MiniMax-M2.1-lightning`

Sau đó kiểm tra lại bằng:

```bash
openclaw models list
```