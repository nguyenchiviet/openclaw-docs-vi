---
title: Bộ nhớ đệm Prompt
summary: >-
  Các nút điều chỉnh caching prompt, thứ tự hợp nhất, hành vi nhà cung cấp và
  các mẫu điều chỉnh
read_when:
  - Bạn muốn giảm chi phí token prompt bằng cách giữ lại bộ nhớ cache
  - Bạn cần hành vi bộ nhớ cache cho từng agent trong các thiết lập đa agent
  - Bạn đang điều chỉnh heartbeat và cache-ttl pruning cùng nhau
x-i18n:
  source_path: reference\prompt-caching.md
  source_hash: 7952e90d0d6eb23fee4e0046220dddc7c89dc19aae0129d0619290e081a92778
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:20:06.703Z'
---

# Lưu trữ đệm Prompt

Lưu trữ đệm Prompt có nghĩa là nhà cung cấp mô hình có thể tái sử dụng các tiền tố prompt không thay đổi (thường là hướng dẫn hệ thống/nhà phát triển và ngữ cảnh ổn định khác) trên các lượt thay vì xử lý lại chúng mỗi lần. Yêu cầu khớp đầu tiên ghi các token bộ đệm (`cacheWrite`), và các yêu cầu khớp sau có thể đọc lại chúng (`cacheRead`).

Tại sao điều này quan trọng: chi phí token thấp hơn, phản hồi nhanh hơn và hiệu suất dễ dự đoán hơn cho các phiên chạy dài. Nếu không có lưu trữ đệm, các prompt lặp lại phải trả toàn bộ chi phí prompt trên mỗi lượt ngay cả khi hầu hết đầu vào không thay đổi.

Trang này bao gồm tất cả các tùy chọn liên quan đến bộ đệm ảnh hưởng đến tái sử dụng prompt và chi phí token.

Để biết chi tiết về giá của Anthropic, xem:
[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)
## Các tham số chính

### `cacheRetention` (mô hình và mỗi agent)

Đặt thời gian lưu giữ bộ nhớ đệm trên các tham số mô hình:

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

Per-agent override:

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

Config merge order:

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` (matching agent id; overrides by key)

### Legacy `cacheControlTtl`

Legacy values are still accepted and mapped:

- `5m` -> `short`
- `1h` -> `long`

Prefer `cacheRetention` for new config.

### `contextPruning.mode: "cache-ttl"`

Prunes old tool-result context after cache TTL windows so post-idle requests do not re-cache oversized history.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

See [Session Pruning](/concepts/session-pruning) for full behavior.

### Heartbeat keep-warm

Heartbeat can keep cache windows warm and reduce repeated cache writes after idle gaps.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```
Nhịp tim trên mỗi agent được hỗ trợ tại `agents.list[].heartbeat`.

## Hành vi nhà cung cấp

### Anthropic (API trực tiếp)

- `cacheRetention` được hỗ trợ.
- Với hồ sơ xác thực API-key của Anthropic, OpenClaw khởi tạo `cacheRetention: "short"` cho các tham chiếu mô hình Anthropic khi chưa được đặt.

### Amazon Bedrock

- Các tham chiếu mô hình Anthropic Claude (`amazon-bedrock/*anthropic.claude*`) hỗ trợ truyền qua `cacheRetention` rõ ràng.
- Các mô hình Bedrock không phải Anthropic bị buộc phải `cacheRetention: "none"` tại thời chạy.

### Mô hình Anthropic của OpenRouter

Đối với các tham chiếu mô hình `openrouter/anthropic/*`, OpenClaw tiêm `cache_control` của Anthropic vào các khối lời nhắc hệ thống/nhà phát triển để cải thiện việc tái sử dụng bộ nhớ đệm lời nhắc.

### Các nhà cung cấp khác

Nếu nhà cung cấp không hỗ trợ chế độ bộ nhớ đệm này, `cacheRetention` không có tác dụng.
## Các mẫu điều chỉnh

### Lưu lượng truy cập hỗn hợp (mặc định được khuyến nghị)

Giữ một đường cơ sở dài hạn trên agent chính của bạn, vô hiệu hóa bộ nhớ cache trên các agent thông báo bùng nổ:

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### Cost-first baseline

- Set baseline `cacheRetention: "short"`.
- Enable `contextPruning.mode: "cache-ttl"`.
- Giữ nhịp tim dưới TTL của bạn chỉ cho các agent được hưởng lợi từ bộ nhớ cache ấm.
## Chẩn đoán bộ nhớ đệm

OpenClaw cung cấp chẩn đoán theo dõi bộ nhớ đệm chuyên dụng cho các lần chạy agent nhúng.

### `diagnostics.cacheTrace` config

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # optional
    includeMessages: false # default true
    includePrompt: false # default true
    includeSystem: false # default true
```

Defaults:

- `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
- `includeMessages`: `true`
- `includePrompt`: `true`
- `includeSystem`: `true`

### Env toggles (one-off debugging)

- `OPENCLAW_CACHE_TRACE=1` enables cache tracing.
- `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` overrides output path.
- `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` toggles full message payload capture.
- `OPENCLAW_CACHE_TRACE_PROMPT=0|1` toggles prompt text capture.
- `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` toggles system prompt capture.

### What to inspect

- Cache trace events are JSONL and include staged snapshots like `session:loaded`, `prompt:before`, `stream:context`, and `session:after`.
- Per-turn cache token impact is visible in normal usage surfaces via `cacheRead` and `cacheWrite` (for example `/usage đầy đủ` và tóm tắt sử dụng phiên).
## Khắc phục sự cố nhanh

- `cacheWrite` cao trên hầu hết các lượt: kiểm tra các đầu vào system-prompt không ổn định và xác minh mô hình/nhà cung cấp hỗ trợ cài đặt bộ nhớ đệm của bạn.
- Không có hiệu quả từ `cacheRetention`: xác nhận khóa mô hình khớp với `agents.defaults.models["provider/model"]`.
- Yêu cầu Bedrock Nova/Mistral với cài đặt bộ nhớ đệm: thời gian chạy dự kiến buộc phải `none`.

Tài liệu liên quan:

- [Anthropic](/providers/anthropic)
- [Sử dụng Token và Chi phí](/reference/token-use)
- [Cắt tỉa Phiên](/concepts/session-pruning)
- [Tham chiếu Cấu hình Gateway](/gateway/configuration-reference)