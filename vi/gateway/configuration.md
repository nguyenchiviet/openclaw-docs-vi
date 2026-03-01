---
summary: >-
  Tổng quan cấu hình: các tác vụ phổ biến, thiết lập nhanh và liên kết đến tài
  liệu tham khảo đầy đủ
read_when:
  - Thiết lập OpenClaw lần đầu tiên
  - Tìm kiếm các mẫu cấu hình phổ biến
  - Điều hướng đến các phần cấu hình cụ thể
title: Cấu hình
x-i18n:
  source_path: gateway\configuration.md
  source_hash: 0063bd36211014a347419ae85da32a0fe6f3e641baabd751edf9a04e70202295
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:57:08.208Z'
---

# Cấu hình

OpenClaw đọc một tệp cấu hình <Tooltip tip="JSON5 hỗ trợ nhận xét và dấu phẩy ở cuối">**JSON5**</Tooltip> tùy chọn từ `~/.openclaw/openclaw.json`.

Nếu tệp bị thiếu, OpenClaw sử dụng các giá trị mặc định an toàn. Những lý do phổ biến để thêm cấu hình:

- Kết nối các kênh và kiểm soát ai có thể gửi tin nhắn cho bot
- Đặt mô hình, công cụ, sandboxing hoặc tự động hóa (cron, hooks)
- Tinh chỉnh phiên, phương tiện, mạng hoặc giao diện người dùng

Xem [tài liệu tham khảo đầy đủ](/gateway/configuration-reference) để biết mọi trường có sẵn.

<Tip>
**Mới bắt đầu với cấu hình?** Bắt đầu với `openclaw onboard` để thiết lập tương tác, hoặc xem hướng dẫn [Ví dụ Cấu hình](/gateway/configuration-examples) để có các cấu hình sao chép đầy đủ.
</Tip>
## Cấu hình tối thiểu

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```
## Chỉnh sửa cấu hình

<Tabs>
  <Tab title="Trình hướng dẫn tương tác">
    ```bash
    openclaw onboard       # full setup wizard
    openclaw configure     # config wizard
    ```
  </Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    Open [http://127.0.0.1:18789](http://127.0.0.1:18789) and use the **Config** tab.
    The Control UI renders a form from the config schema, with a **Raw JSON** editor as an escape hatch.
  </Tab>
  <Tab title="Direct edit">
    Edit `~/.openclaw/openclaw.json` trực tiếp. Gateway theo dõi tệp và áp dụng các thay đổi tự động (xem [tải lại nóng](#config-hot-reload)).
  </Tab>
</Tabs>
## Xác thực nghiêm ngặt

<Warning>
OpenClaw chỉ chấp nhận các cấu hình hoàn toàn phù hợp với schema. Các khóa không xác định, kiểu dữ liệu không đúng định dạng hoặc giá trị không hợp lệ khiến Gateway **từ chối khởi động**. Ngoại lệ duy nhất ở cấp root là `$schema` (string), để các trình chỉnh sửa có thể đính kèm siêu dữ liệu JSON Schema.
</Warning>

Khi xác thực không thành công:

- Gateway không khởi động
- Chỉ các lệnh chẩn đoán hoạt động (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Chạy `openclaw doctor` để xem các vấn đề chính xác
- Chạy `openclaw doctor --fix` (hoặc `--yes`) để áp dụng các sửa chữa
## Các tác vụ phổ biến

<AccordionGroup>
  <Accordion title="Thiết lập kênh (WhatsApp, Telegram, Discord, v.v.)">
    Mỗi kênh có phần cấu hình riêng của nó dưới `channels.<provider>`. Xem trang kênh chuyên dụng để biết các bước thiết lập:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Tất cả các kênh chia sẻ cùng một mẫu chính sách tin nhắn riêng:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Choose and configure models">
    Set the primary model and optional fallbacks:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` defines the model catalog and acts as the allowlist for `/model`.
    - Model refs use `provider/model` format (e.g. `anthropic/claude-opus-4-6`).
    - `agents.defaults.imageMaxDimensionPx` controls transcript/tool image downscaling (default `1200`); các giá trị thấp hơn thường giảm mức sử dụng vision-token trên các lần chạy nặng ảnh chụp màn hình.
    - Xem [Models CLI](/concepts/models) để chuyển đổi mô hình trong trò chuyện và [Model Failover](/concepts/model-failover) để xoay vòng xác thực và hành vi dự phòng.
    - Đối với các nhà cung cấp tùy chỉnh/tự lưu trữ, xem [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) trong tài liệu tham khảo.

  </Accordion>
