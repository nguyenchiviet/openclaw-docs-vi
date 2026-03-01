---
summary: Cron jobs + đánh thức cho bộ lập lịch Gateway
read_when:
  - Lập lịch các công việc nền hoặc đánh thức
  - Kết nối tự động hóa nên chạy cùng với hoặc song song với heartbeat
  - Lựa chọn giữa heartbeat và cron cho các tác vụ được lên lịch
title: Công việc Cron
x-i18n:
  source_path: automation\cron-jobs.md
  source_hash: fa14d2ac586659948c16443a9ab2fef5e7550a78acb3737342665680f9b78ae0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:06:58.665Z'
---

# Cron job (Bộ lập lịch Gateway)

> **Cron vs Heartbeat?** Xem [Cron vs Heartbeat](/automation/cron-vs-heartbeat) để được hướng dẫn khi nào nên sử dụng từng loại.

Cron là bộ lập lịch tích hợp sẵn của Gateway. Nó lưu trữ các công việc, đánh thức agent vào đúng thời điểm, và có thể tùy chọn gửi kết quả trở lại chat.

Nếu bạn muốn _"chạy cái này mỗi sáng"_ hoặc _"nhắc nhở agent sau 20 phút"_, cron job là cơ chế phù hợp.

Khắc phục sự cố: [/automation/troubleshooting](/automation/troubleshooting)
## Tóm tắt

- Cron job chạy **bên trong Gateway** (không chạy bên trong mô hình).
- Các công việc được lưu trữ dưới `~/.openclaw/cron/` để việc khởi động lại không làm mất lịch trình.
- Hai kiểu thực thi:
  - **Phiên chính**: đưa sự kiện hệ thống vào hàng đợi, sau đó chạy vào nhịp tim tiếp theo.
  - **Cô lập**: chạy một lượt agent riêng biệt trong `cron:<jobId>`, với việc gửi (thông báo theo mặc định hoặc không có).
- Đánh thức là tính năng hạng nhất: một công việc có thể yêu cầu "đánh thức ngay" so với "nhịp tim tiếp theo".
- Đăng webhook được thực hiện cho từng công việc thông qua `delivery.mode = "webhook"` + `delivery.to = "<url>"`.
- Vẫn giữ lại cơ chế dự phòng cũ cho các công việc đã lưu trữ với `notify: true` khi `cron.webhook` được đặt, hãy di chuyển những công việc đó sang chế độ gửi webhook.
## Bắt đầu nhanh (có thể thực hiện)

Tạo một lời nhắc một lần, xác minh nó tồn tại, và chạy ngay lập tức:

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

