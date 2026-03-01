---
summary: 'Chế độ Talk: các cuộc trò chuyện nói liên tục với ElevenLabs TTS'
read_when:
  - Triển khai chế độ Talk trên macOS/iOS/Android
  - Thay đổi hành vi giọng nói/TTS/ngắt
title: Chế độ Nói chuyện
x-i18n:
  source_path: nodes\talk.md
  source_hash: ecbc3701c9e9502970cf13227fedbc9714d13668d8f4f3988fef2a4d68116a42
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:48.235Z'
---

# Chế độ Nói chuyện

Chế độ nói chuyện là một vòng lặp hội thoại giọng nói liên tục:

1. Lắng nghe giọng nói
2. Gửi bản ghi âm đến mô hình (phiên chính, chat.send)
3. Chờ phản hồi
4. Phát nó qua ElevenLabs (phát lại truyền phát)
## Hành vi (macOS)

- **Lớp phủ luôn bật** khi chế độ Talk được bật.
- **Nghe → Suy nghĩ → Nói** chuyển đổi giai đoạn.
- Khi có **tạm dừng ngắn** (cửa sổ im lặng), bản ghi hiện tại sẽ được gửi.
- Các câu trả lời được **ghi vào WebChat** (giống như gõ).
- **Ngắt khi có tiếng nói** (bật theo mặc định): nếu người dùng bắt đầu nói trong khi trợ lý đang nói, chúng tôi sẽ dừng phát lại và ghi lại dấu thời gian ngắt cho lời nhắc tiếp theo.
## Chỉ thị giọng nói trong các câu trả lời

Trợ lý có thể đặt tiền tố cho câu trả lời của nó bằng **một dòng JSON duy nhất** để kiểm soát giọng nói:

```json
{ "voice": "<voice-id>", "once": true }
```

Rules:

- First non-empty line only.
- Unknown keys are ignored.
- `once: true` applies to the current reply only.
- Without `once`, the voice becomes the new default for Talk mode.
- The JSON line is stripped before TTS playback.

Supported keys:

- `voice` / `voice_id` / `voiceId`
- `model` / `model_id` / `modelId`
- `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
- `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
- `once`
## Cấu hình (`~/.openclaw/openclaw.json`)

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

Defaults:

- `interruptOnSpeech`: true
- `voiceId`: falls back to `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` (or first ElevenLabs voice when API key is available)
- `modelId`: defaults to `eleven_v3` when unset
- `apiKey`: falls back to `ELEVENLABS_API_KEY` (or gateway shell profile if available)
- `outputFormat`: defaults to `pcm_44100` on macOS/iOS and `pcm_24000` on Android (set `mp3_*` để buộc truyền phát MP3)
## macOS UI

- Menu bar toggle: **Talk**
- Config tab: **Talk Mode** group (voice id + interrupt toggle)
- Overlay:
  - **Listening**: cloud pulses with mic level
  - **Thinking**: sinking animation
  - **Speaking**: radiating rings
  - Click cloud: stop speaking
  - Click X: exit Talk mode
## Ghi chú

- Yêu cầu quyền Lời nói + Microphone.
- Sử dụng `chat.send` với khóa phiên `main`.
- TTS sử dụng API truyền phát ElevenLabs với `ELEVENLABS_API_KEY` và phát lại tăng dần trên macOS/iOS/Android để giảm độ trễ.
- `stability` cho `eleven_v3` được xác thực thành `0.0`, `0.5`, hoặc `1.0`; các mô hình khác chấp nhận `0..1`.
- `latency_tier` được xác thực thành `0..4` khi được đặt.
- Android hỗ trợ `pcm_16000`, `pcm_22050`, `pcm_24000`, và `pcm_44100` định dạng đầu ra cho truyền phát AudioTrack độ trễ thấp.