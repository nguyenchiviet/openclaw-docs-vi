---
summary: >-
  Sổ tay khắc phục sự cố chuyên sâu cho Gateway, kênh, tự động hóa, nút và trình
  duyệt
read_when:
  - Trung tâm khắc phục sự cố đã hướng dẫn bạn đến đây để chẩn đoán sâu hơn.
  - >-
    Bạn cần các phần runbook ổn định dựa trên triệu chứng với các lệnh chính
    xác.
title: Khắc phục sự cố
x-i18n:
  source_path: gateway\troubleshooting.md
  source_hash: 5cab2ea22a0a13baf1f0b907a1ac018a0d216eca08344906c463c433348248cf
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:16:00.960Z'
---

# Khắc phục sự cố Gateway

Trang này là sổ tay vận hành chuyên sâu.
Hãy bắt đầu tại [/help/troubleshooting](__OC_I19N_0000__) nếu bạn muốn quy trình phân loại nhanh trước.

## Thứ tự lệnh

Chạy các lệnh này trước, theo thứ tự sau:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Expected healthy signals:

- `openclaw gateway status` shows `Thời gian chạy: đang chạy` and `Kiểm tra RPC: OK`.
- `openclaw doctor` reports no blocking config/service issues.
- `openclaw channels status --probe` hiển thị các kênh đã kết nối/sẵn sàng.
## Không có phản hồi

Nếu các kênh đã hoạt động nhưng không có gì trả lời, hãy kiểm tra định tuyến và chính sách trước khi kết nối lại bất cứ thứ gì.

``__OC_I19N_0000__`__OC_I19N_0001__requireMention__OC_I19N_0002__mentionPatterns__OC_I19N_0003__drop guild message (mention required__OC_I19N_0004__pairing request__OC_I19N_0005__blocked__OC_I19N_0006__allowlist` → người gửi/kênh đã bị lọc bởi chính sách.

Liên quan:

- [/channels/troubleshooting](__OC_I19N_0007__)
- [/channels/pairing](__OC_I19N_0008__)
- [/channels/groups](__OC_I19N_0009__)

## Kết nối giao diện người dùng điều khiển bảng điều khiển

Khi giao diện người dùng bảng điều khiển/điều khiển không kết nối được, hãy xác thực URL, chế độ xác thực và các giả định về ngữ cảnh bảo mật.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Look for:

- Correct probe URL and dashboard URL.
- Auth mode/token mismatch between client and gateway.
- HTTP usage where device identity is required.

Common signatures:

- `cần có định danh thiết bị` → non-secure context or missing device auth.
- `cần có nonce thiết bị` / `nonce thiết bị không khớp` → client is not completing the
  challenge-based device auth flow (`kết nối.thử thách` + `thiết bị.nonce`).
- `chữ ký thiết bị không hợp lệ` / `chữ ký thiết bị đã hết hạn` → client signed the wrong
  payload (or stale timestamp) for the current handshake.
- `không được ủy quyền` / reconnect loop → token/password mismatch.
- `kết nối Gateway thất bại:` → sai máy chủ/cổng/mục tiêu URL.

Kiểm tra di chuyển xác thực thiết bị v2:
`bash
openclaw --version
openclaw doctor
openclaw gateway status
```

If logs show nonce/signature errors, update the connecting client and verify it:

1. waits for `connect.challenge`
2. signs the challenge-bound payload
3. sends `connect.params.device.nonce` với cùng một nonce thử thách

Liên quan:

- [/web/giao-dien-dieu-khien](/web/control-ui)
- [/gateway/xac-thuc](/gateway/authentication)
- [/gateway/tu-xa](/gateway/remote)
## Dịch vụ Gateway không chạy

Sử dụng điều này khi dịch vụ đã được cài đặt nhưng tiến trình không duy trì hoạt động.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Look for:

- `Thời gian chạy: đã dừng` with exit hints.
- Service config mismatch (`Cấu hình (CLI)` vs `Cấu hình (dịch vụ)`).
- Port/listener conflicts.

Common signatures:

- `Khởi động Gateway bị chặn: đặt gateway.mode=local` → local gateway mode is not enabled. Fix: set `gateway.mode="local"` in your config (or run `openclaw configure`). If you are running OpenClaw via Podman using the dedicated `openclaw` user, the config lives at `~openclaw/.openclaw/openclaw.json`.
- `từ chối liên kết gateway ... không có xác thực` → non-loopback bind without token/password.
- `một phiên bản gateway khác đã đang lắng nghe` / `EADDRINUSE` → xung đột cổng.

Liên quan:

- [/gateway/background-process](/gateway/background-process)
- [/gateway/configuration](/gateway/configuration)
- [/gateway/doctor](/gateway/doctor)

## Tin nhắn kênh đã kết nối không truyền tải

Nếu trạng thái kênh đã kết nối nhưng luồng tin nhắn bị ngưng trệ, hãy tập trung vào chính sách, quyền và các quy tắc phân phối cụ thể của kênh.

``bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Look for:

- DM policy (`ghép nối`, `danh sách cho phép`, `mở`, `bị vô hiệu hóa`).
- Group allowlist and mention requirements.
- Missing channel API permissions/scopes.

Common signatures:

- `yêu cầu nhắc đến` → message ignored by group mention policy.
- `ghép nối` / pending approval traces → sender is not approved.
- `thiếu phạm vi`, `không có trong kênh`, `Bị cấm`, `401/403` → vấn đề về xác thực/quyền của kênh.

Liên quan:

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/whatsapp](/channels/whatsapp)
- [/channels/telegram](/channels/telegram)
- [/channels/discord](/channels/discord)
## Phân phối Cron và heartbeat

Nếu cron hoặc heartbeat không chạy hoặc không được phân phối, trước tiên hãy xác minh trạng thái bộ lập lịch, sau đó là đích phân phối.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Look for:

- Cron enabled and next wake present.
- Job run history status (`ok`, `skipped`, `error`).
- Heartbeat skip reasons (`quiet-hours__OC_I19N_0005__requests-in-flight`, `alerts-disabled`).

Common signatures:

- `cron: scheduler disabled; jobs will not run automatically` → cron disabled.
- `cron: timer tick failed` → scheduler tick failed; check file/log/runtime errors.
- `heartbeat skipped` with `reason=quiet-hours` → outside active hours window.
- `heartbeat: unknown accountId` → invalid account id for heartbeat delivery target.
- `heartbeat skipped` with `reason=dm-blocked` → heartbeat target resolved to a DM-style destination while `agents.defaults.heartbeat.directPolicy` (or per-agent override) is set to `block`.

Liên quan:

- [/automation/troubleshooting](/automation/troubleshooting)
- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)

## Công cụ được ghép nối với node bị lỗi

Nếu một node đã được ghép nối nhưng các công cụ bị lỗi, hãy cô lập trạng thái nền trước, quyền và phê duyệt.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Look for:

- Node online with expected capabilities.
- OS permission grants for camera/mic/location/screen.
- Exec approvals and allowlist state.

Common signatures:

- `NODE_BACKGROUND_UNAVAILABLE__OC_I19N_0002__*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → missing OS permission.
- `SYSTEM_RUN_DENIED: approval required` → exec approval pending.
- `SYSTEM_RUN_DENIED: allowlist miss` → lệnh bị chặn bởi danh sách cho phép.

Liên quan:

- [/nodes/khắc-phục-sự-cố](__OC_I19N_0000__)
- [/nodes/tổng-quan](__OC_I19N_0001__)
- [/tools/phê-duyệt-thực-thi](__OC_I19N_0002__)
## Công cụ trình duyệt bị lỗi

Sử dụng hướng dẫn này khi các hành động của công cụ trình duyệt bị lỗi mặc dù Gateway vẫn hoạt động bình thường.

``__OC_I19N_0000__`__OC_I19N_0001__profile="chrome"__OC_I19N_0002__Không thể khởi động Chrome CDP trên cổng__OC_I19N_0003__Không tìm thấy browser.executablePath__OC_I19N_0004__Chuyển tiếp tiện ích mở rộng Chrome đang chạy, nhưng không có tab nào được kết nối__OC_I19N_0005__Chế độ Browser attachOnly được bật ... không thể truy cập` → hồ sơ chỉ đính kèm không có mục tiêu có thể truy cập.

Liên quan:

- [/tools/browser-linux-troubleshooting](__OC_I19N_0006__)
- [/tools/chrome-extension](__OC_I19N_0007__)
- [/tools/browser](__OC_I19N_0008__)
## Nếu bạn nâng cấp và có gì đó đột ngột bị lỗi

Hầu hết các lỗi sau nâng cấp là do cấu hình bị lệch hoặc các giá trị mặc định nghiêm ngặt hơn hiện đang được áp dụng.

### 1) Hành vi ghi đè xác thực và URL đã thay đổi

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

What to check:

- If `gateway.mode=remote`, CLI calls may be targeting remote while your local service is fine.
- Explicit `--url__OC_I19N_0003__gateway connect failed:` → wrong URL target.
- `unauthorized` → endpoint reachable but wrong auth.

### 2) Bind and auth guardrails are stricter

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```
Những điều cần kiểm tra:

- Các liên kết không phải local loopback (`lan`, `tailnet`, `custom`) cần được cấu hình xác thực.
- Các khóa cũ như `gateway.token` không thay thế `gateway.auth.token`.

Các dấu hiệu phổ biến:

- `refusing to bind gateway ... without auth` → không khớp liên kết+xác thực.
- `RPC probe: failed` trong khi runtime đang chạy → gateway đang hoạt động nhưng không thể truy cập bằng xác thực/URL hiện tại.

### 3) Trạng thái ghép nối và nhận dạng thiết bị đã thay đổi

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

What to check:

- Pending device approvals for dashboard/nodes.
- Pending DM pairing approvals after policy or identity changes.

Common signatures:

- `yêu cầu nhận dạng thiết bị` → device auth not satisfied.
- `yêu cầu ghép nối` → người gửi/thiết bị phải được chấp thuận.

Nếu cấu hình dịch vụ và thời gian chạy vẫn không khớp sau khi kiểm tra, hãy cài đặt lại siêu dữ liệu dịch vụ từ cùng thư mục hồ sơ/trạng thái:

```bash
openclaw gateway install --force
openclaw gateway restart
```

Liên quan:

- [/gateway/pairing](/gateway/pairing)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/background-process](/gateway/background-process)