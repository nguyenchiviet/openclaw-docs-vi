---
summary: Các tin nhắn polling nhịp tim và quy tắc thông báo
read_when:
  - Điều chỉnh nhịp độ nhịp tim hoặc tin nhắn
  - Quyết định giữa heartbeat và cron cho các tác vụ được lên lịch
title: Nhịp tim
x-i18n:
  source_path: gateway\heartbeat.md
  source_hash: e893b56b39499d661108878d1958fca4eb5e7f4cc8176f2a18845346c53caee3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:58:05.856Z'
---

# Nhịp tim (Gateway)

> **Nhịp tim vs Cron?** Xem [Cron vs Nhịp tim](/automation/cron-vs-heartbeat) để biết hướng dẫn về thời điểm sử dụng từng loại.

Nhịp tim chạy **các lượt agent định kỳ** trong phiên chính để mô hình có thể
nêu bất cứ điều gì cần chú ý mà không làm bạn bị spam.

Khắc phục sự cố: [/automation/troubleshooting](/automation/troubleshooting)
## Bắt đầu nhanh (người mới bắt đầu)

1. Để bật heartbeats (mặc định là `30m`, hoặc `1h` cho Anthropic OAuth/setup-token) hoặc đặt nhịp độ của riêng bạn.
2. Tạo một danh sách kiểm tra nhỏ `HEARTBEAT.md` trong không gian làm việc agent (tùy chọn nhưng được khuyến nghị).
3. Quyết định nơi các tin nhắn heartbeat sẽ được gửi (`target: "none"` là mặc định; đặt `target: "last"` để định tuyến đến liên hệ cuối cùng).
4. Tùy chọn: bật phân phối lý do heartbeat để minh bạch.
5. Tùy chọn: giới hạn heartbeats trong giờ hoạt động (giờ địa phương).

Ví dụ cấu hình:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // explicit delivery to last contact (default is "none")
        directPolicy: "allow", // default: allow direct/DM targets; set "block" to suppress
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // optional: send separate `Reasoning:` message too
      },
    },
  },
}
```
## Mặc định

- Khoảng thời gian: `30m` (hoặc `1h` khi Anthropic OAuth/setup-token là chế độ xác thực được phát hiện). Đặt `agents.defaults.heartbeat.every` hoặc `agents.list[].heartbeat.every` cho từng agent; sử dụng `0m` để tắt.
- Nội dung prompt (có thể cấu hình qua `agents.defaults.heartbeat.prompt`):
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- Prompt heartbeat được gửi **nguyên văn** dưới dạng tin nhắn của người dùng. System prompt bao gồm phần "Heartbeat" và lần chạy được đánh dấu nội bộ.
- Giờ hoạt động (`heartbeat.activeHours`) được kiểm tra trong múi giờ được cấu hình.
  Ngoài cửa sổ thời gian, các heartbeat sẽ bị bỏ qua cho đến tick tiếp theo trong cửa sổ.
## Mục đích của heartbeat prompt

Prompt mặc định được thiết kế có ý để rộng:

- **Các tác vụ nền**: "Consider outstanding tasks" nhắc nhở agent xem xét các
  công việc tiếp theo (hộp thư, lịch, nhắc nhở, công việc trong hàng đợi) và nêu bật bất kỳ điều gì khẩn cấp.
- **Kiểm tra con người**: "Checkup sometimes on your human during day time" nhắc nhở một
  tin nhắn nhẹ nhàng "anything you need?" thỉnh thoảng, nhưng tránh spam vào ban đêm
  bằng cách sử dụng múi giờ local của bạn (xem [/concepts/timezone](/concepts/timezone)).

Nếu bạn muốn heartbeat thực hiện một điều gì đó rất cụ thể (ví dụ: "check Gmail PubSub
stats" hoặc "verify gateway health"), hãy đặt `agents.defaults.heartbeat.prompt` (hoặc
`agents.list[].heartbeat.prompt`) thành một nội dung tùy chỉnh (được gửi nguyên văn).
## Hợp đồng phản hồi

- Nếu không cần chú ý gì, hãy trả lời bằng **`HEARTBEAT_OK`**.
- Trong các lần chạy heartbeat, OpenClaw coi `HEARTBEAT_OK` là một xác nhận khi nó xuất hiện
  ở **đầu hoặc cuối** của phản hồi. Token được loại bỏ và phản hồi bị
  loại bỏ nếu nội dung còn lại là **≤ `ackMaxChars`** (mặc định: 300).
- Nếu `HEARTBEAT_OK` xuất hiện ở **giữa** một phản hồi, nó không được coi là
  đặc biệt.
- Đối với cảnh báo, **không** bao gồm `HEARTBEAT_OK`; chỉ trả về văn bản cảnh báo.

Ngoài heartbeats, `HEARTBEAT_OK` lạc lõng ở đầu/cuối của một tin nhắn bị loại bỏ
và ghi nhật ký; một tin nhắn chỉ là `HEARTBEAT_OK` bị loại bỏ.
## Cấu hình

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // default: 30m (0m disables)
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // default: false (deliver separate Reasoning: message when available)
        target: "last", // default: none | options: last | none | <channel id> (core or plugin, e.g. "bluebubbles")
        to: "+15551234567", // optional channel-specific override
        accountId: "ops-bot", // optional multi-account channel id
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // max chars allowed after HEARTBEAT_OK
      },
    },
  },
}
```

