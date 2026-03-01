---
summary: 'Các plugin/extension của OpenClaw: khám phá, cấu hình và an toàn'
read_when:
  - Thêm hoặc sửa đổi plugins/extensions
  - Quy tắc cài đặt hoặc tải plugin
title: Plugin
x-i18n:
  source_path: tools\plugin.md
  source_hash: 104d052fc10d21cc7066b29bb7e6c2264719d2df45422a3f5e47defd48a732d6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:32:46.904Z'
---

# Plugin (Tiện ích mở rộng)

## Bắt đầu nhanh (mới sử dụng plugin?)

Plugin chỉ là một **mô-đun mã nhỏ** mở rộng OpenClaw với các tính năng bổ sung (lệnh, công cụ và Gateway RPC).

Hầu hết thời gian, bạn sẽ sử dụng plugin khi bạn muốn một tính năng chưa được tích hợp sẵn trong OpenClaw cốt lõi (hoặc bạn muốn giữ các tính năng tùy chọn ngoài cài đặt chính của mình).

Đường dẫn nhanh:

1. Xem những gì đã được tải:

```bash
openclaw plugins list
```

2. Install an official plugin (example: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

Npm specs are **registry-only** (package name + optional version/tag). Git/URL/file
specs are rejected.

3. Restart the Gateway, then configure under `plugins.entries.<id>.config`.

Xem [Voice Call](/plugins/voice-call) để xem một ví dụ plugin cụ thể.
Đang tìm danh sách của bên thứ ba? Xem [Plugin cộng đồng](/plugins/community).
## Các plugin có sẵn (chính thức)

- Microsoft Teams chỉ là plugin kể từ 2026.1.15; cài đặt `@openclaw/msteams` nếu bạn sử dụng Teams.
- Memory (Core) — plugin tìm kiếm bộ nhớ được đóng gói (được bật theo mặc định qua `plugins.slots.memory`)
- Memory (LanceDB) — plugin bộ nhớ dài hạn được đóng gói (tự động gọi lại/ghi lại; đặt `plugins.slots.memory = "memory-lancedb"`)
- [Voice Call](/plugins/voice-call) — `@openclaw/voice-call`
- [Zalo Personal](/plugins/zalouser) — `@openclaw/zalouser`
- [Matrix](/channels/matrix) — `@openclaw/matrix`
- [Nostr](/channels/nostr) — `@openclaw/nostr`
- [Zalo](/channels/zalo) — `@openclaw/zalo`
- [Microsoft Teams](/channels/msteams) — `@openclaw/msteams`
- Google Antigravity OAuth (xác thực nhà cung cấp) — được đóng gói dưới dạng `google-antigravity-auth` (bị vô hiệu hóa theo mặc định)
- Gemini CLI OAuth (xác thực nhà cung cấp) — được đóng gói dưới dạng `google-gemini-cli-auth` (bị vô hiệu hóa theo mặc định)
- Qwen OAuth (xác thực nhà cung cấp) — được đóng gói dưới dạng `qwen-portal-auth` (bị vô hiệu hóa theo mặc định)
- Copilot Proxy (xác thực nhà cung cấp) — cầu nối local VS Code Copilot Proxy; khác biệt với đăng nhập thiết bị `github-copilot` được tích hợp sẵn (được đóng gói, bị vô hiệu hóa theo mặc định)

Các plugin OpenClaw là **các mô-đun TypeScript** được tải tại thời gian chạy qua jiti. **Xác thực cấu hình không thực thi mã plugin**; nó sử dụng bản kê khai plugin và JSON Schema thay thế. Xem [Plugin manifest](/plugins/manifest).

Các plugin có thể đăng ký:

- Các phương thức Gateway RPC
- Các trình xử lý HTTP Gateway
- Các công cụ agent
- Các lệnh CLI
- Các dịch vụ nền
- Xác thực cấu hình tùy chọn
- **Skills** (bằng cách liệt kê các thư mục `skills` trong bản kê khai plugin)
- **Các lệnh tự động trả lời** (thực thi mà không gọi agent AI)

Các plugin chạy **trong quy trình** với Gateway, vì vậy hãy coi chúng là mã đáng tin cậy.
Hướng dẫn tạo công cụ: [Plugin agent tools](/plugins/agent-tools).
## Trợ giúp Runtime

Các plugin có thể truy cập các trợ giúp cốt lõi được chọn thông qua `api.runtime`. Đối với TTS điện thoại:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes:

- Uses core `messages.tts` cấu hình (OpenAI hoặc ElevenLabs).
- Trả về bộ đệm âm thanh PCM + tốc độ mẫu. Các plugin phải lấy mẫu lại/mã hóa cho các nhà cung cấp.
- Edge TTS không được hỗ trợ cho điện thoại.
## Khám phá + giao thức truyền tải & thứ tự ưu tiên

OpenClaw quét, theo thứ tự:

1. Đường dẫn cấu hình

- `plugins.load.paths` (tệp hoặc thư mục)

2. Tiện ích mở rộng không gian làm việc

- `<workspace>/.openclaw/extensions/*.ts`
- `<workspace>/.openclaw/extensions/*/index.ts`

3. Tiện ích mở rộng toàn cục

- `~/.openclaw/extensions/*.ts`
- `~/.openclaw/extensions/*/index.ts`

4. Tiện ích mở rộng được đóng gói (được cung cấp với OpenClaw, **bị vô hiệu hóa theo mặc định**)

- `<openclaw>/extensions/*`

Plugin được đóng gói phải được bật rõ ràng thông qua `plugins.entries.<id>.enabled`
hoặc `openclaw plugins enable <id>`. Plugin được cài đặt được bật theo mặc định,
nhưng có thể bị vô hiệu hóa theo cách tương tự.

Ghi chú về tăng cường bảo mật:

- Nếu `plugins.allow` trống và plugin không được đóng gói có thể khám phá được, OpenClaw ghi lại cảnh báo khởi động với id plugin và nguồn.
- Các đường dẫn ứng cử được kiểm tra an toàn trước khi chấp nhận khám phá. OpenClaw chặn các ứng cử khi:
  - mục nhập tiện ích mở rộng phân giải bên ngoài gốc plugin (bao gồm thoát symlink/path traversal),
  - đường dẫn gốc/nguồn plugin có thể ghi được bởi mọi người,
  - quyền sở hữu đường dẫn đáng ngờ đối với plugin không được đóng gói (chủ sở hữu POSIX không phải là uid hiện tại cũng không phải root).
- Plugin không được đóng gói được tải mà không có nguồn gốc cài đặt/tải-đường dẫn phát ra cảnh báo để bạn có thể ghim tin tưởng (`plugins.allow`) hoặc cài đặt theo dõi (`plugins.installs`).

Mỗi plugin phải bao gồm tệp `openclaw.plugin.json` trong gốc của nó. Nếu một đường dẫn
trỏ đến một tệp, gốc plugin là thư mục của tệp và phải chứa
bản kê khai.

Nếu nhiều plugin phân giải thành cùng một id, kết quả khớp đầu tiên theo thứ tự trên
sẽ thắng và các bản sao có mức ưu tiên thấp hơn sẽ bị bỏ qua.

### Gói pack

Thư mục plugin có thể bao gồm `package.json` với `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Each entry becomes a plugin. If the pack lists multiple extensions, the plugin id
becomes `name/<fileBase>`.

If your plugin imports npm deps, install them in that directory so
`node_modules` is available (`npm install` / `pnpm install`).
Guardrail bảo mật: mỗi `openclaw.extensions` entry phải nằm trong thư mục plugin sau khi giải quyết symlink. Các entry thoát ra khỏi thư mục package sẽ bị từ chối.

Lưu ý bảo mật: `openclaw plugins install` cài đặt các phụ thuộc plugin với `npm install --ignore-scripts` (không có lifecycle scripts). Giữ các cây phụ thuộc plugin "pure JS/TS" và tránh các package yêu cầu `postinstall` builds.

### Siêu dữ liệu danh mục kênh

Các plugin kênh có thể quảng cáo siêu dữ liệu thiết lập ban đầu thông qua `openclaw.channel` và gợi ý cài đặt thông qua `openclaw.install`. Điều này giữ cho dữ liệu danh mục cốt lõi không bị ảnh hưởng.

Ví dụ:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Self-hosted chat via Nextcloud Talk webhook bots.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw can also merge **external channel catalogs** (for example, an MPM
registry export). Drop a JSON file at one of:

- `~/.openclaw/mpm/plugins.json`
- `~/.openclaw/mpm/catalog.json`
- `~/.openclaw/plugins/catalog.json`

Or point `OPENCLAW_PLUGIN_CATALOG_PATHS` (or `OPENCLAW_MPM_CATALOG_PATHS`) at
one or more JSON files (comma/semicolon/`PATH`-delimited). Each file should
contain `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.
## ID Plugin

ID plugin mặc định:

- Package packs: `package.json` `name`
- Tệp độc lập: tên cơ sở tệp (`~/.../voice-call.ts` → `voice-call`)

Nếu một plugin xuất `id`, OpenClaw sử dụng nó nhưng cảnh báo khi nó không khớp với
id được cấu hình.
## Cấu hình

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

Fields:

- `enabled`: master toggle (default: true)
- `allow`: allowlist (optional)
- `deny`: denylist (optional; deny wins)
- `load.paths`: extra plugin files/dirs
- `entries.<id>`: per‑plugin toggles + config

Config changes **require a gateway restart**.

Validation rules (strict):

- Unknown plugin ids in `entries`, `allow`, `deny`, or `slots` are **errors**.
- Unknown `channels.<id>` keys are **errors** unless a plugin manifest declares
  the channel id.
- Plugin config is validated using the JSON Schema embedded in
  `openclaw.plugin.json` (`configSchema`).
- Nếu một plugin bị vô hiệu hóa, cấu hình của nó được bảo toàn và một **cảnh báo** được phát hành.
## Khe cắm plugin (danh mục độc quyền)

Một số danh mục plugin là **độc quyền** (chỉ một hoạt động tại một thời điểm). Sử dụng
`plugins.slots` để chọn plugin nào sở hữu khe cắm:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core", // or "none" to disable memory plugins
    },
  },
}
```

If multiple plugins declare `kind: "memory"`, chỉ cái được chọn mới tải. Những cái khác
bị vô hiệu hóa với chẩn đoán.
## Control UI (schema + labels)

Control UI sử dụng `config.schema` (JSON Schema + `uiHints`) để hiển thị các biểu mẫu tốt hơn.

OpenClaw tăng cường `uiHints` tại thời điểm chạy dựa trên các plugin được khám phá:

- Thêm nhãn cho từng plugin cho `plugins.entries.<id>` / `.enabled` / `.config`
- Hợp nhất các gợi ý trường cấu hình do plugin cung cấp tùy chọn dưới:
  `plugins.entries.<id>.config.<field>`

Nếu bạn muốn các trường cấu hình plugin của mình hiển thị nhãn/trình giữ chỗ tốt (và đánh dấu các bí mật là nhạy cảm),
hãy cung cấp `uiHints` cùng với JSON Schema của bạn trong bản kê khai plugin.

Ví dụ:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "Region", "placeholder": "us-east-1" }
  }
}
```
## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copy a local file/dir into ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # relative path ok
openclaw plugins install ./plugin.tgz           # install from a local tarball
openclaw plugins install ./plugin.zip           # install from a local zip
openclaw plugins install -l ./extensions/voice-call # link (no copy) for dev
openclaw plugins install @openclaw/voice-call # install from npm
openclaw plugins install @openclaw/voice-call --pin # store exact resolved name@version
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` only works for npm installs tracked under `plugins.installs`.
If stored integrity metadata changes between updates, OpenClaw warns and asks for confirmation (use global `--yes` to bypass prompts).

Plugins may also register their own top‑level commands (example: `openclaw voicecall`).
## Plugin API (tổng quan)

Các plugin xuất ra một trong những cách sau:

- Một hàm: `(api) => { ... }`
- Một đối tượng: `{ id, name, configSchema, register(api) { ... } }`
## Plugin hooks

Các plugin có thể đăng ký hooks tại thời gian chạy. Điều này cho phép một plugin gói tự động hóa được điều khiển bởi sự kiện mà không cần cài đặt hook pack riêng biệt.

### Ví dụ

```ts
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // Hook logic here.
    },
    {
      name: "my-plugin.command-new",
      description: "Runs when /new is invoked",
    },
  );
}
```

Notes:

- Register hooks explicitly via `api.registerHook(...)`.
- Hook eligibility rules still apply (OS/bins/env/config requirements).
- Plugin-managed hooks show up in `openclaw hooks list` with `plugin:<id>`.
- You cannot enable/disable plugin-managed hooks via `openclaw hooks`; bật/tắt plugin thay thế.
## Plugin nhà cung cấp (xác thực mô hình)

Plugin có thể đăng ký **luồng xác thực nhà cung cấp mô hình** để người dùng có thể chạy OAuth hoặc
thiết lập khóa API bên trong OpenClaw (không cần các tập lệnh bên ngoài).

Đăng ký một nhà cung cấp thông qua `api.registerProvider(...)`. Mỗi nhà cung cấp hiển thị một
hoặc nhiều phương thức xác thực (OAuth, khóa API, mã thiết bị, v.v.). Các phương thức này cung cấp:

- `openclaw models auth login --provider <id> [--method <id>]`

Ví dụ:

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Run OAuth flow and return auth profiles.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notes:

- `run` receives a `ProviderAuthContext` with `prompter`, `runtime`,
  `openUrl`, and `oauth.createVpsAwareHandlers` helpers.
- Return `configPatch` when you need to add default models or provider config.
- Return `defaultModel` so `--set-default` can update agent defaults.

### Register a messaging channel

Plugins can register **channel plugins** that behave like built‑in channels
(WhatsApp, Telegram, etc.). Channel config lives under `channels.<id>` and is
validated by your channel plugin code.

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```
### Hướng dẫn thiết lập ban đầu kênh

