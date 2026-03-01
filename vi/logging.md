---
summary: 'Tổng quan về Logging: file logs, console output, CLI tailing, và Control UI'
read_when:
  - Bạn cần một cái nhìn tổng quan về logging thân thiện với người mới bắt đầu
  - Bạn muốn cấu hình mức độ ghi nhật ký hoặc định dạng
  - Bạn đang khắc phục sự cố và cần tìm nhật ký nhanh chóng
title: Ghi nhật ký
x-i18n:
  source_path: logging.md
  source_hash: 11907b8364e374c6eb1f157f380c2c14aced3053ef28c0f90e732ab475c21f93
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:30.244Z'
---

# Ghi nhật ký

OpenClaw ghi nhật ký ở hai nơi:

- **Nhật ký tệp** (JSON lines) được viết bởi Gateway.
- **Đầu ra bảng điều khiển** hiển thị trong các terminal và Control UI.

Trang này giải thích vị trí lưu trữ nhật ký, cách đọc chúng và cách định cấu hình mức độ ghi nhật ký và định dạng.
## Nơi lưu trữ nhật ký

Theo mặc định, Gateway ghi tệp nhật ký lăn dưới:

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

Ngày sử dụng múi giờ địa phương của máy chủ gateway.

Bạn có thể ghi đè điều này trong `~/.openclaw/openclaw.json`:

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```
## Cách đọc nhật ký

### CLI: theo dõi trực tiếp (được khuyến nghị)

Sử dụng CLI để theo dõi tệp nhật ký gateway thông qua RPC:

```bash
openclaw logs --follow
```

Output modes:

- **TTY sessions**: pretty, colorized, structured log lines.
- **Non-TTY sessions**: plain text.
- `--json`: line-delimited JSON (one log event per line).
- `--plain`: force plain text in TTY sessions.
- `--no-color`: disable ANSI colors.

In JSON mode, the CLI emits `type`-tagged objects:

- `meta`: stream metadata (file, cursor, size)
- `log`: parsed log entry
- `notice`: truncation / rotation hints
- `raw`: unparsed log line

If the Gateway is unreachable, the CLI prints a short hint to run:

```bash
openclaw doctor
```

### Control UI (web)

The Control UI’s **Logs** tab tails the same file using `logs.tail`.
See [/web/control-ui](/web/control-ui) for how to open it.

### Channel-only logs

To filter channel activity (WhatsApp/Telegram/etc), use:

```bash
openclaw channels logs --channel whatsapp
```
## Định dạng nhật ký

### Nhật ký tệp (JSONL)

Mỗi dòng trong tệp nhật ký là một đối tượng JSON. CLI và Control UI phân tích các mục này để hiển thị đầu ra có cấu trúc (thời gian, mức độ, hệ thống con, thông báo).

### Đầu ra bảng điều khiển

Nhật ký bảng điều khiển **nhận biết TTY** và được định dạng để dễ đọc:

- Tiền tố hệ thống con (ví dụ: `gateway/channels/whatsapp`)
- Tô màu mức độ (info/warn/error)
- Chế độ compact hoặc JSON tùy chọn

Định dạng bảng điều khiển được kiểm soát bởi `logging.consoleStyle`.
## Cấu hình ghi nhật ký

Tất cả cấu hình ghi nhật ký nằm dưới `logging` trong `~/.openclaw/openclaw.json`.

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": ["sk-.*"]
  }
}
```

### Log levels

- `logging.level`: **file logs** (JSONL) level.
- `logging.consoleLevel`: **console** verbosity level.

You can override both via the **`OPENCLAW_LOG_LEVEL`** environment variable (e.g. `OPENCLAW_LOG_LEVEL=debug`). The env var takes precedence over the config file, so you can raise verbosity for a single run without editing `openclaw.json`. You can also pass the global CLI option **`--log-level <level>`** (for example, `openclaw --log-level debug gateway run`), which overrides the environment variable for that command.

`--verbose` only affects console output; it does not change file log levels.

### Console styles

`logging.consoleStyle`:

- `pretty`: human-friendly, colored, with timestamps.
- `compact`: tighter output (best for long sessions).
- `json`: JSON per line (for log processors).

### Redaction

Tool summaries can redact sensitive tokens before they hit the console:

