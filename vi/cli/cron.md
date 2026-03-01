---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw cron` (lên lịch và chạy các công việc
  nền)
read_when:
  - Bạn muốn các công việc được lên lịch và đánh thức
  - Bạn đang gỡ lỗi thực thi cron và nhật ký
title: cron
x-i18n:
  source_path: cli\cron.md
  source_hash: 2aaf3baaaf433349df3b553c8528bbc3f22eda7de470a20423abda2b80ce2243
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:24.238Z'
---

# `openclaw cron`

Quản lý cron job cho bộ lập lịch Gateway.

Liên quan:

- Cron jobs: [Cron jobs](/automation/cron-jobs)

Mẹo: chạy `openclaw cron --help` để xem toàn bộ lệnh.

Lưu ý: cron job `cron add` được cô lập mặc định sử dụng `--announce` delivery. Sử dụng `--no-deliver` để giữ
output nội bộ. `--deliver` vẫn là bí danh không dùng nữa cho `--announce`.

Lưu ý: cron job một lần (`--at`) xóa sau khi thành công theo mặc định. Sử dụng `--keep-after-run` để giữ chúng.

Lưu ý: cron job định kỳ hiện sử dụng exponential retry backoff sau các lỗi liên tiếp (30s → 1m → 5m → 15m → 60m), sau đó quay lại lịch trình bình thường sau lần chạy thành công tiếp theo.

Lưu ý: retention/pruning được kiểm soát trong config:

- `cron.sessionRetention` (mặc định `24h`) xóa các phiên chạy cô lập đã hoàn thành.
- `cron.runLog.maxBytes` + `cron.runLog.keepLines` xóa `~/.openclaw/cron/runs/<jobId>.jsonl`.

## Chỉnh sửa phổ biến

Cập nhật cài đặt delivery mà không thay đổi tin nhắn:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

Disable delivery for an isolated job:

```bash
openclaw cron edit <job-id> --no-deliver
```

Announce to a specific channel:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```