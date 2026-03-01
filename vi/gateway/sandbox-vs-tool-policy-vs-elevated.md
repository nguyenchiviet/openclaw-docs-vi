---
title: Sandbox vs Tool Policy vs Elevated
summary: >-
  Tại sao một công cụ bị chặn: sandbox runtime, tool allow/deny policy, và
  elevated exec gates
read_when: >-
  You hit 'sandbox jail' or see a tool/elevated refusal and want the exact
  config key to change.
status: active
x-i18n:
  source_path: gateway\sandbox-vs-tool-policy-vs-elevated.md
  source_hash: 863ea5e6d137dfb61f12bd686b9557d6df1fd0c13ba5f15861bf72248bc975f1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:07.198Z'
---

# Sandbox vs Tool Policy vs Elevated

OpenClaw có ba điều khiển liên quan (nhưng khác nhau):

1. **Sandbox** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) quyết định **nơi các công cụ chạy** (Docker vs host).
2. **Tool policy** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) quyết định **công cụ nào có sẵn/được phép**.
3. **Elevated** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) là một **cách thoát chỉ dành cho exec** để chạy trên host khi bạn đang ở trong sandbox.
## Gỡ lỗi nhanh

Sử dụng inspector để xem OpenClaw _thực sự_ đang làm gì:

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Nó in ra:

- chế độ sandbox hiệu quả/phạm vi/quyền truy cập không gian làm việc
- liệu phiên hiện tại có được sandboxed hay không (main vs non-main)
- công cụ sandbox hiệu quả cho phép/từ chối (và liệu nó có đến từ agent/global/default)
- các cổng nâng cao và đường dẫn khóa fix-it
## Sandbox: nơi các công cụ chạy

Sandboxing được kiểm soát bởi `agents.defaults.sandbox.mode`:

- `"off"`: mọi thứ chạy trên máy chủ.
- `"non-main"`: chỉ các phiên không phải phiên chính được sandboxed (trường hợp "bất ngờ" phổ biến cho nhóm/kênh).
- `"all"`: mọi thứ được sandboxed.

Xem [Sandboxing](/gateway/sandboxing) để xem ma trận đầy đủ (phạm vi, gắn kết không gian làm việc, hình ảnh).

### Bind mounts (kiểm tra bảo mật nhanh)

- `docker.binds` _xuyên qua_ hệ thống tệp sandbox: bất cứ điều gì bạn gắn kết đều hiển thị bên trong container với chế độ bạn đặt (`:ro` hoặc `:rw`).
- Mặc định là đọc-ghi nếu bạn bỏ qua chế độ; ưu tiên `:ro` cho nguồn/bí mật.
- `scope: "shared"` bỏ qua các bind theo agent (chỉ áp dụng các bind toàn cục).
- Gắn kết `/var/run/docker.sock` có hiệu lực trao quyền kiểm soát máy chủ cho sandbox; chỉ làm điều này có chủ ý.
- Truy cập không gian làm việc (`workspaceAccess: "ro"`/`"rw"`) độc lập với các chế độ bind.
## Chính sách công cụ: công cụ nào tồn tại/có thể gọi được

Hai lớp quan trọng:

- **Hồ sơ công cụ**: `tools.profile` và `agents.list[].tools.profile` (danh sách cho phép cơ bản)
- **Hồ sơ công cụ nhà cung cấp**: `tools.byProvider[provider].profile` và `agents.list[].tools.byProvider[provider].profile`
- **Chính sách công cụ toàn cầu/trên mỗi agent**: `tools.allow`/`tools.deny` và `agents.list[].tools.allow`/`agents.list[].tools.deny`
- **Chính sách công cụ nhà cung cấp**: `tools.byProvider[provider].allow/deny` và `agents.list[].tools.byProvider[provider].allow/deny`
- **Chính sách công cụ sandbox** (chỉ áp dụng khi được sandboxed): `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` và `agents.list[].tools.sandbox.tools.*`

Quy tắc chung:

- `deny` luôn thắng.
- Nếu `allow` không trống, mọi thứ khác được coi là bị chặn.
- Chính sách công cụ là điểm dừng cứng: `/exec` không thể ghi đè công cụ `exec` bị từ chối.
- `/exec` chỉ thay đổi mặc định phiên cho những người gửi được phép; nó không cấp quyền truy cập công cụ.
  Các khóa công cụ nhà cung cấp chấp nhận `provider` (ví dụ `google-antigravity`) hoặc `provider/model` (ví dụ `openai/gpt-5.2`).

### Nhóm công cụ (viết tắt)

Chính sách công cụ (toàn cầu, agent, sandbox) hỗ trợ các mục `group:*` mở rộng thành nhiều công cụ:

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

Available groups:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: tất cả các công cụ OpenClaw tích hợp sẵn (không bao gồm plugin nhà cung cấp)
## Elevated: exec-only "run on host"

Elevated không cấp thêm công cụ; nó chỉ ảnh hưởng đến `exec`.

- Nếu bạn đang ở trong sandbox, `/elevated on` (hoặc `exec` với `elevated: true`) chạy trên host (phê duyệt vẫn có thể áp dụng).
- Sử dụng `/elevated full` để bỏ qua phê duyệt exec cho phiên.
- Nếu bạn đã chạy trực tiếp, elevated về cơ bản là không có tác dụng (vẫn bị kiểm soát).
- Elevated **không** được phạm vi kỹ năng và **không** ghi đè cho phép/từ chối công cụ.
- `/exec` tách biệt với elevated. Nó chỉ điều chỉnh các giá trị mặc định exec cho mỗi phiên cho những người gửi được phép.

Cổng:

- Kích hoạt: `tools.elevated.enabled` (và tùy chọn `agents.list[].tools.elevated.enabled`)
- Danh sách cho phép người gửi: `tools.elevated.allowFrom.<provider>` (và tùy chọn `agents.list[].tools.elevated.allowFrom.<provider>`)

Xem [Chế độ Elevated](/tools/elevated).
## Các cách khắc phục "sandbox jail" phổ biến

### "Tool X bị chặn bởi chính sách công cụ sandbox"

Các khóa khắc phục (chọn một):

- Vô hiệu hóa sandbox: `agents.defaults.sandbox.mode=off` (hoặc theo agent `agents.list[].sandbox.mode=off`)
- Cho phép công cụ bên trong sandbox:
  - xóa nó khỏi `tools.sandbox.tools.deny` (hoặc theo agent `agents.list[].tools.sandbox.tools.deny`)
  - hoặc thêm nó vào `tools.sandbox.tools.allow` (hoặc cho phép theo agent)

### "Tôi nghĩ đây là main, tại sao nó lại được sandboxed?"

Ở chế độ `"non-main"`, các khóa group/channel _không phải_ main. Sử dụng khóa phiên main (được hiển thị bởi `sandbox explain`) hoặc chuyển chế độ sang `"off"`.