---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw status` (chẩn đoán, kiểm tra, ảnh chụp sử
  dụng)
read_when:
  - >-
    Bạn muốn chẩn đoán nhanh tình trạng kênh + những người nhận phiên làm việc
    gần đây
  - Bạn muốn một trạng thái "all" có thể dán được để gỡ lỗi
title: trạng thái
x-i18n:
  source_path: cli\status.md
  source_hash: 2bbf5579c48034fc15c2cbd5506c50456230b17e4a74c06318968c590d8f1501
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:38.598Z'
---

# `openclaw status`

Chẩn đoán cho các kênh + phiên.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes:

- `--deep` runs live probes (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
- Output includes per-agent session stores when multiple agents are configured.
- Overview includes Gateway + node host service install/runtime status when available.
- Overview includes update channel + git SHA (for source checkouts).
- Update info surfaces in the Overview; if an update is available, status prints a hint to run `openclaw update` (xem [Cập nhật](/install/updating)).