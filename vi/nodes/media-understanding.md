---
summary: >-
  Hiểu biết hình ảnh/âm thanh/video đến (tùy chọn) với các nhà cung cấp + CLI
  fallbacks
read_when:
  - Thiết kế hoặc tái cấu trúc khả năng hiểu nội dung đa phương tiện
  - Tinh chỉnh xử lý trước inbound audio/video/image
title: Hiểu Biết Về Phương Tiện Truyền Thông
x-i18n:
  source_path: nodes\media-understanding.md
  source_hash: 440ba1b35f398a5d3f9ce9656d344e2c74943604c7741bebbf8aa793307c1b15
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:11:13.585Z'
---

# Hiểu Nội Dung Phương Tiện (Đến) — 2026-01-17

OpenClaw có thể **tóm tắt phương tiện đến** (hình ảnh/âm thanh/video) trước khi quy trình trả lời chạy. Nó tự động phát hiện khi các công cụ cục bộ hoặc khóa nhà cung cấp có sẵn, và có thể bị tắt hoặc tùy chỉnh. Nếu tắt tính năng hiểu nội dung, các mô hình vẫn nhận được các tệp/URL gốc như bình thường.
## Mục tiêu

- Tùy chọn: tiền xử lý phương tiện đến thành văn bản ngắn để định tuyến nhanh hơn + phân tích lệnh tốt hơn.
- Bảo toàn việc cung cấp phương tiện gốc cho mô hình (luôn luôn).
- Hỗ trợ **API nhà cung cấp** và **CLI fallback**.
- Cho phép nhiều mô hình với fallback có thứ tự (lỗi/kích thước/timeout).
## Hành vi cấp cao

