---
summary: 'Tài liệu tham khảo OpenClaw CLI cho các lệnh `openclaw`, lệnh con và tùy chọn'
read_when:
  - Thêm hoặc sửa đổi các lệnh hoặc tùy chọn CLI
  - Ghi chép các bề mặt lệnh mới
title: Tham Chiếu CLI
x-i18n:
  source_path: cli\index.md
  source_hash: 07513dc7c8d1a7d4ad3f174f87a3764dfaeab5193a6be8688a4839d2e43d4d53
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:36:28.049Z'
---

# Tham chiếu CLI

Trang này mô tả hành vi CLI hiện tại. Nếu các lệnh thay đổi, hãy cập nhật tài liệu này.
## Trang lệnh

- [`setup`](/cli/setup)
- [`onboard`](/cli/onboard)
- [`configure`](/cli/configure)
- [`config`](/cli/config)
- [`completion`](/cli/completion)
- [`doctor`](/cli/doctor)
- [`dashboard`](/cli/dashboard)
- [`reset`](/cli/reset)
- [`uninstall`](/cli/uninstall)
- [`update`](/cli/update)
- [`message`](/cli/message)
- [`agent`](/cli/agent)
- [`agents`](/cli/agents)
- [`acp`](/cli/acp)
- [`status`](/cli/status)
- [`health`](/cli/health)
- [`sessions`](/cli/sessions)
- [`gateway`](/cli/gateway)
- [`logs`](/cli/logs)
- [`system`](/cli/system)
- [`models`](/cli/models)
- [`memory`](/cli/memory)
- [`directory`](/cli/directory)
- [`nodes`](/cli/nodes)
- [`devices`](/cli/devices)
- [`node`](/cli/node)
- [`approvals`](/cli/approvals)
- [`sandbox`](/cli/sandbox)
- [`tui`](/cli/tui)
- [`browser`](/cli/browser)
- [`cron`](/cli/cron)
- [`dns`](/cli/dns)
- [`docs`](/cli/docs)
- [`hooks`](/cli/hooks)
- [`webhooks`](/cli/webhooks)
- [`pairing`](/cli/pairing)
- [`qr`](/cli/qr)
- [`plugins`](/cli/plugins) (lệnh plugin)
- [`channels`](/cli/channels)
- [`security`](/cli/security)
- [`secrets`](/cli/secrets)
- [`skills`](/cli/skills)
- [`daemon`](/cli/daemon) (bí danh cũ cho lệnh dịch vụ Gateway)
- [`clawbot`](/cli/clawbot) (không gian tên bí danh cũ)
- [`voicecall`](/cli/voicecall) (plugin; nếu được cài đặt)
## Cờ toàn cục

- `--dev`: cô lập trạng thái dưới `~/.openclaw-dev` và thay đổi cổng mặc định.
- `--profile <name>`: cô lập trạng thái dưới `~/.openclaw-<name>`.
- `--no-color`: vô hiệu hóa màu ANSI.
- `--update`: viết tắt cho `openclaw update` (chỉ cài đặt từ nguồn).
- `-V`, `--version`, `-v`: in phiên bản và thoát.
## Kiểu dáng đầu ra

- Màu ANSI và chỉ báo tiến độ chỉ hiển thị trong các phiên TTY.
- Siêu liên kết OSC-8 hiển thị dưới dạng liên kết có thể nhấp trong các terminal được hỗ trợ; nếu không, chúng tôi sẽ quay lại các URL thuần túy.
- `--json` (và `--plain` nếu được hỗ trợ) vô hiệu hóa kiểu dáng để có đầu ra sạch sẽ.
- `--no-color` vô hiệu hóa kiểu dáng ANSI; `NO_COLOR=1` cũng được tôn trọng.
- Các lệnh chạy lâu dài hiển thị chỉ báo tiến độ (OSC 9;4 khi được hỗ trợ).
## Bảng màu

OpenClaw sử dụng bảng màu tôm hùm cho đầu ra CLI.