- `logging.redactSensitive`: `off` | `tools` (default: `tools`)
- `logging.redactPatterns`: danh sách các chuỗi regex để ghi đè bộ mặc định

Biên tập ảnh hưởng đến **đầu ra bảng điều khiển chỉ** và không thay đổi nhật ký tệp.
## Chẩn đoán + OpenTelemetry

Chẩn đoán là các sự kiện có cấu trúc, có thể đọc được bởi máy cho các lần chạy mô hình **và**
telemetry luồng tin nhắn (webhooks, queueing, trạng thái phiên). Chúng **không**
thay thế nhật ký; chúng tồn tại để cung cấp cho các chỉ số, dấu vết và các trình xuất khác.

Các sự kiện chẩn đoán được phát ra trong quá trình, nhưng các trình xuất chỉ được đính kèm khi
chẩn đoán + plugin trình xuất được bật.

### OpenTelemetry vs OTLP

- **OpenTelemetry (OTel)**: mô hình dữ liệu + SDK cho dấu vết, chỉ số và nhật ký.
- **OTLP**: giao thức dây được sử dụng để xuất dữ liệu OTel tới bộ sưu tập/backend.
- OpenClaw xuất qua **OTLP/HTTP (protobuf)** ngày hôm nay.

### Tín hiệu được xuất

- **Chỉ số**: bộ đếm + biểu đồ (sử dụng token, luồng tin nhắn, queueing).
- **Dấu vết**: khoảng thời gian cho sử dụng mô hình + xử lý webhook/tin nhắn.
- **Nhật ký**: được xuất qua OTLP khi `diagnostics.otel.logs` được bật. Khối lượng nhật ký
  có thể cao; hãy ghi nhớ `logging.level` và các bộ lọc trình xuất.

### Danh mục sự kiện chẩn đoán

Sử dụng mô hình:

- `model.usage`: token, chi phí, thời lượng, ngữ cảnh, nhà cung cấp/mô hình/kênh, id phiên.

Luồng tin nhắn:

- `webhook.received`: webhook ingress trên mỗi kênh.
- `webhook.processed`: webhook được xử lý + thời lượng.
- `webhook.error`: lỗi trình xử lý webhook.
- `message.queued`: tin nhắn được xếp hàng để xử lý.
- `message.processed`: kết quả + thời lượng + lỗi tùy chọn.

Hàng đợi + phiên:

- `queue.lane.enqueue`: làn hàng đợi lệnh enqueue + độ sâu.
- `queue.lane.dequeue`: làn hàng đợi lệnh dequeue + thời gian chờ.
- `session.state`: chuyển đổi trạng thái phiên + lý do.
- `session.stuck`: cảnh báo phiên bị kẹt + tuổi.
- `run.attempt`: siêu dữ liệu thử lại/nỗ lực chạy.
- `diagnostic.heartbeat`: bộ đếm tổng hợp (webhooks/hàng đợi/phiên).

### Bật chẩn đoán (không có trình xuất)

Sử dụng điều này nếu bạn muốn các sự kiện chẩn đoán có sẵn cho các plugin hoặc bộ chứa tùy chỉnh:

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

### Diagnostics flags (targeted logs)

Use flags to turn on extra, targeted debug logs without raising `logging.level`.
```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Env override (one-off):

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Notes:

- Flag logs go to the standard log file (same as `logging.file`).
- Output is still redacted according to `logging.redactSensitive`.
- Full guide: [/diagnostics/flags](/diagnostics/flags).

### Export to OpenTelemetry

Diagnostics can be exported via the `diagnostics-otel` plugin (OTLP/HTTP). This
works with any OpenTelemetry collector/backend that accepts OTLP/HTTP.

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

Notes:

- You can also enable the plugin with `openclaw plugins enable diagnostics-otel`.
- `protocol` currently supports `http/protobuf` only. `grpc` bị bỏ qua.
- Các chỉ số bao gồm sử dụng token, chi phí, kích thước ngữ cảnh, thời gian chạy và bộ đếm/biểu đồ luồng tin nhắn (webhook, xếp hàng, trạng thái phiên, độ sâu hàng đợi/thời gian chờ).
- Traces/metrics có thể được bật/tắt với `traces` / `metrics` (mặc định: bật). Traces bao gồm các span sử dụng mô hình cộng với các span xử lý webhook/tin nhắn khi được bật.
- Đặt `headers` khi bộ thu thập của bạn yêu cầu xác thực.
- Biến môi trường được hỗ trợ: `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

