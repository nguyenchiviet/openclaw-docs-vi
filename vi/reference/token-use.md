---
summary: Cách OpenClaw xây dựng ngữ cảnh prompt và báo cáo sử dụng token + chi phí
read_when:
  - 'Giải thích về sử dụng token, chi phí hoặc cửa sổ ngữ cảnh'
  - Gỡ lỗi hành vi tăng trưởng hoặc nén ngữ cảnh
title: Sử dụng Token và Chi phí
x-i18n:
  source_path: reference\token-use.md
  source_hash: e5732050dd7e1fbda296c63de47a148a7205a882ca060303798c82ff5080b98b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:22:35.814Z'
---

# Sử dụng token & chi phí

OpenClaw theo dõi **token**, không phải ký tự. Token là dành riêng cho từng mô hình, nhưng hầu hết các mô hình kiểu OpenAI trung bình ~4 ký tự trên mỗi token cho văn bản tiếng Anh.
## Cách xây dựng lời nhắc hệ thống

OpenClaw tự lắp ráp lời nhắc hệ thống của nó trên mỗi lần chạy. Nó bao gồm:

- Danh sách công cụ + mô tả ngắn
- Danh sách Skills (chỉ siêu dữ liệu; hướng dẫn được tải theo yêu cầu với `read`)
- Hướng dẫn tự cập nhật
- Workspace + tệp bootstrap (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` khi mới, cộng với `MEMORY.md` và/hoặc `memory.md` khi có). Các tệp lớn được cắt ngắn bởi `agents.defaults.bootstrapMaxChars` (mặc định: 20000), và tổng số tiêm bootstrap bị giới hạn bởi `agents.defaults.bootstrapTotalMaxChars` (mặc định: 150000). Các tệp `memory/*.md` được tải theo yêu cầu thông qua các công cụ bộ nhớ và không được tiêm tự động.
- Thời gian (UTC + múi giờ người dùng)
- Thẻ trả lời + hành vi nhịp tim
- Siêu dữ liệu thời chạy (máy chủ/HĐH/mô hình/tư duy)

Xem phân tích đầy đủ trong [System Prompt](/concepts/system-prompt).
## Những gì tính vào cửa sổ ngữ cảnh

Mọi thứ mô hình nhận được đều tính vào giới hạn ngữ cảnh:

- System prompt (tất cả các phần được liệt kê ở trên)
- Lịch sử cuộc trò chuyện (tin nhắn của người dùng + trợ lý)
- Lệnh gọi công cụ và kết quả công cụ
- Tệp đính kèm/bản ghi chép (hình ảnh, âm thanh, tệp)
- Tóm tắt nén và các tạo tác cắt tỉa
- Trình bao bọc nhà cung cấp hoặc tiêu đề bảo mật (không hiển thị, nhưng vẫn được tính)

Đối với hình ảnh, OpenClaw giảm tỷ lệ các tải trọng hình ảnh công cụ/bản ghi chép trước các lệnh gọi nhà cung cấp.
Sử dụng `agents.defaults.imageMaxDimensionPx` (mặc định: `1200`) để điều chỉnh:

- Các giá trị thấp hơn thường giảm mức sử dụng token vision và kích thước tải trọng.
- Các giá trị cao hơn bảo toàn nhiều chi tiết hình ảnh hơn cho các ảnh chụp màn hình nặng OCR/UI.

Để có bảng phân tích thực tế (cho mỗi tệp được tiêm, công cụ, Skills và kích thước system prompt), sử dụng `/context list` hoặc `/context detail`. Xem [Context](/concepts/context).
## Cách xem mức sử dụng token hiện tại

Sử dụng những lệnh này trong chat:

- `/status` → **thẻ trạng thái giàu emoji** với mô hình phiên, mức sử dụng ngữ cảnh,
  token đầu vào/đầu ra của phản hồi cuối cùng, và **chi phí ước tính** (chỉ khóa API).
- `/usage off|tokens|full` → thêm **chân trang sử dụng cho mỗi phản hồi** vào mọi trả lời.
  - Tồn tại theo phiên (được lưu trữ dưới dạng `responseUsage`).
  - Xác thực OAuth **ẩn chi phí** (chỉ token).
- `/usage cost` → hiển thị tóm tắt chi phí cục bộ từ nhật ký phiên OpenClaw.

Các bề mặt khác:

- **TUI/Web TUI:** `/status` + `/usage` được hỗ trợ.
- **CLI:** `openclaw status --usage` và `openclaw channels list` hiển thị
  cửa sổ hạn ngạch nhà cung cấp (không phải chi phí cho mỗi phản hồi).
## Ước tính chi phí (khi hiển thị)

Chi phí được ước tính từ cấu hình giá mô hình của bạn:

```
models.providers.<provider>.models[].cost
```

These are **USD per 1M tokens** for `input`, `output`, `cacheRead`, and
`cacheWrite`. Nếu thiếu thông tin giá, OpenClaw chỉ hiển thị token. Các token OAuth không bao giờ hiển thị chi phí theo đô la.
## Tác động của Cache TTL và pruning

Nhà cung cấp prompt caching chỉ áp dụng trong cửa sổ cache TTL. OpenClaw có thể
tùy chọn chạy **cache-ttl pruning**: nó pruning phiên một khi cache TTL
đã hết hạn, sau đó đặt lại cửa sổ cache để các yêu cầu tiếp theo có thể tái sử dụng
ngữ cảnh được lưu vào cache mới thay vì lưu vào cache lại toàn bộ lịch sử. Điều này giữ chi phí ghi cache
thấp hơn khi một phiên không hoạt động quá TTL.

Cấu hình nó trong [Cấu hình Gateway](/gateway/configuration) và xem chi tiết
hành vi trong [Session pruning](/concepts/session-pruning).

Heartbeat có thể giữ cache **ấm** qua các khoảng không hoạt động. Nếu cache TTL mô hình của bạn
là `1h`, đặt khoảng thời gian heartbeat chỉ dưới đó (ví dụ: `55m`) có thể tránh
lưu vào cache lại toàn bộ prompt, giảm chi phí ghi cache.

Trong các thiết lập đa agent, bạn có thể giữ một cấu hình mô hình được chia sẻ và điều chỉnh hành vi cache
cho mỗi agent với `agents.list[].params.cacheRetention`.

Để có hướng dẫn chi tiết từng nút, xem [Prompt Caching](/reference/prompt-caching).

Đối với giá API của Anthropic, đọc cache rẻ hơn đáng kể so với input
tokens, trong khi ghi cache được tính theo bội số cao hơn. Xem giá prompt caching
của Anthropic để biết tỷ giá và bội số TTL mới nhất:
[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### Ví dụ: giữ cache 1h ấm với heartbeat

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### Example: mixed traffic with per-agent cache strategy

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # default baseline for most agents
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # keep long cache warm for deep sessions
    - id: "alerts"
      params:
        cacheRetention: "none" # avoid cache writes for bursty notifications
```
`agents.list[].params` hợp nhất trên `params` của mô hình được chọn, vì vậy bạn có thể
ghi đè chỉ `cacheRetention` và kế thừa các giá trị mặc định của mô hình khác không thay đổi.

### Ví dụ: bật tiêu đề beta ngữ cảnh 1M của Anthropic

Cửa sổ ngữ cảnh 1M của Anthropic hiện đang bị giới hạn beta. OpenClaw có thể tiêm
giá trị `anthropic-beta` cần thiết khi bạn bật `context1m` trên các mô hình Opus
hoặc Sonnet được hỗ trợ.

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

This maps to Anthropic's `context-1m-2025-08-07` beta header.

If you authenticate Anthropic with OAuth/subscription tokens (`sk-ant-oat-*`),
OpenClaw skips the `context-1m-*` tiêu đề beta vì Anthropic hiện tại
từ chối kết hợp đó với HTTP 401.
## Mẹo giảm áp lực token

- Sử dụng `/compact` để tóm tắt các phiên dài.
- Cắt bớt đầu ra công cụ lớn trong quy trình làm việc của bạn.
- Hạ thấp `agents.defaults.imageMaxDimensionPx` cho các phiên có nhiều ảnh chụp màn hình.
- Giữ mô tả skill ngắn (danh sách skill được chèn vào prompt).
- Ưu tiên các mô hình nhỏ hơn cho công việc dài dòng, khám phá.

Xem [Skills](/tools/skills) để biết công thức tính toán chi phí danh sách skill chính xác.