Các plugin kênh có thể định nghĩa các hướng dẫn thiết lập ban đầu tùy chọn trên `plugin.onboarding`:

- `configure(ctx)` là luồng thiết lập cơ bản.
- `configureInteractive(ctx)` có thể hoàn toàn kiểm soát thiết lập tương tác cho cả trạng thái đã cấu hình và chưa cấu hình.
- `configureWhenConfigured(ctx)` có thể ghi đè hành vi chỉ cho các kênh đã được cấu hình.

Thứ tự ưu tiên trong trình hướng dẫn:

1. `configureInteractive` (nếu có)
2. `configureWhenConfigured` (chỉ khi trạng thái kênh đã được cấu hình)
3. quay lại `configure`

Chi tiết ngữ cảnh:

- `configureInteractive` và `configureWhenConfigured` nhận:
  - `configured` (`true` hoặc `false`)
  - `label` (tên kênh hướng tới người dùng được sử dụng bởi các lời nhắc)
  - cộng với các trường cấu hình/runtime/prompter/options được chia sẻ
- Trả về `"skip"` để lựa chọn và theo dõi tài khoản không thay đổi.
- Trả về `{ cfg, accountId? }` để áp dụng cập nhật cấu hình và ghi lại lựa chọn tài khoản.

### Viết một kênh nhắn tin mới (từng bước)

