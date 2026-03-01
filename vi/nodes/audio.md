---
summary: >-
  Cách tải xuống, chuyển đổi thành văn bản và chèn âm thanh/ghi chú thoại đến
  vào các câu trả lời
read_when:
  - Thay đổi chuyển đổi âm thanh thành văn bản hoặc xử lý phương tiện
title: Âm thanh và Ghi chú Thoại
x-i18n:
  source_path: nodes\audio.md
  source_hash: c5c7e0edf6619192bb1fd864943a31ff6a942db197d0974b0fbbd9c054b9d9da
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:15.107Z'
---

# Audio / Voice Notes — 2026-01-17

## What works

- **Media understanding (audio)**: Nếu hiểu biết về âm thanh được bật (hoặc tự động phát hiện), OpenClaw:
  1. Định vị tệp đính kèm âm thanh đầu tiên (đường dẫn cục bộ hoặc URL) và tải xuống nếu cần.
  2. Thực thi `maxBytes` trước khi gửi đến từng mục mô hình.
  3. Chạy mục mô hình đủ điều kiện đầu tiên theo thứ tự (nhà cung cấp hoặc CLI).
  4. Nếu nó thất bại hoặc bỏ qua (kích thước/hết thời gian), nó sẽ thử mục tiếp theo.
  5. Khi thành công, nó thay thế `Body` bằng khối `[Audio]` và đặt `{{Transcript}}`.
- **Command parsing**: Khi chuyển đổi giọng nói thành văn bản thành công, `CommandBody`/`RawBody` được đặt thành bản ghi âm để các lệnh gạch chéo vẫn hoạt động.
- **Verbose logging**: Trong `--verbose`, chúng tôi ghi nhật ký khi chuyển đổi giọng nói thành văn bản chạy và khi nó thay thế nội dung.
## Tự động phát hiện (mặc định)

Nếu bạn **không cấu hình mô hình** và `tools.media.audio.enabled` **không** được đặt thành `false`,
OpenClaw tự động phát hiện theo thứ tự này và dừng lại ở tùy chọn đầu tiên hoạt động:

1. **CLI cục bộ** (nếu được cài đặt)
   - `sherpa-onnx-offline` (yêu cầu `SHERPA_ONNX_MODEL_DIR` với encoder/decoder/joiner/tokens)
   - `whisper-cli` (từ `whisper-cpp`; sử dụng `WHISPER_CPP_MODEL` hoặc mô hình nhỏ được đóng gói)
   - `whisper` (CLI Python; tải xuống mô hình tự động)
2. **Gemini CLI** (`gemini`) sử dụng `read_many_files`
3. **Khóa nhà cung cấp** (OpenAI → Groq → Deepgram → Google)

Để vô hiệu hóa tự động phát hiện, đặt `tools.media.audio.enabled: false`.
Để tùy chỉnh, đặt `tools.media.audio.models`.
Lưu ý: Phát hiện nhị phân là nỗ lực tốt nhất trên macOS/Linux/Windows; đảm bảo CLI nằm trên `PATH` (chúng tôi mở rộng `~`), hoặc đặt một mô hình CLI rõ ràng với đường dẫn lệnh đầy đủ.
## Ví dụ cấu hình

### Nhà cung cấp + CLI dự phòng (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Provider-only with scope gating

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Provider-only (Deepgram)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```
### Nhà cung cấp duy nhất (Mistral Voxtral)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```
## Ghi chú & giới hạn

- Xác thực nhà cung cấp tuân theo thứ tự xác thực mô hình tiêu chuẩn (hồ sơ xác thực, biến môi trường, `models.providers.*.apiKey`).
- Deepgram nhận `DEEPGRAM_API_KEY` khi `provider: "deepgram"` được sử dụng.
- Chi tiết thiết lập Deepgram: [Deepgram (/providers/deepgram).
- Chi tiết thiết lập Mistral: [Mistral](/providers/mistral).
- Các nhà cung cấp âm thanh có thể ghi đè `baseUrl`, `headers`, và `providerOptions` thông qua `tools.media.audio`.
- Giới hạn kích thước mặc định là 20MB (`tools.media.audio.maxBytes`). Âm thanh vượt quá kích thước sẽ bị bỏ qua cho mô hình đó và mục nhập tiếp theo sẽ được thử.
- `maxChars` mặc định cho âm thanh là **chưa được đặt** (bản ghi đầy đủ). Đặt `tools.media.audio.maxChars` hoặc `maxChars` cho từng mục nhập để cắt ngắn đầu ra.
- Mặc định tự động OpenAI là `gpt-4o-mini-transcribe`; đặt `model: "gpt-4o-transcribe"` để có độ chính xác cao hơn.
- Sử dụng `tools.media.audio.attachments` để xử lý nhiều ghi chú thoại (`mode: "all"` + `maxAttachments`).
- Bản ghi đã có sẵn cho các mẫu dưới dạng `{{Transcript}}`.
- Đầu ra CLI stdout bị giới hạn (5MB); giữ đầu ra CLI ngắn gọn.
## Phát hiện Mention trong Nhóm

Khi `requireMention: true` được đặt cho một cuộc trò chuyện nhóm, OpenClaw giờ đây chuyển đổi âm thanh **trước khi** kiểm tra mention. Điều này cho phép các ghi chú thoại được xử lý ngay cả khi chúng chứa mention.

**Cách hoạt động:**

1. Nếu một tin nhắn thoại không có nội dung văn bản và nhóm yêu cầu mention, OpenClaw thực hiện chuyển đổi "preflight".
2. Bản chuyển đổi được kiểm tra để tìm các mẫu mention (ví dụ: `@BotName`, kích hoạt emoji).
3. Nếu tìm thấy mention, tin nhắn sẽ tiếp tục qua toàn bộ quy trình trả lời.
4. Bản chuyển đổi được sử dụng để phát hiện mention để các ghi chú thoại có thể vượt qua cổng mention.

**Hành vi dự phòng:**

- Nếu chuyển đổi không thành công trong quá trình preflight (timeout, lỗi API, v.v.), tin nhắn được xử lý dựa trên phát hiện mention chỉ dựa trên văn bản.
- Điều này đảm bảo rằng các tin nhắn hỗn hợp (văn bản + âm thanh) không bao giờ bị loại bỏ không chính xác.

**Ví dụ:** Một người dùng gửi một ghi chú thoại nói "Hey @Claude, what's the weather?" trong một nhóm Telegram với `requireMention: true`. Ghi chú thoại được chuyển đổi, mention được phát hiện, và agent trả lời.
## Những điều cần lưu ý

- Quy tắc phạm vi sử dụng nguyên tắc khớp đầu tiên. `chatType` được chuẩn hóa thành `direct`, `group`, hoặc `room`.
- Đảm bảo CLI của bạn thoát với mã 0 và in văn bản thuần túy; JSON cần được xử lý qua `jq -r .text`.
- Giữ cho thời gian chờ hợp lý (`timeoutSeconds`, mặc định 60 giây) để tránh chặn hàng đợi trả lời.
- Phiên bản sơ bộ chỉ xử lý **tệp đính kèm âm thanh đầu tiên** để phát hiện đề cập. Âm thanh bổ sung được xử lý trong giai đoạn hiểu phương tiện chính.