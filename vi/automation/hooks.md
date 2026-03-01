---
summary: 'Hooks: tự động hóa theo sự kiện cho các lệnh và sự kiện vòng đời'
read_when:
  - >-
    Bạn muốn tự động hóa theo sự kiện cho /new, /reset, /stop, và các sự kiện
    vòng đời của agent
  - 'Bạn muốn xây dựng, cài đặt, hoặc debug các hook'
title: Hooks
x-i18n:
  source_path: automation\hooks.md
  source_hash: 4c210f3a2cf7be0a6d5098c567b0ff902f95b9d8ccc4fc26cef3b4b987bdcff0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:11:58.660Z'
---

# Hooks

Hooks cung cấp một hệ thống mở rộng dựa trên sự kiện để tự động hóa các hành động phản hồi lại các lệnh và sự kiện của agent. Hooks được tự động khám phá từ các thư mục và có thể được quản lý thông qua các lệnh CLI, tương tự như cách Skills hoạt động trong OpenClaw.
## Định hướng

Hooks là các script nhỏ chạy khi có sự kiện xảy ra. Có hai loại:

- **Hooks** (trang này): chạy bên trong Gateway khi các sự kiện agent kích hoạt, như `/new`, `/reset`, `/stop`, hoặc các sự kiện vòng đời.
- **Webhooks**: các webhook HTTP bên ngoài cho phép các hệ thống khác kích hoạt công việc trong OpenClaw. Xem [Webhook Hooks](/automation/webhook) hoặc sử dụng `openclaw webhooks` cho các lệnh hỗ trợ Gmail.

