---
summary: 'Lệnh Slash: văn bản so với gốc, cấu hình và các lệnh được hỗ trợ'
read_when:
  - Sử dụng hoặc cấu hình các lệnh chat
  - Gỡ lỗi định tuyến lệnh hoặc quyền hạn
title: Lệnh Gạch Chéo
x-i18n:
  source_path: tools\slash-commands.md
  source_hash: 70a8087a2315f36dcd8573634d87895b9a32a03a22b6af2b3fc65af1756e34ee
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:41.109Z'
---

# Lệnh Slash

Các lệnh được xử lý bởi Gateway. Hầu hết các lệnh phải được gửi dưới dạng **tin nhắn độc lập** bắt đầu bằng `/`.
Lệnh bash chat chỉ dành cho host sử dụng `! <cmd>` (với `/bash <cmd>` làm bí danh).

Có hai hệ thống liên quan:

- **Lệnh**: tin nhắn `/...` độc lập.
- **Chỉ thị**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  - Các chỉ thị được loại bỏ khỏi tin nhắn trước khi mô hình nhìn thấy nó.
  - Trong các tin nhắn trò chuyện bình thường (không phải chỉ thị duy nhất), chúng được coi là "gợi ý nội tuyến" và **không** duy trì cài đặt phiên.
  - Trong các tin nhắn chỉ thị duy nhất (tin nhắn chỉ chứa các chỉ thị), chúng duy trì phiên và trả lời bằng xác nhận.
  - Các chỉ thị chỉ được áp dụng cho **những người gửi được phép**. Nếu `commands.allowFrom` được đặt, đó là danh sách cho phép duy nhất được sử dụng; nếu không, ủy quyền đến từ danh sách cho phép kênh/ghép nối cộng với `commands.useAccessGroups`.
    Những người gửi không được phép sẽ thấy các chỉ thị được coi là văn bản thuần túy.