Schedule a recurring isolated job with delivery:

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```
## Tương đương lệnh gọi công cụ (Công cụ cron Gateway)

Để xem các hình dạng JSON chuẩn và ví dụ, hãy xem [Lược đồ JSON cho lệnh gọi công cụ](/automation/cron-jobs#json-schema-for-tool-calls).
## Nơi lưu trữ cron job

Cron job được lưu trữ trên máy chủ Gateway tại `~/.openclaw/cron/jobs.json` theo mặc định.
Gateway tải tệp vào bộ nhớ và ghi lại khi có thay đổi, vì vậy việc chỉnh sửa thủ công
chỉ an toàn khi Gateway đã dừng. Nên sử dụng `openclaw cron add/edit` hoặc API gọi công cụ cron
để thực hiện thay đổi.
## Tổng quan thân thiện với người mới bắt đầu

Hãy nghĩ về cron job như: **khi nào** chạy + **làm gì**.

1. **Chọn lịch trình**
   - Nhắc nhở một lần → `schedule.kind = "at"` (CLI: `--at`)
   - Công việc lặp lại → `schedule.kind = "every"` hoặc `schedule.kind = "cron"`
   - Nếu dấu thời gian ISO của bạn bỏ qua múi giờ, nó sẽ được coi là **UTC**.

2. **Chọn nơi chạy**
   - `sessionTarget: "main"` → chạy trong lần heartbeat tiếp theo với ngữ cảnh chính.
   - `sessionTarget: "isolated"` → chạy một lượt agent riêng biệt trong `cron:<jobId>`.

3. **Chọn payload**
   - Phiên chính → `payload.kind = "systemEvent"`
   - Phiên cô lập → `payload.kind = "agentTurn"`

Tùy chọn: các công việc một lần (`schedule.kind = "at"`) sẽ bị xóa sau khi thành công theo mặc định. Đặt
`deleteAfterRun: false` để giữ chúng (chúng sẽ bị vô hiệu hóa sau khi thành công).
## Khái niệm

### Jobs

Một cron job là một bản ghi được lưu trữ với:

- một **lịch trình** (khi nào nó nên chạy),
- một **payload** (nó nên làm gì),
- **chế độ giao hàng** tùy chọn (`announce`, `webhook`, hoặc `none`).
- **ràng buộc agent** tùy chọn (`agentId`): chạy job dưới một agent cụ thể; nếu
  thiếu hoặc không xác định, gateway sẽ quay về agent mặc định.

Jobs được xác định bằng một `jobId` ổn định (được sử dụng bởi CLI/Gateway APIs).
Trong các lời gọi công cụ agent, `jobId` là chuẩn; `id` cũ được chấp nhận để tương thích.
Jobs một lần tự động xóa sau khi thành công theo mặc định; đặt `deleteAfterRun: false` để giữ chúng.

### Lịch trình

Cron hỗ trợ ba loại lịch trình:

- `at`: dấu thời gian một lần qua `schedule.at` (ISO 8601).
- `every`: khoảng thời gian cố định (ms).
- `cron`: biểu thức cron 5 trường (hoặc 6 trường với giây) với múi giờ IANA tùy chọn.

Biểu thức cron sử dụng `croner`. Nếu múi giờ bị bỏ qua, múi giờ
cục bộ của máy chủ Gateway sẽ được sử dụng.

Để giảm các đỉnh tải đầu giờ trên nhiều gateway, OpenClaw áp dụng một
cửa sổ trễ xác định cho mỗi job lên đến 5 phút cho các biểu thức
đầu giờ định kỳ (ví dụ `0 * * * *`, `0 */2 * * *`). Các biểu thức
giờ cố định như `0 7 * * *` vẫn chính xác.

Đối với bất kỳ lịch trình cron nào, bạn có thể đặt cửa sổ trễ rõ ràng với `schedule.staggerMs`
(`0` giữ thời gian chính xác). Phím tắt CLI:

- `--stagger 30s` (hoặc `1m`, `5m`) để đặt cửa sổ trễ rõ ràng.
- `--exact` để buộc `staggerMs = 0`.

### Thực thi chính vs cô lập
#### Công việc phiên chính (sự kiện hệ thống)

Các công việc chính đưa một sự kiện hệ thống vào hàng đợi và tùy chọn đánh thức trình chạy heartbeat.
Chúng phải sử dụng `payload.kind = "systemEvent"`.

- `wakeMode: "now"` (mặc định): sự kiện kích hoạt một lần chạy heartbeat ngay lập tức.
- `wakeMode: "next-heartbeat"`: sự kiện chờ đợi heartbeat được lên lịch tiếp theo.

Đây là lựa chọn tốt nhất khi bạn muốn prompt heartbeat bình thường + ngữ cảnh phiên chính.
Xem [Heartbeat](/gateway/heartbeat).

#### Công việc cô lập (phiên cron chuyên dụng)

Các công việc cô lập chạy một lượt agent chuyên dụng trong phiên `cron:<jobId>`.

Hành vi chính:

- Prompt được thêm tiền tố với `[cron:<jobId> <job name>]` để có thể truy vết.
- Mỗi lần chạy bắt đầu một **id phiên mới** (không có cuộc trò chuyện trước đó được chuyển tiếp).
- Hành vi mặc định: nếu `delivery` bị bỏ qua, các công việc cô lập thông báo một bản tóm tắt (`delivery.mode = "announce"`).
- `delivery.mode` chọn điều gì sẽ xảy ra:
  - `announce`: gửi bản tóm tắt đến kênh đích và đăng bản tóm tắt ngắn gọn lên phiên chính.
  - `webhook`: POST payload sự kiện đã hoàn thành đến `delivery.to` khi sự kiện đã hoàn thành bao gồm bản tóm tắt.
  - `none`: chỉ nội bộ (không gửi, không có bản tóm tắt phiên chính).
- `wakeMode` kiểm soát khi nào bản tóm tắt phiên chính được đăng:
  - `now`: heartbeat ngay lập tức.
  - `next-heartbeat`: chờ đợi heartbeat được lên lịch tiếp theo.

Sử dụng các công việc cô lập cho những tác vụ ồn ào, thường xuyên, hoặc "công việc nền" không nên làm spam lịch sử trò chuyện chính của bạn.

### Hình dạng payload (những gì chạy)

Hai loại payload được hỗ trợ:

- `systemEvent`: chỉ phiên chính, được định tuyến qua prompt heartbeat.
- `agentTurn`: chỉ phiên cô lập, chạy một lượt agent chuyên dụng.

Các trường `agentTurn` chung:
- `message`: lời nhắc văn bản bắt buộc.
- `model` / `thinking`: ghi đè tùy chọn (xem bên dưới).
- `timeoutSeconds`: ghi đè thời gian chờ tùy chọn.

Cấu hình giao hàng:

- `delivery.mode`: `none` | `announce` | `webhook`.
- `delivery.channel`: `last` hoặc một kênh cụ thể.
- `delivery.to`: mục tiêu cụ thể của kênh (announce) hoặc URL webhook (chế độ webhook).
- `delivery.bestEffort`: tránh làm thất bại công việc nếu giao hàng announce thất bại.

Giao hàng announce ngăn chặn việc gửi công cụ nhắn tin cho lần chạy; sử dụng `delivery.channel`/`delivery.to`
để nhắm mục tiêu vào cuộc trò chuyện thay thế. Khi `delivery.mode = "none"`, không có tóm tắt nào được đăng lên phiên chính.

Nếu `delivery` bị bỏ qua cho các công việc cô lập, OpenClaw mặc định là `announce`.

#### Luồng giao hàng announce

Khi `delivery.mode = "announce"`, cron giao hàng trực tiếp thông qua các bộ điều hợp kênh đi.
Agent chính không được khởi động để tạo hoặc chuyển tiếp tin nhắn.

Chi tiết hành vi:

- Nội dung: giao hàng sử dụng các tải trọng đi của lần chạy cô lập (văn bản/phương tiện) với việc phân đoạn bình thường và
  định dạng kênh.
- Các phản hồi chỉ có heartbeat (`HEARTBEAT_OK` không có nội dung thực) không được giao hàng.
- Nếu lần chạy cô lập đã gửi tin nhắn đến cùng mục tiêu thông qua công cụ tin nhắn, việc giao hàng sẽ
  bị bỏ qua để tránh trùng lặp.
- Các mục tiêu giao hàng bị thiếu hoặc không hợp lệ sẽ làm thất bại công việc trừ khi `delivery.bestEffort = true`.
- Một tóm tắt ngắn được đăng lên phiên chính chỉ khi `delivery.mode = "announce"`.
- Tóm tắt phiên chính tuân thủ `wakeMode`: `now` kích hoạt heartbeat ngay lập tức và
  `next-heartbeat` chờ heartbeat được lên lịch tiếp theo.

#### Luồng giao hàng webhook

Khi `delivery.mode = "webhook"`, cron đăng tải trọng sự kiện hoàn thành lên `delivery.to` khi sự kiện hoàn thành bao gồm tóm tắt.

Chi tiết hành vi:

- Điểm cuối phải là URL HTTP(S) hợp lệ.
- Không có giao hàng kênh nào được thử trong chế độ webhook.
- Không có tóm tắt phiên chính nào được đăng trong chế độ webhook.
- Nếu `cron.webhookToken` được đặt, header xác thực là `Authorization: Bearer <cron.webhookToken>`.
- Fallback đã lỗi thời: các công việc cũ được lưu trữ với `notify: true` vẫn đăng lên `cron.webhook` (nếu được cấu hình), với cảnh báo để bạn có thể di chuyển sang `delivery.mode = "webhook"`.

### Ghi đè mô hình và suy nghĩ

Các công việc cô lập (`agentTurn`) có thể ghi đè mô hình và mức độ suy nghĩ:

- `model`: Chuỗi nhà cung cấp/mô hình (ví dụ: `anthropic/claude-sonnet-4-20250514`) hoặc bí danh (ví dụ: `opus`)
- `thinking`: Mức độ suy nghĩ (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`; chỉ dành cho mô hình GPT-5.2 + Codex)

