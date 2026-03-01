---
summary: Sử dụng OpenCode Zen (các mô hình được lựa chọn) với OpenClaw
read_when:
  - Bạn muốn OpenCode Zen để truy cập mô hình
  - >-
    Bạn muốn một danh sách được lựa chọn kỹ lưỡng về các mô hình thân thiện với
    lập trình
title: OpenCode Zen
x-i18n:
  source_path: providers\opencode.md
  source_hash: b3b5c640ac32f3177f6f4ffce766f3f57ff75c6ca918822c817d9a18f680be8f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:47.379Z'
---

# OpenCode Zen

OpenCode Zen là một **danh sách các mô hình được lựa chọn** được đề xuất bởi nhóm OpenCode cho các agent mã hóa.
Đây là một đường dẫn truy cập mô hình tùy chọn, được lưu trữ sử dụng khóa API và nhà cung cấp `opencode`.
Zen hiện đang ở phiên bản beta.

## Thiết lập CLI

```bash
openclaw onboard --auth-choice opencode-zen
# or non-interactive
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## Config snippet

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## Notes

- `OPENCODE_ZEN_API_KEY` cũng được hỗ trợ.
- Bạn đăng nhập vào Zen, thêm chi tiết thanh toán và sao chép khóa API của bạn.
- OpenCode Zen tính phí theo từng yêu cầu; kiểm tra bảng điều khiển OpenCode để biết chi tiết.