### Các chỉ số được xuất (tên + loại)

Sử dụng mô hình:

- `openclaw.tokens` (counter, attrs: `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
- `openclaw.cost.usd` (counter, attrs: `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
- `openclaw.run.duration_ms` (histogram, attrs: `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
- `openclaw.context.tokens` (histogram, attrs: `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

Luồng tin nhắn:

- `openclaw.webhook.received` (counter, attrs: `openclaw.channel`,
  `openclaw.webhook`)
- `openclaw.webhook.error` (counter, attrs: `openclaw.channel`,
  `openclaw.webhook`)
- `openclaw.webhook.duration_ms` (histogram, attrs: `openclaw.channel`,
  `openclaw.webhook`)
- `openclaw.message.queued` (counter, attrs: `openclaw.channel`,
  `openclaw.source`)
- `openclaw.message.processed` (counter, attrs: `openclaw.channel`,
  `openclaw.outcome`)
- `openclaw.message.duration_ms` (histogram, attrs: `openclaw.channel`,
  `openclaw.outcome`)

Hàng đợi + phiên:

- `openclaw.queue.lane.enqueue` (counter, attrs: `openclaw.lane`)
- `openclaw.queue.lane.dequeue` (counter, attrs: `openclaw.lane`)
- `openclaw.queue.depth` (histogram, attrs: `openclaw.lane` hoặc
  `openclaw.channel=heartbeat`)
- `openclaw.queue.wait_ms` (histogram, attrs: `openclaw.lane`)
- `openclaw.session.state` (counter, attrs: `openclaw.state`, `openclaw.reason`)
- `openclaw.session.stuck` (counter, attrs: `openclaw.state`)
- `openclaw.session.stuck_age_ms` (histogram, attrs: `openclaw.state`)
- `openclaw.run.attempt` (counter, attrs: `openclaw.attempt`)

### Các span được xuất (tên + các thuộc tính chính)

- `openclaw.model.usage`
  - `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  - `openclaw.sessionKey`, `openclaw.sessionId`
  - `openclaw.tokens.*` (input/output/cache_read/cache_write/total)
- `openclaw.webhook.processed`
  - `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
- `openclaw.webhook.error`
  - `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
- `openclaw.message.processed`
  - `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
`openclaw.reason`
- `openclaw.session.stuck`
  - `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

### Lấy mẫu + xả dữ liệu

- Lấy mẫu theo dõi: `diagnostics.otel.sampleRate` (0.0–1.0, chỉ các span gốc).
- Khoảng thời gian xuất metric: `diagnostics.otel.flushIntervalMs` (tối thiểu 1000ms).

### Ghi chú về giao thức

- Các điểm cuối OTLP/HTTP có thể được đặt thông qua `diagnostics.otel.endpoint` hoặc
  `OTEL_EXPORTER_OTLP_ENDPOINT`.
- Nếu điểm cuối đã chứa `/v1/traces` hoặc `/v1/metrics`, nó sẽ được sử dụng nguyên trạng.
- Nếu điểm cuối đã chứa `/v1/logs`, nó sẽ được sử dụng nguyên trạng cho nhật ký.
- `diagnostics.otel.logs` cho phép xuất nhật ký OTLP cho đầu ra logger chính.

### Hành vi xuất nhật ký

- Nhật ký OTLP sử dụng các bản ghi có cấu trúc tương tự được ghi vào `logging.file`.
- Tôn trọng `logging.level` (mức độ nhật ký tệp). Biên tập lại bảng điều khiển **không** áp dụng
  cho nhật ký OTLP.
- Các cài đặt khối lượng cao nên ưu tiên lấy mẫu/lọc bộ thu thập OTLP.
## Mẹo khắc phục sự cố

- **Gateway không thể truy cập được?** Chạy `openclaw doctor` trước tiên.
- **Nhật ký trống?** Kiểm tra xem Gateway có đang chạy và ghi vào đường dẫn tệp trong `logging.file`.
- **Cần chi tiết hơn?** Đặt `logging.level` thành `debug` hoặc `trace` và thử lại.