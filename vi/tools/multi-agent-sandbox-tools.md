---
summary: 'Sandbox riêng cho từng agent + hạn chế công cụ, thứ tự ưu tiên và ví dụ'
title: Hộp Cát Đa Tác Nhân & Công Cụ
read_when: >-
  You want per-agent sandboxing or per-agent tool allow/deny policies in a
  multi-agent gateway.
status: active
x-i18n:
  source_path: tools\multi-agent-sandbox-tools.md
  source_hash: 3e4de14530209ddb899d2366a40981b9063a081cf515cfd770f1d2144359acb9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:32:11.929Z'
---

# Cấu hình Sandbox & Công cụ Đa Agent

## Tổng quan

Mỗi agent trong thiết lập đa agent hiện có thể có:

- **Cấu hình sandbox** (`agents.list[].sandbox` ghi đè `agents.defaults.sandbox`)
- **Hạn chế công cụ** (`tools.allow` / `tools.deny`, cộng với `agents.list[].tools`)

Điều này cho phép bạn chạy nhiều agent với các hồ sơ bảo mật khác nhau:

- Trợ lý cá nhân có quyền truy cập đầy đủ
- Agent gia đình/công việc với công cụ bị hạn chế
- Agent công khai trong sandbox

`setupCommand` nằm dưới `sandbox.docker` (toàn cục hoặc cho từng agent) và chạy một lần
khi container được tạo.

Xác thực là cho từng agent: mỗi agent đọc từ kho lưu trữ xác thực `agentDir` của riêng nó tại:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Credentials are **not** shared between agents. Never reuse `agentDir` across agents.
If you want to share creds, copy `auth-profiles.json` into the other agent's `agentDir`.

For how sandboxing behaves at runtime, see [Sandboxing](/gateway/sandboxing).
For debugging “why is this blocked?”, see [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated) and `openclaw sandbox explain`.

---
## Ví dụ Cấu hình

### Ví dụ 1: Agent Cá nhân + Gia đình Bị hạn chế

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Result:**

- `main` agent: Runs on host, full tool access
- `family` agent: Runs in Docker (one container per agent), only `read` tool

---

### Example 2: Work Agent with Shared Sandbox

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```
### Ví dụ 2b: Hồ sơ mã hóa toàn cầu + agent chỉ nhắn tin

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Result:**

- default agents get coding tools
- `support` agent is messaging-only (+ Slack tool)

---

### Example 3: Different Sandbox Modes per Agent

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // Override: public always sandboxed
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```
## Thứ tự ưu tiên cấu hình

Khi cả cấu hình toàn cục (`agents.defaults.*`) và cấu hình dành riêng cho agent (`agents.list[].*`) tồn tại:

### Cấu hình Sandbox

Các cài đặt dành riêng cho agent ghi đè cài đặt toàn cục:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Notes:**

- `agents.list[].sandbox.{docker,browser,prune}.*` overrides `agents.defaults.sandbox.{docker,browser,prune}.*` for that agent (ignored when sandbox scope resolves to `"shared"`).

### Tool Restrictions

The filtering order is:

1. **Tool profile** (`tools.profile` or `agents.list[].tools.profile`)
2. **Provider tool profile** (`tools.byProvider[provider].profile` or `agents.list[].tools.byProvider[provider].profile`)
3. **Global tool policy** (`tools.allow` / `tools.deny`)
4. **Provider tool policy** (`tools.byProvider[provider].allow/deny`)
5. **Agent-specific tool policy** (`agents.list[].tools.allow/deny`)
6. **Agent provider policy** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Sandbox tool policy** (`tools.sandbox.tools` or `agents.list[].tools.sandbox.tools`)
8. **Subagent tool policy** (`tools.subagents.tools`, if applicable)

Each level can further restrict tools, but cannot grant back denied tools from earlier levels.
If `agents.list[].tools.sandbox.tools` is set, it replaces `tools.sandbox.tools` for that agent.
If `agents.list[].tools.profile` is set, it overrides `tools.profile` for that agent.
Provider tool keys accept either `provider` (e.g. `google-antigravity`) or `provider/model` (e.g. `openai/gpt-5.2`).

### Tool groups (shorthands)

Tool policies (global, agent, sandbox) support `group:*` entries that expand to multiple concrete tools:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: all built-in OpenClaw tools (excludes provider plugins)

### Elevated Mode

`tools.elevated` is the global baseline (sender-based allowlist). `agents.list[].tools.elevated` có thể hạn chế thêm elevated cho các agent cụ thể (cả hai phải cho phép).
Các mẫu giảm thiểu rủi ro:

- Từ chối `exec` cho các agent không đáng tin cậy (`agents.list[].tools.deny: ["exec"]`)
- Tránh danh sách cho phép những người gửi định tuyến đến các agent bị hạn chế
- Vô hiệu hóa elevated toàn cầu (`tools.elevated.enabled: false`) nếu bạn chỉ muốn thực thi sandbox
- Vô hiệu hóa elevated cho từng agent (`agents.list[].tools.elevated.enabled: false`) cho các hồ sơ nhạy cảm

---
## Di chuyển từ Single Agent

**Trước đây (single agent):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**After (multi-agent with different profiles):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Legacy `agent.*` configs are migrated by `openclaw doctor`; prefer `agents.defaults` + `agents.list` từ bây giờ trở đi.

---
## Ví dụ về Hạn chế Công cụ

### Agent Chỉ đọc

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### Safe Execution Agent (no file modifications)

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### Communication-only Agent

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

---
## Cạm Bẫy Phổ Biến: "non-main"

`agents.defaults.sandbox.mode: "non-main"` dựa trên `session.mainKey` (mặc định `"main"`),
không phải ID agent. Các phiên nhóm/kênh luôn nhận các khóa riêng của chúng, vì vậy
chúng được coi là non-main và sẽ được sandboxing. Nếu bạn muốn một agent không bao giờ
sandboxing, hãy đặt `agents.list[].sandbox.mode: "off"`.

---
## Kiểm tra

Sau khi cấu hình sandbox đa agent và công cụ:

1. **Kiểm tra phân giải agent:**

   ```exec
   openclaw agents list --bindings
   ```

2. **Verify sandbox containers:**

   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **Test tool restrictions:**
   - Send a message requiring restricted tools
   - Verify the agent cannot use denied tools

4. **Monitor logs:**

   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

---
## Khắc phục sự cố

### Agent không được sandboxing mặc dù `mode: "all"`

- Kiểm tra xem có biến môi trường toàn cục `agents.defaults.sandbox.mode` ghi đè nó không
- Cấu hình dành riêng cho agent có ưu tiên cao hơn, vì vậy hãy đặt `agents.list[].sandbox.mode: "all"`

### Công cụ vẫn khả dụng mặc dù danh sách từ chối

- Kiểm tra thứ tự lọc công cụ: toàn cục → agent → sandbox → subagent
- Mỗi cấp chỉ có thể hạn chế thêm, không thể cấp lại
- Xác minh bằng nhật ký: `[tools] filtering tools for agent:${agentId}`

### Container không được cách ly theo agent

- Đặt `scope: "agent"` trong cấu hình sandbox dành riêng cho agent
- Mặc định là `"session"` tạo một container cho mỗi phiên

---
## Xem thêm

- [Định tuyến đa Agent](/concepts/multi-agent)
- [Cấu hình Sandbox](/gateway/configuration#agentsdefaults-sandbox)
- [Quản lý Phiên](/concepts/session)