Hooks cũng có thể được đóng gói bên trong các plugin; xem [Plugins](/tools/plugin#plugin-hooks).

Các cách sử dụng phổ biến:

- Lưu ảnh chụp bộ nhớ khi bạn đặt lại phiên
- Giữ nhật ký kiểm toán các lệnh để khắc phục sự cố hoặc tuân thủ
- Kích hoạt tự động hóa tiếp theo khi phiên bắt đầu hoặc kết thúc
- Ghi tệp vào không gian làm việc của agent hoặc gọi API bên ngoài khi các sự kiện kích hoạt

Nếu bạn có thể viết một hàm TypeScript nhỏ, bạn có thể viết một hook. Hooks được khám phá tự động, và bạn bật hoặc tắt chúng thông qua CLI.
## Tổng quan

Hệ thống hooks cho phép bạn:

- Lưu ngữ cảnh phiên vào bộ nhớ khi `/new` được thực hiện
- Ghi lại tất cả các lệnh để kiểm toán
- Kích hoạt các tự động hóa tùy chỉnh trên các sự kiện vòng đời agent
- Mở rộng hành vi của OpenClaw mà không cần sửa đổi mã lõi
## Bắt đầu

### Hooks được tích hợp sẵn

OpenClaw đi kèm với bốn hooks được tích hợp sẵn và tự động khám phá:

- **💾 session-memory**: Lưu ngữ cảnh phiên vào không gian làm việc agent của bạn (mặc định `~/.openclaw/workspace/memory/`) khi bạn thực hiện `/new`
- **📎 bootstrap-extra-files**: Chèn các tệp bootstrap không gian làm việc bổ sung từ các mẫu glob/đường dẫn đã cấu hình trong quá trình `agent:bootstrap`
- **📝 command-logger**: Ghi lại tất cả các sự kiện lệnh vào `~/.openclaw/logs/commands.log`
- **🚀 boot-md**: Chạy `BOOT.md` khi Gateway khởi động (yêu cầu bật hooks nội bộ)

Liệt kê các hooks có sẵn:

```bash
openclaw hooks list
```

Enable a hook:

```bash
openclaw hooks enable session-memory
```

Check hook status:

```bash
openclaw hooks check
```

Get detailed information:

```bash
openclaw hooks info session-memory
```

### Onboarding

During onboarding (`openclaw onboard`), bạn sẽ được nhắc bật các hooks được khuyến nghị. Trình hướng dẫn tự động khám phá các hooks đủ điều kiện và trình bày chúng để lựa chọn.
## Khám phá Hook

Các hook được tự động khám phá từ ba thư mục (theo thứ tự ưu tiên):

1. **Hook không gian làm việc**: `<workspace>/hooks/` (theo agent, ưu tiên cao nhất)
2. **Hook được quản lý**: `~/.openclaw/hooks/` (do người dùng cài đặt, chia sẻ giữa các không gian làm việc)
3. **Hook đi kèm**: `<openclaw>/dist/hooks/bundled/` (được cung cấp cùng với OpenClaw)

Các thư mục hook được quản lý có thể là **hook đơn lẻ** hoặc **gói hook** (thư mục gói).

Mỗi hook là một thư mục chứa:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```
## Hook Packs (npm/archives)

Hook packs là các gói npm tiêu chuẩn xuất một hoặc nhiều hook thông qua `openclaw.hooks` trong
`package.json`. Cài đặt chúng bằng:

```bash
openclaw hooks install <path-or-spec>
```

Npm specs are registry-only (package name + optional version/tag). Git/URL/file specs are rejected.

Example `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Each entry points to a hook directory containing `HOOK.md` and `handler.ts` (or `index.ts`).
Hook packs can ship dependencies; they will be installed under `~/.openclaw/hooks/<id>`.
Each `openclaw.hooks` entry must stay inside the package directory after symlink
resolution; entries that escape are rejected.

Security note: `openclaw hooks install` installs dependencies with `npm install --ignore-scripts`
(no lifecycle scripts). Keep hook pack dependency trees "pure JS/TS" and avoid packages that rely
on `postinstall` builds.
## Cấu trúc Hook

### Định dạng HOOK.md

Tệp `HOOK.md` chứa metadata trong YAML frontmatter cộng với tài liệu Markdown:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### Metadata Fields

The `metadata.openclaw` object supports:

- **`emoji`**: Display emoji for CLI (e.g., `"💾"`)
- **`events`**: Array of events to listen for (e.g., `["command:new", "command:reset"]`)
- **`export`**: Named export to use (defaults to `"default"`)
- **`homepage`**: Documentation URL
- **`requires`**: Optional requirements
  - **`bins`**: Required binaries on PATH (e.g., `["git", "node"]`)
  - **`anyBins`**: At least one of these binaries must be present
  - **`env`**: Required environment variables
  - **`config`**: Required config paths (e.g., `["workspace.dir"]`)
  - **`os`**: Required platforms (e.g., `["darwin", "linux"]`)
- **`always`**: Bypass eligibility checks (boolean)
- **`install`**: Installation methods (for bundled hooks: `[{"id":"bundled","kind":"bundled"}]`)

### Handler Implementation

The `handler.ts` file exports a `HookHandler` function:

```typescript
const myHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] Lệnh mới được kích hoạt`);
  console.log(`  Phiên: ${event.sessionKey}`);
  console.log(`  Thời gian: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```
#### Ngữ cảnh Sự kiện

Mỗi sự kiện bao gồm:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // e.g., 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    // Command events:
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // Message events (see Message Events section for full details):
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```
## Loại sự kiện

### Sự kiện lệnh

Được kích hoạt khi các lệnh agent được thực hiện:

- **`command`**: Tất cả sự kiện lệnh (trình lắng nghe chung)
- **`command:new`**: Khi lệnh `/new` được thực hiện
- **`command:reset`**: Khi lệnh `/reset` được thực hiện
- **`command:stop`**: Khi lệnh `/stop` được thực hiện

### Sự kiện Agent

- **`agent:bootstrap`**: Trước khi các tệp khởi tạo workspace được chèn vào (hooks có thể thay đổi `context.bootstrapFiles`)

### Sự kiện Gateway

Được kích hoạt khi gateway khởi động:

- **`gateway:startup`**: Sau khi các kênh khởi động và hooks được tải

### Sự kiện tin nhắn

Được kích hoạt khi tin nhắn được nhận hoặc gửi:

- **`message`**: Tất cả sự kiện tin nhắn (trình lắng nghe chung)
- **`message:received`**: Khi một tin nhắn đến được nhận từ bất kỳ kênh nào
- **`message:sent`**: Khi một tin nhắn đi được gửi thành công

#### Ngữ cảnh sự kiện tin nhắn

Sự kiện tin nhắn bao gồm ngữ cảnh phong phú về tin nhắn:

```typescript
// message:received context
{
  from: string,           // Sender identifier (phone number, user ID, etc.)
  content: string,        // Message content
  timestamp?: number,     // Unix timestamp when received
  channelId: string,      // Channel (e.g., "whatsapp", "telegram", "discord")
  accountId?: string,     // Provider account ID for multi-account setups
  conversationId?: string, // Chat/conversation ID
  messageId?: string,     // Message ID from the provider
  metadata?: {            // Additional provider-specific data
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// message:sent context
{
  to: string,             // Recipient identifier
  content: string,        // Message content that was sent
  success: boolean,       // Whether the send succeeded
  error?: string,         // Error message if sending failed
  channelId: string,      // Channel (e.g., "whatsapp", "telegram", "discord")
  accountId?: string,     // Provider account ID
  conversationId?: string, // Chat/conversation ID
  messageId?: string,     // Message ID returned by the provider
}
```
#### Ví dụ: Hook Ghi nhật ký Tin nhắn

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Received from ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Sent to ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### Tool Result Hooks (Plugin API)

These hooks are not event-stream listeners; they let plugins synchronously adjust tool results before OpenClaw persists them.

- **`tool_result_persist`**: transform tool results before they are written to the session transcript. Must be synchronous; return the updated tool result payload or `undefined` to keep it as-is. See [Agent Loop](/concepts/agent-loop).

### Future Events

Planned event types:

- **`session:start`**: When a new session begins
- **`session:end`**: When a session ends
- **`agent:error`**: Khi một agent gặp lỗi
## Tạo Hook Tùy chỉnh

### 1. Chọn Vị trí

- **Hook workspace** (`<workspace>/hooks/`): Theo từng agent, ưu tiên cao nhất
- **Hook được quản lý** (`~/.openclaw/hooks/`): Chia sẻ giữa các workspace

### 2. Tạo Cấu trúc Thư mục

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. Create HOOK.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. Create handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### 5. Enable and Test

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```
## Cấu hình

### Định dạng cấu hình mới (Khuyến nghị)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Per-Hook Configuration

Hooks can have custom configuration:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Extra Directories

Load hooks from additional directories:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Định dạng cấu hình cũ (Vẫn được hỗ trợ)

Định dạng cấu hình cũ vẫn hoạt động để đảm bảo tương thích ngược:
```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

Note: `module` phải là đường dẫn tương đối với workspace. Đường dẫn tuyệt đối và việc truy cập ra ngoài workspace sẽ bị từ chối.

**Di chuyển**: Sử dụng hệ thống dựa trên khám phá mới cho các hook mới. Các handler cũ sẽ được tải sau các hook dựa trên thư mục.
## Lệnh CLI

### Liệt kê Hook

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Verbose output (show missing requirements)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

### Hook Information

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### Check Eligibility

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### Enable/Disable

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```
## Tham chiếu hook tích hợp sẵn

### session-memory

Lưu ngữ cảnh phiên vào bộ nhớ khi bạn thực hiện `/new`.

**Sự kiện**: `command:new`

**Yêu cầu**: `workspace.dir` phải được cấu hình

**Đầu ra**: `<workspace>/memory/YYYY-MM-DD-slug.md` (mặc định là `~/.openclaw/workspace`)

**Chức năng**:

1. Sử dụng mục phiên trước khi đặt lại để xác định vị trí bản ghi chính xác
2. Trích xuất 15 dòng cuối cùng của cuộc trò chuyện
3. Sử dụng LLM để tạo slug tên tệp mô tả
4. Lưu siêu dữ liệu phiên vào tệp bộ nhớ có ngày tháng

**Ví dụ đầu ra**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Filename examples**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (fallback timestamp if slug generation fails)

**Enable**:

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

Injects additional bootstrap files (for example monorepo-local `AGENTS.md` / `TOOLS.md`) during `agent:bootstrap`.

**Events**: `agent:bootstrap`

**Requirements**: `workspace.dir` must be configured

**Output**: No files written; bootstrap context is modified in-memory only.

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```
**Ghi chú**:

- Đường dẫn được giải quyết tương đối với workspace.
- Tệp phải nằm trong workspace (được kiểm tra realpath).
- Chỉ các basename bootstrap được nhận dạng mới được tải.
- Danh sách cho phép subagent được bảo toàn (chỉ `AGENTS.md` và `TOOLS.md`).

**Kích hoạt**:

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

Logs all command events to a centralized audit file.

**Events**: `command`

**Requirements**: None

**Output**: `~/.openclaw/logs/commands.log`

**What it does**:

1. Captures event details (command action, timestamp, session key, sender ID, source)
2. Appends to log file in JSONL format
3. Runs silently in the background

**Example log entries**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**View logs**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Enable**:

```bash
openclaw hooks enable command-logger
```

### boot-md

Runs `BOOT.md` khi gateway khởi động (sau khi các kênh khởi động).
Các hook nội bộ phải được kích hoạt để chạy.
**Sự kiện**: `gateway:startup`

**Yêu cầu**: `workspace.dir` phải được cấu hình

**Chức năng**:

1. Đọc `BOOT.md` từ không gian làm việc của bạn
2. Chạy các hướng dẫn thông qua trình chạy agent
3. Gửi bất kỳ tin nhắn đi nào được yêu cầu thông qua công cụ tin nhắn

**Kích hoạt**:

```bash
openclaw hooks enable boot-md
```
## Thực hành tốt nhất

### Giữ cho Handlers nhanh

Hooks chạy trong quá trình xử lý lệnh. Hãy giữ chúng nhẹ:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Handle Errors Gracefully

Always wrap risky operations:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### Filter Events Early

Return early if the event isn't relevant:

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Use Specific Event Keys

Specify exact events in metadata when possible:

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

Rather than:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```
## Gỡ lỗi

### Bật ghi log Hook

Gateway ghi log việc tải hook khi khởi động:

```
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Check Discovery

List all discovered hooks:

```bash
openclaw hooks list --verbose
```

### Check Registration

In your handler, log when it's called:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

Check why a hook isn't eligible:

```bash
openclaw hooks info my-hook
```

Tìm kiếm các yêu cầu bị thiếu trong đầu ra.
## Kiểm thử

### Nhật ký Gateway

Theo dõi nhật ký gateway để xem việc thực thi hook:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Test Hooks Directly

Test your handlers in isolation:

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // Assert side effects
});
```
## Kiến trúc

### Các thành phần cốt lõi

- **`src/hooks/types.ts`**: Định nghĩa kiểu dữ liệu
- **`src/hooks/workspace.ts`**: Quét và tải thư mục
- **`src/hooks/frontmatter.ts`**: Phân tích metadata HOOK.md
- **`src/hooks/config.ts`**: Kiểm tra tính đủ điều kiện
- **`src/hooks/hooks-status.ts`**: Báo cáo trạng thái
- **`src/hooks/loader.ts`**: Trình tải module động
- **`src/cli/hooks-cli.ts`**: Lệnh CLI
- **`src/gateway/server-startup.ts`**: Tải hooks khi khởi động gateway
- **`src/auto-reply/reply/commands-core.ts`**: Kích hoạt sự kiện lệnh

### Luồng khám phá

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### Event Flow

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```
## Khắc phục sự cố

### Hook không được khám phá

1. Kiểm tra cấu trúc thư mục:

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. Verify HOOK.md format:

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. List all discovered hooks:

   ```bash
   openclaw hooks list
   ```

### Hook Not Eligible

Check requirements:

```bash
openclaw hooks info my-hook
```

Look for missing:

- Binaries (check PATH)
- Environment variables
- Config values
- OS compatibility

### Hook Not Executing

1. Verify hook is enabled:

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. Restart your gateway process so hooks reload.

3. Check gateway logs for errors:

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Lỗi Handler

Kiểm tra lỗi TypeScript/import:
```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```
## Hướng dẫn Di chuyển

### Từ Cấu hình Cũ sang Khám phá thiết bị

**Trước đây**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**After**:

1. Create hook directory:

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. Create HOOK.md:

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. Update config:

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```
4. Xác minh và khởi động lại tiến trình gateway của bạn:

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Lợi ích của việc di chuyển**:

- Khám phá thiết bị tự động
- Quản lý CLI
- Kiểm tra tính đủ điều kiện
- Tài liệu tốt hơn
- Cấu trúc nhất quán
## Xem thêm

- [Tham khảo CLI: hooks](/cli/hooks)
- [README Hooks tích hợp](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Cấu hình](/gateway/configuration#hooks)