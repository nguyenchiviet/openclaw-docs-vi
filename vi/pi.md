---
title: Kiến trúc Tích hợp Pi
summary: Kiến trúc tích hợp Pi agent nhúng của OpenClaw và vòng đời phiên làm việc
read_when:
  - Hiểu về thiết kế tích hợp Pi SDK trong OpenClaw
  - >-
    Sửa đổi vòng đời phiên làm việc của agent, tooling hoặc provider wiring cho
    Pi
x-i18n:
  source_path: pi.md
  source_hash: 7b7c7e6cc736d448e85de8361d4dd0542f3e1e9ceec2ac78faf95c0b2b2b5a16
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:04.692Z'
---

# Kiến trúc tích hợp Pi

Tài liệu này mô tả cách OpenClaw tích hợp với [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) và các gói liên quan (`pi-ai`, `pi-agent-core`, `pi-tui`) để cung cấp các khả năng agent AI.
## Tổng quan

OpenClaw sử dụng Pi SDK để nhúng một agent mã hóa AI vào kiến trúc gateway nhắn tin của nó. Thay vì tạo Pi như một tiến trình con hoặc sử dụng chế độ RPC, OpenClaw trực tiếp nhập và khởi tạo `AgentSession` của Pi thông qua `createAgentSession()`. Cách tiếp cận nhúng này cung cấp:

- Kiểm soát toàn bộ vòng đời phiên và xử lý sự kiện
- Tiêm công cụ tùy chỉnh (nhắn tin, sandbox, hành động dành riêng cho kênh)
- Tùy chỉnh lời nhắc hệ thống cho mỗi kênh/ngữ cảnh
- Tính bền vững của phiên với hỗ trợ nhánh/nén
- Xoay vòng hồ sơ xác thực đa tài khoản với chuyển đổi dự phòng
- Chuyển đổi mô hình không phụ thuộc vào nhà cung cấp
## Các Phụ Thuộc Gói

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| Package           | Purpose                                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------ |
| `pi-ai`           | Core LLM abstractions: `Model`, `streamSimple`, message types, provider APIs                           |
| `pi-agent-core`   | Agent loop, tool execution, `AgentMessage` types                                                       |
| `pi-coding-agent` | High-level SDK: `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, built-in tools |
| `pi-tui`          | Các thành phần Terminal UI (được sử dụng trong chế độ TUI cục bộ của OpenClaw)                                             |
## Cấu trúc tệp

```
src/agents/
├── pi-embedded-runner.ts          # Re-exports from pi-embedded-runner/
├── pi-embedded-runner/
│   ├── run.ts                     # Main entry: runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # Single attempt logic with session setup
│   │   ├── params.ts              # RunEmbeddedPiAgentParams type
│   │   ├── payloads.ts            # Build response payloads from run results
│   │   ├── images.ts              # Vision model image injection
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # Abort error detection
│   ├── cache-ttl.ts               # Cache TTL tracking for context pruning
│   ├── compact.ts                 # Manual/auto compaction logic
│   ├── extensions.ts              # Load pi extensions for embedded runs
│   ├── extra-params.ts            # Provider-specific stream params
│   ├── google.ts                  # Google/Gemini turn ordering fixes
│   ├── history.ts                 # History limiting (DM vs group)
│   ├── lanes.ts                   # Session/global command lanes
│   ├── logger.ts                  # Subsystem logger
│   ├── model.ts                   # Model resolution via ModelRegistry
│   ├── runs.ts                    # Active run tracking, abort, queue
│   ├── sandbox-info.ts            # Sandbox info for system prompt
│   ├── session-manager-cache.ts   # SessionManager instance caching
│   ├── session-manager-init.ts    # Session file initialization
│   ├── system-prompt.ts           # System prompt builder
│   ├── tool-split.ts              # Split tools into builtIn vs custom
│   ├── types.ts                   # EmbeddedPiAgentMeta, EmbeddedPiRunResult
│   └── utils.ts                   # ThinkLevel mapping, error description
├── pi-embedded-subscribe.ts       # Session event subscription/dispatch
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # Event handler factory
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # Streaming block reply chunking
├── pi-embedded-messaging.ts       # Messaging tool sent tracking
├── pi-embedded-helpers.ts         # Error classification, turn validation
├── pi-embedded-helpers/           # Helper modules
├── pi-embedded-utils.ts           # Formatting utilities
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # AbortSignal wrapping for tools
├── pi-tools.policy.ts             # Tool allowlist/denylist policy
├── pi-tools.read.ts               # Read tool customizations
├── pi-tools.schema.ts             # Tool schema normalization
├── pi-tools.types.ts              # AnyAgentTool type alias
├── pi-tool-definition-adapter.ts  # AgentTool -> ToolDefinition adapter
├── pi-settings.ts                 # Settings overrides
├── pi-extensions/                 # Custom pi extensions
│   ├── compaction-safeguard.ts    # Safeguard extension
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # Cache-TTL context pruning extension
│   └── context-pruning/
├── model-auth.ts                  # Auth profile resolution
├── auth-profiles.ts               # Profile store, cooldown, failover
├── model-selection.ts             # Default model resolution
├── models-config.ts               # models.json generation
├── model-catalog.ts               # Model catalog cache
├── context-window-guard.ts        # Context window validation
├── failover-error.ts              # FailoverError class
├── defaults.ts                    # DEFAULT_PROVIDER, DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # System prompt parameter resolution
├── system-prompt-report.ts        # Debug report generation
├── tool-summaries.ts              # Tool description summaries
├── tool-policy.ts                 # Tool policy resolution
├── transcript-policy.ts           # Transcript validation policy
├── skills.ts                      # Skill snapshot/prompt building
├── skills/                        # Skill subsystem
├── sandbox.ts                     # Sandbox context resolution
├── sandbox/                       # Sandbox subsystem
├── channel-tools.ts               # Channel-specific tool injection
├── openclaw-tools.ts              # OpenClaw-specific tools
├── bash-tools.ts                  # exec/process tools
├── apply-patch.ts                 # apply_patch tool (OpenAI)
├── tools/                         # Individual tool implementations
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```
## Luồng Tích hợp Cốt lõi

### 1. Chạy một Agent Nhúng

Điểm vào chính là `runEmbeddedPiAgent()` trong `pi-embedded-runner/run.ts`:

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2. Session Creation

Inside `runEmbeddedAttempt()` (called by `runEmbeddedPiAgent()`), the pi SDK is used:

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```
### 3. Event Subscription

