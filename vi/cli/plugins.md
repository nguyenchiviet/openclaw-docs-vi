---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw plugins` (list, install, uninstall,
  enable/disable, doctor)
read_when:
  - Bạn muốn cài đặt hoặc quản lý các plugin Gateway trong quá trình xử lý
  - Bạn muốn gỡ lỗi các lỗi tải plugin
title: plugin
x-i18n:
  source_path: cli\plugins.md
  source_hash: b72bc4d177690ba02e3f409fe557345b43a1634af102fda5f49d32dd2ef5b4ba
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:06.474Z'
---

# `openclaw plugins`

Quản lý các plugin/extension của Gateway (được tải trong quá trình chạy).

Liên quan:

- Hệ thống plugin: [Plugins](/tools/plugin)
- Manifest plugin + schema: [Plugin manifest](/plugins/manifest)
- Tăng cường bảo mật: [Security](/gateway/security)
## Lệnh

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Bundled plugins ship with OpenClaw but start disabled. Use `plugins enable` to
activate them.

All plugins must ship a `openclaw.plugin.json` file with an inline JSON Schema
(`configSchema`, even if empty). Missing/invalid manifests or schemas prevent
the plugin from loading and fail config validation.

### Install

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

Security note: treat plugin installs like running code. Prefer pinned versions.

Npm specs are **registry-only** (package name + optional version/tag). Git/URL/file
specs are rejected. Dependency installs run with `--ignore-scripts` for safety.

Supported archives: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Use `--link` to avoid copying a local directory (adds to `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

Use `--pin` on npm installs to save the resolved exact spec (`name@version`) in
`plugins.installs` while keeping the default behavior unpinned.

### Uninstall

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` removes plugin records from `plugins.entries`, `plugins.installs`,
the plugin allowlist, and linked `plugins.load.paths` entries when applicable.
For active memory plugins, the memory slot resets to `memory-core`.

By default, uninstall also removes the plugin install directory under the active
state dir extensions root (`$OPENCLAW_STATE_DIR/extensions/<id>`). Use
`--keep-files` to keep files on disk.

`--keep-config` is supported as a deprecated alias for `--keep-files`.
### Cập nhật

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Updates only apply to plugins installed from npm (tracked in `plugins.installs`).

When a stored integrity hash exists and the fetched artifact hash changes,
OpenClaw prints a warning and asks for confirmation before proceeding. Use
global `--yes` để bỏ qua các lời nhắc trong các lần chạy CI/không tương tác.