Lưu ý: Bạn cũng có thể đặt `model` cho các công việc phiên chính, nhưng nó sẽ thay đổi mô hình phiên chính được chia sẻ. Chúng tôi khuyến nghị chỉ ghi đè mô hình cho các công việc cô lập để tránh những thay đổi ngữ cảnh không mong muốn.

Thứ tự ưu tiên giải quyết:

1. Ghi đè payload công việc (cao nhất)
2. Mặc định cụ thể của hook (ví dụ: `hooks.gmail.model`)
3. Mặc định cấu hình agent

### Giao hàng (kênh + đích)

Các công việc cô lập có thể giao đầu ra đến một kênh thông qua cấu hình `delivery` cấp cao nhất:

- `delivery.mode`: `announce` (giao hàng kênh), `webhook` (HTTP POST), hoặc `none`.
- `delivery.channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`.
- `delivery.to`: đích người nhận cụ thể của kênh.

Giao hàng `announce` chỉ hợp lệ cho các công việc cô lập (`sessionTarget: "isolated"`).
Giao hàng `webhook` hợp lệ cho cả công việc chính và cô lập.

Nếu `delivery.channel` hoặc `delivery.to` bị bỏ qua, cron có thể quay lại "tuyến đường cuối cùng" của phiên chính (nơi cuối cùng agent đã trả lời).