1. Thu thập các tệp đính kèm đến (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Đối với mỗi khả năng được bật (hình ảnh/âm thanh/video), chọn các tệp đính kèm theo chính sách (mặc định: **đầu tiên**).
3. Chọn mục mô hình đủ điều kiện đầu tiên (kích thước + khả năng + xác thực).
4. Nếu mô hình không thành công hoặc phương tiện quá lớn, **quay lại mục tiếp theo**.
5. Khi thành công:
   - `Body` trở thành khối `[Image]`, `[Audio]`, hoặc `[Video]`.
   - Âm thanh đặt `{{Transcript}}`; phân tích cú pháp lệnh sử dụng văn bản chú thích khi có, nếu không thì sử dụng bản ghi âm.
   - Chú thích được bảo toàn dưới dạng `User text:` bên trong khối.

Nếu hiểu không thành công hoặc bị tắt, **luồng trả lời tiếp tục** với nội dung gốc + các tệp đính kèm.
## Tổng quan cấu hình

`tools.media` hỗ trợ **mô hình được chia sẻ** cộng với ghi đè cho từng khả năng:

- `tools.media.models`: danh sách mô hình được chia sẻ (sử dụng `capabilities` để kiểm soát).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - mặc định (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - ghi đè nhà cung cấp (`baseUrl`, `headers`, `providerOptions`)
  - tùy chọn âm thanh Deepgram thông qua `tools.media.audio.providerOptions.deepgram`
  - danh sách **`models` cho từng khả năng tùy chọn** (ưu tiên trước mô hình được chia sẻ)
  - chính sách `attachments` (`mode`, `maxAttachments`, `prefer`)
  - `scope` (kiểm soát tùy chọn theo khóa kênh/chatType/phiên)
- `tools.media.concurrency`: số lần chạy khả năng tối đa đồng thời (mặc định **2**).

```json5
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### Model entries

Each `models[]` entry can be **provider** or **CLI**:

```json5
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```
CLI templates cũng có thể sử dụng:

- `{{MediaDir}}` (thư mục chứa tệp phương tiện)
- `{{OutputDir}}` (thư mục tạm được tạo cho lần chạy này)
- `{{OutputBase}}` (đường dẫn tệp tạm cơ sở, không có phần mở rộng)
## Mặc định và giới hạn

Các mặc định được khuyến nghị:

- `maxChars`: **500** cho hình ảnh/video (ngắn, thân thiện với lệnh)
- `maxChars`: **chưa đặt** cho âm thanh (toàn bộ bản ghi âm trừ khi bạn đặt giới hạn)
- `maxBytes`:
  - hình ảnh: **10MB**
  - âm thanh: **20MB**
  - video: **50MB**

Quy tắc:

- Nếu phương tiện vượt quá `maxBytes`, mô hình đó sẽ bị bỏ qua và **mô hình tiếp theo sẽ được thử**.
- Nếu mô hình trả về nhiều hơn `maxChars`, đầu ra sẽ bị cắt ngắn.
- `prompt` mặc định là "Describe the {media}." đơn giản cộng với hướng dẫn `maxChars` (chỉ hình ảnh/video).
- Nếu `<capability>.enabled: true` nhưng không có mô hình nào được cấu hình, OpenClaw sẽ thử
  **mô hình trả lời hoạt động** khi nhà cung cấp của nó hỗ trợ khả năng này.

### Tự động phát hiện hiểu biết phương tiện (mặc định)

Nếu `tools.media.<capability>.enabled` **không** được đặt thành `false` và bạn chưa
cấu hình mô hình, OpenClaw tự động phát hiện theo thứ tự này và **dừng lại ở tùy chọn đầu tiên hoạt động**:

1. **CLI cục bộ** (chỉ âm thanh; nếu được cài đặt)
   - `sherpa-onnx-offline` (yêu cầu `SHERPA_ONNX_MODEL_DIR` với bộ mã hóa/giải mã/bộ nối/token)
   - `whisper-cli` (`whisper-cpp`; sử dụng `WHISPER_CPP_MODEL` hoặc mô hình nhỏ được đóng gói)
   - `whisper` (CLI Python; tự động tải xuống mô hình)
2. **CLI Gemini** (`gemini`) sử dụng `read_many_files`
3. **Khóa nhà cung cấp**
   - Âm thanh: OpenAI → Groq → Deepgram → Google
   - Hình ảnh: OpenAI → Anthropic → Google → MiniMax
   - Video: Google

Để vô hiệu hóa tự động phát hiện, hãy đặt:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Note: Binary detection is best-effort across macOS/Linux/Windows; ensure the CLI is on `PATH` (we expand `~`), hoặc đặt một mô hình CLI rõ ràng với đường dẫn lệnh đầy đủ.
## Khả năng (tùy chọn)

Nếu bạn đặt `capabilities`, mục nhập chỉ chạy cho các loại phương tiện đó. Đối với danh sách được chia sẻ, OpenClaw có thể suy ra các giá trị mặc định:

- `openai`, `anthropic`, `minimax`: **hình ảnh**
- `google` (Gemini API): **hình ảnh + âm thanh + video**
- `groq`: **âm thanh**
- `deepgram`: **âm thanh**

Đối với các mục nhập CLI, **đặt `capabilities` một cách rõ ràng** để tránh các kết quả khớp bất ngờ.
Nếu bạn bỏ qua `capabilities`, mục nhập đủ điều kiện cho danh sách mà nó xuất hiện.
## Ma trận hỗ trợ nhà cung cấp (Tích hợp OpenClaw)

| Khả năng | Tích hợp nhà cung cấp                            | Ghi chú                                                   |
| ---------- | ------------------------------------------------ | --------------------------------------------------------- |
| Hình ảnh      | OpenAI / Anthropic / Google / những nhà cung cấp khác qua `pi-ai` | Bất kỳ mô hình nào có khả năng xử lý hình ảnh trong sổ đăng ký đều hoạt động.            |
| Âm thanh      | OpenAI, Groq, Deepgram, Google, Mistral          | Chuyển đổi giọng nói của nhà cung cấp (Whisper/Deepgram/Gemini/Voxtral). |
| Video      | Google (Gemini API)                              | Hiểu video của nhà cung cấp.                             |
## Các nhà cung cấp được khuyến nghị

**Hình ảnh**

- Ưu tiên mô hình hoạt động của bạn nếu nó hỗ trợ hình ảnh.
- Các giá trị mặc định tốt: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Âm thanh**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, `deepgram/nova-3`, hoặc `mistral/voxtral-mini-latest`.
- Dự phòng CLI: `whisper-cli` (whisper-cpp) hoặc `whisper`.
- Thiết lập Deepgram: [Deepgram (/providers/deepgram).

**Video**

- `google/gemini-3-flash-preview` (nhanh), `google/gemini-3-pro-preview` (phong phú hơn).
- Dự phòng CLI: `gemini` CLI (hỗ trợ `read_file` trên video/âm thanh).
## Chính sách tệp đính kèm

Mỗi khả năng `attachments` kiểm soát những tệp đính kèm nào được xử lý:

- `mode`: `first` (mặc định) hoặc `all`
- `maxAttachments`: giới hạn số lượng được xử lý (mặc định **1**)
- `prefer`: `first`, `last`, `path`, `url`

Khi `mode: "all"`, các đầu ra được gắn nhãn `[Image 1/2]`, `[Audio 2/2]`, v.v.
## Ví dụ cấu hình

### 1) Danh sách mô hình được chia sẻ + ghi đè

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) Audio + Video only (image off)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```
### 3) Hiểu hình ảnh tùy chọn

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) Multi‑modal single entry (explicit capabilities)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```
## Kết quả trạng thái

Khi hiểu thông tin phương tiện chạy, `/status` bao gồm một dòng tóm tắt ngắn:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Điều này hiển thị kết quả cho từng khả năng và nhà cung cấp/mô hình được chọn khi áp dụng.
## Ghi chú

- Hiểu biết là **nỗ lực tốt nhất**. Lỗi không chặn các phản hồi.
- Tệp đính kèm vẫn được chuyển đến các mô hình ngay cả khi tắt hiểu biết.
- Sử dụng `scope` để giới hạn nơi hiểu biết chạy (ví dụ: chỉ tin nhắn riêng).
## Tài liệu liên quan

- [Cấu hình](/gateway/configuration)
- [Hỗ trợ Hình ảnh & Phương tiện](/nodes/images)