### Scope and precedence

- `agents.defaults.heartbeat` sets global heartbeat behavior.
- `agents.list[].heartbeat` merges on top; if any agent has a `heartbeat` block, **only those agents** run heartbeats.
- `channels.defaults.heartbeat` sets visibility defaults for all channels.
- `channels.<channel>.heartbeat` overrides channel defaults.
- `channels.<channel>.accounts.<id>.heartbeat` (multi-account channels) overrides per-channel settings.

### Per-agent heartbeats

If any `agents.list[]` entry includes a `heartbeat` block, **only those agents**
run heartbeats. The per-agent block merges on top of `agents.defaults.heartbeat`
(so you can set shared defaults once and override per agent).

Example: two agents, only the second agent runs heartbeats.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // explicit delivery to last contact (default is "none")
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```
### Ví dụ về giờ hoạt động

Hạn chế nhịp tim trong giờ làm việc ở múi giờ cụ thể:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // explicit delivery to last contact (default is "none")
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // optional; uses your userTimezone if set, otherwise host tz
        },
      },
    },
  },
}
```

Outside this window (before 9am or after 10pm Eastern), heartbeats are skipped. The next scheduled tick inside the window will run normally.

### 24/7 setup

If you want heartbeats to run all day, use one of these patterns:

- Omit `activeHours` entirely (no time-window restriction; this is the default behavior).
- Set a full-day window: `activeHours: { start: "00:00", end: "24:00" }`.

Do not set the same `start` and `end` time (for example `08:00` to `08:00`).
That is treated as a zero-width window, so heartbeats are always skipped.

### Multi account example

Use `accountId` to target a specific account on multi-account channels like Telegram:

```json5
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // optional: route to a specific topic/thread
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```
### Ghi chú trường

- `every`: khoảng thời gian heartbeat (chuỗi thời gian; đơn vị mặc định = phút).
- `model`: ghi đè mô hình tùy chọn cho các lần chạy heartbeat (`provider/model`).
- `includeReasoning`: khi được bật, cũng gửi thêm tin nhắn `Reasoning:` riêng biệt khi có sẵn (cùng hình dạng với `/reasoning on`).
- `session`: khóa phiên tùy chọn cho các lần chạy heartbeat.
  - `main` (mặc định): phiên chính của agent.
  - Khóa phiên rõ ràng (sao chép từ `openclaw sessions --json` hoặc [CLI sessions](/cli/sessions)).
  - Định dạng khóa phiên: xem [Sessions](/concepts/session) và [Groups](/channels/groups).
- `target`:
  - `last`: gửi đến kênh bên ngoài được sử dụng lần cuối.
  - kênh rõ ràng: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  - `none` (mặc định): chạy heartbeat nhưng **không gửi** bên ngoài.
- `directPolicy`: kiểm soát hành vi gửi trực tiếp/tin nhắn riêng:
  - `allow` (mặc định): cho phép gửi heartbeat trực tiếp/tin nhắn riêng.
  - `block`: chặn gửi trực tiếp/tin nhắn riêng (`reason=dm-blocked`).
