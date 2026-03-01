---
summary: 'Các kênh Stable, beta và dev: ngữ nghĩa, chuyển đổi và gắn thẻ'
read_when:
  - Bạn muốn chuyển đổi giữa stable/beta/dev
  - Bạn đang gắn thẻ hoặc xuất bản các bản phát hành sơ bộ
title: Kênh Phát Triển
x-i18n:
  source_path: install\development-channels.md
  source_hash: 86d9e7131dff1307ef38f53b8bd8e0b6a0b567c8a9faa2ca195474baed1bbc9e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:07:23.433Z'
---

# Kênh phát triển

Cập nhật lần cuối: 2026-01-21

OpenClaw cung cấp ba kênh cập nhật:

- **stable**: npm dist-tag `latest`.
- **beta**: npm dist-tag `beta` (các bản dựng đang được kiểm tra).
- **dev**: đầu di động của `main` (git). npm dist-tag: `dev` (khi được xuất bản).

Chúng tôi gửi các bản dựng đến **beta**, kiểm tra chúng, sau đó **nâng cấp một bản dựng đã được xác minh lên `latest`**
mà không thay đổi số phiên bản — dist-tags là nguồn sự thật cho các cài đặt npm.
## Chuyển đổi kênh

Git checkout:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

- `stable`/`beta` check out the latest matching tag (often the same tag).
- `dev` switches to `main` and rebases on the upstream.

npm/pnpm global install:

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

This updates via the corresponding npm dist-tag (`latest`, `beta`, `dev`).

When you **explicitly** switch channels with `--channel`, OpenClaw also aligns
the install method:

- `dev` ensures a git checkout (default `~/openclaw`, override with `OPENCLAW_GIT_DIR`),
  updates it, and installs the global CLI from that checkout.
- `stable`/`beta` cài đặt từ npm bằng cách sử dụng dist-tag phù hợp.

Mẹo: nếu bạn muốn stable + dev song song, hãy giữ hai bản sao và trỏ Gateway của bạn đến bản sao stable.
## Plugins và kênh

Khi bạn chuyển đổi kênh với `openclaw update`, OpenClaw cũng đồng bộ hóa các nguồn plugin:

- `dev` ưu tiên các plugin được đóng gói từ git checkout.
- `stable` và `beta` khôi phục các gói plugin được cài đặt npm.
## Các phương pháp hay nhất cho gắn thẻ

- Gắn thẻ các bản phát hành mà bạn muốn git checkouts hạ cánh (`vYYYY.M.D` cho ổn định, `vYYYY.M.D-beta.N` cho beta).
- `vYYYY.M.D.beta.N` cũng được công nhận để tương thích, nhưng ưu tiên `-beta.N`.
- Các thẻ `vYYYY.M.D-<patch>` cũ vẫn được công nhận là ổn định (không phải beta).
- Giữ các thẻ không thay đổi: không bao giờ di chuyển hoặc tái sử dụng một thẻ.
- npm dist-tags vẫn là nguồn sự thật cho các cài đặt npm:
  - `latest` → ổn định
  - `beta` → bản dựng ứng cử viên
  - `dev` → ảnh chụp nhanh chính (tùy chọn)
## Tính khả dụng của ứng dụng macOS

Các bản beta và dev có thể **không** bao gồm bản phát hành ứng dụng macOS. Điều đó không sao:

- Git tag và npm dist-tag vẫn có thể được xuất bản.
- Ghi chú "không có bản dựng macOS cho beta này" trong ghi chú phát hành hoặc changelog.