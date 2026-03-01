---
summary: Áp dụng các bản vá đa tệp với công cụ apply_patch
read_when:
  - Bạn cần chỉnh sửa tệp có cấu trúc trên nhiều tệp
  - Bạn muốn ghi chép hoặc gỡ lỗi các chỉnh sửa dựa trên patch
title: Công cụ apply_patch
x-i18n:
  source_path: tools\apply-patch.md
  source_hash: a1b251e8327228ff327eda8989626421edbe75cd1483acfc6a2f2fd31ed6cfc2
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:27:47.622Z'
---

# công cụ apply_patch

Áp dụng các thay đổi tệp bằng định dạng patch có cấu trúc. Điều này lý tưởng cho các chỉnh sửa đa tệp
hoặc đa hunk nơi một lệnh gọi `edit` duy nhất sẽ dễ bị lỗi.

Công cụ chấp nhận một chuỗi `input` duy nhất bao bọc một hoặc nhiều thao tác tệp:

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## Parameters

- `input` (required): Full patch contents including `*** Begin Patch` and `*** End Patch`.

## Notes

- Patch paths support relative paths (from the workspace directory) and absolute paths.
- `tools.exec.applyPatch.workspaceOnly` defaults to `true` (workspace-contained). Set it to `false` only if you intentionally want `apply_patch` to write/delete outside the workspace directory.
- Use `*** Move to:` within an `*** Update File:` hunk to rename files.
- `*** End of File` marks an EOF-only insert when needed.
- Experimental and disabled by default. Enable with `tools.exec.applyPatch.enabled`.
- OpenAI-only (including OpenAI Codex). Optionally gate by model via
  `tools.exec.applyPatch.allowModels`.
- Config is only under `tools.exec`.

## Example

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```