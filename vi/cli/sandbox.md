---
title: Sandbox CLI
summary: Quản lý các container sandbox và kiểm tra chính sách sandbox hiệu quả
read_when: You are managing sandbox containers or debugging sandbox/tool-policy behavior.
status: active
x-i18n:
  source_path: cli\sandbox.md
  source_hash: 6e1186f26c77e188206ce5e198ab624d6b38bc7bb7c06e4d2281b6935c39e347
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:37:24.068Z'
---

# Sandbox CLI

Quản lý các container sandbox dựa trên Docker để thực thi agent bị cô lập.
## Tổng quan

OpenClaw có thể chạy các agent trong các container Docker được cách ly để đảm bảo bảo mật. Các lệnh `sandbox` giúp bạn quản lý các container này, đặc biệt là sau khi cập nhật hoặc thay đổi cấu hình.
## Lệnh

### `openclaw sandbox explain`

Kiểm tra chế độ sandbox **hiệu quả**, phạm vi truy cập, chính sách công cụ sandbox và các cổng nâng cao (với đường dẫn khóa cấu hình sửa chữa).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### `openclaw sandbox list`

List all sandbox containers with their status and configuration.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # List only browser containers
openclaw sandbox list --json     # JSON output
```

**Output includes:**

- Container name and status (running/stopped)
- Docker image and whether it matches config
- Age (time since creation)
- Idle time (time since last use)
- Associated session/agent

### `openclaw sandbox recreate`

Remove sandbox containers to force recreation with updated images/config.

```bash
openclaw sandbox recreate --all                # Recreate all containers
openclaw sandbox recreate --session main       # Specific session
openclaw sandbox recreate --agent mybot        # Specific agent
openclaw sandbox recreate --browser            # Only browser containers
openclaw sandbox recreate --all --force        # Skip confirmation
```

**Options:**

- `--all`: Recreate all sandbox containers
- `--session <key>`: Recreate container for specific session
- `--agent <id>`: Recreate containers for specific agent
- `--browser`: Only recreate browser containers
- `--force`: Bỏ qua lời nhắc xác nhận

**Quan trọng:** Các container được tự động tạo lại khi agent tiếp theo được sử dụng.
## Trường hợp sử dụng

### Sau khi cập nhật hình ảnh Docker

```bash
# Pull new image
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Update config to use new image
# Edit config: agents.defaults.sandbox.docker.image (or agents.list[].sandbox.docker.image)

# Recreate containers
openclaw sandbox recreate --all
```

### After changing sandbox configuration

```bash
# Edit config: agents.defaults.sandbox.* (or agents.list[].sandbox.*)

# Recreate to apply new config
openclaw sandbox recreate --all
```

### After changing setupCommand

```bash
openclaw sandbox recreate --all
# or just one agent:
openclaw sandbox recreate --agent family
```

### For a specific agent only

```bash
# Update only one agent's containers
openclaw sandbox recreate --agent alfred
```
## Tại sao điều này cần thiết?

**Vấn đề:** Khi bạn cập nhật sandbox Docker images hoặc cấu hình:

- Các container hiện có tiếp tục chạy với cài đặt cũ
- Các container chỉ được xóa sau 24 giờ không hoạt động
- Các agent được sử dụng thường xuyên giữ các container cũ chạy vô thời hạn

**Giải pháp:** Sử dụng `openclaw sandbox recreate` để buộc xóa các container cũ. Chúng sẽ được tạo lại tự động với cài đặt hiện tại khi cần thiết tiếp theo.

Mẹo: ưu tiên `openclaw sandbox recreate` hơn `docker rm` thủ công. Nó sử dụng đặt tên container của Gateway và tránh không khớp khi các khóa scope/session thay đổi.
## Cấu hình

Cài đặt sandbox nằm trong `~/.openclaw/openclaw.json` dưới `agents.defaults.sandbox` (các ghi đè cho từng agent nằm trong `agents.list[].sandbox`):

```jsonc
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... more Docker options
        },
        "prune": {
          "idleHours": 24, // Auto-prune after 24h idle
          "maxAgeDays": 7, // Auto-prune after 7 days
        },
      },
    },
  },
}
```
## Xem thêm

- [Tài liệu Sandbox](/gateway/sandboxing)
- [Cấu hình Agent](/concepts/agent-workspace)
- [Lệnh Doctor](/gateway/doctor) - Kiểm tra thiết lập sandbox