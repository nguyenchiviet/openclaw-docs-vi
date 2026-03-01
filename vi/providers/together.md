---
summary: Thiết lập Together AI (xác thực + lựa chọn mô hình)
read_when:
  - Bạn muốn sử dụng Together AI với OpenClaw
  - Bạn cần biến môi trường API key hoặc lựa chọn xác thực CLI
x-i18n:
  source_path: providers\together.md
  source_hash: 4f2ba5a12b03d0140feba4f54e0540bb57237cd131c8f1d826bc3629fde2d111
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:17:02.278Z'
---

# Together AI

[Together AI](https://together.ai) cung cấp quyền truy cập vào các mô hình mã nguồn mở hàng đầu bao gồm Llama, DeepSeek, Kimi và nhiều mô hình khác thông qua một API thống nhất.

- Nhà cung cấp: `together`
- Xác thực: `TOGETHER_API_KEY`
- API: Tương thích OpenAI

## Bắt đầu nhanh

1. Đặt khóa API (khuyến nghị: lưu trữ nó cho Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Set a default model:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Non-interactive example

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

This will set `together/moonshotai/Kimi-K2.5` as the default model.

## Environment note

If the Gateway runs as a daemon (launchd/systemd), make sure `TOGETHER_API_KEY`
is available to that process (for example, in `~/.openclaw/.env` or via
`env.shellEnv`).

## Các mô hình có sẵn

Together AI cung cấp quyền truy cập vào nhiều mô hình mã nguồn mở phổ biến:

- **GLM 4.7 Fp8** - Mô hình mặc định với cửa sổ ngữ cảnh 200K
- **Llama 3.3 70B Instruct Turbo** - Tuân theo hướng dẫn nhanh chóng và hiệu quả
- **Llama 4 Scout** - Mô hình thị giác với khả năng hiểu hình ảnh
- **Llama 4 Maverick** - Thị giác và lập luận nâng cao
- **DeepSeek V3.1** - Mô hình mạnh mẽ cho mã hóa và lập luận
- **DeepSeek R1** - Mô hình lập luận nâng cao
- **Kimi K2 Instruct** - Mô hình hiệu suất cao với cửa sổ ngữ cảnh 262K

Tất cả các mô hình hỗ trợ hoàn thành trò chuyện tiêu chuẩn và tương thích với API OpenAI.