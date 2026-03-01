---
summary: Chuyển đổi văn bản thành giọng nói (TTS) cho các phản hồi gửi đi
read_when:
  - Bật tính năng chuyển đổi văn bản thành giọng nói cho các câu trả lời
  - Cấu hình các nhà cung cấp TTS hoặc giới hạn
  - Sử dụng các lệnh /tts
title: Chuyển đổi Văn bản thành Giọng nói
x-i18n:
  source_path: tts.md
  source_hash: 187466289606535eec153b15c733959e27a1f47be0f22d471afa405cd8e2c177
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:34:31.235Z'
---

# Chuyển đổi văn bản thành giọng nói (TTS)

OpenClaw có thể chuyển đổi các câu trả lời gửi đi thành âm thanh bằng cách sử dụng ElevenLabs, OpenAI hoặc Edge TTS.
Nó hoạt động ở bất kỳ nơi nào OpenClaw có thể gửi âm thanh; Telegram nhận được một bong bóng ghi âm tròn.
## Các dịch vụ được hỗ trợ

- **ElevenLabs** (nhà cung cấp chính hoặc dự phòng)
- **OpenAI** (nhà cung cấp chính hoặc dự phòng; cũng được sử dụng cho các bản tóm tắt)
- **Edge TTS** (nhà cung cấp chính hoặc dự phòng; sử dụng `node-edge-tts`, mặc định khi không có khóa API)

### Ghi chú về Edge TTS

Edge TTS sử dụng dịch vụ TTS neural trực tuyến của Microsoft Edge thông qua thư viện `node-edge-tts`. Đây là một dịch vụ được lưu trữ (không phải cục bộ), sử dụng các điểm cuối của Microsoft, và không yêu cầu khóa API. `node-edge-tts` cung cấp các tùy chọn cấu hình lời nói và định dạng đầu ra, nhưng không phải tất cả các tùy chọn đều được dịch vụ Edge hỗ trợ. citeturn2search0

Vì Edge TTS là một dịch vụ web công cộng không có SLA hoặc hạn ngạch được công bố, hãy coi nó là best-effort. Nếu bạn cần các giới hạn được đảm bảo và hỗ trợ, hãy sử dụng OpenAI hoặc ElevenLabs. API REST Speech của Microsoft ghi lại giới hạn âm thanh 10 phút trên mỗi yêu cầu; Edge TTS không công bố giới hạn, vì vậy hãy giả định các giới hạn tương tự hoặc thấp hơn. citeturn0search3
## Các khóa tùy chọn

Nếu bạn muốn sử dụng OpenAI hoặc ElevenLabs:

- `ELEVENLABS_API_KEY` (hoặc `XI_API_KEY`)
- `OPENAI_API_KEY`

Edge TTS **không** yêu cầu khóa API. Nếu không tìm thấy khóa API nào, OpenClaw sẽ mặc định
sử dụng Edge TTS (trừ khi bị vô hiệu hóa qua `messages.tts.edge.enabled=false`).

Nếu cấu hình nhiều nhà cung cấp, nhà cung cấp được chọn sẽ được sử dụng trước tiên và những nhà cung cấp khác là các tùy chọn dự phòng.
Tóm tắt tự động sử dụng `summaryModel` được cấu hình (hoặc `agents.defaults.model.primary`),
vì vậy nhà cung cấp đó cũng phải được xác thực nếu bạn bật tính năng tóm tắt.
## Liên kết dịch vụ

