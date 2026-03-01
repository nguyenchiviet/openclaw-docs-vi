---
summary: Di chuyển (di cư) một cài đặt OpenClaw từ máy này sang máy khác
read_when:
  - Bạn đang di chuyển OpenClaw sang một máy tính xách tay/máy chủ mới
  - 'Bạn muốn bảo tồn các phiên, xác thực và đăng nhập kênh (WhatsApp, v.v.).'
title: Hướng dẫn Di chuyển
x-i18n:
  source_path: install\migrating.md
  source_hash: 604d862c4bf86e7924d09028db8cc2514ca6f1d64ebe8bb7d1e2dde57ef70caa
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:09:03.904Z'
---

# Di chuyển OpenClaw sang máy mới

Hướng dẫn này di chuyển OpenClaw Gateway từ máy này sang máy khác **mà không cần làm lại thiết lập ban đầu**.

Khái niệm di chuyển rất đơn giản:

- Sao chép **thư mục trạng thái** (`$OPENCLAW_STATE_DIR`, mặc định: `~/.openclaw/`) — bao gồm cấu hình, xác thực, phiên và trạng thái kênh.
- Sao chép **không gian làm việc** (`~/.openclaw/workspace/` theo mặc định) — bao gồm các tệp agent của bạn (bộ nhớ, lời nhắc, v.v.).

Nhưng có những lỗi phổ biến xung quanh **hồ sơ**, **quyền**, và **sao chép một phần**.
## Trước khi bắt đầu (những gì bạn đang di chuyển)

### 1) Xác định thư mục trạng thái của bạn

Hầu hết các cài đặt sử dụng mặc định:

- **State dir:** `~/.openclaw/`

Nhưng nó có thể khác nếu bạn sử dụng:

- `--profile <name>` (thường trở thành `~/.openclaw-<profile>/`)
- `OPENCLAW_STATE_DIR=/some/path`

Nếu bạn không chắc chắn, hãy chạy trên máy **cũ**:

```bash
openclaw status
```

Look for mentions of `OPENCLAW_STATE_DIR` / profile in the output. If you run multiple gateways, repeat for each profile.

### 2) Identify your workspace

Common defaults:

- `~/.openclaw/workspace/` (recommended workspace)
- a custom folder you created

Your workspace is where files like `MEMORY.md`, `USER.md`, and `memory/*.md` live.

### 3) Understand what you will preserve

If you copy **both** the state dir and workspace, you keep:

- Gateway configuration (`openclaw.json`)
- Auth profiles / API keys / OAuth tokens
- Session history + agent state
- Channel state (e.g. WhatsApp login/session)
- Your workspace files (memory, skills notes, etc.)

If you copy **only** the workspace (e.g., via Git), you do **not** preserve:

- sessions
- credentials
- channel logins

Those live under `$OPENCLAW_STATE_DIR`.
## Các bước di chuyển (được khuyến nghị)

### Bước 0 — Tạo bản sao lưu (máy cũ)

Trên máy **cũ**, dừng gateway trước để các tệp không thay đổi trong quá trình sao chép:

```bash
openclaw gateway stop
```

(Optional but recommended) archive the state dir and workspace:

```bash
# Adjust paths if you use a profile or custom locations
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

If you have multiple profiles/state dirs (e.g. `~/.openclaw-main`, `~/.openclaw-work`), archive each.

### Step 1 — Install OpenClaw on the new machine

On the **new** machine, install the CLI (and Node if needed):

- See: [Install](/install)

At this stage, it’s OK if onboarding creates a fresh `~/.openclaw/` — you will overwrite it in the next step.

### Step 2 — Copy the state dir + workspace to the new machine

Copy **both**:

- `$OPENCLAW_STATE_DIR` (default `~/.openclaw/`)
- your workspace (default `~/.openclaw/workspace/`)

Common approaches:

- `scp` the tarballs and extract
- `rsync -a` over SSH
- external drive

After copying, ensure:

- Hidden directories were included (e.g. `.openclaw/`)
- File ownership is correct for the user running the gateway

### Step 3 — Run Doctor (migrations + service repair)

On the **new** machine:

```bash
openclaw doctor
```

Doctor là lệnh "an toàn và đơn giản". Nó sửa chữa các dịch vụ, áp dụng các bản di chuyển cấu hình và cảnh báo về các sự không khớp.

Sau đó:
`bash
openclaw gateway restart
openclaw status
`
## Những lỗi thường gặp (và cách tránh chúng)

### Lỗi: profile / state-dir không khớp

Nếu bạn chạy gateway cũ với một profile (hoặc `OPENCLAW_STATE_DIR`), và gateway mới sử dụng một profile khác, bạn sẽ thấy các triệu chứng như:

- các thay đổi cấu hình không có hiệu lực
- các kênh bị mất / đã đăng xuất
- lịch sử phiên trống

Cách khắc phục: chạy gateway/service bằng cách sử dụng **cùng** profile/state dir mà bạn đã di chuyển, sau đó chạy lại:

```bash
openclaw doctor
```

### Footgun: copying only `openclaw.json`

`openclaw.json` is not enough. Many providers store state under:

- `$OPENCLAW_STATE_DIR/credentials/`
- `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

Always migrate the entire `$OPENCLAW_STATE_DIR` folder.

### Footgun: permissions / ownership

If you copied as root or changed users, the gateway may fail to read credentials/sessions.

Fix: ensure the state dir + workspace are owned by the user running the gateway.

### Footgun: migrating between remote/local modes

- If your UI (WebUI/TUI) points at a **remote** gateway, the remote host owns the session store + workspace.
- Migrating your laptop won’t move the remote gateway’s state.

If you’re in remote mode, migrate the **gateway host**.

### Footgun: secrets in backups

`$OPENCLAW_STATE_DIR` chứa các bí mật (khóa API, mã thông báo OAuth, thông tin đăng nhập WhatsApp). Coi các bản sao lưu như các bí mật sản xuất:

- lưu trữ được mã hóa
- tránh chia sẻ qua các kênh không an toàn
- xoay khóa nếu bạn nghi ngờ bị lộ
## Danh sách kiểm tra xác minh

Trên máy mới, xác nhận:

- `openclaw status` hiển thị gateway đang chạy
- Các kênh của bạn vẫn được kết nối (ví dụ: WhatsApp không yêu cầu ghép lại)
- Bảng điều khiển mở và hiển thị các phiên hiện có
- Các tệp không gian làm việc của bạn (bộ nhớ, cấu hình) có mặt
## Liên quan

- [Doctor](/gateway/doctor)
- [Khắc phục sự cố Gateway](/gateway/troubleshooting)
- [OpenClaw lưu trữ dữ liệu của nó ở đâu?](/help/faq#where-does-openclaw-store-its-data)