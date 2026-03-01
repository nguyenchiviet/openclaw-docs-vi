---
summary: 'Cách sử dụng công cụ Exec, các chế độ stdin và hỗ trợ TTY'
read_when:
  - Sử dụng hoặc sửa đổi công cụ exec
  - Gỡ lỗi hành vi stdin hoặc TTY
title: Công cụ Exec
x-i18n:
  source_path: tools\exec.md
  source_hash: 474c337e5a1c262c7f717231b8e74ee3035fc37bd3288fe2cae5b9521dfbd361
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:30:54.335Z'
---

# Công cụ Exec

Chạy các lệnh shell trong không gian làm việc. Hỗ trợ thực thi nền trước + nền thông qua `process`.
Nếu `process` bị cấm, `exec` chạy đồng bộ và bỏ qua `yieldMs`/`background`.
Các phiên nền được phân loại theo agent; `process` chỉ nhìn thấy các phiên từ cùng một agent.
## Tham số

- `command` (bắt buộc)
- `workdir` (mặc định là cwd)
- `env` (ghi đè khóa/giá trị)
- `yieldMs` (mặc định 10000): tự động chạy nền sau khoảng thời gian
- `background` (bool): chạy nền ngay lập tức
- `timeout` (giây, mặc định 1800): dừng khi hết hạn
- `pty` (bool): chạy trong pseudo-terminal khi có sẵn (CLI chỉ TTY, agent mã hóa, giao diện terminal)
- `host` (`sandbox | gateway | node`): nơi thực thi
- `security` (`deny | allowlist | full`): chế độ thực thi cho `gateway`/`node`
- `ask` (`off | on-miss | always`): lời nhắc phê duyệt cho `gateway`/`node`
- `node` (string): id/tên node cho `host=node`
- `elevated` (bool): yêu cầu chế độ nâng cao (gateway host); `security=full` chỉ được buộc khi nâng cao phân giải thành `full`

Ghi chú:

- `host` mặc định là `sandbox`.
- `elevated` bị bỏ qua khi sandboxing tắt (exec đã chạy trên host).
- `gateway`/`node` phê duyệt được kiểm soát bởi `~/.openclaw/exec-approvals.json`.
- `node` yêu cầu một node được ghép nối (ứng dụng đi kèm hoặc host node headless).
- Nếu có nhiều node khả dụng, hãy đặt `exec.node` hoặc `tools.exec.node` để chọn một.
- Trên host không phải Windows, exec sử dụng `SHELL` khi được đặt; nếu `SHELL` là `fish`, nó ưu tiên `bash` (hoặc `sh`)
  từ `PATH` để tránh script không tương thích với fish, sau đó quay lại `SHELL` nếu không có cái nào tồn tại.
- Trên host Windows, exec ưu tiên PowerShell 7 (`pwsh`) discovery (Program Files, ProgramW6432, sau đó PATH),
  sau đó quay lại Windows PowerShell 5.1.
- Thực thi host (`gateway`/`node`) từ chối `env.PATH` và ghi đè loader (`LD_*`/`DYLD_*`) để
  ngăn chặn binary hijacking hoặc code được tiêm.
- Quan trọng: sandboxing **tắt theo mặc định**. Nếu sandboxing tắt và `host=sandbox` được cấu hình/yêu cầu rõ ràng,
  exec giờ đây thất bại đóng thay vì im lặng chạy trên gateway host.
  Bật sandboxing hoặc sử dụng `host=gateway` với phê duyệt.
- Kiểm tra preflight script (cho các lỗi cú pháp shell Python/Node phổ biến) chỉ kiểm tra các tệp bên trong
  ranh giới `workdir` hiệu lực. Nếu đường dẫn script phân giải bên ngoài `workdir`, preflight bị bỏ qua cho
  tệp đó.
## Cấu hình