Sử dụng điều này khi bạn muốn một **bề mặt trò chuyện mới** (một "kênh nhắn tin"), không phải nhà cung cấp mô hình.
Tài liệu nhà cung cấp mô hình nằm dưới `/providers/*`.

1. Chọn một id + hình dạng cấu hình

- Tất cả cấu hình kênh nằm dưới `channels.<id>`.
- Ưu tiên `channels.<id>.accounts.<accountId>` cho các thiết lập đa tài khoản.

2. Định nghĩa siêu dữ liệu kênh

- `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` kiểm soát danh sách CLI/UI.
- `meta.docsPath` nên trỏ tới một trang tài liệu như `/channels/<id>`.
- `meta.preferOver` cho phép một plugin thay thế một kênh khác (tự động bật ưu tiên nó).
- `meta.detailLabel` và `meta.systemImage` được sử dụng bởi các UI cho văn bản chi tiết/biểu tượng.

3. Triển khai các bộ điều hợp bắt buộc

- `config.listAccountIds` + `config.resolveAccount`
- `capabilities` (loại trò chuyện, phương tiện, luồng, v.v.)
- `outbound.deliveryMode` + `outbound.sendText` (để gửi cơ bản)

4. Thêm các bộ điều hợp tùy chọn khi cần

- `setup` (trình hướng dẫn), `security` (chính sách tin nhắn riêng), `status` (sức khỏe/chẩn đoán)
- `gateway` (bắt đầu/dừng/đăng nhập), `mentions`, `threading`, `streaming`
- `actions` (hành động tin nhắn), `commands` (hành vi lệnh gốc)
5. Đăng ký kênh trong plugin của bạn