`subscribeEmbeddedPiSession()` đăng ký các sự kiện `AgentSession` của pi:

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

Events handled include:

- `message_start` / `message_end` / `message_update` (streaming text/thinking)
- `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
- `turn_start` / `turn_end`
- `agent_start` / `agent_end`
- `auto_compaction_start` / `auto_compaction_end`

### 4. Prompting

After setup, the session is prompted:

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

The SDK handles the full agent loop: sending to LLM, executing tool calls, streaming responses.

Image injection is prompt-local: OpenClaw loads image refs from the current prompt and
passes them via `images` cho lượt đó. Nó không quét lại các lượt lịch sử cũ hơn để tái chèn các tải trọng hình ảnh.
## Kiến trúc Công cụ

### Quy trình Công cụ

1. **Công cụ Cơ bản**: `codingTools` của pi (read, bash, edit, write)
2. **Thay thế Tùy chỉnh**: OpenClaw thay thế bash bằng `exec`/`process`, tùy chỉnh read/edit/write cho sandbox
3. **Công cụ OpenClaw**: messaging, browser, canvas, sessions, cron, gateway, v.v.
4. **Công cụ Kênh**: Công cụ hành động dành riêng cho Discord/Telegram/Slack/WhatsApp
5. **Lọc Chính sách**: Công cụ được lọc theo chính sách hồ sơ, nhà cung cấp, agent, nhóm, sandbox
6. **Chuẩn hóa Lược đồ**: Lược đồ được làm sạch cho các đặc điểm của Gemini/OpenAI
7. **Bao bọc AbortSignal**: Công cụ được bao bọc để tôn trọng các tín hiệu hủy bỏ

### Bộ Chuyển đổi Định nghĩa Công cụ

`AgentTool` của pi-agent-core có `execute` khác so với `ToolDefinition` của pi-coding-agent. Bộ chuyển đổi trong `pi-tool-definition-adapter.ts` cầu nối điều này:

```typescript
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // pi-coding-agent signature differs from pi-agent-core
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### Tool Split Strategy