Lời nhắc định dạng đích:

- Các đích Slack/Discord/Mattermost (plugin) nên sử dụng tiền tố rõ ràng (ví dụ: `channel:<id>`, `user:<id>`) để tránh nhầm lẫn.
- Các chủ đề Telegram nên sử dụng dạng `:topic:` (xem bên dưới).
#### Mục tiêu gửi Telegram (chủ đề / luồng diễn đàn)

Telegram hỗ trợ chủ đề diễn đàn thông qua `message_thread_id`. Đối với gửi cron job, bạn có thể mã hóa
chủ đề/luồng vào trường `to`:

- `-1001234567890` (chỉ có id chat)
- `-1001234567890:topic:123` (ưu tiên: đánh dấu chủ đề rõ ràng)
- `-1001234567890:123` (viết tắt: hậu tố số)

Các mục tiêu có tiền tố như `telegram:...` / `telegram:group:...` cũng được chấp nhận:

- `telegram:group:-1001234567890:topic:123`
## Lược đồ JSON cho lời gọi công cụ

Sử dụng các hình dạng này khi gọi trực tiếp các công cụ Gateway `cron.*` (lời gọi công cụ agent hoặc RPC).
Các cờ CLI chấp nhận thời lượng dạng con người như `20m`, nhưng lời gọi công cụ nên sử dụng chuỗi ISO 8601
cho `schedule.at` và mili giây cho `schedule.everyMs`.

### Tham số cron.add

Công việc một lần, phiên chính (sự kiện hệ thống):

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

Recurring, isolated job with delivery:

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates."
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```
Ghi chú:

- `schedule.kind`: `at` (`at`), `every` (`everyMs`), hoặc `cron` (`expr`, tùy chọn `tz`).
- `schedule.at` chấp nhận ISO 8601 (múi giờ tùy chọn; được coi là UTC khi bỏ qua).
- `everyMs` tính bằng mili giây.
- `sessionTarget` phải là `"main"` hoặc `"isolated"` và phải khớp với `payload.kind`.
- Các trường tùy chọn: `agentId`, `description`, `enabled`, `deleteAfterRun` (mặc định là true cho `at`),
  `delivery`.
- `wakeMode` mặc định là `"now"` khi bỏ qua.

### Tham số cron.update

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

Notes:

- `jobId` is canonical; `id` is accepted for compatibility.
- Use `agentId: null` in the patch to clear an agent binding.

### cron.run and cron.remove params

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```
## Lưu trữ & lịch sử