Ngoài ra còn có một vài **phím tắt nội tuyến** (chỉ những người gửi được phép/được ủy quyền): `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Chúng chạy ngay lập tức, được loại bỏ trước khi mô hình nhìn thấy tin nhắn, và văn bản còn lại tiếp tục thông qua luồng bình thường.
## Cấu hình

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

- `commands.text` (default `true`) enables parsing `/...` in chat messages.
  - On surfaces without native commands (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), text commands still work even if you set this to `false`.
- `commands.native` (default `"auto"`) registers native commands.
  - Auto: on for Discord/Telegram; off for Slack (until you add slash commands); ignored for providers without native support.
  - Set `channels.discord.commands.native`, `channels.telegram.commands.native`, or `channels.slack.commands.native` to override per provider (bool or `"auto"`).
  - `false` clears previously registered commands on Discord/Telegram at startup. Slack commands are managed in the Slack app and are not removed automatically.
- `commands.nativeSkills` (default `"auto"`) registers **skill** commands natively when supported.
  - Auto: on for Discord/Telegram; off for Slack (Slack requires creating a slash command per skill).
  - Set `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills`, or `channels.slack.commands.nativeSkills` to override per provider (bool or `"auto"`).
- `commands.bash` (default `false`) enables `! <cmd>` to run host shell commands (`/bash <cmd>` is an alias; requires `tools.elevated` allowlists).
- `commands.bashForegroundMs` (default `2000`) controls how long bash waits before switching to background mode (`0` backgrounds immediately).
- `commands.config` (default `false`) enables `/config` (reads/writes `openclaw.json`).
- `commands.debug` (default `false`) enables `/debug` (runtime-only overrides).
- `commands.allowFrom` (optional) sets a per-provider allowlist for command authorization. When configured, it is the
  only authorization source for commands and directives (channel allowlists/pairing and `commands.useAccessGroups`
  are ignored). Use `"*"` for a global default; provider-specific keys override it.
- `commands.useAccessGroups` (default `true`) enforces allowlists/policies for commands when `commands.allowFrom` chưa được đặt.
## Danh sách lệnh

Text + native (khi được bật):

- `/help`
- `/commands`
- `/skill <name> [input]` (chạy một skill theo tên)
- `/status` (hiển thị trạng thái hiện tại; bao gồm mức sử dụng/hạn ngạch nhà cung cấp cho nhà cung cấp mô hình hiện tại khi có sẵn)
- `/allowlist` (liệt kê/thêm/xóa các mục danh sách cho phép)
- `/approve <id> allow-once|allow-always|deny` (giải quyết các lời nhắc phê duyệt exec)
- `/context [list|detail|json]` (giải thích "context"; `detail` hiển thị kích thước mỗi tệp + mỗi công cụ + mỗi skill + system prompt)
- `/export-session [path]` (bí danh: `/export`) (xuất phiên hiện tại sang HTML với system prompt đầy đủ)
- `/whoami` (hiển thị ID người gửi của bạn; bí danh: `/id`)
- `/session idle <duration|off>` (quản lý tự động unfocus do không hoạt động cho các ràng buộc luồng tập trung)
- `/session max-age <duration|off>` (quản lý tự động unfocus max-age cứng cho các ràng buộc luồng tập trung)
- `/subagents list|kill|log|info|send|steer|spawn` (kiểm tra, kiểm soát hoặc tạo các lần chạy sub-agent cho phiên hiện tại)
- `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions` (kiểm tra và kiểm soát các phiên runtime ACP)
- `/agents` (liệt kê các agent được ràng buộc với luồng cho phiên này)
- `/focus <target>` (Discord: ràng buộc luồng này hoặc một luồng mới với mục tiêu phiên/subagent)
- `/unfocus` (Discord: xóa ràng buộc luồng hiện tại)
- `/kill <id|#|all>` (hủy bỏ ngay một hoặc tất cả các sub-agent đang chạy cho phiên này; không có thông báo xác nhận)
- `/steer <id|#> <message>` (điều hướng một sub-agent đang chạy ngay lập tức: trong lần chạy khi có thể, nếu không hãy hủy bỏ công việc hiện tại và khởi động lại trên thông báo điều hướng)
- `/tell <id|#> <message>` (bí danh cho `/steer`)
- `/config show|get|set|unset` (lưu cấu hình vào đĩa, chỉ chủ sở hữu; yêu cầu `commands.config: true`)
- `/debug show|set|unset|reset` (ghi đè runtime, chỉ chủ sở hữu; yêu cầu `commands.debug: true`)
- `/usage off|tokens|full|cost` (chân trang sử dụng mỗi phản hồi hoặc tóm tắt chi phí cục bộ)
- `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (kiểm soát TTS; xem [/tts](/tts))
  - Discord: lệnh native là `/voice` (Discord dành riêng `/tts`); text `/tts` vẫn hoạt động.
- `/stop`
- `/restart`
- `/dock-telegram` (bí danh: `/dock_telegram`) (chuyển câu trả lời sang Telegram)
- `/dock-discord` (bí danh: `/dock_discord`) (chuyển câu trả lời sang Discord)
- `/dock-slack` (bí danh: `/dock_slack`) (chuyển câu trả lời sang Slack)
- `/activation mention|always` (chỉ nhóm)
- `/send on|off|inherit` (chỉ chủ sở hữu)
- `/reset` hoặc `/new [model]` (gợi ý mô hình tùy chọn; phần còn lại được chuyển qua)
- `/think <off|minimal|low|medium|high|xhigh>` (lựa chọn động theo mô hình/nhà cung cấp; bí danh: `/thinking`, `/t`)
- `/verbose on|full|off` (bí danh: `/v`)
- `/reasoning on|off|stream` (bí danh: `/reason`; khi bật, gửi một thông báo riêng biệt có tiền tố `Reasoning:`; `stream` = chỉ bản nháp Telegram)
- `/elevated on|off|ask|full` (bí danh: `/elev`; `full` bỏ qua phê duyệt exec)
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` (gửi `/exec` để hiển thị hiện tại)
- `/model <name>` (bí danh: `/models`; hoặc `/<alias>` từ `agents.defaults.models.*.alias`)
- `/queue <mode>` (cộng với các tùy chọn như `debounce:2s cap:25 drop:summarize`; gửi `/queue` để xem cài đặt hiện tại)
- `/bash <command>` (chỉ host; bí danh cho `! <command>`; yêu cầu `commands.bash: true` + `tools.elevated` danh sách cho phép)