`splitSdkTools()` passes all tools via `customTools`:

```typescript
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // Empty. We override everything
    customTools: toToolDefinitions(options.tools),
  };
}
```

Điều này đảm bảo lọc chính sách, tích hợp sandbox và bộ công cụ mở rộng của OpenClaw vẫn nhất quán trên các nhà cung cấp.
## Xây dựng System Prompt

System prompt được xây dựng trong `buildAgentSystemPrompt()` (`system-prompt.ts`). Nó lắp ráp một prompt đầy đủ với các phần bao gồm Tooling, Tool Call Style, Safety guardrails, OpenClaw CLI reference, Skills, Docs, Workspace, Sandbox, Messaging, Reply Tags, Voice, Silent Replies, Heartbeats, Runtime metadata, cộng với Memory và Reactions khi được bật, và các tệp ngữ cảnh tùy chọn và nội dung system prompt bổ sung. Các phần được cắt ngắn cho chế độ prompt tối thiểu được sử dụng bởi các subagents.

Prompt được áp dụng sau khi tạo phiên thông qua `applySystemPromptOverrideToSession()`:

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```
## Quản lý Phiên

### Tệp Phiên

Các phiên là tệp JSONL có cấu trúc cây (liên kết id/parentId). `SessionManager` của Pi xử lý tính bền vững:

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

OpenClaw wraps this with `guardSessionManager()` for tool result safety.

### Session Caching

`session-manager-cache.ts` caches SessionManager instances to avoid repeated file parsing:

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### History Limiting

`limitHistoryTurns()` trims conversation history based on channel type (DM vs group).

### Compaction

Auto-compaction triggers on context overflow. `compactEmbeddedPiSessionDirect()` handles manual compaction:

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```
## Xác thực & Phân giải Mô hình

### Hồ sơ Xác thực

OpenClaw duy trì một kho lưu trữ hồ sơ xác thực với nhiều khóa API cho mỗi nhà cung cấp:

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

Profiles rotate on failures with cooldown tracking:

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### Model Resolution

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// Uses pi's ModelRegistry and AuthStorage
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### Failover

`FailoverError` triggers model fallback when configured:

```typescript
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```
## Tiện ích mở rộng Pi

OpenClaw tải các tiện ích mở rộng pi tùy chỉnh để có hành vi chuyên biệt:

### Safeguard Nén dữ liệu

`src/agents/pi-extensions/compaction-safeguard.ts` thêm các biện pháp bảo vệ vào nén dữ liệu, bao gồm ngân sách token thích ứng cộng với tóm tắt lỗi công cụ và hoạt động tệp:

```typescript
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### Context Pruning

`src/agents/pi-extensions/context-pruning.ts` implements cache-TTL based context pruning:

```typescript
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```
## Truyền phát & Phản hồi theo khối

### Phân chia khối

`EmbeddedBlockChunker` quản lý truyền phát văn bản thành các khối phản hồi riêng biệt:

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### Thinking/Final Tag Stripping

Streaming output is processed to strip `<think>`/`<thinking>` blocks and extract `<final>` content:

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // Strip <think>...</think> content
  // If enforceFinalTag, only return <final>...</final> content
};
```

### Reply Directives

Reply directives like `[[media:url]]`, `[[voice]]`, `[[reply:id]]` are parsed and extracted:

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```
## Xử lý Lỗi

### Phân loại Lỗi

`pi-embedded-helpers.ts` phân loại các lỗi để xử lý phù hợp:

```typescript
isContextOverflowError(errorText)     // Context too large
isCompactionFailureError(errorText)   // Compaction failed
isAuthAssistantError(lastAssistant)   // Auth failure
isRateLimitAssistantError(...)        // Rate limited
isFailoverAssistantError(...)         // Should failover
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### Thinking Level Fallback