- [Hướng dẫn OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
- [Tài liệu tham khảo OpenAI Audio API](https://platform.openai.com/docs/api-reference/audio)
- [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
- [Xác thực ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
- [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
- [Định dạng đầu ra Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)
## Nó có được bật theo mặc định không?

Không. Auto‑TTS **tắt** theo mặc định. Bật nó trong cấu hình với
`messages.tts.auto` hoặc cho mỗi phiên với `/tts always` (bí danh: `/tts on`).

Edge TTS **được** bật theo mặc định khi TTS bật, và được sử dụng tự động
khi không có khóa API OpenAI hoặc ElevenLabs.
## Cấu hình

Cấu hình TTS nằm dưới `messages.tts` trong `openclaw.json`.
Lược đồ đầy đủ nằm trong [Cấu hình Gateway](/gateway/configuration).

### Cấu hình tối thiểu (bật + nhà cung cấp)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI primary with ElevenLabs fallback

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTS primary (no API key)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```
### Tắt Edge TTS

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### Custom limits + prefs path

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### Only reply with audio after an inbound voice note

```json5
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### Disable auto-summary for long replies

```json5
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

Then run:

```
/tts summary off
```
### Ghi chú về các trường

- `auto`: chế độ TTS tự động (`off`, `always`, `inbound`, `tagged`).
  - `inbound` chỉ gửi âm thanh sau khi nhận được ghi chú thoại đến.
  - `tagged` chỉ gửi âm thanh khi phản hồi bao gồm các thẻ `[[tts]]`.
- `enabled`: công tắc kế thừa (doctor chuyển đổi điều này thành `auto`).
- `mode`: `"final"` (mặc định) hoặc `"all"` (bao gồm phản hồi công cụ/khối).
- `provider`: `"elevenlabs"`, `"openai"`, hoặc `"edge"` (dự phòng là tự động).
- Nếu `provider` **chưa được đặt**, OpenClaw ưu tiên `openai` (nếu có khóa), sau đó `elevenlabs` (nếu có khóa),
  nếu không thì `edge`.
- `summaryModel`: mô hình rẻ tiền tùy chọn để tóm tắt tự động; mặc định là `agents.defaults.model.primary`.
  - Chấp nhận `provider/model` hoặc bí danh mô hình được cấu hình.
- `modelOverrides`: cho phép mô hình phát hành các chỉ thị TTS (bật theo mặc định).
  - `allowProvider` mặc định là `false` (chuyển đổi nhà cung cấp là tùy chọn).
- `maxTextLength`: giới hạn cứng cho đầu vào TTS (ký tự). `/tts audio` không thành công nếu vượt quá.
- `timeoutMs`: thời gian chờ yêu cầu (ms).
- `prefsPath`: ghi đè đường dẫn JSON tùy chọn cục bộ (nhà cung cấp/giới hạn/tóm tắt).
- Các giá trị `apiKey` quay lại các biến môi trường (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
- `elevenlabs.baseUrl`: ghi đè URL cơ sở API ElevenLabs.
- `elevenlabs.voiceSettings`:
  - `stability`, `similarityBoost`, `style`: `0..1`
  - `useSpeakerBoost`: `true|false`
  - `speed`: `0.5..2.0` (1.0 = bình thường)
- `elevenlabs.applyTextNormalization`: `auto|on|off`
- `elevenlabs.languageCode`: ISO 639-1 2 chữ cái (ví dụ: `en`, `de`)
- `elevenlabs.seed`: số nguyên `0..4294967295` (xác định tốt nhất)
- `edge.enabled`: cho phép sử dụng Edge TTS (mặc định `true`; không có khóa API).
- `edge.voice`: tên giọng nói neural Edge (ví dụ: `en-US-MichelleNeural`).
- `edge.lang`: mã ngôn ngữ (ví dụ: `en-US`).
- `edge.outputFormat`: định dạng đầu ra Edge (ví dụ: `audio-24khz-48kbitrate-mono-mp3`).
  - Xem định dạng đầu ra Microsoft Speech để biết các giá trị hợp lệ; không phải tất cả các định dạng đều được Edge hỗ trợ.
- `edge.rate` / `edge.pitch` / `edge.volume`: chuỗi phần trăm (ví dụ: `+10%`, `-5%`).
- `edge.saveSubtitles`: ghi phụ đề JSON cùng với tệp âm thanh.
- `edge.proxy`: URL proxy cho các yêu cầu Edge TTS.
- `edge.timeoutMs`: ghi đè thời gian chờ yêu cầu (ms).
## Ghi đè do mô hình điều khiển (bật theo mặc định)

Theo mặc định, mô hình **có thể** phát hành các chỉ thị TTS cho một câu trả lời duy nhất.
Khi `messages.tts.auto` là `tagged`, các chỉ thị này được yêu cầu để kích hoạt âm thanh.

Khi được bật, mô hình có thể phát hành các chỉ thị `[[tts:...]]` để ghi đè giọng nói
cho một câu trả lời duy nhất, cộng với một khối `[[tts:text]]...[[/tts:text]]` tùy chọn để
cung cấp các thẻ biểu cảm (tiếng cười, gợi ý hát, v.v.) chỉ nên xuất hiện trong
âm thanh.

Các chỉ thị `provider=...` bị bỏ qua trừ khi `modelOverrides.allowProvider: true`.

Ví dụ tải trọng trả lời:

```
Here you go.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

Available directive keys (when enabled):

- `provider` (`openai` | `elevenlabs` | `edge`, requires `allowProvider: true`)
- `voice` (OpenAI voice) or `voiceId` (ElevenLabs)
- `model` (OpenAI TTS model or ElevenLabs model id)
- `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
- `applyTextNormalization` (`auto|on|off`)
- `languageCode` (ISO 639-1)
- `seed`

Disable all model overrides:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

Optional allowlist (enable provider switching while keeping other knobs configurable):

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: true,
        allowSeed: false,
      },
    },
  },
}
```
## Tùy chọn cho từng người dùng

Các lệnh slash ghi các ghi đè cục bộ vào `prefsPath` (mặc định:
`~/.openclaw/settings/tts.json`, ghi đè bằng `OPENCLAW_TTS_PREFS` hoặc
`messages.tts.prefsPath`).

Các trường được lưu trữ:

- `enabled`
- `provider`
- `maxLength` (ngưỡng tóm tắt; mặc định 1500 ký tự)
- `summarize` (mặc định `true`)

Những cái này ghi đè `messages.tts.*` cho máy chủ đó.
## Định dạng đầu ra (cố định)

- **Telegram**: Ghi âm thoại Opus (`opus_48000_64` từ ElevenLabs, `opus` từ OpenAI).
  - 48kHz / 64kbps là sự cân bằng tốt cho ghi âm thoại và bắt buộc cho bong bóng tròn.
- **Các kênh khác**: MP3 (`mp3_44100_128` từ ElevenLabs, `mp3` từ OpenAI).
  - 44.1kHz / 128kbps là cân bằng mặc định cho độ rõ ràng của lời nói.
- **Edge TTS**: sử dụng `edge.outputFormat` (mặc định `audio-24khz-48kbitrate-mono-mp3`).
  - `node-edge-tts` chấp nhận một `outputFormat`, nhưng không phải tất cả các định dạng đều có sẵn
    từ dịch vụ Edge. citeturn2search0
  - Các giá trị định dạng đầu ra tuân theo định dạng đầu ra Lời nói của Microsoft (bao gồm Ogg/WebM Opus). citeturn1search0
  - Telegram `sendVoice` chấp nhận OGG/MP3/M4A; sử dụng OpenAI/ElevenLabs nếu bạn cần
    ghi âm thoại Opus được đảm bảo. citeturn1search1
  - Nếu định dạng đầu ra Edge được cấu hình không thành công, OpenClaw sẽ thử lại với MP3.

Các định dạng OpenAI/ElevenLabs được cố định; Telegram mong đợi Opus cho trải nghiệm ghi âm thoại.
## Hành vi Tự động TTS

Khi được bật, OpenClaw:

- bỏ qua TTS nếu câu trả lời đã chứa media hoặc chỉ thị `MEDIA:`.
- bỏ qua các câu trả lời rất ngắn (< 10 ký tự).
- tóm tắt các câu trả lời dài khi được bật bằng cách sử dụng `agents.defaults.model.primary` (hoặc `summaryModel`).
- đính kèm âm thanh được tạo vào câu trả lời.

Nếu câu trả lời vượt quá `maxLength` và tính năng tóm tắt bị tắt (hoặc không có khóa API cho mô hình tóm tắt), âm thanh sẽ bị bỏ qua và câu trả lời văn bản bình thường sẽ được gửi.
## Sơ đồ luồng

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```
## Sử dụng lệnh Slash

Có một lệnh duy nhất: `/tts`.
Xem [Lệnh Slash](/tools/slash-commands) để biết chi tiết về cách bật.

Lưu ý Discord: `/tts` là lệnh tích hợp của Discord, vì vậy OpenClaw đăng ký
`/voice` làm lệnh gốc ở đó. Văn bản `/tts ...` vẫn hoạt động.

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Notes:

- Commands require an authorized sender (allowlist/owner rules still apply).
- `commands.text` or native command registration must be enabled.
- `off|always|inbound|tagged` are per‑session toggles (`/tts on` is an alias for `/tts always`).
- `limit` and `summary` are stored in local prefs, not the main config.
- `/tts audio` tạo ra một phản hồi âm thanh một lần (không bật TTS).
## Công cụ Agent

Công cụ `tts` chuyển đổi văn bản thành giọng nói và trả về đường dẫn `MEDIA:`. Khi kết quả tương thích với Telegram, công cụ bao gồm `[[audio_as_voice]]` để Telegram gửi một bong bóng giọng nói.
## Gateway RPC

Các phương thức Gateway:

- `tts.status`
- `tts.enable`
- `tts.disable`
- `tts.convert`
- `tts.setProvider`
- `tts.providers`