- `to`: ghi đè người nhận tùy chọn (id cụ thể của kênh, ví dụ E.164 cho WhatsApp hoặc id trò chuyện Telegram). Đối với các chủ đề/luồng Telegram, hãy sử dụng `<chatId>:topic:<messageThreadId>`.
- `accountId`: id tài khoản tùy chọn cho các kênh đa tài khoản. Khi `target: "last"`, id tài khoản áp dụng cho kênh cuối cùng được giải quyết nếu nó hỗ trợ tài khoản; nếu không, nó sẽ bị bỏ qua. Nếu id tài khoản không khớp với tài khoản được cấu hình cho kênh được giải quyết, việc gửi sẽ bị bỏ qua.
- `prompt`: ghi đè nội dung lời nhắc mặc định (không được hợp nhất).
- `ackMaxChars`: số ký tự tối đa được phép sau `HEARTBEAT_OK` trước khi gửi.
- `suppressToolErrorWarnings`: khi đúng, chặn các tải trọng cảnh báo lỗi công cụ trong các lần chạy heartbeat.
- `activeHours`: hạn chế các lần chạy heartbeat trong một cửa sổ thời gian. Đối tượng với `start` (HH:MM, bao gồm; sử dụng `00:00` cho đầu ngày), `end` (HH:MM loại trừ; `24:00` được phép cho cuối ngày), và `timezone` tùy chọn.
  - Bị bỏ qua hoặc `"user"`: sử dụng `agents.defaults.userTimezone` của bạn nếu được đặt, nếu không sẽ quay lại múi giờ hệ thống máy chủ.
  - `"local"`: luôn sử dụng múi giờ hệ thống máy chủ.
  - Bất kỳ định danh IANA nào (ví dụ `America/New_York`): được sử dụng trực tiếp; nếu không hợp lệ, sẽ quay lại hành vi `"user"` ở trên.
  - `start` và `end` không được bằng nhau cho một cửa sổ hoạt động; các giá trị bằng nhau được coi là chiều rộng bằng không (luôn ngoài cửa sổ).
  - Ngoài cửa sổ hoạt động, các heartbeat sẽ bị bỏ qua cho đến lần đánh dấu tiếp theo bên trong cửa sổ.
## Hành vi gửi

- Heartbeats chạy trong phiên chính của agent theo mặc định (`agent:<id>:<mainKey>`),
  hoặc `global` khi `session.scope = "global"`. Đặt `session` để ghi đè thành
  một phiên kênh cụ thể (Discord/WhatsApp/v.v.).
- `session` chỉ ảnh hưởng đến ngữ cảnh chạy; việc gửi được kiểm soát bởi `target` và `to`.
- Để gửi đến một kênh/người nhận cụ thể, đặt `target` + `to`. Với
  `target: "last"`, việc gửi sử dụng kênh bên ngoài cuối cùng cho phiên đó.
- Các lần gửi heartbeat cho phép các mục tiêu trực tiếp/tin nhắn riêng theo mặc định. Đặt `directPolicy: "block"` để chặn các lần gửi mục tiêu trực tiếp trong khi vẫn chạy lượt heartbeat.
- Nếu hàng đợi chính bận, heartbeat sẽ bị bỏ qua và thử lại sau.
- Nếu `target` không phân giải thành đích bên ngoài nào, lần chạy vẫn xảy ra nhưng không có
  tin nhắn gửi đi.
- Các phản hồi chỉ heartbeat **không** giữ phiên hoạt động; `updatedAt` cuối cùng
  được khôi phục để hành vi hết hạn nhàn rỗi hoạt động bình thường.
## Kiểm soát khả năng hiển thị

Theo mặc định, `HEARTBEAT_OK` acknowledgments bị ẩn trong khi nội dung cảnh báo được
gửi. Bạn có thể điều chỉnh điều này cho mỗi kênh hoặc cho mỗi tài khoản:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false # Hide HEARTBEAT_OK (default)
      showAlerts: true # Show alert messages (default)
      useIndicator: true # Emit indicator events (default)
  telegram:
    heartbeat:
      showOk: true # Show OK acknowledgments on Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Suppress alert delivery for this account
