---
summary: 'Bun workflow (thử nghiệm): cài đặt và những điểm cần lưu ý so với pnpm'
read_when:
  - Bạn muốn có vòng lặp phát triển cục bộ nhanh nhất (bun + watch)
  - Bạn gặp phải các vấn đề liên quan đến script cài đặt/vá lỗi/vòng đời của Bun
title: Bun (Thử nghiệm)
x-i18n:
  source_path: install\bun.md
  source_hash: eb3f4c222b6bae49938d8bf53a0818fe5f5e0c0c3c1adb3e0a832ce8f785e1e3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:07:17.646Z'
---

# Bun (thử nghiệm)

Mục tiêu: chạy repo này với **Bun** (tùy chọn, không được khuyến nghị cho WhatsApp/Telegram)
mà không phân kỳ khỏi quy trình làm việc của pnpm.

⚠️ **Không được khuyến nghị cho runtime Gateway** (lỗi WhatsApp/Telegram). Sử dụng Node cho production.

## Trạng thái

- Bun là một runtime cục bộ tùy chọn để chạy TypeScript trực tiếp (`bun run …`, `bun --watch …`).
- `pnpm` là mặc định cho các bản dựng và vẫn được hỗ trợ đầy đủ (và được sử dụng bởi một số công cụ tài liệu).
- Bun không thể sử dụng `pnpm-lock.yaml` và sẽ bỏ qua nó.

## Cài đặt

Mặc định:

```sh
bun install
```

Note: `bun.lock`/`bun.lockb` are gitignored, so there’s no repo churn either way. If you want _no lockfile writes_:

```sh
bun install --no-save
```

## Build / Test (Bun)

```sh
bun run build
bun run vitest run
```

## Bun lifecycle scripts (blocked by default)

Bun may block dependency lifecycle scripts unless explicitly trusted (`bun pm untrusted` / `bun pm trust`).
For this repo, the commonly blocked scripts are not required:

- `@whiskeysockets/baileys` `preinstall`: checks Node major >= 20 (we run Node 22+).
- `protobufjs` `postinstall`: emits warnings about incompatible version schemes (no build artifacts).

If you hit a real runtime issue that requires these scripts, trust them explicitly:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## Caveats

- Some scripts still hardcode pnpm (e.g. `docs:build`, `ui:*`, `protocol:check`). Chạy những cái này qua pnpm hiện tại.