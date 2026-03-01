---
summary: >-
  Tài liệu tham khảo CLI cho `openclaw security` (kiểm tra và sửa các lỗ hổng
  bảo mật phổ biến)
read_when:
  - Bạn muốn chạy một cuộc kiểm toán bảo mật nhanh trên config/state
  - >-
    Bạn muốn áp dụng các gợi ý "sửa chữa" an toàn (chmod, siết chặt các cài đặt
    mặc định)
title: bảo mật
x-i18n:
  source_path: cli\security.md
  source_hash: 0f3a5c6f9847962056fd68c3fe4aa49d8613734e32ac6d7a82a61163b4748fee
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:23.747Z'
---

# `openclaw security`

Công cụ bảo mật (kiểm toán + sửa chữa tùy chọn).

Liên quan:

- Hướng dẫn bảo mật: [Security](/gateway/security)
## Kiểm toán

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

The audit warns when multiple DM senders share the main session and recommends **secure DM mode**: `session.dmScope="per-channel-peer"` (or `per-account-channel-peer` for multi-account channels) for shared inboxes.
This is for cooperative/shared inbox hardening. A single Gateway shared by mutually untrusted/adversarial operators is not a recommended setup; split trust boundaries with separate gateways (or separate OS users/hosts).
It also emits `security.trust_model.multi_user_heuristic` when config suggests likely shared-user ingress (for example open DM/group policy, configured group targets, or wildcard sender rules), and reminds you that OpenClaw is a personal-assistant trust model by default.
For intentional shared-user setups, the audit guidance is to sandbox all sessions, keep filesystem access workspace-scoped, and keep personal/private identities or credentials off that runtime.
It also warns when small models (`<=300B`) are used without sandboxing and with web/browser tools enabled.
For webhook ingress, it warns when `hooks.defaultSessionKey` is unset, when request `sessionKey` overrides are enabled, and when overrides are enabled without `hooks.allowedSessionKeyPrefixes`.
It also warns when sandbox Docker settings are configured while sandbox mode is off, when `gateway.nodes.denyCommands` uses ineffective pattern-like/unknown entries (exact node command-name matching only, not shell-text filtering), when `gateway.nodes.allowCommands` explicitly enables dangerous node commands, when global `tools.profile="minimal"` is overridden by agent tool profiles, when open groups expose runtime/filesystem tools without sandbox/workspace guards, and when installed extension plugin tools may be reachable under permissive tool policy.
It also flags `gateway.allowRealIpFallback=true` (header-spoofing risk if proxies are misconfigured) and `discovery.mdns.mode="full"` (metadata leakage via mDNS TXT records).
It also warns when sandbox browser uses Docker `bridge` network without `sandbox.browser.cdpSourceRange`.
It also flags dangerous sandbox Docker network modes (including `host` and `container:*` namespace joins).
It also warns when existing sandbox browser Docker containers have missing/stale hash labels (for example pre-migration containers missing `openclaw.browserConfigEpoch`) and recommends `openclaw sandbox recreate --browser --all`.
It also warns when npm-based plugin/hook install records are unpinned, missing integrity metadata, or drift from currently installed package versions.
It warns when channel allowlists rely on mutable names/emails/tags instead of stable IDs (Discord, Slack, Google Chat, MS Teams, Mattermost, IRC scopes where applicable).
It warns when `gateway.auth.mode="none"` leaves Gateway HTTP APIs reachable without a shared secret (`/tools/invoke` plus any enabled `/v1/*` endpoint).
Settings prefixed with `dangerous`/`dangerously` là các ghi đè toán tử break-glass rõ ràng; bật một trong số chúng không phải là, bản thân nó, một báo cáo lỗ hổng bảo mật.
Để xem danh sách đầy đủ các tham số nguy hiểm, hãy xem phần "Tóm tắt các cờ không an toàn hoặc nguy hiểm" trong [Bảo mật](/gateway/security).
## Đầu ra JSON

Sử dụng `--json` cho các kiểm tra CI/chính sách:

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

If `--fix` and `--json` are combined, output includes both fix actions and final report:

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```
## `--fix` thay đổi những gì

`--fix` áp dụng các biện pháp khắc phục an toàn, xác định:

- đảo ngược các `groupPolicy="open"` phổ biến thành `groupPolicy="allowlist"` (bao gồm các biến thể tài khoản trong các kênh được hỗ trợ)
- đặt `logging.redactSensitive` từ `"off"` thành `"tools"`
- siết chặt quyền cho trạng thái/cấu hình và các tệp nhạy cảm phổ biến (`credentials/*.json`, `auth-profiles.json`, `sessions.json`, phiên `*.jsonl`)

`--fix` **không**:

- xoay vòng token/mật khẩu/khóa API
- vô hiệu hóa công cụ (`gateway`, `cron`, `exec`, v.v.)
- thay đổi các lựa chọn liên kết/xác thực/tiếp xúc mạng của gateway
- xóa hoặc viết lại plugin/Skills