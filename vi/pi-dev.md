---
title: Quy Trình Phát Triển Pi
summary: 'Quy trình phát triển cho tích hợp Pi: xây dựng, kiểm thử và xác thực trực tiếp'
read_when:
  - Đang làm việc trên mã tích hợp Pi hoặc các bài kiểm tra
  - 'Chạy các luồng lint, typecheck và live test cụ thể cho Pi'
x-i18n:
  source_path: pi-dev.md
  source_hash: 497f962ca431f46046a5cb7267e73a6a92cc6c4c35608cdf77f1d7e128c8d01f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:11:30.221Z'
---

# Quy trình phát triển Pi

Hướng dẫn này tóm tắt một quy trình hợp lý để làm việc với tích hợp Pi trong OpenClaw.
## Kiểm tra kiểu dữ liệu và Linting

- Kiểm tra kiểu dữ liệu và xây dựng: `pnpm build`
- Lint: `pnpm lint`
- Kiểm tra định dạng: `pnpm format`
- Kiểm tra đầy đủ trước khi đẩy: `pnpm lint && pnpm build && pnpm test`
## Chạy Kiểm Thử Pi

Chạy bộ kiểm thử tập trung vào Pi trực tiếp với Vitest:

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

To include the live provider exercise:

```bash
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

This covers the main Pi unit suites:

- `src/agents/pi-*.test.ts`
- `src/agents/pi-embedded-*.test.ts`
- `src/agents/pi-tools*.test.ts`
- `src/agents/pi-settings.test.ts`
- `src/agents/pi-tool-definition-adapter.test.ts`
- `src/agents/pi-extensions/*.test.ts`
## Kiểm tra thủ công

Quy trình được khuyến nghị:

- Chạy gateway ở chế độ phát triển:
  - `pnpm gateway:dev`
- Kích hoạt agent trực tiếp:
  - `pnpm openclaw agent --message "Hello" --thinking low`
- Sử dụng TUI để gỡ lỗi tương tác:
  - `pnpm tui`

Đối với hành vi gọi công cụ, hãy nhắc nhở một hành động `read` hoặc `exec` để bạn có thể thấy truyền phát công cụ và xử lý tải trọng.
## Đặt lại trạng thái sạch sẽ

Trạng thái được lưu trữ trong thư mục trạng thái OpenClaw. Mặc định là `~/.openclaw`. Nếu `OPENCLAW_STATE_DIR` được đặt, hãy sử dụng thư mục đó thay thế.

Để đặt lại mọi thứ:

- `openclaw.json` cho cấu hình
- `credentials/` cho các hồ sơ xác thực và mã thông báo
- `agents/<agentId>/sessions/` cho lịch sử phiên agent
- `agents/<agentId>/sessions.json` cho chỉ mục phiên
- `sessions/` nếu các đường dẫn cũ tồn tại
- `workspace/` nếu bạn muốn một không gian làm việc trống

Nếu bạn chỉ muốn đặt lại các phiên, hãy xóa `agents/<agentId>/sessions/` và `agents/<agentId>/sessions.json` cho agent đó. Giữ `credentials/` nếu bạn không muốn xác thực lại.
## Tài liệu tham khảo

- [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
- [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)