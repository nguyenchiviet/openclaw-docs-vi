---
summary: >-
  Cách hoạt động của sandboxing trong OpenClaw: các chế độ, phạm vi, quyền truy
  cập không gian làm việc và hình ảnh
title: Cách ly (Sandboxing)
read_when: >-
  You want a dedicated explanation of sandboxing or need to tune
  agents.defaults.sandbox.
status: active
x-i18n:
  source_path: gateway\sandboxing.md
  source_hash: 62713697eb36c4ce1d664f818e1c12f06fe86d8321eb8d24a6e5e6e78cda464a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:59:17.156Z'
---

# Sandboxing

OpenClaw có thể chạy **các công cụ bên trong các container Docker** để giảm phạm vi ảnh hưởng.
Đây là **tùy chọn** và được kiểm soát bằng cấu hình (`agents.defaults.sandbox` hoặc
`agents.list[].sandbox`). Nếu sandboxing bị tắt, các công cụ sẽ chạy trên máy chủ.
Gateway ở trên máy chủ; thực thi công cụ chạy trong một sandbox cách ly
khi được bật.

Đây không phải là một ranh giới bảo mật hoàn hảo, nhưng nó giới hạn đáng kể quyền truy cập vào hệ thống tệp
và quy trình khi mô hình làm điều gì đó không thông minh.
## Những gì được sandboxing

- Thực thi công cụ (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, v.v.).
- Trình duyệt sandbox tùy chọn (`agents.defaults.sandbox.browser`).
  - Theo mặc định, trình duyệt sandbox tự động khởi động (đảm bảo CDP có thể truy cập được) khi công cụ trình duyệt cần nó.
    Cấu hình thông qua `agents.defaults.sandbox.browser.autoStart` và `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
  - Theo mặc định, các container trình duyệt sandbox sử dụng mạng Docker riêng (`openclaw-sandbox-browser`) thay vì mạng `bridge` toàn cầu.
    Cấu hình với `agents.defaults.sandbox.browser.network`.
  - `agents.defaults.sandbox.browser.cdpSourceRange` tùy chọn hạn chế lưu lượng CDP từ container đến cạnh với danh sách cho phép CIDR (ví dụ `172.21.0.1/32`).
  - Truy cập noVNC observer được bảo vệ bằng mật khẩu theo mặc định; OpenClaw phát hành URL token có thời hạn ngắn phân giải thành phiên observer.
  - `agents.defaults.sandbox.browser.allowHostControl` cho phép các phiên sandbox nhắm mục tiêu trình duyệt máy chủ một cách rõ ràng.
  - Danh sách cho phép tùy chọn kiểm soát `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Không được sandboxing:

- Chính quá trình Gateway.
- Bất kỳ công cụ nào được phép rõ ràng chạy trên máy chủ (ví dụ `tools.elevated`).
  - **Các lần chạy exec nâng cao chạy trên máy chủ và bỏ qua sandboxing.**
  - Nếu sandboxing bị tắt, `tools.elevated` không thay đổi thực thi (đã trên máy chủ). Xem [Chế độ Nâng cao](/tools/elevated).
## Chế độ

`agents.defaults.sandbox.mode` kiểm soát **khi nào** sandboxing được sử dụng:

- `"off"`: không có sandboxing.
- `"non-main"`: sandbox chỉ các phiên **không phải chính** (mặc định nếu bạn muốn các cuộc trò chuyện bình thường trên host).
- `"all"`: mọi phiên đều chạy trong một sandbox.
  Lưu ý: `"non-main"` dựa trên `session.mainKey` (mặc định `"main"`), không phải id agent.
  Các phiên nhóm/kênh sử dụng các khóa riêng của chúng, vì vậy chúng được tính là không phải chính và sẽ được sandboxing.
## Phạm vi

`agents.defaults.sandbox.scope` kiểm soát **có bao nhiêu container** được tạo:

- `"session"` (mặc định): một container cho mỗi phiên.
- `"agent"`: một container cho mỗi agent.
- `"shared"`: một container được chia sẻ bởi tất cả các phiên sandbox.
## Truy cập Workspace

`agents.defaults.sandbox.workspaceAccess` kiểm soát **những gì sandbox có thể nhìn thấy**:

- `"none"` (mặc định): các công cụ nhìn thấy một workspace sandbox dưới `~/.openclaw/sandboxes`.
- `"ro"`: gắn kết workspace agent ở chế độ chỉ đọc tại `/agent` (vô hiệu hóa `write`/`edit`/`apply_patch`).
- `"rw"`: gắn kết workspace agent ở chế độ đọc/ghi tại `/workspace`.

Phương tiện nội bộ được sao chép vào workspace sandbox hoạt động (`media/inbound/*`).
Lưu ý về Skills: công cụ `read` có gốc sandbox. Với `workspaceAccess: "none"`,
OpenClaw sao chép các Skills đủ điều kiện vào workspace sandbox (`.../skills`) để
chúng có thể được đọc. Với `"rw"`, các Skills workspace có thể được đọc từ
`/workspace/skills`.
## Custom bind mounts