</AccordionGroup>
```
  <Accordion title="Kiểm soát ai có thể nhắn tin cho bot">
    Quyền truy cập DM được kiểm soát cho mỗi kênh thông qua `dmPolicy`:

    - `"pairing"` (mặc định): những người gửi không xác định nhận được mã ghép nối một lần để phê duyệt
    - `"allowlist"`: chỉ những người gửi trong `allowFrom` (hoặc kho lưu trữ cho phép được ghép nối)
    - `"open"`: cho phép tất cả DM đến (yêu cầu `allowFrom: ["*"]`)
    - `"disabled"`: bỏ qua tất cả DM

    Đối với nhóm, sử dụng `groupPolicy` + `groupAllowFrom` hoặc danh sách cho phép dành riêng cho kênh.

    Xem [tài liệu tham khảo đầy đủ](/gateway/configuration-reference#dm-and-group-access) để biết chi tiết từng kênh.

  </Accordion>

  <Accordion title="Thiết lập gating nhắc đến trong trò chuyện nhóm">
    Tin nhắn nhóm mặc định yêu cầu **nhắc đến**. Cấu hình các mẫu cho mỗi agent:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Metadata mentions**: native @-mentions (WhatsApp tap-to-mention, Telegram @bot, etc.)
    - **Text patterns**: regex patterns in `mentionPatterns`
    - See [full reference](/gateway/configuration-reference#group-chat-mention-gating) for per-channel overrides and self-chat mode.

  </Accordion>

  <Accordion title="Configure sessions and resets">
    Sessions control conversation continuity and isolation:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // recommended for multi-user
        threadBindings: {
          enabled: true,
          idleHours: 24,
          maxAgeHours: 0,
        },
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

  </Accordion>
```
- `dmScope`: `main` (được chia sẻ) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
- `threadBindings`: giá trị mặc định toàn cục cho định tuyến phiên liên kết với luồng (Discord hỗ trợ `/focus`, `/unfocus`, `/agents`, `/session idle`, và `/session max-age`).
- Xem [Quản lý phiên](/concepts/session) để biết về phạm vi, liên kết danh tính và chính sách gửi.
- Xem [tham chiếu đầy đủ](/gateway/configuration-reference#session) để biết tất cả các trường.

  </Accordion>

  <Accordion title="Bật sandboxing">
    Chạy phiên agent trong các container Docker cách ly:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    Build the image first: `scripts/sandbox-setup.sh`

    See [Sandboxing](/gateway/sandboxing) for the full guide and [full reference](/gateway/configuration-reference#sandbox) for all options.

  </Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: duration string (`30m`, `2h`). Set `0m` to disable.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - `directPolicy`: `allow` (default) or `block` for DM-style heartbeat targets
    - See [Heartbeat](/gateway/heartbeat) for the full guide.

  </Accordion>

  <Accordion title="Configure cron jobs">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
        runLog: {
          maxBytes: "2mb",
          keepLines: 2000,
        },
      },
    }
    ```
- `sessionRetention`: loại bỏ các phiên chạy cô lập đã hoàn thành khỏi `sessions.json` (mặc định `24h`; đặt `false` để tắt).
    - `runLog`: loại bỏ `cron/runs/<jobId>.jsonl` theo kích thước và dòng được giữ lại.
    - Xem [Cron jobs](/automation/cron-jobs) để xem tổng quan tính năng và ví dụ CLI.

  </Accordion>

  <Accordion title="Thiết lập webhooks (hooks)">
    Bật các điểm cuối webhook HTTP trên Gateway:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    See [full reference](/gateway/configuration-reference#hooks) for all mapping options and Gmail integration.

  </Accordion>

  <Accordion title="Configure multi-agent routing">
    Run multiple isolated agents with separate workspaces and sessions:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    See [Multi-Agent](/concepts/multi-agent) and [full reference](/gateway/configuration-reference#multi-agent-routing) for binding rules and per-agent access profiles.

  </Accordion>

  <Accordion title="Split config into multiple files ($include)">
    Use `$include` để tổ chức các cấu hình lớn:
```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

- **Tệp đơn**: thay thế đối tượng chứa nó
- **Mảng tệp**: được hợp nhất sâu theo thứ tự (giá trị sau cùng thắng)
- **Khóa anh chị em**: được hợp nhất sau khi bao gồm (ghi đè các giá trị được bao gồm)
- **Bao gồm lồng nhau**: được hỗ trợ lên đến 10 cấp độ sâu
- **Đường dẫn tương đối**: được phân giải tương đối với tệp bao gồm
- **Xử lý lỗi**: các lỗi rõ ràng cho các tệp bị thiếu, lỗi phân tích cú pháp và bao gồm vòng tròn

  </Accordion>
</AccordionGroup>
## Tải lại cấu hình nóng

Gateway theo dõi `~/.openclaw/openclaw.json` và áp dụng các thay đổi tự động — không cần khởi động lại thủ công cho hầu hết các cài đặt.

### Chế độ tải lại

