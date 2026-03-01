---
summary: Gmail Pub/Sub push được kết nối vào OpenClaw webhooks thông qua gogcli
read_when:
  - Kết nối các trigger hộp thư đến Gmail với OpenClaw
  - Thiết lập Pub/Sub push để đánh thức agent
title: Gmail PubSub
x-i18n:
  source_path: automation\gmail-pubsub.md
  source_hash: 0c8a87516e12091f96209f570012cdba895265af3d48ba848e0260535535bd18
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:10:52.459Z'
---

# Gmail Pub/Sub -> OpenClaw

Mục tiêu: Gmail watch -> Pub/Sub push -> `gog gmail watch serve` -> OpenClaw webhook.
## Yêu cầu tiên quyết

- `gcloud` đã cài đặt và đăng nhập ([hướng dẫn cài đặt](https://docs.cloud.google.com/sdk/docs/install-sdk)).
- `gog` (gogcli) đã cài đặt và được ủy quyền cho tài khoản Gmail ([gogcli.sh](https://gogcli.sh/)).
- Hooks OpenClaw đã được bật (xem [Webhooks](/automation/webhook)).
- `tailscale` đã đăng nhập ([tailscale.com](https://tailscale.com/)). Thiết lập được hỗ trợ sử dụng Tailscale Funnel cho điểm cuối HTTPS công khai.
  Các dịch vụ tunnel khác có thể hoạt động, nhưng là DIY/không được hỗ trợ và yêu cầu kết nối thủ công.
  Hiện tại, Tailscale là những gì chúng tôi hỗ trợ.

Ví dụ cấu hình hook (bật ánh xạ preset Gmail):

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

To deliver the Gmail summary to a chat surface, override the preset with a mapping
that sets `deliver` + optional `channel`/`to`:

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

If you want a fixed channel, set `channel` + `to`. Otherwise `channel: "last"`
uses the last delivery route (falls back to WhatsApp).

To force a cheaper model for Gmail runs, set `model` in the mapping
(`provider/model` or alias). If you enforce `agents.defaults.models`, include it there.

To set a default model and thinking level specifically for Gmail hooks, add
`hooks.gmail.model` / `hooks.gmail.thinking` in your config:

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```
Ghi chú:

- `model`/`thinking` cho từng hook trong ánh xạ vẫn ghi đè các giá trị mặc định này.
- Thứ tự dự phòng: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → chính (auth/rate-limit/timeouts).
- Nếu `agents.defaults.models` được đặt, mô hình Gmail phải có trong danh sách cho phép.
- Nội dung hook Gmail được bao bọc với các ranh giới an toàn nội dung bên ngoài theo mặc định.
  Để vô hiệu hóa (nguy hiểm), đặt `hooks.gmail.allowUnsafeExternalContent: true`.

Để tùy chỉnh xử lý payload thêm, thêm `hooks.mappings` hoặc một module chuyển đổi JS/TS
dưới `~/.openclaw/hooks/transforms` (xem [Webhooks](/automation/webhook)).
## Trình hướng dẫn (khuyến nghị)

Sử dụng trình trợ giúp OpenClaw để kết nối mọi thứ với nhau (cài đặt các phụ thuộc trên macOS qua brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Defaults:

- Uses Tailscale Funnel for the public push endpoint.
- Writes `hooks.gmail` config for `openclaw webhooks gmail run`.
- Enables the Gmail hook preset (`hooks.presets: ["gmail"]`).

Path note: when `tailscale.mode` is enabled, OpenClaw automatically sets
`hooks.gmail.serve.path` to `/` and keeps the public path at
`hooks.gmail.tailscale.path` (default `/gmail-pubsub`) because Tailscale
strips the set-path prefix before proxying.
If you need the backend to receive the prefixed path, set
`hooks.gmail.tailscale.target` (or `--tailscale-target`) to a full URL like
`http://127.0.0.1:8788/gmail-pubsub` and match `hooks.gmail.serve.path`.

Want a custom endpoint? Use `--push-endpoint <url>` or `--tailscale off`.

Platform note: on macOS the wizard installs `gcloud`, `gogcli`, and `tailscale`
via Homebrew; on Linux install them manually first.

Gateway auto-start (recommended):

- When `hooks.enabled=true` and `hooks.gmail.account` is set, the Gateway starts
  `gog gmail watch serve` on boot and auto-renews the watch.
- Set `OPENCLAW_SKIP_GMAIL_WATCHER=1` to opt out (useful if you run the daemon yourself).
- Do not run the manual daemon at the same time, or you will hit
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Manual daemon (starts `gog gmail watch serve` + auto-renew):

```bash
openclaw webhooks gmail run
```
## Thiết lập một lần

1. Chọn dự án GCP **sở hữu OAuth client** được sử dụng bởi `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Note: Gmail watch requires the Pub/Sub topic to live in the same project as the OAuth client.

2. Enable APIs:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Create a topic:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Allow Gmail push to publish:

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```
## Bắt đầu theo dõi

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Save the `history_id` từ đầu ra (để gỡ lỗi).
## Chạy trình xử lý push

Ví dụ cục bộ (xác thực token chia sẻ):

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Notes:

- `--token` protects the push endpoint (`x-gog-token` or `?token=`).
- `--hook-url` points to OpenClaw `/hooks/gmail` (mapped; isolated run + summary to main).
- `--include-body` and `--max-bytes` control the body snippet sent to OpenClaw.

Recommended: `openclaw webhooks gmail run` bao bọc cùng một luồng và tự động gia hạn watch.
## Expose the handler (nâng cao, không được hỗ trợ)

Nếu bạn cần một đường hầm không phải Tailscale, hãy kết nối thủ công và sử dụng URL công khai trong
đăng ký push (không được hỗ trợ, không có biện pháp bảo vệ):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Use the generated URL as the push endpoint:

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Production: use a stable HTTPS endpoint and configure Pub/Sub OIDC JWT, then run:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```
## Kiểm tra

Gửi tin nhắn đến hộp thư được theo dõi:

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Check watch state and history:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```
## Khắc phục sự cố

- `Invalid topicName`: dự án không khớp (chủ đề không có trong dự án OAuth client).
- `User not authorized`: thiếu `roles/pubsub.publisher` trên chủ đề.
- Tin nhắn trống: Gmail push chỉ cung cấp `historyId`; tải về qua `gog gmail history`.
## Dọn dẹp

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```