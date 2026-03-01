---
summary: >-
  Plugin Cuộc gọi thoại: cuộc gọi đi + cuộc gọi đến qua Twilio/Telnyx/Plivo (cài
  đặt plugin + cấu hình + CLI)
read_when:
  - Bạn muốn thực hiện một cuộc gọi thoại đi từ OpenClaw
  - Bạn đang cấu hình hoặc phát triển plugin voice-call
title: Plugin Cuộc Gọi Thoại
x-i18n:
  source_path: plugins\voice-call.md
  source_hash: 5417fbfcdca2ed10a1e478715604cdc4e9fdace2fa0d50924e6a1da58d604444
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:03.822Z'
---

# Voice Call (plugin)

Cuộc gọi thoại cho OpenClaw thông qua một plugin. Hỗ trợ thông báo gửi đi và các cuộc trò chuyện nhiều lượt với chính sách nhận đến.

Các nhà cung cấp hiện tại:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (dev/no network)

Mô hình tư duy nhanh:

- Cài đặt plugin
- Khởi động lại Gateway
- Cấu hình dưới `plugins.entries.voice-call.config`
- Sử dụng `openclaw voicecall ...` hoặc công cụ `voice_call`
## Nơi nó chạy (cục bộ vs từ xa)

Plugin Voice Call chạy **bên trong quy trình Gateway**.

Nếu bạn sử dụng Gateway từ xa, hãy cài đặt/cấu hình plugin trên **máy chạy Gateway**, sau đó khởi động lại Gateway để tải nó.
## Cài đặt

### Tùy chọn A: cài đặt từ npm (được khuyến nghị)

```bash
openclaw plugins install @openclaw/voice-call
```

Restart the Gateway afterwards.

### Option B: install from a local folder (dev, no copying)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Khởi động lại Gateway sau đó.
## Cấu hình

Đặt cấu hình dưới `plugins.entries.voice-call.config`:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // or "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Telnyx webhook public key from the Telnyx Mission Control Portal
            // (Base64 string; can also be set via TELNYX_PUBLIC_KEY).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Webhook security (recommended for tunnels/proxies)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Public exposure (pick one)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```
Ghi chú:

- Twilio/Telnyx yêu cầu một **webhook URL có thể truy cập công khai**.
- Plivo yêu cầu một **webhook URL có thể truy cập công khai**.
- `mock` là nhà cung cấp dev cục bộ (không có lệnh gọi mạng).
- Telnyx yêu cầu `telnyx.publicKey` (hoặc `TELNYX_PUBLIC_KEY`) trừ khi `skipSignatureVerification` là true.
- `skipSignatureVerification` chỉ dành cho kiểm tra cục bộ.
- Nếu bạn sử dụng ngrok free tier, hãy đặt `publicUrl` thành URL ngrok chính xác; xác minh chữ ký luôn được thực thi.
- `tunnel.allowNgrokFreeTierLoopbackBypass: true` cho phép webhook Twilio có chữ ký không hợp lệ **chỉ khi** `tunnel.provider="ngrok"` và `serve.bind` là local loopback (ngrok local agent). Chỉ sử dụng cho dev cục bộ.
- URL ngrok free tier có thể thay đổi hoặc thêm hành vi interstitial; nếu `publicUrl` thay đổi, chữ ký Twilio sẽ thất bại. Đối với production, hãy ưu tiên một domain ổn định hoặc Tailscale funnel.
- Mặc định bảo mật truyền phát:
  - `streaming.preStartTimeoutMs` đóng các socket không bao giờ gửi một khung `start` hợp lệ.
  - `streaming.maxPendingConnections` giới hạn tổng số socket pre-start chưa được xác thực.
  - `streaming.maxPendingConnectionsPerIp` giới hạn socket pre-start chưa được xác thực trên mỗi IP nguồn.
  - `streaming.maxConnections` giới hạn tổng số socket media stream mở (pending + active).
## Stale call reaper

Sử dụng `staleCallReaperSeconds` để kết thúc các cuộc gọi không bao giờ nhận được webhook cuối cùng
(ví dụ: các cuộc gọi ở chế độ thông báo không bao giờ hoàn thành). Mặc định là `0`
(tắt).

Các phạm vi được khuyến nghị:

- **Production:** `120`–`300` giây cho các luồng kiểu thông báo.
- Giữ giá trị này **cao hơn `maxDurationSeconds`** để các cuộc gọi bình thường có thể
  hoàn thành. Một điểm bắt đầu tốt là `maxDurationSeconds + 30–60` giây.

Ví dụ:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```
## Bảo mật Webhook

Khi một proxy hoặc tunnel nằm phía trước Gateway, plugin sẽ tái tạo URL công khai để xác minh chữ ký. Các tùy chọn này kiểm soát các tiêu đề được chuyển tiếp nào được tin tưởng.

`webhookSecurity.allowedHosts` danh sách cho phép các máy chủ từ các tiêu đề được chuyển tiếp.

`webhookSecurity.trustForwardingHeaders` tin tưởng các tiêu đề được chuyển tiếp mà không cần danh sách cho phép.

`webhookSecurity.trustedProxyIPs` chỉ tin tưởng các tiêu đề được chuyển tiếp khi IP từ xa của yêu cầu khớp với danh sách.

Bảo vệ phát lại webhook được bật cho Twilio và Plivo. Các yêu cầu webhook hợp lệ được phát lại sẽ được xác nhận nhưng bỏ qua các tác dụng phụ.

Các lượt hội thoại Twilio bao gồm một token cho mỗi lượt trong các callback `<Gather>`, vì vậy các callback lời nói cũ/được phát lại không thể thỏa mãn một lượt phiên bản trang chủ đang chờ xử lý mới hơn.

Ví dụ với một máy chủ công khai ổn định:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```
## TTS cho cuộc gọi

Voice Call sử dụng cấu hình `messages.tts` cốt lõi (OpenAI hoặc ElevenLabs) để
truyền phát phát biểu trên cuộc gọi. Bạn có thể ghi đè nó trong cấu hình plugin với
**hình dạng giống nhau** — nó hợp nhất sâu với `messages.tts`.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

Notes:

- **Edge TTS is ignored for voice calls** (telephony audio needs PCM; Edge output is unreliable).
- Core TTS is used when Twilio media streaming is enabled; otherwise calls fall back to provider native voices.

### More examples

Use core TTS only (no override):

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

Override to ElevenLabs just for calls (keep core default elsewhere):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```
Ghi đè chỉ mô hình OpenAI cho các cuộc gọi (ví dụ về deep‑merge):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```
## Cuộc gọi đến

Chính sách đến mặc định là `disabled`. Để bật cuộc gọi đến, hãy đặt:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?",
}
```

Auto-responses use the agent system. Tune with:

- `responseModel`
- `responseSystemPrompt`
- `responseTimeoutMs`
## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```
## Agent tool

Tool name: `voice_call`

Actions:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

This repo ships a matching skill doc at `skills/voice-call/SKILL.md`.
## Gateway RPC

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)