`agents.defaults.sandbox.docker.binds` gắn kết các thư mục máy chủ bổ sung vào container.
Định dạng: `host:container:mode` (ví dụ: `"/home/user/source:/source:rw"`).

Các bind toàn cục và theo agent được **hợp nhất** (không thay thế). Dưới `scope: "shared"`, các bind theo agent bị bỏ qua.

`agents.defaults.sandbox.browser.binds` gắn kết các thư mục máy chủ bổ sung vào container **trình duyệt sandbox** chỉ.

- Khi được đặt (bao gồm `[]`), nó thay thế `agents.defaults.sandbox.docker.binds` cho container trình duyệt.
- Khi bị bỏ qua, container trình duyệt quay lại `agents.defaults.sandbox.docker.binds` (tương thích ngược).

Ví dụ (nguồn chỉ đọc + một thư mục dữ liệu bổ sung):

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

Security notes:

- Binds bypass the sandbox filesystem: they expose host paths with whatever mode you set (`:ro` or `:rw`).
- OpenClaw blocks dangerous bind sources (for example: `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev`, and parent mounts that would expose them).
- Sensitive mounts (secrets, SSH keys, service credentials) should be `:ro` unless absolutely required.
- Combine with `workspaceAccess: "ro"` nếu bạn chỉ cần quyền truy cập đọc vào workspace; các chế độ bind vẫn độc lập.
- Xem [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated) để biết cách các bind tương tác với chính sách công cụ và exec nâng cao.
## Hình ảnh + thiết lập

Hình ảnh mặc định: `openclaw-sandbox:bookworm-slim`

Xây dựng một lần:

```bash
scripts/sandbox-setup.sh
```

Note: the default image does **not** include Node. If a skill needs Node (or
other runtimes), either bake a custom image or install via
`sandbox.docker.setupCommand` (requires network egress + writable root +
root user).

Sandboxed browser image:

```bash
scripts/sandbox-browser-setup.sh
```

By default, sandbox containers run with **no network**.
Override with `agents.defaults.sandbox.docker.network`.

Security defaults:

- `network: "host"` is blocked.
- `network: "container:<id>"` is blocked by default (namespace join bypass risk).
- Break-glass override: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.

Các cài đặt Docker và gateway được đóng gói nằm ở đây:
[Docker](/install/docker)
## setupCommand (thiết lập container một lần)

`setupCommand` chạy **một lần** sau khi container sandbox được tạo (không chạy lại mỗi lần).
Nó thực thi bên trong container thông qua `sh -lc`.

Đường dẫn:

- Toàn cục: `agents.defaults.sandbox.docker.setupCommand`
- Theo agent: `agents.list[].sandbox.docker.setupCommand`

Những lỗi thường gặp:

- `docker.network` mặc định là `"none"` (không có egress), vì vậy cài đặt gói sẽ thất bại.
- `docker.network: "container:<id>"` yêu cầu `dangerouslyAllowContainerNamespaceJoin: true` và chỉ dành cho trường hợp khẩn cấp.
- `readOnlyRoot: true` ngăn chặn ghi; đặt `readOnlyRoot: false` hoặc tạo một image tùy chỉnh.
- `user` phải là root để cài đặt gói (bỏ qua `user` hoặc đặt `user: "0:0"`).
- Sandbox exec **không** kế thừa `process.env` từ host. Sử dụng
  `agents.defaults.sandbox.docker.env` (hoặc một image tùy chỉnh) cho các khóa API của skill.
## Chính sách công cụ + cách thoát

Chính sách cho phép/từ chối công cụ vẫn áp dụng trước các quy tắc sandbox. Nếu một công cụ bị từ chối trên toàn cầu hoặc cho mỗi agent, sandboxing không mang nó trở lại.

`tools.elevated` là một cách thoát rõ ràng chạy `exec` trên máy chủ.
`/exec` chỉ áp dụng cho những người gửi được phép và tồn tại cho mỗi phiên; để vô hiệu hóa cứng `exec`, hãy sử dụng từ chối chính sách công cụ (xem [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated)).

Gỡ lỗi:

- Sử dụng `openclaw sandbox explain` để kiểm tra chế độ sandbox hiệu quả, chính sách công cụ và các khóa cấu hình fix-it.
- Xem [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated) để hiểu mô hình tư duy "tại sao điều này bị chặn?".
  Giữ nó bị khóa.
## Ghi đè đa agent

Mỗi agent có thể ghi đè sandbox + công cụ:
`agents.list[].sandbox` và `agents.list[].tools` (cộng với `agents.list[].tools.sandbox.tools` cho chính sách công cụ sandbox).
Xem [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) để biết thứ tự ưu tiên.
## Ví dụ bật tối thiểu

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```
## Tài liệu liên quan

- [Cấu hình Sandbox](/gateway/configuration#agentsdefaults-sandbox)
- [Sandbox & Công cụ Đa Agent](/tools/multi-agent-sandbox-tools)
- [Bảo mật](/gateway/security)