- `api.registerChannel({ plugin })`

Ví dụ cấu hình tối thiểu:

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true },
      },
    },
  },
}
```

Minimal channel plugin (outbound‑only):

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // deliver `text` to your channel here
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Load the plugin (extensions dir or `plugins.load.paths`), restart the gateway,
then configure `channels.<id>` trong cấu hình của bạn.

### Công cụ agent

Xem hướng dẫn chuyên dụng: [Công cụ agent plugin](/plugins/agent-tools).
### Đăng ký phương thức RPC của Gateway

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

### Register CLI commands

```ts
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### Register auto-reply commands

Plugins can register custom slash commands that execute **without invoking the
AI agent**. This is useful for toggle commands, status checks, or quick actions
that don't need LLM processing.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

Command handler context:

- `senderId`: The sender's ID (if available)
- `channel`: The channel where the command was sent
- `isAuthorizedSender`: Whether the sender is an authorized user
- `args`: Arguments passed after the command (if `acceptsArgs: true`)
- `commandBody`: The full command text
- `config`: The current OpenClaw config

Command options:

- `name`: Command name (without the leading `/`)
- `description`: Help text shown in command lists
- `acceptsArgs`: Whether the command accepts arguments (default: false). If false and arguments are provided, the command won't match and the message falls through to other handlers
- `requireAuth`: Whether to require authorized sender (default: true)
- `handler`: Function that returns `{ text: string }` (có thể là async)
Ví dụ với xác thực và đối số:

```ts
api.registerCommand({
  name: "setmode",
  description: "Set plugin mode",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Chế độ được đặt thành: ${mode}` };
  },
});
```

Notes:

- Plugin commands are processed **before** built-in commands and the AI agent
- Commands are registered globally and work across all channels
- Command names are case-insensitive (`/MyStatus` matches `/mystatus`)
- Command names must start with a letter and contain only letters, numbers, hyphens, and underscores
- Reserved command names (like `help`, `status`, `reset`, etc.) cannot be overridden by plugins
- Duplicate command registration across plugins will fail with a diagnostic error

### Register background services

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```
## Quy ước đặt tên

- Gateway methods: `pluginId.action` (ví dụ: `voicecall.status`)
- Tools: `snake_case` (ví dụ: `voice_call`)
- CLI commands: kebab hoặc camel, nhưng tránh xung đột với các lệnh cốt lõi
## Skills

Các plugin có thể cung cấp một Skill trong repo (`skills/<name>/SKILL.md`).
Bật nó bằng `plugins.entries.<id>.enabled` (hoặc các cổng cấu hình khác) và đảm bảo
nó có mặt trong các vị trí Skills được quản lý/workspace của bạn.
## Phân phối (npm)

Đóng gói được khuyến nghị:

- Gói chính: `openclaw` (kho này)
- Plugins: các gói npm riêng biệt dưới `@openclaw/*` (ví dụ: `@openclaw/voice-call`)

Hợp đồng xuất bản:

- Plugin `package.json` phải bao gồm `openclaw.extensions` với một hoặc nhiều tệp entry.
- Các tệp entry có thể là `.js` hoặc `.ts` (jiti tải TS tại thời điểm chạy).
- `openclaw plugins install <npm-spec>` sử dụng `npm pack`, trích xuất vào `~/.openclaw/extensions/<id>/`, và bật nó trong cấu hình.
- Ổn định khóa cấu hình: các gói có phạm vi được chuẩn hóa thành id **không có phạm vi** cho `plugins.entries.*`.
## Plugin ví dụ: Cuộc gọi thoại

Repo này bao gồm một plugin cuộc gọi thoại (Twilio hoặc fallback ghi nhật ký):

- Nguồn: `extensions/voice-call`
- Skill: `skills/voice-call`
- CLI: `openclaw voicecall start|status`
- Công cụ: `voice_call`
- RPC: `voicecall.start`, `voicecall.status`
- Cấu hình (twilio): `provider: "twilio"` + `twilio.accountSid/authToken/from` (tùy chọn `statusCallbackUrl`, `twimlUrl`)
- Cấu hình (dev): `provider: "log"` (không có mạng)

Xem [Cuộc gọi thoại](/plugins/voice-call) và `extensions/voice-call/README.md` để biết cách thiết lập và sử dụng.
## Ghi chú về bảo mật

Các plugin chạy trong cùng một tiến trình với Gateway. Hãy coi chúng là mã đáng tin cậy:

- Chỉ cài đặt các plugin mà bạn tin tưởng.
- Ưu tiên `plugins.allow` allowlists.
- Khởi động lại Gateway sau khi thay đổi.
## Kiểm tra plugins

Plugins có thể (và nên) đi kèm với các bài kiểm tra:

- Các plugins trong repo có thể giữ các bài kiểm tra Vitest dưới `src/**` (ví dụ: `src/plugins/voice-call.plugin.test.ts`).
- Các plugins được xuất bản riêng biệt nên chạy CI của riêng chúng (lint/build/test) và xác thực `openclaw.extensions` trỏ đến điểm vào được xây dựng (`dist/index.js`).