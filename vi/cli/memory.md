---
summary: Tài liệu tham khảo CLI cho `openclaw memory` (status/index/search)
read_when:
  - Bạn muốn lập chỉ mục hoặc tìm kiếm bộ nhớ ngữ nghĩa
  - Bạn đang gỡ lỗi tính khả dụng bộ nhớ hoặc lập chỉ mục
title: bộ nhớ
x-i18n:
  source_path: cli\memory.md
  source_hash: 2909de890a072ca58ecd25f45ba5ea108bd8ade0e193fd742f7668ee6368920d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:36:36.210Z'
---

# `openclaw memory`

Quản lý lập chỉ mục và tìm kiếm bộ nhớ ngữ nghĩa.
Được cung cấp bởi plugin bộ nhớ hoạt động (mặc định: `memory-core`; đặt `plugins.slots.memory = "none"` để vô hiệu hóa).

Liên quan:

- Khái niệm bộ nhớ: [Memory](/concepts/memory)
- Plugins: [Plugins](/tools/plugin)

## Ví dụ

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory index
openclaw memory index --verbose
openclaw memory search "release checklist"
openclaw memory search --query "release checklist"
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## Options

Common:

- `--agent <id>`: scope to a single agent (default: all configured agents).
- `--verbose`: emit detailed logs during probes and indexing.

`memory search`:

- Query input: pass either positional `[query]` or `--query <text>`.
- If both are provided, `--query` wins.
- If neither is provided, the command exits with an error.

Notes:

- `memory status --deep` probes vector + embedding availability.
- `memory status --deep --index` runs a reindex if the store is dirty.
- `memory index --verbose` prints per-phase details (provider, model, sources, batch activity).
- `memory status` includes any extra paths configured via `memorySearch.extraPaths`.