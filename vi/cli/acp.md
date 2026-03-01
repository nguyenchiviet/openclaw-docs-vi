---
summary: Chạy cầu nối ACP cho tích hợp IDE
read_when:
  - Thiết lập tích hợp IDE dựa trên ACP
  - Gỡ lỗi định tuyến phiên ACP đến Gateway
title: >-
  I need the English text to translate to Vietnamese. You've only provided "acp"
  which appears to be an abbreviation or command. Please provide the full
  English text you'd like me to translate.
x-i18n:
  source_path: cli\acp.md
  source_hash: 2ac039c16613f0c0460c7ff2ba5a62135fa17d5a68ed1bf0086ad397e93ed8fa
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:31:16.689Z'
---

# acp

Chạy cầu nối [Agent Client Protocol (https://agentclientprotocol.com/) để giao tiếp với OpenClaw Gateway.

Lệnh này sử dụng ACP qua stdio cho các IDE và chuyển tiếp các prompt tới Gateway
qua WebSocket. Nó duy trì ánh xạ các phiên ACP với các khóa phiên Gateway.
## Sử dụng

```bash
openclaw acp

# Remote Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# Remote Gateway (token from file)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Attach to an existing session key
openclaw acp --session agent:main:main

# Attach by label (must already exist)
openclaw acp --session-label "support inbox"

# Reset the session key before the first prompt
openclaw acp --session agent:main:main --reset-session
```
## ACP client (debug)

Sử dụng ACP client tích hợp để kiểm tra cài đặt bridge mà không cần IDE.
Nó khởi chạy ACP bridge và cho phép bạn nhập prompts một cách tương tác.

```bash
openclaw acp client

# Point the spawned bridge at a remote Gateway
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Override the server command (default: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

Permission model (client debug mode):

- Auto-approval is allowlist-based and only applies to trusted core tool IDs.
- `read` auto-approval is scoped to the current working directory (`--cwd` when set).
- Unknown/non-core tool names, out-of-scope reads, and dangerous tools always require explicit prompt approval.
- Server-provided `toolCall.kind` được coi là metadata không đáng tin cậy (không phải nguồn ủy quyền).
## Cách sử dụng

Sử dụng ACP khi một IDE (hoặc client khác) hỗ trợ Agent Client Protocol và bạn muốn nó điều khiển một phiên Gateway OpenClaw.

1. Đảm bảo Gateway đang chạy (local hoặc remote).
2. Cấu hình target Gateway (config hoặc flags).
3. Trỏ IDE của bạn để chạy `openclaw acp` qua stdio.

Ví dụ config (được lưu trữ):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Example direct run (no config write):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# preferred for local process safety
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```
## Chọn agent

ACP không chọn agent trực tiếp. Nó định tuyến theo khóa phiên của Gateway.

Sử dụng khóa phiên có phạm vi agent để nhắm đến một agent cụ thể:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Each ACP session maps to a single Gateway session key. One agent can have many
sessions; ACP defaults to an isolated `acp:<uuid>` trừ khi bạn ghi đè khóa hoặc nhãn.
## Thiết lập trình soạn thảo Zed

Thêm một agent ACP tùy chỉnh trong `~/.config/zed/settings.json` (hoặc sử dụng giao diện Settings của Zed):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

To target a specific Gateway or agent:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Trong Zed, mở bảng Agent và chọn "OpenClaw ACP" để bắt đầu một cuộc trò chuyện.
## Ánh xạ phiên

Theo mặc định, các phiên ACP sẽ nhận một khóa phiên Gateway riêng biệt với tiền tố `acp:`.
Để tái sử dụng một phiên đã biết, hãy truyền một khóa phiên hoặc nhãn:

- `--session <key>`: sử dụng một khóa phiên Gateway cụ thể.
- `--session-label <label>`: phân giải một phiên hiện có theo nhãn.
- `--reset-session`: tạo một id phiên mới cho khóa đó (cùng khóa, bản ghi mới).

Nếu client ACP của bạn hỗ trợ metadata, bạn có thể ghi đè cho từng phiên:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

Tìm hiểu thêm về các khóa phiên tại [/concepts/session](/concepts/session).
## Tùy chọn

- `--url <url>`: URL WebSocket của Gateway (mặc định là gateway.remote.url khi được cấu hình).
- `--token <token>`: Token xác thực Gateway.
- `--token-file <path>`: đọc token xác thực Gateway từ tệp.
- `--password <password>`: Mật khẩu xác thực Gateway.
- `--password-file <path>`: đọc mật khẩu xác thực Gateway từ tệp.
- `--session <key>`: khóa phiên mặc định.
- `--session-label <label>`: nhãn phiên mặc định để phân giải.
- `--require-existing`: thất bại nếu khóa/nhãn phiên không tồn tại.
- `--reset-session`: đặt lại khóa phiên trước khi sử dụng lần đầu.
- `--no-prefix-cwd`: không thêm tiền tố thư mục làm việc vào lời nhắc.
- `--verbose, -v`: ghi log chi tiết vào stderr.

Lưu ý bảo mật:

- `--token` và `--password` có thể hiển thị trong danh sách tiến trình cục bộ trên một số hệ thống.
- Nên sử dụng `--token-file`/`--password-file` hoặc biến môi trường (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`).

### Tùy chọn `acp client`

- `--cwd <dir>`: thư mục làm việc cho phiên ACP.
- `--server <command>`: lệnh máy chủ ACP (mặc định: `openclaw`).
- `--server-args <args...>`: các tham số bổ sung được truyền cho máy chủ ACP.
- `--server-verbose`: bật ghi log chi tiết trên máy chủ ACP.
- `--verbose, -v`: ghi log chi tiết của client.