| Chế độ                   | Hành vi                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (mặc định) | Áp dụng nóng các thay đổi an toàn ngay lập tức. Tự động khởi động lại cho những thay đổi quan trọng.           |
| **`hot`**              | Chỉ áp dụng nóng các thay đổi an toàn. Ghi nhật ký cảnh báo khi cần khởi động lại — bạn xử lý. |
| **`restart`**          | Khởi động lại Gateway khi có bất kỳ thay đổi cấu hình nào, an toàn hay không.                                 |
| **`off`**              | Vô hiệu hóa theo dõi tệp. Các thay đổi có hiệu lực khi khởi động lại thủ công tiếp theo.                 |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### What hot-applies vs what needs a restart

Most fields hot-apply without downtime. In `hybrid` mode, restart-required changes are handled automatically.

| Category            | Fields                                                               | Restart needed? |
| ------------------- | -------------------------------------------------------------------- | --------------- |
| Channels            | `channels.*`, `web` (WhatsApp) — all built-in and extension channels | No              |
| Agent & models      | `agent`, `agents`, `models`, `routing`                               | No              |
| Automation          | `hooks`, `cron`, `agent.heartbeat`                                   | No              |
| Sessions & messages | `session`, `messages`                                                | No              |
| Tools & media       | `tools`, `browser`, `skills`, `audio`, `talk`                        | No              |
| UI & misc           | `ui`, `logging`, `identity`, `bindings`                              | No              |
| Gateway server      | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                 | **Yes**         |
| Infrastructure      | `discovery`, `canvasHost`, `plugins`                                 | **Yes**         |

<Note>
`gateway.reload` and `gateway.remote` là những ngoại lệ — thay đổi chúng **không** kích hoạt khởi động lại.
</Note>
## Config RPC (cập nhật lập trình)

<Note>
Control-plane write RPCs (`config.apply`, `config.patch`, `update.run`) bị giới hạn tốc độ ở **3 yêu cầu trên 60 giây** trên mỗi `deviceId+clientIp`. Khi bị giới hạn, RPC trả về `UNAVAILABLE` với `retryAfterMs`.
</Note>

<AccordionGroup>
  <Accordion title="config.apply (thay thế toàn bộ)">
    Xác thực + ghi toàn bộ cấu hình và khởi động lại Gateway trong một bước.

    <Warning>
    `config.apply` thay thế **toàn bộ cấu hình**. Sử dụng `config.patch` để cập nhật một phần, hoặc `openclaw config set` cho các khóa riêng lẻ.
    </Warning>

    Tham số:

    - `raw` (string) — JSON5 payload cho toàn bộ cấu hình
    - `baseHash` (tùy chọn) — hash cấu hình từ `config.get` (bắt buộc khi cấu hình tồn tại)
    - `sessionKey` (tùy chọn) — khóa phiên cho ping wake-up sau khi khởi động lại
    - `note` (tùy chọn) — ghi chú cho sentinel khởi động lại
    - `restartDelayMs` (tùy chọn) — độ trễ trước khi khởi động lại (mặc định 2000)

    Các yêu cầu khởi động lại được hợp nhất trong khi một yêu cầu đang chờ/đang thực hiện, và thời gian chờ 30 giây được áp dụng giữa các chu kỳ khởi động lại.

    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```

  </Accordion>

  <Accordion title="config.patch (partial update)">
    Merges a partial update into the existing config (JSON merge patch semantics):

    - Objects merge recursively
    - `null` deletes a key
    - Arrays replace

    Params:

    - `raw` (string) — JSON5 with just the keys to change
    - `baseHash` (required) — config hash from `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — same as `config.apply`

    Restart behavior matches `config.apply`: coalesced pending restarts plus a 30-second cooldown between restart cycles.

    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```

  </Accordion>
</AccordionGroup>
## Biến môi trường

OpenClaw đọc biến môi trường từ quy trình cha cũng như:

- `.env` từ thư mục làm việc hiện tại (nếu có)
- `~/.openclaw/.env` (fallback toàn cục)

Không có tệp nào ghi đè biến môi trường hiện có. Bạn cũng có thể đặt biến môi trường nội tuyến trong cấu hình:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  If enabled and expected keys aren't set, OpenClaw runs your login shell and imports only the missing keys:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var equivalent: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Env var substitution in config values">
  Reference env vars in any config string value with `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Rules:

- Only uppercase names matched: `[A-Z_][A-Z0-9_]*`
- Missing/empty vars throw an error at load time
- Escape with `$${VAR}` for literal output
- Works inside `$include` files
- Inline substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

<Accordion title="Secret refs (env, file, exec)">
  For fields that support SecretRef objects, you can use:

```json5
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```
Chi tiết SecretRef (bao gồm `secrets.providers` cho `env`/`file`/`exec`) có trong [Quản lý Bí mật](/gateway/secrets).
</Accordion>

Xem [Môi trường](/help/environment) để biết đầy đủ thứ tự ưu tiên và nguồn.
## Tham chiếu đầy đủ

Để xem tham chiếu đầy đủ từng trường, hãy xem **[Tham chiếu Cấu hình](/gateway/configuration-reference)**.

---

_Liên quan: [Ví dụ Cấu hình](/gateway/configuration-examples) · [Tham chiếu Cấu hình](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_