Chỉ text:

- `/compact [instructions]` (xem [/concepts/compaction](/concepts/compaction))
- `! <command>` (chỉ host; một lần một; sử dụng `!poll` + `!stop` cho các công việc chạy lâu dài)
- `!poll` (kiểm tra đầu ra / trạng thái; chấp nhận `sessionId` tùy chọn; `/bash poll` cũng hoạt động)
- `!stop` (dừng công việc bash đang chạy; chấp nhận `sessionId` tùy chọn; `/bash stop` cũng hoạt động)

Ghi chú:

- Các lệnh chấp nhận một `:` tùy chọn giữa lệnh và đối số (ví dụ: `/think: high`, `/send: on`, `/help:`).
- `/new <model>` chấp nhận bí danh mô hình, `provider/model`, hoặc tên nhà cung cấp (khớp mờ); nếu không có kết quả khớp, văn bản được coi là nội dung thông báo.
- Để xem chi tiết sử dụng nhà cung cấp đầy đủ, sử dụng `openclaw status --usage`.
- `/allowlist add|remove` yêu cầu `commands.config=true` và tôn trọng kênh `configWrites`.
- `/usage` kiểm soát chân trang sử dụng mỗi phản hồi; `/usage cost` in tóm tắt chi phí cục bộ từ nhật ký phiên OpenClaw.
- `/restart` được bật theo mặc định; đặt `commands.restart: false` để tắt nó.
- Lệnh native chỉ dành cho Discord: `/vc join|leave|status` kiểm soát các kênh thoại (yêu cầu `channels.discord.voice` và các lệnh native; không khả dụng dưới dạng văn bản).
- Các lệnh liên kết thread của Discord (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`) yêu cầu các liên kết thread hiệu quả được bật (`session.threadBindings.enabled` và/hoặc `channels.discord.threadBindings.enabled`).
- Tham chiếu lệnh ACP và hành vi runtime: [ACP Agents](/tools/acp-agents).
- `/verbose` được dùng để gỡ lỗi và tăng khả năng hiển thị; giữ nó **tắt** trong sử dụng bình thường.
- Các bản tóm tắt lỗi công cụ vẫn được hiển thị khi có liên quan, nhưng văn bản lỗi chi tiết chỉ được đưa vào khi `/verbose` là `on` hoặc `full`.
- `/reasoning` (và `/verbose`) có rủi ro trong các cài đặt nhóm: chúng có thể tiết lộ lý luận nội bộ hoặc đầu ra công cụ mà bạn không dự định để lộ. Tốt hơn là để chúng tắt, đặc biệt là trong các cuộc trò chuyện nhóm.
- **Đường dẫn nhanh:** các tin nhắn chỉ lệnh từ những người gửi được phép liệt kê được xử lý ngay lập tức (bỏ qua hàng đợi + mô hình).
- **Gating nhắc nhở nhóm:** các tin nhắn chỉ lệnh từ những người gửi được phép liệt kê bỏ qua các yêu cầu nhắc nhở.
- **Phím tắt nội tuyến (chỉ những người gửi được phép liệt kê):** một số lệnh cũng hoạt động khi được nhúng trong một tin nhắn bình thường và được loại bỏ trước khi mô hình nhìn thấy văn bản còn lại.
  - Ví dụ: `hey /status` kích hoạt một phản hồi trạng thái, và văn bản còn lại tiếp tục thông qua luồng bình thường.
- Hiện tại: `/help`, `/commands`, `/status`, `/whoami` (`/id`).
- Các tin nhắn lệnh không được phép chỉ được bỏ qua im lặng, và các token `/...` nội tuyến được coi là văn bản thuần túy.
- **Lệnh Skill:** các skill `user-invocable` được hiển thị dưới dạng các lệnh slash. Tên được làm sạch thành `a-z0-9_` (tối đa 32 ký tự); các va chạm nhận được hậu tố số (ví dụ `_2`).
  - `/skill <name> [input]` chạy một skill theo tên (hữu ích khi các giới hạn lệnh native ngăn chặn các lệnh cho từng skill).
  - Theo mặc định, các lệnh skill được chuyển tiếp đến mô hình dưới dạng một yêu cầu bình thường.
  - Các skill có thể tùy chọn khai báo `command-dispatch: tool` để định tuyến lệnh trực tiếp đến một công cụ (xác định, không có mô hình).
  - Ví dụ: `/prose` (plugin OpenProse) — xem [OpenProse](/prose).
- **Đối số lệnh native:** Discord sử dụng tự động hoàn thành cho các tùy chọn động (và menu nút khi bạn bỏ qua các đối số bắt buộc). Telegram và Slack hiển thị menu nút khi một lệnh hỗ trợ các lựa chọn và bạn bỏ qua đối số.
## Bề mặt sử dụng (hiển thị ở đâu)

- **Sử dụng nhà cung cấp/hạn ngạch** (ví dụ: "Claude 80% còn lại") hiển thị trong `/status` cho nhà cung cấp mô hình hiện tại khi bật theo dõi sử dụng.
- **Token/chi phí trên mỗi phản hồi** được kiểm soát bởi `/usage off|tokens|full` (được thêm vào các phản hồi bình thường).
- `/model status` là về **mô hình/xác thực/điểm cuối**, không phải sử dụng.
## Lựa chọn mô hình (`/model`)

`/model` được triển khai dưới dạng một chỉ thị.

Ví dụ:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notes:

- `/model` and `/model list` show a compact, numbered picker (model family + available providers).
- On Discord, `/model` and `/models` open an interactive picker with provider and model dropdowns plus a Submit step.
- `/model <#>` selects from that picker (and prefers the current provider when possible).
- `/model status` shows the detailed view, including configured provider endpoint (`baseUrl`) and API mode (`api`) khi có sẵn.
## Ghi đè gỡ lỗi

`/debug` cho phép bạn đặt các ghi đè cấu hình **chỉ lúc chạy** (bộ nhớ, không phải đĩa). Chỉ chủ sở hữu. Bị vô hiệu hóa theo mặc định; bật bằng `commands.debug: true`.

Ví dụ:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Notes:

- Overrides apply immediately to new config reads, but do **not** write to `openclaw.json`.
- Use `/debug reset` để xóa tất cả các ghi đè và quay lại cấu hình trên đĩa.
## Cập nhật cấu hình

`/config` ghi vào cấu hình trên đĩa của bạn (`openclaw.json`). Chỉ dành cho chủ sở hữu. Bị vô hiệu hóa theo mặc định; bật bằng `commands.config: true`.

Ví dụ:

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Notes:

- Config is validated before write; invalid changes are rejected.
- `/config` các cập nhật được duy trì qua các lần khởi động lại.
## Ghi chú bề mặt

- **Lệnh văn bản** chạy trong phiên chat bình thường (Tin nhắn riêng chia sẻ `main`, các nhóm có phiên riêng của chúng).
- **Lệnh gốc** sử dụng các phiên cô lập:
  - Discord: `agent:<agentId>:discord:slash:<userId>`
  - Slack: `agent:<agentId>:slack:slash:<userId>` (tiền tố có thể cấu hình qua `channels.slack.slashCommand.sessionPrefix`)
  - Telegram: `telegram:slash:<userId>` (nhắm đến phiên chat qua `CommandTargetSessionKey`)
- **`/stop`** nhắm đến phiên chat hoạt động để nó có thể hủy bỏ lần chạy hiện tại.
- **Slack:** `channels.slack.slashCommand` vẫn được hỗ trợ cho một lệnh kiểu `/openclaw`. Nếu bạn bật `commands.native`, bạn phải tạo một lệnh gạch chéo Slack cho mỗi lệnh tích hợp (cùng tên với `/help`). Các menu đối số lệnh cho Slack được cung cấp dưới dạng các nút Block Kit tạm thời.