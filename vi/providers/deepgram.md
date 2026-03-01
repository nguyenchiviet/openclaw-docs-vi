---
summary: Phiên âm Deepgram cho các ghi chú thoại đến
read_when:
  - Bạn muốn Deepgram speech-to-text cho các tệp đính kèm âm thanh
  - Bạn cần một ví dụ cấu hình Deepgram nhanh chóng
title: Deepgram
x-i18n:
  source_path: providers\deepgram.md
  source_hash: dabd1f6942c339fbd744fbf38040b6a663b06ddf4d9c9ee31e3ac034de9e79d9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:42.498Z'
---

# Deepgram (Chuyển đổi âm thanh thành văn bản)

Deepgram là một API chuyển đổi giọng nói thành văn bản. Trong OpenClaw, nó được sử dụng cho **chuyển đổi âm thanh/ghi chú giọng nói đến** thông qua `tools.media.audio`.

Khi được bật, OpenClaw tải tệp âm thanh lên Deepgram và chèn bản ghi chép vào quy trình trả lời (`{{Transcript}}` + `[Audio]` block). Đây **không phải là truyền phát**; nó sử dụng điểm cuối chuyển đổi được ghi âm trước.

Website: [https://deepgram.com](https://deepgram.com)  
Docs: [https://developers.deepgram.com](https://developers.deepgram.com)
## Bắt đầu nhanh

1. Đặt khóa API của bạn:

```
DEEPGRAM_API_KEY=dg_...
```

2. Enable the provider:

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
## Tùy chọn

- `model`: ID mô hình Deepgram (mặc định: `nova-3`)
- `language`: gợi ý ngôn ngữ (tùy chọn)
- `tools.media.audio.providerOptions.deepgram.detect_language`: bật phát hiện ngôn ngữ (tùy chọn)
- `tools.media.audio.providerOptions.deepgram.punctuate`: bật dấu câu (tùy chọn)
- `tools.media.audio.providerOptions.deepgram.smart_format`: bật định dạng thông minh (tùy chọn)

Ví dụ với ngôn ngữ:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

Example with Deepgram options:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```
## Ghi chú

- Xác thực tuân theo thứ tự xác thực nhà cung cấp tiêu chuẩn; `DEEPGRAM_API_KEY` là đường dẫn đơn giản nhất.
- Ghi đè các điểm cuối hoặc tiêu đề bằng `tools.media.audio.baseUrl` và `tools.media.audio.headers` khi sử dụng proxy.
- Đầu ra tuân theo các quy tắc âm thanh giống như các nhà cung cấp khác (giới hạn kích thước, hết thời gian chờ, tiêm bản ghi).