```

Precedence: per-account → per-channel → channel defaults → built-in defaults.

### What each flag does

- `showOk`: sends a `HEARTBEAT_OK` acknowledgment when the model returns an OK-only reply.
- `showAlerts`: sends the alert content when the model returns a non-OK reply.
- `useIndicator`: emits indicator events for UI status surfaces.

If **all three** are false, OpenClaw skips the heartbeat run entirely (no model call).

### Per-channel vs per-account examples

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # all Slack accounts
    accounts:
      ops:
        heartbeat:
          showAlerts: false # suppress alerts for the ops account only
  telegram:
    heartbeat:
      showOk: true
```

### Common patterns

| Goal                                     | Config                                                                                   |
| ---------------------------------------- | ---------------------------------------------------------------------------------------- |
| Default behavior (silent OKs, alerts on) | _(no config needed)_                                                                     |
| Fully silent (no messages, no indicator) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Indicator-only (no messages)             | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }`  |
| OKs in one channel only                  | `channels.telegram.heartbeat: { showOk: true }`                                          |

## HEARTBEAT.md (tùy chọn)

Nếu tệp `HEARTBEAT.md` tồn tại trong workspace, prompt mặc định sẽ yêu cầu
agent đọc nó. Hãy coi nó như "danh sách kiểm tra nhịp tim" của bạn: nhỏ gọn, ổn định và
an toàn để đưa vào mỗi 30 phút.

Nếu `HEARTBEAT.md` tồn tại nhưng về cơ bản trống rỗng (chỉ có dòng trống và tiêu đề markdown
như `# Heading`), OpenClaw sẽ bỏ qua lần chạy heartbeat để tiết kiệm lệnh gọi API.
Nếu tệp bị thiếu, heartbeat vẫn chạy và mô hình sẽ quyết định phải làm gì.

Giữ nó nhỏ gọn (danh sách kiểm tra hoặc nhắc nhở ngắn) để tránh làm phình prompt.

Ví dụ `HEARTBEAT.md`:

```md
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it’s daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### Can the agent update HEARTBEAT.md?

Yes — if you ask it to.

`HEARTBEAT.md` is just a normal file in the agent workspace, so you can tell the
agent (in a normal chat) something like:

- “Update `HEARTBEAT.md` to add a daily calendar check.”
- “Rewrite `HEARTBEAT.md` so it’s shorter and focused on inbox follow-ups.”

If you want this to happen proactively, you can also include an explicit line in
your heartbeat prompt like: “If the checklist becomes stale, update HEARTBEAT.md
with a better one.”

Safety note: don’t put secrets (API keys, phone numbers, private tokens) into
`HEARTBEAT.md` — nó trở thành một phần của ngữ cảnh prompt.
## Đánh thức thủ công (theo yêu cầu)

Bạn có thể xếp hàng một sự kiện hệ thống và kích hoạt một nhịp tim ngay lập tức với:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

If multiple agents have `heartbeat` configured, a manual wake runs each of those
agent heartbeats immediately.

Use `--mode next-heartbeat` để chờ đợi tick được lên lịch tiếp theo.
## Truyền phát lý do (tùy chọn)

Theo mặc định, heartbeats chỉ truyền phát payload "answer" cuối cùng.

Nếu bạn muốn tính minh bạch, hãy bật:

- `agents.defaults.heartbeat.includeReasoning: true`

Khi được bật, heartbeats cũng sẽ truyền phát một tin nhắn riêng biệt có tiền tố
`Reasoning:` (cùng hình dạng với `/reasoning on`). Điều này có thể hữu ích khi agent
đang quản lý nhiều phiên/codexes và bạn muốn xem tại sao nó quyết định ping
bạn — nhưng nó cũng có thể tiết lộ nhiều chi tiết nội bộ hơn bạn muốn. Tốt hơn là giữ nó
tắt trong các cuộc trò chuyện nhóm.
## Nhận thức về chi phí

Heartbeats chạy các lượt agent đầy đủ. Các khoảng thời gian ngắn hơn tiêu tốn nhiều token hơn. Giữ
`HEARTBEAT.md` nhỏ và cân nhắc sử dụng `model` hoặc `target: "none"` rẻ hơn nếu bạn
chỉ muốn cập nhật trạng thái nội bộ.