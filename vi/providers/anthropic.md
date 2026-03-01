---
summary: Sử dụng Anthropic Claude thông qua API keys hoặc setup-token trong OpenClaw
read_when:
  - Bạn muốn sử dụng các mô hình của Anthropic trong OpenClaw
  - Bạn muốn setup-token thay vì API keys
title: Anthropic
x-i18n:
  source_path: providers\anthropic.md
  source_hash: c651eaa6367eac4880c566a924b308b8b99cd0119f6fc25746c7e0b381e84083
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:28.356Z'
---

# Anthropic (Claude)

Anthropic xây dựng họ mô hình **Claude** và cung cấp quyền truy cập thông qua API.
Trong OpenClaw, bạn có thể xác thực bằng khóa API hoặc **setup-token**.
## Tùy chọn A: Khóa API Anthropic

**Tốt nhất cho:** truy cập API tiêu chuẩn và thanh toán theo mức sử dụng.
Tạo khóa API của bạn trong Bảng điều khiển Anthropic.

### Thiết lập CLI

```bash
openclaw onboard
# choose: Anthropic API key

# or non-interactive
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### Config snippet

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```
## Lưu trữ đệm lời nhắc (Anthropic API)

OpenClaw hỗ trợ tính năng lưu trữ đệm lời nhắc của Anthropic. Đây là **chỉ API**; xác thực đăng ký không tuân theo cài đặt bộ nhớ đệm.

### Cấu hình

Sử dụng tham số `cacheRetention` trong cấu hình mô hình của bạn:

| Giá trị   | Thời gian lưu trữ đệm | Mô tả                         |
| ------- | -------------- | ----------------------------------- |
| `none`  | Không lưu trữ đệm     | Vô hiệu hóa lưu trữ đệm lời nhắc              |
| `short` | 5 phút      | Mặc định cho xác thực API Key            |
| `long`  | 1 giờ         | Bộ nhớ đệm mở rộng (yêu cầu cờ beta) |

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### Defaults

When using Anthropic API Key authentication, OpenClaw automatically applies `cacheRetention: "short"` (5-minute cache) for all Anthropic models. You can override this by explicitly setting `cacheRetention` in your config.

### Per-agent cacheRetention overrides

Use model-level params as your baseline, then override specific agents via `agents.list[].params`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // baseline for most agents
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // override for this agent only
    ],
  },
}
```

Config merge order for cache-related params:

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` (matching `id`, ghi đè theo khóa)
Điều này cho phép một agent duy trì bộ nhớ cache lâu dài trong khi một agent khác trên cùng mô hình vô hiệu hóa bộ nhớ cache để tránh chi phí ghi trên lưu lượng bursty/low-reuse.

### Ghi chú Bedrock Claude

- Các mô hình Anthropic Claude trên Bedrock (`amazon-bedrock/*anthropic.claude*`) chấp nhận `cacheRetention` pass-through khi được cấu hình.
- Các mô hình Bedrock không phải Anthropic bị buộc phải `cacheRetention: "none"` tại thời gian chạy.
- Các giá trị mặc định thông minh khóa API Anthropic cũng khởi tạo `cacheRetention: "short"` cho các tham chiếu mô hình Claude-on-Bedrock khi không có giá trị rõ ràng nào được đặt.

### Tham số cũ

Tham số `cacheControlTtl` cũ hơn vẫn được hỗ trợ để tương thích ngược:

- `"5m"` ánh xạ tới `short`
- `"1h"` ánh xạ tới `long`

Chúng tôi khuyên bạn nên di chuyển sang tham số `cacheRetention` mới.

OpenClaw bao gồm cờ beta `extended-cache-ttl-2025-04-11` cho các yêu cầu API Anthropic; giữ nó nếu bạn ghi đè các tiêu đề nhà cung cấp (xem [/gateway/configuration](/gateway/configuration)).
## Cửa sổ ngữ cảnh 1M (Anthropic beta)

Cửa sổ ngữ cảnh 1M của Anthropic được kiểm soát bằng beta. Trong OpenClaw, bật nó cho từng mô hình bằng `params.context1m: true` cho các mô hình Opus/Sonnet được hỗ trợ.

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw maps this to `anthropic-beta: context-1m-2025-08-07` on Anthropic
requests.

Note: Anthropic currently rejects `context-1m-*` beta requests when using
OAuth/subscription tokens (`sk-ant-oat-*`). OpenClaw tự động bỏ qua tiêu đề beta context1m cho xác thực OAuth và giữ lại các beta OAuth cần thiết.
## Tùy chọn B: Claude setup-token

**Tốt nhất cho:** sử dụng đăng ký Claude của bạn.

### Nơi lấy setup-token

Setup-tokens được tạo bởi **Claude Code CLI**, không phải bằng Anthropic Console. Bạn có thể chạy lệnh này trên **bất kỳ máy nào**:

```bash
claude setup-token
```

Paste the token into OpenClaw (wizard: **Anthropic token (paste setup-token)**), or run it on the gateway host:

```bash
openclaw models auth setup-token --provider anthropic
```

If you generated the token on a different machine, paste it:

```bash
openclaw models auth paste-token --provider anthropic
```

### CLI setup (setup-token)

```bash
# Paste a setup-token during onboarding
openclaw onboard --auth-choice setup-token
```

### Config snippet (setup-token)

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```
## Ghi chú

- Tạo setup-token với `claude setup-token` và dán nó, hoặc chạy `openclaw models auth setup-token` trên máy chủ gateway.
- Nếu bạn thấy "OAuth token refresh failed …" trên đăng ký Claude, hãy xác thực lại bằng setup-token. Xem [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription).
- Chi tiết xác thực + quy tắc tái sử dụng nằm trong [/concepts/oauth](/concepts/oauth).
## Khắc phục sự cố

**Lỗi 401 / token đột ngột không hợp lệ**

- Xác thực đăng ký Claude có thể hết hạn hoặc bị thu hồi. Chạy lại `claude setup-token`
  và dán nó vào **gateway host**.
- Nếu đăng nhập Claude CLI nằm trên một máy khác, hãy sử dụng
  `openclaw models auth paste-token --provider anthropic` trên gateway host.

**Không tìm thấy API key cho nhà cung cấp "anthropic"**

- Xác thực là **cho mỗi agent**. Các agent mới không kế thừa các khóa của agent chính.
- Chạy lại thiết lập ban đầu cho agent đó, hoặc dán setup-token / API key trên
  gateway host, sau đó xác minh bằng `openclaw models status`.

**Không tìm thấy thông tin xác thực cho hồ sơ `anthropic:default`**

- Chạy `openclaw models status` để xem hồ sơ xác thực nào đang hoạt động.
- Chạy lại thiết lập ban đầu, hoặc dán setup-token / API key cho hồ sơ đó.

**Không có hồ sơ xác thực khả dụng (tất cả đều trong thời gian chờ/không khả dụng)**

- Kiểm tra `openclaw models status --json` cho `auth.unusableProfiles`.
- Thêm một hồ sơ Anthropic khác hoặc chờ thời gian chờ.

Thêm: [/gateway/troubleshooting](/gateway/troubleshooting) và [/help/faq](/help/faq).