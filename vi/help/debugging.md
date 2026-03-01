---
summary: 'Công cụ gỡ lỗi: chế độ watch, luồng mô hình thô, và theo dõi rò rỉ lý luận'
read_when:
  - Bạn cần kiểm tra đầu ra mô hình thô để phát hiện rò rỉ lý luận.
  - Bạn muốn chạy Gateway ở chế độ watch trong khi lặp lại
  - Bạn cần một quy trình gỡ lỗi có thể lặp lại
title: Gỡ lỗi
x-i18n:
  source_path: help\debugging.md
  source_hash: 9496410cc38a8024a8c49403af657099fc0a155425e15b7f77c964d15e82afb4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:00:22.028Z'
---

# Gỡ lỗi

Trang này bao gồm các công cụ hỗ trợ gỡ lỗi cho đầu ra truyền phát, đặc biệt là khi một nhà cung cấp trộn lập luận vào văn bản thông thường.
## Ghi đè debug thời chạy

Sử dụng `/debug` trong chat để đặt **ghi đè cấu hình chỉ thời chạy** (bộ nhớ, không phải đĩa).
`/debug` bị vô hiệu hóa theo mặc định; bật bằng `commands.debug: true`.
Điều này rất hữu ích khi bạn cần chuyển đổi các cài đặt không rõ ràng mà không cần chỉnh sửa `openclaw.json`.

Ví dụ:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` xóa tất cả các ghi đè và quay lại cấu hình trên đĩa.
## Chế độ theo dõi Gateway

Để lặp lại nhanh chóng, chạy gateway dưới trình theo dõi tệp:

```bash
pnpm gateway:watch
```

This maps to:

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

Add any gateway CLI flags after `gateway:watch` và chúng sẽ được chuyển qua
trên mỗi lần khởi động lại.
## Hồ sơ dev + gateway dev (--dev)

Sử dụng hồ sơ dev để cô lập trạng thái và thiết lập một môi trường an toàn, có thể loại bỏ để gỡ lỗi. Có **hai** `--dev` cờ:

- **`--dev` toàn cục (hồ sơ):** cô lập trạng thái dưới `~/.openclaw-dev` và đặt cổng gateway mặc định thành `19001` (các cổng dẫn xuất thay đổi theo).
- **`gateway --dev`: yêu cầu Gateway tự động tạo cấu hình mặc định + không gian làm việc** khi bị thiếu (và bỏ qua BOOTSTRAP.md).

Quy trình được khuyến nghị (hồ sơ dev + bootstrap dev):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

If you don’t have a global install yet, run the CLI via `pnpm openclaw ...`.

What this does:

1. **Profile isolation** (global `--dev`)
   - `OPENCLAW_PROFILE=dev`
   - `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   - `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   - `OPENCLAW_GATEWAY_PORT=19001` (browser/canvas shift accordingly)

2. **Dev bootstrap** (`gateway --dev`)
   - Writes a minimal config if missing (`gateway.mode=local`, bind loopback).
   - Sets `agent.workspace` to the dev workspace.
   - Sets `agent.skipBootstrap=true` (no BOOTSTRAP.md).
   - Seeds the workspace files if missing:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   - Default identity: **C3‑PO** (protocol droid).
   - Skips channel providers in dev mode (`OPENCLAW_SKIP_CHANNELS=1`).

Reset flow (fresh start):

```bash
pnpm gateway:dev:reset
```

Note: `--dev` is a **global** profile flag and gets eaten by some runners.
If you need to spell it out, use the env var form:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` wipes config, credentials, sessions, and the dev workspace (using
`trash`, not `rm`), then recreates the default dev setup.

Tip: if a non‑dev gateway is already running (launchd/systemd), stop it first:

```bash
openclaw gateway stop
```
## Ghi nhật ký luồng thô (OpenClaw)

OpenClaw có thể ghi nhật ký **luồng trợ lý thô** trước khi có bất kỳ lọc/định dạng nào.
Đây là cách tốt nhất để xem liệu lý luận có đến dưới dạng các delta văn bản thuần túy
(hoặc dưới dạng các khối suy nghĩ riêng biệt).

Bật nó qua CLI:

```bash
pnpm gateway:watch --raw-stream
```

Optional path override:

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Equivalent env vars:

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Default file:

`~/.openclaw/logs/raw-stream.jsonl`
## Ghi nhật ký khối thô (pi-mono)

Để ghi lại **các khối tương thích OpenAI thô** trước khi chúng được phân tích cú pháp thành các khối,
pi-mono cung cấp một logger riêng biệt:

```bash
PI_RAW_STREAM=1
```

Optional path:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Default file:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Note: this is only emitted by processes using pi-mono’s
> `openai-completions` provider.
## Ghi chú về bảo mật

- Nhật ký luồng thô có thể bao gồm các lời nhắc đầy đủ, kết quả công cụ và dữ liệu người dùng.
- Giữ nhật ký cục bộ và xóa chúng sau khi gỡ lỗi.
- Nếu bạn chia sẻ nhật ký, hãy xóa các bí mật và thông tin cá nhân trước tiên.