- `accent` (#FF5A2D): tiêu đề, nhãn, điểm nhấn chính.
- `accentBright` (#FF7A3D): tên lệnh, nhấn mạnh.
- `accentDim` (#D14A22): văn bản điểm nhấn thứ cấp.
- `info` (#FF8A5B): giá trị thông tin.
- `success` (#2FBF71): trạng thái thành công.
- `warn` (#FFB020): cảnh báo, dự phòng, chú ý.
- `error` (#E23D2D): lỗi, thất bại.
- `muted` (#8B7F77): giảm nhấn, siêu dữ liệu.

Nguồn sự thật của bảng màu: `src/terminal/palette.ts` (hay còn gọi là "đường may tôm hùm").
## Cây lệnh

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```
Lưu ý: các plugin có thể thêm các lệnh cấp cao nhất bổ sung (ví dụ `openclaw voicecall`).

## Bảo mật

- `openclaw security audit` — kiểm tra cấu hình + trạng thái cục bộ để tìm các lỗ hổng bảo mật phổ biến.
- `openclaw security audit --deep` — kiểm tra Gateway trực tiếp theo cách tốt nhất.
- `openclaw security audit --fix` — tăng cường các giá trị mặc định an toàn và chmod trạng thái/cấu hình.
## Bí mật

- `openclaw secrets reload` — giải quyết lại các tham chiếu và hoán đổi nguyên tử ảnh chụp thời gian chạy.
- `openclaw secrets audit` — quét các phần dư văn bản thuần túy, các tham chiếu chưa giải quyết và sự trôi dạo ưu tiên.
- `openclaw secrets configure` — trình hỗ trợ tương tác để thiết lập nhà cung cấp + ánh xạ SecretRef + kiểm tra trước/áp dụng.
- `openclaw secrets apply --from <plan.json>` — áp dụng một kế hoạch được tạo trước đó (`--dry-run` được hỗ trợ).
## Plugin

Quản lý các tiện ích mở rộng và cấu hình của chúng:

- `openclaw plugins list` — khám phá plugin (sử dụng `--json` để xuất dữ liệu máy).
- `openclaw plugins info <id>` — hiển thị chi tiết cho một plugin.
- `openclaw plugins install <path|.tgz|npm-spec>` — cài đặt một plugin (hoặc thêm đường dẫn plugin vào `plugins.load.paths`).
- `openclaw plugins enable <id>` / `disable <id>` — bật/tắt `plugins.entries.<id>.enabled`.
- `openclaw plugins doctor` — báo cáo lỗi tải plugin.

Hầu hết các thay đổi plugin yêu cầu khởi động lại gateway. Xem [/plugin](/tools/plugin).
## Bộ nhớ

Tìm kiếm vector trên `MEMORY.md` + `memory/*.md`:

- `openclaw memory status` — hiển thị thống kê chỉ mục.
- `openclaw memory index` — lập chỉ mục lại các tệp bộ nhớ.
- `openclaw memory search "<query>"` (hoặc `--query "<query>"`) — tìm kiếm ngữ nghĩa trên bộ nhớ.
## Lệnh slash trong chat

Tin nhắn chat hỗ trợ `/...` commands (text và native). Xem [/tools/slash-commands](/tools/slash-commands).

Điểm nổi bật:

- `/status` để chẩn đoán nhanh.
- `/config` để thay đổi cấu hình lâu dài.
- `/debug` để ghi đè cấu hình chỉ lúc chạy (bộ nhớ, không phải đĩa; yêu cầu `commands.debug: true`).
## Thiết lập + thiết lập ban đầu

### `setup`

Khởi tạo cấu hình + không gian làm việc.

Tùy chọn:

- `--workspace <dir>`: đường dẫn không gian làm việc agent (mặc định `~/.openclaw/workspace`).
- `--wizard`: chạy trình hướng dẫn thiết lập ban đầu.
- `--non-interactive`: chạy trình hướng dẫn mà không cần nhắc.
- `--mode <local|remote>`: chế độ trình hướng dẫn.
- `--remote-url <url>`: URL Gateway từ xa.
- `--remote-token <token>`: token Gateway từ xa.

Trình hướng dẫn tự động chạy khi có bất kỳ cờ trình hướng dẫn nào (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### `onboard`

Trình hướng dẫn tương tác để thiết lập gateway, không gian làm việc và Skills.

Tùy chọn:

- `--workspace <dir>`
- `--reset` (đặt lại cấu hình + thông tin xác thực + phiên trước khi chạy trình hướng dẫn)
- `--reset-scope <config|config+creds+sessions|full>` (mặc định `config+creds+sessions`; sử dụng `full` để cũng xóa không gian làm việc)
- `--non-interactive`
- `--mode <local|remote>`
- `--flow <quickstart|advanced|manual>` (manual là bí danh cho advanced)
- `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
- `--token-provider <id>` (không tương tác; được sử dụng với `--auth-choice token`)
- `--token <token>` (không tương tác; được sử dụng với `--auth-choice token`)
- `--token-profile-id <id>` (không tương tác; mặc định: `<provider>:manual`)
- `--token-expires-in <duration>` (không tương tác; ví dụ `365d`, `12h`)
- `--secret-input-mode <plaintext|ref>` (mặc định `plaintext`; sử dụng `ref` để lưu trữ tham chiếu biến môi trường nhà cung cấp mặc định thay vì khóa văn bản thuần)
- `--anthropic-api-key <key>`
- `--openai-api-key <key>`
- `--mistral-api-key <key>`
- `--openrouter-api-key <key>`
- `--ai-gateway-api-key <key>`
- `--moonshot-api-key <key>`
- `--kimi-code-api-key <key>`
- `--gemini-api-key <key>`
- `--zai-api-key <key>`
- `--minimax-api-key <key>`
- `--opencode-zen-api-key <key>`
- `--custom-base-url <url>` (không tương tác; được sử dụng với `--auth-choice custom-api-key`)
- `--custom-model-id <id>` (không tương tác; được sử dụng với `--auth-choice custom-api-key`)
- `--custom-api-key <key>` (không tương tác; tùy chọn; được sử dụng với `--auth-choice custom-api-key`; quay lại `CUSTOM_API_KEY` khi bị bỏ qua)
- `--custom-provider-id <id>` (không tương tác; id nhà cung cấp tùy chỉnh tùy chọn)
- `--custom-compatibility <openai|anthropic>` (không tương tác; tùy chọn; mặc định `openai`)
- `--gateway-port <port>`
- `--gateway-bind <loopback|lan|tailnet|auto|custom>`
- `--gateway-auth <token|password>`
- `--gateway-token <token>`
- `--gateway-password <password>`
- `--remote-url <url>`
- `--remote-token <token>`
- `--tailscale <off|serve|funnel>`
- `--tailscale-reset-on-exit`
- `--install-daemon`
- `--no-install-daemon` (bí danh: `--skip-daemon`)
- `--daemon-runtime <node|bun>`
- `--skip-channels`
- `--skip-skills`
- `--skip-health`
- `--skip-ui`
- `--node-manager <npm|pnpm|bun>` (khuyến nghị pnpm; không khuyến nghị bun cho Gateway runtime)
- `--json`

### `configure`

Trình hướng dẫn cấu hình tương tác (mô hình, kênh, Skills, Gateway).

### `config`

Trợ giúp cấu hình không tương tác (get/set/unset). Chạy `openclaw config` mà không có
lệnh con sẽ khởi chạy trình hướng dẫn.

Lệnh con:

- `config get <path>`: in giá trị cấu hình (đường dẫn dấu chấm/dấu ngoặc).
- `config set <path> <value>`: đặt giá trị (JSON5 hoặc chuỗi thô).
- `config unset <path>`: xóa giá trị.

### `doctor`

Kiểm tra sức khỏe + sửa chữa nhanh (cấu hình + Gateway + dịch vụ cũ).

Tùy chọn:

- `--no-workspace-suggestions`: tắt gợi ý bộ nhớ không gian làm việc.
- `--yes`: chấp nhận mặc định mà không nhắc (chế độ không đầu).
- `--non-interactive`: bỏ qua lời nhắc; chỉ áp dụng các di chuyển an toàn.
- `--deep`: quét dịch vụ hệ thống để tìm các cài đặt Gateway bổ sung.
## Trợ giúp kênh

### `channels`

Quản lý tài khoản kênh trò chuyện (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams).

Lệnh con:

- `channels list`: hiển thị các kênh được cấu hình và hồ sơ xác thực.
- `channels status`: kiểm tra khả năng tiếp cận gateway và tình trạng kênh (`--probe` chạy các kiểm tra bổ sung; sử dụng `openclaw health` hoặc `openclaw status --deep` để kiểm tra tình trạng gateway).
- Mẹo: `channels status` in ra cảnh báo với các sửa chữa được đề xuất khi nó có thể phát hiện các cấu hình sai phổ biến (sau đó hướng bạn đến `openclaw doctor`).
- `channels logs`: hiển thị nhật ký kênh gần đây từ tệp nhật ký gateway.
- `channels add`: thiết lập theo kiểu trình hướng dẫn khi không có cờ nào được truyền; các cờ chuyển sang chế độ không tương tác.
  - Khi thêm tài khoản không phải mặc định vào kênh vẫn sử dụng cấu hình cấp cao nhất tài khoản đơn, OpenClaw di chuyển các giá trị có phạm vi tài khoản vào `channels.<channel>.accounts.default` trước khi viết tài khoản mới.
  - `channels add` không tương tác không tự động tạo/nâng cấp liên kết; các liên kết chỉ kênh tiếp tục khớp với tài khoản mặc định.
- `channels remove`: tắt theo mặc định; chuyển `--delete` để xóa các mục cấu hình mà không cần nhắc.
- `channels login`: đăng nhập kênh tương tác (WhatsApp Web chỉ).
- `channels logout`: đăng xuất khỏi phiên kênh (nếu được hỗ trợ).

Tùy chọn chung:

- `--channel <name>`: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
- `--account <id>`: id tài khoản kênh (mặc định `default`)
- `--name <label>`: tên hiển thị cho tài khoản

Tùy chọn `channels login`:

- `--channel <channel>` (mặc định `whatsapp`; hỗ trợ `whatsapp`/`web`)
- `--account <id>`
- `--verbose`

Tùy chọn `channels logout`:

- `--channel <channel>` (mặc định `whatsapp`)
- `--account <id>`

Tùy chọn `channels list`:

- `--no-usage`: bỏ qua ảnh chụp nhanh sử dụng/hạn ngạch nhà cung cấp mô hình (chỉ OAuth/API).
- `--json`: xuất JSON (bao gồm sử dụng trừ khi `--no-usage` được đặt).

Tùy chọn `channels logs`:

- `--channel <name|all>` (mặc định `all`)
- `--lines <n>` (mặc định `200`)
- `--json`

Chi tiết hơn: [/concepts/oauth](/concepts/oauth)

Ví dụ:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### `skills`
Liệt kê và kiểm tra các Skills có sẵn cùng thông tin sẵn sàng.

Các lệnh con:

- `skills list`: liệt kê các Skills (mặc định khi không có lệnh con).
- `skills info <name>`: hiển thị chi tiết cho một Skill.
- `skills check`: tóm tắt các yêu cầu sẵn sàng so với thiếu.

Tùy chọn:

- `--eligible`: chỉ hiển thị các Skills sẵn sàng.
- `--json`: xuất JSON (không có kiểu dáng).
- `-v`, `--verbose`: bao gồm chi tiết yêu cầu thiếu.

Mẹo: sử dụng `npx clawhub` để tìm kiếm, cài đặt và đồng bộ hóa các Skills.

### `pairing`

Phê duyệt các yêu cầu ghép nối tin nhắn riêng trên các kênh.

Các lệnh con:

- `pairing list [channel] [--channel <channel>] [--account <id>] [--json]`
- `pairing approve <channel> <code> [--account <id>] [--notify]`
- `pairing approve --channel <channel> [--account <id>] <code> [--notify]`

### `devices`

Quản lý các mục ghép nối thiết bị Gateway và các token thiết bị theo vai trò.

Các lệnh con:

- `devices list [--json]`
- `devices approve [requestId] [--latest]`
- `devices reject <requestId>`
- `devices remove <deviceId>`
- `devices clear --yes [--pending]`
- `devices rotate --device <id> --role <role> [--scope <scope...>]`
- `devices revoke --device <id> --role <role>`

### `webhooks gmail`

Thiết lập hook Gmail Pub/Sub + trình chạy. Xem [/automation/gmail-pubsub](/automation/gmail-pubsub).

Các lệnh con:

- `webhooks gmail setup` (yêu cầu `--account <email>`; hỗ trợ `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
- `webhooks gmail run` (ghi đè thời gian chạy cho các cờ tương tự)

### `dns setup`

Trợ giúp DNS khám phá khu vực rộng (CoreDNS + Tailscale). Xem [/gateway/discovery](/gateway/discovery).

Tùy chọn:

- `--apply`: cài đặt/cập nhật cấu hình CoreDNS (yêu cầu sudo; chỉ macOS).
## Nhắn tin + agent

### `message`

Nhắn tin gửi đi thống nhất + hành động kênh.

Xem: [/cli/message](/cli/message)

Các lệnh con:

- `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
- `message thread <create|list|reply>`
- `message emoji <list|upload>`
- `message sticker <send|upload>`
- `message role <info|add|remove>`
- `message channel <info|list>`
- `message member info`
- `message voice status`
- `message event <list|create>`

Ví dụ:

- `openclaw message send --target +15555550123 --message "Hi"`
- `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

### `agent`

Chạy một lượt agent thông qua Gateway (hoặc `--local` nhúng).

Bắt buộc:

- `--message <text>`

Tùy chọn:

- `--to <dest>` (cho khóa phiên và giao hàng tùy chọn)
- `--session-id <id>`
- `--thinking <off|minimal|low|medium|high|xhigh>` (chỉ các mô hình GPT-5.2 + Codex)
- `--verbose <on|full|off>`
- `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
- `--local`
- `--deliver`
- `--json`
- `--timeout <seconds>`

### `agents`

Quản lý các agent cô lập (không gian làm việc + xác thực + định tuyến).

#### `agents list`

Liệt kê các agent được cấu hình.

Tùy chọn:

- `--json`
- `--bindings`

#### `agents add [name]`
Thêm một agent cô lập mới. Chạy trình hướng dẫn được hướng dẫn trừ khi các cờ (hoặc `--non-interactive`) được truyền; `--workspace` là bắt buộc ở chế độ không tương tác.

Tùy chọn:

- `--workspace <dir>`
- `--model <id>`
- `--agent-dir <dir>`
- `--bind <channel[:accountId]>` (có thể lặp lại)
- `--non-interactive`
- `--json`

Các thông số kỹ thuật ràng buộc sử dụng `channel[:accountId]`. Khi `accountId` bị bỏ qua, OpenClaw có thể phân giải phạm vi tài khoản thông qua các giá trị mặc định kênh/hook plugin; nếu không, đó là ràng buộc kênh mà không có phạm vi tài khoản rõ ràng.

#### `agents bindings`

Liệt kê các ràng buộc định tuyến.

Tùy chọn:

- `--agent <id>`
- `--json`

#### `agents bind`

Thêm ràng buộc định tuyến cho một agent.

Tùy chọn:

- `--agent <id>`
- `--bind <channel[:accountId]>` (có thể lặp lại)
- `--json`

#### `agents unbind`

Xóa ràng buộc định tuyến cho một agent.

Tùy chọn:

- `--agent <id>`
- `--bind <channel[:accountId]>` (có thể lặp lại)
- `--all`
- `--json`

#### `agents delete <id>`

Xóa một agent và dọn sạch không gian làm việc + trạng thái của nó.

Tùy chọn:

- `--force`
- `--json`

### `acp`

Chạy cầu ACP kết nối các IDE với Gateway.

Xem [`acp`](/cli/acp) để biết đầy đủ các tùy chọn và ví dụ.

### `status`
Hiển thị tình trạng phiên được liên kết và những người nhận gần đây.

Tùy chọn:

- `--json`
- `--all` (chẩn đoán đầy đủ; chỉ đọc, có thể dán)
- `--deep` (kiểm tra kênh)
- `--usage` (hiển thị mức sử dụng/hạn ngạch nhà cung cấp mô hình)
- `--timeout <ms>`
- `--verbose`
- `--debug` (bí danh cho `--verbose`)

Ghi chú:

- Tổng quan bao gồm trạng thái dịch vụ Gateway + node host khi có sẵn.

### Theo dõi sử dụng

OpenClaw có thể hiển thị mức sử dụng/hạn ngạch nhà cung cấp khi có sẵn thông tin xác thực OAuth/API.

Hiển thị:

- `/status` (thêm một dòng sử dụng nhà cung cấp ngắn khi có sẵn)
- `openclaw status --usage` (in ra bảng phân tích nhà cung cấp đầy đủ)
- thanh menu macOS (phần Sử dụng trong Bối cảnh)

Ghi chú:

- Dữ liệu đến trực tiếp từ các điểm cuối sử dụng nhà cung cấp (không có ước tính).
- Nhà cung cấp: Anthropic, GitHub Copilot, OpenAI Codex OAuth, cộng với Gemini CLI/Antigravity khi các plugin nhà cung cấp đó được bật.
- Nếu không có thông tin xác thực phù hợp, mức sử dụng sẽ bị ẩn.
- Chi tiết: xem [Theo dõi sử dụng](/concepts/usage-tracking).

### `health`

Lấy tình trạng từ Gateway đang chạy.

Tùy chọn:

- `--json`
- `--timeout <ms>`
- `--verbose`

### `sessions`

Liệt kê các phiên trò chuyện được lưu trữ.

Tùy chọn:

- `--json`
- `--verbose`
- `--store <path>`
- `--active <minutes>`
## Đặt lại / Gỡ cài đặt

### `reset`

Đặt lại cấu hình/trạng thái cục bộ (giữ CLI được cài đặt).

Tùy chọn:

- `--scope <config|config+creds+sessions|full>`
- `--yes`
- `--non-interactive`
- `--dry-run`

Ghi chú:

- `--non-interactive` yêu cầu `--scope` và `--yes`.

### `uninstall`

Gỡ cài đặt dịch vụ gateway + dữ liệu cục bộ (CLI vẫn giữ lại).

Tùy chọn:

- `--service`
- `--state`
- `--workspace`
- `--app`
- `--all`
- `--yes`
- `--non-interactive`
- `--dry-run`

Ghi chú:

- `--non-interactive` yêu cầu `--yes` và các phạm vi rõ ràng (hoặc `--all`).
## Gateway

### `gateway`

Chạy WebSocket Gateway.

Tùy chọn:

- `--port <port>`
- `--bind <loopback|tailnet|lan|auto|custom>`
- `--token <token>`
- `--auth <token|password>`
- `--password <password>`
- `--tailscale <off|serve|funnel>`
- `--tailscale-reset-on-exit`
- `--allow-unconfigured`
- `--dev`
- `--reset` (đặt lại cấu hình dev + thông tin xác thực + phiên + không gian làm việc)
- `--force` (kết thúc trình lắng nghe hiện có trên cổng)
- `--verbose`
- `--claude-cli-logs`
- `--ws-log <auto|full|compact>`
- `--compact` (bí danh cho `--ws-log compact`)
- `--raw-stream`
- `--raw-stream-path <path>`

### `gateway service`

Quản lý dịch vụ Gateway (launchd/systemd/schtasks).

Lệnh con:

- `gateway status` (kiểm tra Gateway RPC theo mặc định)
- `gateway install` (cài đặt dịch vụ)
- `gateway uninstall`
- `gateway start`
- `gateway stop`
- `gateway restart`

Ghi chú:

- `gateway status` kiểm tra Gateway RPC theo mặc định bằng cách sử dụng cổng/cấu hình được phân giải của dịch vụ (ghi đè bằng `--url/--token/--password`).
- `gateway status` hỗ trợ `--no-probe`, `--deep`, và `--json` để viết kịch bản.
- `gateway status` cũng hiển thị các dịch vụ gateway cũ hoặc bổ sung khi có thể phát hiện chúng (`--deep` thêm quét cấp hệ thống). Các dịch vụ OpenClaw được đặt tên theo hồ sơ được coi là hạng nhất và không được đánh dấu là "bổ sung".
- `gateway status` in ra đường dẫn cấu hình mà CLI sử dụng so với cấu hình mà dịch vụ có khả năng sử dụng (env dịch vụ), cộng với URL đích kiểm tra được phân giải.
- `gateway install|uninstall|start|stop|restart` hỗ trợ `--json` để viết kịch bản (đầu ra mặc định vẫn thân thiện với con người).
- `gateway install` mặc định là Node runtime; bun **không được khuyến nghị** (lỗi WhatsApp/Telegram).
- `gateway install` tùy chọn: `--port`, `--runtime`, `--token`, `--force`, `--json`.

### `logs`

Theo dõi nhật ký tệp Gateway qua RPC.

Ghi chú:

- Các phiên TTY hiển thị chế độ xem có cấu trúc, được tô màu; không phải TTY sẽ quay lại văn bản thuần túy.
- `--json` phát ra JSON được phân tách theo dòng (một sự kiện nhật ký trên mỗi dòng).

Ví dụ:
```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### `gateway <subcommand>`

Gateway CLI helpers (use `--url`, `--token`, `--password`, `--timeout`, `--expect-final` for RPC subcommands).
When you pass `--url`, the CLI does not auto-apply config or environment credentials.
Include `--token` or `--password` explicitly. Missing explicit credentials is an error.

Subcommands:

- `gateway call <method> [--params <json>]`
- `gateway health`
- `gateway status`
- `gateway probe`
- `gateway discover`
- `gateway install|uninstall|start|stop|restart`
- `gateway run`

Common RPCs:

- `config.apply` (validate + write config + restart + wake)
- `config.patch` (merge a partial update + restart + wake)
- `update.run` (run update + restart + wake)

Tip: when calling `config.set`/`config.apply`/`config.patch` directly, pass `baseHash` from
`config.get` nếu cấu hình đã tồn tại.
## Mô hình

Xem [/concepts/models](/concepts/models) để biết hành vi dự phòng và chiến lược quét.

Xác thực Anthropic ưu tiên (setup-token):

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

### `models` (root)

`openclaw models` is an alias for `models status`.

Root options:

- `--status-json` (alias for `models status --json`)
- `--status-plain` (alias for `models status --plain`)

### `models list`

Options:

- `--all`
- `--local`
- `--provider <name>`
- `--json`
- `--plain`

### `models status`

Options:

- `--json`
- `--plain`
- `--check` (exit 1=expired/missing, 2=expiring)
- `--probe` (live probe of configured auth profiles)
- `--probe-provider <name>`
- `--probe-profile <id>` (repeat or comma-separated)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`

Always includes the auth overview and OAuth expiry status for profiles in the auth store.
`--probe` runs live requests (may consume tokens and trigger rate limits).

### `models set <model>`

Set `agents.defaults.model.primary`.

### `models set-image <model>`

Set `agents.defaults.imageModel.primary`.

### `models aliases list|add|remove`

Tùy chọn:
- `list`: `--json`, `--plain`
- `add <alias> <model>`
- `remove <alias>`

### `models fallbacks list|add|remove|clear`

Tùy chọn:

- `list`: `--json`, `--plain`
- `add <model>`
- `remove <model>`
- `clear`

### `models image-fallbacks list|add|remove|clear`

Tùy chọn:

- `list`: `--json`, `--plain`
- `add <model>`
- `remove <model>`
- `clear`

### `models scan`

Tùy chọn:

- `--min-params <b>`
- `--max-age-days <days>`
- `--provider <name>`
- `--max-candidates <n>`
- `--timeout <ms>`
- `--concurrency <n>`
- `--no-probe`
- `--yes`
- `--no-input`
- `--set-default`
- `--set-image`
- `--json`

### `models auth add|setup-token|paste-token`

Tùy chọn:

- `add`: trình hỗ trợ xác thực tương tác
- `setup-token`: `--provider <name>` (mặc định `anthropic`), `--yes`
- `paste-token`: `--provider <name>`, `--profile-id <id>`, `--expires-in <duration>`

### `models auth order get|set|clear`

Tùy chọn:

- `get`: `--provider <name>`, `--agent <id>`, `--json`
- `set`: `--provider <name>`, `--agent <id>`, `<profileIds...>`
- `clear`: `--provider <name>`, `--agent <id>`
## Hệ thống

### `system event`

Xếp hàng một sự kiện hệ thống và tùy chọn kích hoạt nhịp tim (Gateway RPC).

Bắt buộc:

- `--text <text>`

Tùy chọn:

- `--mode <now|next-heartbeat>`
- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

### `system heartbeat last|enable|disable`

Điều khiển nhịp tim (Gateway RPC).

Tùy chọn:

- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

### `system presence`

Liệt kê các mục hiện diện hệ thống (Gateway RPC).

Tùy chọn:

- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`
## Cron

Quản lý các công việc được lên lịch (Gateway RPC). Xem [/automation/cron-jobs](/automation/cron-jobs).

Các lệnh con:

- `cron status [--json]`
- `cron list [--all] [--json]` (đầu ra dạng bảng theo mặc định; sử dụng `--json` cho dạng thô)
- `cron add` (bí danh: `create`; yêu cầu `--name` và chính xác một trong `--at` | `--every` | `--cron`, và chính xác một payload của `--system-event` | `--message`)
- `cron edit <id>` (các trường vá)
- `cron rm <id>` (bí danh: `remove`, `delete`)
- `cron enable <id>`
- `cron disable <id>`
- `cron runs --id <id> [--limit <n>]`
- `cron run <id> [--force]`

Tất cả các lệnh `cron` chấp nhận `--url`, `--token`, `--timeout`, `--expect-final`.
## Node host

`node` chạy một **node host không giao diện** hoặc quản lý nó như một dịch vụ nền. Xem
[`openclaw node`](/cli/node).

Các lệnh con:

- `node run --host <gateway-host> --port 18789`
- `node status`
- `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
- `node uninstall`
- `node stop`
- `node restart`
## Nodes

`nodes` liên lạc với Gateway và nhắm mục tiêu các node được ghép nối. Xem [/nodes](/nodes).

Các tùy chọn phổ biến:

- `--url`, `--token`, `--timeout`, `--json`

Các lệnh con:

- `nodes status [--connected] [--last-connected <duration>]`
- `nodes describe --node <id|name|ip>`
- `nodes list [--connected] [--last-connected <duration>]`
- `nodes pending`
- `nodes approve <requestId>`
- `nodes reject <requestId>`
- `nodes rename --node <id|name|ip> --name <displayName>`
- `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
- `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (mac node hoặc headless node host)
- `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|active|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (chỉ mac)

Camera:

- `nodes camera list --node <id|name|ip>`
- `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
- `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

Canvas + screen:

- `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
- `nodes canvas present --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
- `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
- `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
- `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

Vị trí:

- `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`
## Trình duyệt

CLI điều khiển trình duyệt (Chrome/Brave/Edge/Chromium chuyên dụng). Xem [`openclaw browser`](/cli/browser) và [công cụ Trình duyệt](/tools/browser).

Các tùy chọn phổ biến:

- `--url`, `--token`, `--timeout`, `--json`
- `--browser-profile <name>`

Quản lý:

- `browser status`
- `browser start`
- `browser stop`
- `browser reset-profile`
- `browser tabs`
- `browser open <url>`
- `browser focus <targetId>`
- `browser close [targetId]`
- `browser profiles`
- `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
- `browser delete-profile --name <name>`

Kiểm tra:

- `browser screenshot [targetId] [--full-page] [--ref <ref>] [--element <selector>] [--type png|jpeg]`
- `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

Hành động:

- `browser navigate <url> [--target-id <id>]`
- `browser resize <width> <height> [--target-id <id>]`
- `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
- `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
- `browser press <key> [--target-id <id>]`
- `browser hover <ref> [--target-id <id>]`
- `browser drag <startRef> <endRef> [--target-id <id>]`
- `browser select <ref> <values...> [--target-id <id>]`
- `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--element <selector>] [--target-id <id>] [--timeout-ms <ms>]`
- `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
- `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
- `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
- `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
- `browser console [--level <error|warn|info>] [--target-id <id>]`
- `browser pdf [--target-id <id>]`
## Tìm kiếm tài liệu

### `docs [query...]`

Tìm kiếm chỉ mục tài liệu trực tiếp.
## TUI

### `tui`

Mở giao diện người dùng terminal được kết nối với Gateway.

Tùy chọn:

- `--url <url>`
- `--token <token>`
- `--password <password>`
- `--session <key>`
- `--deliver`
- `--thinking <level>`
- `--message <text>`
- `--timeout-ms <ms>` (mặc định là `agents.defaults.timeoutSeconds`)
- `--history-limit <n>`