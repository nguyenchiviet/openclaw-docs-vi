---
summary: >-
  Cách chạy kiểm thử cục bộ (vitest) và khi nào nên sử dụng chế độ
  force/coverage
read_when:
  - Chạy hoặc sửa lỗi kiểm thử
title: Kiểm tra
x-i18n:
  source_path: reference\test.md
  source_hash: bab46ea408e14b87590e101242ed43af7934e86c7b275614d8ab1c475239a9b1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:16:37.945Z'
---

# Kiểm tra

- Bộ công cụ kiểm tra đầy đủ (bộ kiểm thử, trực tiếp, Docker): [Kiểm tra](/help/testing)

- `pnpm test:force`: Kết thúc mọi tiến trình gateway còn sót lại đang giữ cổng điều khiển mặc định, sau đó chạy bộ Vitest đầy đủ với một cổng gateway riêng biệt để các bài kiểm tra máy chủ không xung đột với một phiên bản đang chạy. Sử dụng lệnh này khi một lần chạy gateway trước đó đã chiếm cổng 18789.
- `pnpm test:coverage`: Chạy bộ kiểm thử đơn vị với độ phủ V8 (qua `vitest.unit.config.ts`). Ngưỡng toàn cầu là 70% dòng/nhánh/hàm/câu lệnh. Độ phủ loại trừ các điểm vào nặng về tích hợp (kết nối CLI, cầu nối gateway/telegram, máy chủ tĩnh webchat) để giữ mục tiêu tập trung vào logic có thể kiểm thử đơn vị.
- `pnpm test` trên Node 24+: OpenClaw tự động tắt Vitest `vmForks` và sử dụng `forks` để tránh `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`. Bạn có thể buộc hành vi bằng `OPENCLAW_TEST_VM_FORKS=0|1`.
- `pnpm test:e2e`: Chạy các bài kiểm tra khói đầu cuối của gateway (ghép nối WS/HTTP/node đa phiên bản). Mặc định là `vmForks` + các worker thích ứng trong `vitest.e2e.config.ts`; điều chỉnh bằng `OPENCLAW_E2E_WORKERS=<n>` và đặt `OPENCLAW_E2E_VERBOSE=1` để có nhật ký chi tiết.
- `pnpm test:live`: Chạy các bài kiểm tra trực tiếp của nhà cung cấp (minimax/zai). Yêu cầu khóa API và `LIVE=1` (hoặc `*_LIVE_TEST=1` dành riêng cho nhà cung cấp) để bỏ qua.
## Cổng PR cục bộ

Để kiểm tra cổng/đổ bộ PR cục bộ, hãy chạy:

- `pnpm check`
- `pnpm build`
- `pnpm test`
- `pnpm check:docs`

Nếu `pnpm test` bị lỗi trên một máy chủ đang tải, hãy chạy lại một lần trước khi coi đó là lỗi hồi quy, sau đó cô lập bằng `pnpm vitest run <path/to/test>`. Đối với các máy chủ bị hạn chế bộ nhớ, hãy sử dụng:

- `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`
## Kiểm tra độ trễ mô hình (khóa cục bộ)

Script: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

Cách sử dụng:

- `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
- Biến môi trường tùy chọn: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
- Lời nhắc mặc định: “Trả lời bằng một từ duy nhất: ok. Không dấu câu hoặc văn bản thừa.”

Lần chạy cuối cùng (2025-12-31, 20 lần chạy):

- minimax trung vị 1279ms (tối thiểu 1114, tối đa 2431)
- opus trung vị 2454ms (tối thiểu 1224, tối đa 3170)

## Thiết lập ban đầu E2E (Docker)

Docker là tùy chọn; điều này chỉ cần thiết cho các bài kiểm tra khói thiết lập ban đầu trong container.

Quy trình khởi động lạnh hoàn chỉnh trong một container Linux sạch:

```bash
scripts/e2e/onboard-docker.sh
```

This script drives the interactive wizard via a pseudo-tty, verifies config/workspace/session files, then starts the gateway and runs `openclaw health`.
## Kiểm tra nhập QR (Docker)

Đảm bảo `qrcode-terminal` tải dưới Node 22+ trong Docker:

```bash
pnpm test:docker:qr
```