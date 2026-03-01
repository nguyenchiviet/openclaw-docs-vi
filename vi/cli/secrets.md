---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw secrets` (reload, audit, configure,
  apply)
read_when:
  - Giải quyết lại các tham chiếu bí mật tại thời gian chạy
  - >-
    Kiểm tra các phần dư dạng văn bản thuần túy và các tham chiếu chưa được giải
    quyết
  - Cấu hình SecretRefs và áp dụng các thay đổi scrub một chiều
title: bí mật
x-i18n:
  source_path: cli\secrets.md
  source_hash: f12140702d25bd4dd17582bd1b6c00e065ecf95e21038dd2caf4828cfd4b6071
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:31.699Z'
---

# `openclaw secrets`

Sử dụng `openclaw secrets` để di chuyển thông tin xác thực từ văn bản thuần túy sang SecretRefs và giữ cho thời gian chạy bí mật hoạt động lành mạnh.

Vai trò lệnh:

- `reload`: gateway RPC (`secrets.reload`) giải quyết lại các tham chiếu và hoán đổi ảnh chụp thời gian chạy chỉ khi thành công hoàn toàn (không ghi cấu hình).
- `audit`: quét chỉ đọc của cấu hình + cửa hàng xác thực + phần dư cũ (`.env`, `auth.json`) để tìm văn bản thuần túy, tham chiếu chưa được giải quyết và độ trôi ưu tiên.
- `configure`: trình lập kế hoạch tương tác cho thiết lập nhà cung cấp + ánh xạ mục tiêu + kiểm tra trước (yêu cầu TTY).
- `apply`: thực thi một kế hoạch đã lưu (`--dry-run` chỉ để xác thực), sau đó xóa sạch phần dư văn bản thuần túy đã di chuyển.

Vòng lặp toán tử được đề xuất:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

Exit code note for CI/gates:

- `audit --check` returns `1` on findings, `2` khi các tham chiếu chưa được giải quyết.

Liên quan:

- Hướng dẫn bí mật: [Quản lý bí mật](/gateway/secrets)
- Hướng dẫn bảo mật: [Bảo mật](/gateway/security)
## Tải lại ảnh chụp runtime

Giải quyết lại các tham chiếu bí mật và hoán đổi ảnh chụp runtime một cách nguyên tử.

```bash
openclaw secrets reload
openclaw secrets reload --json
```

Notes:

- Uses gateway RPC method `secrets.reload`.
- If resolution fails, gateway keeps last-known-good snapshot and returns an error (no partial activation).
- JSON response includes `warningCount`.
## Kiểm toán

Quét trạng thái OpenClaw để tìm:

- lưu trữ bí mật dưới dạng văn bản thuần
- các tham chiếu chưa được giải quyết
- độ trôi ưu tiên (`auth-profiles` che phủ các tham chiếu cấu hình)
- các dư lượng kế thừa (`auth.json`, ghi chú OAuth ngoài phạm vi)

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

Exit behavior:

- `--check` exits non-zero on findings.
- unresolved refs exit with a higher-priority non-zero code.

Report shape highlights:

- `status`: `clean | findings | unresolved`
- `summary`: `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
- finding codes:
  - `PLAINTEXT_FOUND`
  - `REF_UNRESOLVED`
  - `REF_SHADOWED`
  - `LEGACY_RESIDUE`
## Cấu hình (trình hướng dẫn tương tác)

Xây dựng các thay đổi provider + SecretRef một cách tương tác, chạy kiểm tra trước, và tùy chọn áp dụng:

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --json
```

Flow:

- Provider setup first (`add/edit/remove` for `secrets.providers` aliases).
- Credential mapping second (select fields and assign `{source, provider, id}` refs).
- Preflight and optional apply last.

Flags:

- `--providers-only`: configure `secrets.providers` only, skip credential mapping.
- `--skip-provider-setup`: skip provider setup and map credentials to existing providers.

Notes:

- Requires an interactive TTY.
- You cannot combine `--providers-only` with `--skip-provider-setup`.
- `configure` targets secret-bearing fields in `openclaw.json`.
- Include all secret-bearing fields you intend to migrate (for example both `models.providers.*.apiKey` and `skills.entries.*.apiKey`) so audit can reach a clean state.
- It performs preflight resolution before apply.
- Generated plans default to scrub options (`scrubEnv`, `scrubAuthProfilesForProviderTargets`, `scrubLegacyAuthJson` all enabled).
- Apply path is one-way for migrated plaintext values.
- Without `--apply`, CLI still prompts `Apply this plan now?` after preflight.
- With `--apply` (and no `--yes`), CLI prompts an extra irreversible-migration confirmation.

Exec provider safety note:

- Homebrew installs often expose symlinked binaries under `/opt/homebrew/bin/*`.
- Set `allowSymlinkCommand: true` only when needed for trusted package-manager paths, and pair it with `trustedDirs` (for example `["/opt/homebrew"]`).
## Áp dụng một kế hoạch đã lưu

Áp dụng hoặc kiểm tra trước một kế hoạch được tạo trước đó:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

Plan contract details (allowed target paths, validation rules, and failure semantics):

- [Secrets Apply Plan Contract](/gateway/secrets-plan-contract)

What `apply` may update:

- `openclaw.json` (SecretRef targets + provider upserts/deletes)
- `auth-profiles.json` (provider-target scrubbing)
- legacy `auth.json` residues
- `~/.openclaw/.env` các khóa bí mật đã biết có giá trị được di chuyển
## Tại sao không có bản sao lưu rollback

`secrets apply` cố ý không ghi các bản sao lưu rollback chứa các giá trị plaintext cũ.

Tính an toàn đến từ kiểm tra preflight nghiêm ngặt + áp dụng atomic-ish với nỗ lực khôi phục trong bộ nhớ khi xảy ra lỗi.
## Ví dụ

```bash
# Audit first, then configure, then confirm clean:
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

If `audit --check` still reports plaintext findings after a partial migration, verify you also migrated skill keys (`skills.entries.*.apiKey`) và bất kỳ đường dẫn đích nào khác được báo cáo.