- Kho lưu trữ công việc: `~/.openclaw/cron/jobs.json` (JSON được quản lý bởi Gateway).
- Lịch sử chạy: `~/.openclaw/cron/runs/<jobId>.jsonl` (JSONL, tự động cắt tỉa theo kích thước và số dòng).
- Các phiên chạy cron job cô lập trong `sessions.json` được cắt tỉa bởi `cron.sessionRetention` (mặc định `24h`; đặt `false` để vô hiệu hóa).
- Ghi đè đường dẫn lưu trữ: `cron.store` trong cấu hình.
## Cấu hình

```json5
{
  cron: {
    enabled: true, // default true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // default 1
    webhook: "https://example.invalid/legacy", // deprecated fallback for stored notify:true jobs
    webhookToken: "replace-with-dedicated-webhook-token", // optional bearer token for webhook mode
    sessionRetention: "24h", // duration string or false
    runLog: {
      maxBytes: "2mb", // default 2_000_000 bytes
      keepLines: 2000, // default 2000
    },
  },
}
```

Run-log pruning behavior:

- `cron.runLog.maxBytes`: max run-log file size before pruning.
- `cron.runLog.keepLines`: when pruning, keep only the newest N lines.
- Both apply to `cron/runs/<jobId>.jsonl` files.

Webhook behavior:

- Preferred: set `delivery.mode: "webhook"` with `delivery.to: "https://..."` per job.
- Webhook URLs must be valid `http://` or `https://` URLs.
- When posted, payload is the cron finished event JSON.
- If `cron.webhookToken` is set, auth header is `Authorization: Bearer <cron.webhookToken>`.
- If `cron.webhookToken` is not set, no `Authorization` header is sent.
- Deprecated fallback: stored legacy jobs with `notify: true` still use `cron.webhook` when present.

Disable cron entirely:

- `cron.enabled: false` (config)
- `OPENCLAW_SKIP_CRON=1` (biến môi trường)
## Bảo trì

Cron có hai đường dẫn bảo trì tích hợp: lưu giữ run-session cô lập và cắt tỉa run-log.

### Mặc định

- `cron.sessionRetention`: `24h` (đặt `false` để vô hiệu hóa việc cắt tỉa run-session)
- `cron.runLog.maxBytes`: `2_000_000` bytes
- `cron.runLog.keepLines`: `2000`

### Cách hoạt động

- Các lần chạy cô lập tạo ra các mục session (`...:cron:<jobId>:run:<uuid>`) và các tệp transcript.
- Bộ thu gom loại bỏ các mục run-session đã hết hạn cũ hơn `cron.sessionRetention`.
- Đối với các run session đã bị xóa không còn được tham chiếu bởi session store, OpenClaw lưu trữ các tệp transcript và xóa các kho lưu trữ cũ đã bị xóa trong cùng cửa sổ lưu giữ.
- Sau mỗi lần thêm run, `cron/runs/<jobId>.jsonl` được kiểm tra kích thước:
  - nếu kích thước tệp vượt quá `runLog.maxBytes`, nó sẽ được cắt tỉa xuống `runLog.keepLines` dòng mới nhất.

### Lưu ý về hiệu suất cho các bộ lập lịch khối lượng cao

Các thiết lập cron tần suất cao có thể tạo ra dấu chân run-session và run-log lớn. Bảo trì được tích hợp sẵn, nhưng các giới hạn lỏng lẻo vẫn có thể tạo ra công việc IO và dọn dẹp có thể tránh được.

Những gì cần theo dõi:

- cửa sổ `cron.sessionRetention` dài với nhiều lần chạy cô lập
- `cron.runLog.keepLines` cao kết hợp với `runLog.maxBytes` lớn
- nhiều công việc định kỳ ồn ào ghi vào cùng một `cron/runs/<jobId>.jsonl`

Những gì cần làm:

- giữ `cron.sessionRetention` ngắn nhất có thể theo nhu cầu gỡ lỗi/kiểm toán của bạn
- giữ run logs có giới hạn với `runLog.maxBytes` và `runLog.keepLines` vừa phải
- chuyển các công việc nền ồn ào sang chế độ cô lập với các quy tắc giao hàng tránh những cuộc trò chuyện không cần thiết
- xem xét sự tăng trưởng định kỳ với `openclaw cron runs` và điều chỉnh thời gian lưu giữ trước khi logs trở nên lớn

### Ví dụ tùy chỉnh

Giữ run sessions trong một tuần và cho phép run logs lớn hơn:

```json5
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```
Tắt tính năng dọn dẹp run-session riêng lẻ nhưng giữ tính năng dọn dẹp run-log:

```json5
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

Tune for high-volume cron usage (example):

```json5
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```
## Bắt đầu nhanh CLI

Nhắc nhở một lần (UTC ISO, tự động xóa sau khi thành công):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

One-shot reminder (main session, wake immediately):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Recurring isolated job (announce to WhatsApp):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Cron job định kỳ với độ trễ rõ ràng 30 giây:
```bash
openclaw cron add \
  --name "Minute watcher" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Run minute watcher checks." \
  --announce
```

Recurring isolated job (deliver to a Telegram topic):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Isolated job with model and thinking override:

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```
Lựa chọn agent (thiết lập đa agent):

```bash
# Pin a job to agent "ops" (falls back to default if that agent is missing)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# Switch or clear the agent on an existing job
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

Manual run (force is the default, use `--due` to only run when due):

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

Edit an existing job (patch fields):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Force an existing cron job to run exactly on schedule (no stagger):

```bash
openclaw cron edit <jobId> --exact
```

Run history:

```bash
openclaw cron runs --id <jobId> --limit 50
```
Sự kiện hệ thống tức thì mà không tạo job:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```
## Bề mặt API Gateway

- `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
- `cron.run` (force hoặc due), `cron.runs`
  Đối với các sự kiện hệ thống ngay lập tức không có job, hãy sử dụng [`openclaw system event`](/cli/system).
## Khắc phục sự cố

### "Không có gì chạy"

- Kiểm tra cron đã được bật: `cron.enabled` và `OPENCLAW_SKIP_CRON`.
- Kiểm tra Gateway đang chạy liên tục (cron chạy bên trong tiến trình Gateway).
- Đối với lịch trình `cron`: xác nhận múi giờ (`--tz`) so với múi giờ của máy chủ.

### Công việc định kỳ liên tục bị trì hoãn sau khi thất bại

- OpenClaw áp dụng cơ chế backoff theo cấp số nhân cho các công việc định kỳ sau những lỗi liên tiếp:
  30s, 1m, 5m, 15m, sau đó 60m giữa các lần thử lại.
- Backoff được đặt lại tự động sau lần chạy thành công tiếp theo.
- Các công việc một lần (`at`) sẽ bị vô hiệu hóa sau lần chạy cuối cùng (`ok`, `error`, hoặc `skipped`) và không thử lại.

### Telegram gửi đến sai nơi

- Đối với các chủ đề diễn đàn, sử dụng `-100…:topic:<id>` để rõ ràng và không gây nhầm lẫn.
- Nếu bạn thấy tiền tố `telegram:...` trong logs hoặc các mục tiêu "last route" được lưu trữ, điều đó là bình thường;
  việc gửi cron chấp nhận chúng và vẫn phân tích ID chủ đề một cách chính xác.

### Thử lại gửi thông báo subagent

- Khi một lần chạy subagent hoàn thành, gateway thông báo kết quả cho phiên yêu cầu.
- Nếu luồng thông báo trả về `false` (ví dụ: phiên yêu cầu đang bận), gateway sẽ thử lại tối đa 3 lần với theo dõi qua `announceRetryCount`.
- Các thông báo cũ hơn 5 phút sau `endedAt` sẽ bị hết hạn cưỡng chế để ngăn các mục cũ lặp vô hạn.
- Nếu bạn thấy việc gửi thông báo lặp lại trong logs, hãy kiểm tra registry subagent để tìm các mục có giá trị `announceRetryCount` cao.