- `tools.exec.notifyOnExit` (mặc định: true): khi bật, các phiên exec chạy nền sẽ xếp hàng một sự kiện hệ thống và yêu cầu heartbeat khi thoát.
- `tools.exec.approvalRunningNoticeMs` (mặc định: 10000): phát ra một thông báo "running" duy nhất khi một exec được gated bởi phê duyệt chạy lâu hơn giá trị này (0 để tắt).
- `tools.exec.host` (mặc định: `sandbox`)
- `tools.exec.security` (mặc định: `deny` cho sandbox, `allowlist` cho gateway + node khi không được đặt)
- `tools.exec.ask` (mặc định: `on-miss`)
- `tools.exec.node` (mặc định: không được đặt)
- `tools.exec.pathPrepend`: danh sách các thư mục để thêm vào đầu `PATH` cho các lần chạy exec (chỉ gateway + sandbox).
- `tools.exec.safeBins`: các tệp nhị phân an toàn chỉ stdin có thể chạy mà không cần các mục danh sách cho phép rõ ràng. Để biết chi tiết về hành vi, xem [Safe bins](/tools/exec-approvals#safe-bins-stdin-only).
- `tools.exec.safeBinTrustedDirs`: các thư mục rõ ràng bổ sung được tin tưởng cho các kiểm tra đường dẫn `safeBins`. Các mục `PATH` không bao giờ được tự động tin tưởng. Các giá trị mặc định tích hợp là `/bin` và `/usr/bin`.
- `tools.exec.safeBinProfiles`: chính sách argv tùy chỉnh tùy chọn cho mỗi safe bin (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

Ví dụ:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH handling

- `host=gateway`: merges your login-shell `PATH` into the exec environment. `env.PATH` overrides are
  rejected for host execution. The daemon itself still runs with a minimal `PATH`:
  - macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  - Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
- `host=sandbox`: runs `sh -lc` (login shell) inside the container, so `/etc/profile` may reset `PATH`.
  OpenClaw prepends `env.PATH` after profile sourcing via an internal env var (no shell interpolation);
  `tools.exec.pathPrepend` applies here too.
- `host=node`: only non-blocked env overrides you pass are sent to the node. `env.PATH` overrides are
  rejected for host execution and ignored by node hosts. If you need additional PATH entries on a node,
  configure the node host service environment (systemd/launchd) or install tools in standard locations.

Per-agent node binding (use the agent list index in config):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Giao diện điều khiển: tab Nodes bao gồm một bảng điều khiển nhỏ "Exec node binding" cho các cài đặt tương tự.
## Ghi đè phiên (`/exec`)

Sử dụng `/exec` để đặt các giá trị mặc định **cho mỗi phiên** cho `host`, `security`, `ask`, và `node`.
Gửi `/exec` mà không có đối số để hiển thị các giá trị hiện tại.

Ví dụ:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```
## Mô hình ủy quyền

`/exec` chỉ được áp dụng cho **những người gửi được phép** (danh sách cho phép kênh/ghép nối cộng với `commands.useAccessGroups`).
Nó cập nhật **trạng thái phiên chỉ** và không ghi cấu hình. Để vô hiệu hóa exec cứng, từ chối nó thông qua chính sách công cụ (`tools.deny: ["exec"]` hoặc mỗi agent). Phê duyệt máy chủ vẫn áp dụng trừ khi bạn rõ ràng đặt `security=full` và `ask=off`.
## Phê duyệt Exec (ứng dụng đi kèm / node host)

Các agent sandbox có thể yêu cầu phê duyệt cho mỗi yêu cầu trước khi `exec` chạy trên gateway hoặc node host.
Xem [Phê duyệt Exec](/tools/exec-approvals) để biết chính sách, danh sách cho phép và luồng giao diện người dùng.

Khi cần phê duyệt, công cụ exec trả về ngay lập tức với
`status: "approval-pending"` và một id phê duyệt. Sau khi được phê duyệt (hoặc từ chối / hết thời gian chờ),
Gateway phát hành các sự kiện hệ thống (`Exec finished` / `Exec denied`). Nếu lệnh vẫn
đang chạy sau `tools.exec.approvalRunningNoticeMs`, một thông báo `Exec running` duy nhất sẽ được phát hành.
## Danh sách cho phép + safe bins

Thực thi danh sách cho phép thủ công khớp **chỉ các đường dẫn nhị phân đã giải quyết** (không khớp tên cơ sở). Khi
`security=allowlist`, các lệnh shell được tự động cho phép chỉ khi mọi đoạn đường ống được
liệt kê trong danh sách cho phép hoặc là safe bin. Chuỗi (`;`, `&&`, `||`) và chuyển hướng bị từ chối ở
chế độ danh sách cho phép trừ khi mọi đoạn cấp cao nhất thỏa mãn danh sách cho phép (bao gồm safe bins).
Chuyển hướng vẫn không được hỗ trợ.

`autoAllowSkills` là một đường dẫn tiện lợi riêng biệt trong phê duyệt exec. Nó không giống với
các mục danh sách cho phép đường dẫn thủ công. Để tin tưởng rõ ràng và nghiêm ngặt, hãy giữ `autoAllowSkills` ở trạng thái tắt.

Sử dụng hai điều khiển cho các công việc khác nhau:

- `tools.exec.safeBins`: bộ lọc luồng nhỏ, chỉ stdin.
- `tools.exec.safeBinTrustedDirs`: các thư mục đáng tin cậy bổ sung rõ ràng cho các đường dẫn thực thi safe-bin.
- `tools.exec.safeBinProfiles`: chính sách argv rõ ràng cho các safe bins tùy chỉnh.
- danh sách cho phép: tin tưởng rõ ràng cho các đường dẫn thực thi.

Không coi `safeBins` là danh sách cho phép chung, và không thêm các nhị phân trình thông dịch/thời gian chạy (ví dụ `python3`, `node`, `ruby`, `bash`). Nếu bạn cần những cái đó, hãy sử dụng các mục danh sách cho phép rõ ràng và giữ các lời nhắc phê duyệt ở trạng thái bật.
`openclaw security audit` cảnh báo khi các mục `safeBins` trình thông dịch/thời gian chạy thiếu các hồ sơ rõ ràng, và `openclaw doctor --fix` có thể tạo các mục `safeBinProfiles` tùy chỉnh bị thiếu.

Để biết chi tiết chính sách đầy đủ và ví dụ, hãy xem [Phê duyệt exec](/tools/exec-approvals#safe-bins-stdin-only) và [Safe bins so với danh sách cho phép](/tools/exec-approvals#safe-bins-versus-allowlist).
## Ví dụ

Nền trước:

```json
{ "tool": "exec", "command": "ls -la" }
```

Background + poll:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Send keys (tmux-style):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Submit (send CR only):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

Paste (bracketed by default):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```
## apply_patch (thử nghiệm)

`apply_patch` là một công cụ phụ của `exec` để chỉnh sửa nhiều tệp có cấu trúc.
Bật nó một cách rõ ràng:

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

Notes:

- Only available for OpenAI/OpenAI Codex models.
- Tool policy still applies; `allow: ["exec"]` implicitly allows `apply_patch`.
- Config lives under `tools.exec.applyPatch`.
- `tools.exec.applyPatch.workspaceOnly` defaults to `true` (workspace-contained). Set it to `false` only if you intentionally want `apply_patch` để ghi/xóa bên ngoài thư mục workspace.