If a thinking level is unsupported, it falls back:

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```
## Tích hợp Sandbox

Khi chế độ sandbox được bật, các công cụ và đường dẫn bị hạn chế:

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // Use sandboxed read/edit/write tools
  // Exec runs in container
  // Browser uses bridge URL
}
```
## Xử lý Dành riêng cho Nhà cung cấp

### Anthropic

- Loại bỏ chuỗi ma thuật từ chối
- Xác thực lượt cho các vai trò liên tiếp
- Tương thích tham số Claude Code

### Google/Gemini

- Sửa chữa thứ tự lượt (`applyGoogleTurnOrderingFix`)
- Vệ sinh lược đồ công cụ (`sanitizeToolsForGoogle`)
- Vệ sinh lịch sử phiên (`sanitizeSessionHistory`)

### OpenAI

- `apply_patch` công cụ cho các mô hình Codex
- Xử lý hạ cấp mức tư duy
## Tích hợp TUI

OpenClaw cũng có chế độ TUI cục bộ sử dụng các thành phần pi-tui trực tiếp:

```typescript
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

Điều này cung cấp trải nghiệm terminal tương tác tương tự như chế độ gốc của Pi.
## Những Khác Biệt Chính so với Pi CLI

| Khía cạnh          | Pi CLI                  | OpenClaw Embedded                                                                              |
| --------------- | ----------------------- | ---------------------------------------------------------------------------------------------- |
| Gọi hàm      | `pi` command / RPC      | SDK via `createAgentSession()`                                                                 |
| Công cụ           | Công cụ mã hóa mặc định    | Bộ công cụ OpenClaw tùy chỉnh                                                                     |
| System prompt   | AGENTS.md + prompts     | Động theo kênh/ngữ cảnh                                                                    |
| Lưu trữ phiên | `~/.pi/agent/sessions/` | `~/.openclaw/agents/<agentId>/sessions/` (hoặc `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`) |
| Xác thực            | Thông tin xác thực đơn       | Nhiều hồ sơ với xoay vòng                                                                    |
| Tiện ích mở rộng      | Tải từ đĩa        | Lập trình + đường dẫn đĩa                                                                      |
| Xử lý sự kiện  | Hiển thị TUI           | Dựa trên callback (onBlockReply, v.v.)                                                            |
## Những Cân Nhắc Trong Tương Lai

Các lĩnh vực có khả năng cần làm lại:

1. **Căn chỉnh chữ ký công cụ**: Hiện đang thích ứng giữa các chữ ký của pi-agent-core và pi-coding-agent
2. **Bao bọc trình quản lý phiên**: `guardSessionManager` thêm tính an toàn nhưng tăng độ phức tạp
3. **Tải phần mở rộng**: Có thể sử dụng `ResourceLoader` của pi trực tiếp hơn
4. **Độ phức tạp của trình xử lý truyền phát**: `subscribeEmbeddedPiSession` đã phát triển lớn
5. **Đặc thù của nhà cung cấp**: Nhiều đường mã cụ thể cho từng nhà cung cấp mà pi có thể xử lý
## Kiểm thử

Phạm vi bao phủ tích hợp Pi bao gồm các bộ kiểm thử này:

- `src/agents/pi-*.test.ts`
- `src/agents/pi-auth-json.test.ts`
- `src/agents/pi-embedded-*.test.ts`
- `src/agents/pi-embedded-helpers*.test.ts`
- `src/agents/pi-embedded-runner*.test.ts`
- `src/agents/pi-embedded-runner/**/*.test.ts`
- `src/agents/pi-embedded-subscribe*.test.ts`
- `src/agents/pi-tools*.test.ts`
- `src/agents/pi-tool-definition-adapter*.test.ts`
- `src/agents/pi-settings.test.ts`
- `src/agents/pi-extensions/**/*.test.ts`

Trực tiếp/tùy chọn:

- `src/agents/pi-embedded-runner-extraparams.live.test.ts` (bật `OPENCLAW_LIVE_TEST=1`)

Để xem các lệnh chạy hiện tại, hãy xem [Quy trình phát triển Pi](/pi-dev).