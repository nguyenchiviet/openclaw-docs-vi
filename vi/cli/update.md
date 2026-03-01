---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw update` (cập nhật nguồn an toàn + tự động
  khởi động lại Gateway)
read_when:
  - Bạn muốn cập nhật một source checkout một cách an toàn
  - Bạn cần hiểu hành vi viết tắt của `--update`
title: cập nhật
x-i18n:
  source_path: cli\update.md
  source_hash: f0ab6c4d21952fc637f91e245bcacfe99e18d79d2782076b3bfba35ba4fc4300
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:38:03.469Z'
---

# `openclaw update`

Cập nhật OpenClaw một cách an toàn và chuyển đổi giữa các kênh stable/beta/dev.

Nếu bạn cài đặt qua **npm/pnpm** (cài đặt toàn cục, không có siêu dữ liệu git), các bản cập nhật sẽ diễn ra thông qua quy trình trình quản lý gói trong [Cập nhật](/install/updating).
## Cách sử dụng

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```
## Tùy chọn

- `--no-restart`: bỏ qua việc khởi động lại dịch vụ Gateway sau khi cập nhật thành công.
- `--channel <stable|beta|dev>`: đặt kênh cập nhật (git + npm; được lưu trữ trong cấu hình).
- `--tag <dist-tag|version>`: ghi đè npm dist-tag hoặc phiên bản cho lần cập nhật này.
- `--dry-run`: xem trước các hành động cập nhật được lên kế hoạch (kênh/thẻ/mục tiêu/luồng khởi động lại) mà không cần ghi cấu hình, cài đặt, đồng bộ hóa plugin hoặc khởi động lại.
- `--json`: in `UpdateRunResult` JSON có thể đọc được bằng máy.
- `--timeout <seconds>`: thời gian chờ cho mỗi bước (mặc định là 1200s).

Lưu ý: các bản cập nhật xuống phiên bản cũ hơn yêu cầu xác nhận vì các phiên bản cũ hơn có thể làm hỏng cấu hình.
## `update status`

Hiển thị kênh cập nhật hoạt động + git tag/branch/SHA (cho các bản checkout từ nguồn), cộng với tính khả dụng cập nhật.

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

Options:

- `--json`: print machine-readable status JSON.
- `--timeout <seconds>``: hết thời gian chờ cho các kiểm tra (mặc định là 3 giây).
## `update wizard`

Luồng tương tác để chọn kênh cập nhật và xác nhận có nên khởi động lại Gateway
sau khi cập nhật (mặc định là khởi động lại). Nếu bạn chọn `dev` mà không có git checkout, nó
sẽ đề xuất tạo một cái.
## Chức năng của nó

Khi bạn chuyển kênh một cách rõ ràng (`--channel ...`), OpenClaw cũng giữ cho phương pháp cài đặt được căn chỉnh:

- `dev` → đảm bảo một git checkout (mặc định: `~/openclaw`, ghi đè bằng `OPENCLAW_GIT_DIR`),
  cập nhật nó và cài đặt CLI toàn cục từ checkout đó.
- `stable`/`beta` → cài đặt từ npm bằng cách sử dụng dist-tag phù hợp.

Bộ cập nhật tự động lõi Gateway (khi được bật qua cấu hình) sử dụng lại cùng một đường dẫn cập nhật này.
## Luồng Git checkout

Kênh:

- `stable`: checkout thẻ non-beta mới nhất, sau đó build + doctor.
- `beta`: checkout thẻ `-beta` mới nhất, sau đó build + doctor.
- `dev`: checkout `main`, sau đó fetch + rebase.

Cấp cao:

1. Yêu cầu worktree sạch (không có thay đổi chưa commit).
2. Chuyển sang kênh được chọn (thẻ hoặc nhánh).
3. Fetch upstream (dev only).
4. Dev only: preflight lint + TypeScript build trong worktree tạm thời; nếu tip không thành công, quay lại tối đa 10 commit để tìm build sạch mới nhất.
5. Rebase lên commit được chọn (dev only).
6. Cài đặt deps (pnpm ưu tiên; npm fallback).
7. Build + build Control UI.
8. Chạy `openclaw doctor` làm kiểm tra "safe update" cuối cùng.
9. Đồng bộ hóa plugins với kênh hoạt động (dev sử dụng bundled extensions; stable/beta sử dụng npm) và cập nhật plugins được cài đặt npm.
## Viết tắt `--update`

`openclaw --update` được viết lại thành `openclaw update` (hữu ích cho shell và script khởi chạy).
## Xem thêm

- `openclaw doctor` (cung cấp tùy chọn chạy cập nhật trước trên các bản sao git)
- [Các kênh phát triển](/install/development-channels)
- [Cập nhật](/install/updating)
- [Tham chiếu CLI](/cli)