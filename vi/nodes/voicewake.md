---
summary: >-
  Từ khóa đánh thức giọng nói toàn cầu (do Gateway sở hữu) và cách chúng đồng bộ
  hóa trên các nút
read_when:
  - Thay đổi hành vi hoặc cài đặt mặc định của từ khóa唤醒giọng nói
  - Thêm các nền tảng nút mới cần đồng bộ hóa từ khóa đánh thức
title: Kích hoạt bằng giọng nói
x-i18n:
  source_path: nodes\voicewake.md
  source_hash: eb34f52dfcdc3fc1ae088ae1f621f245546d3cf388299fbeea62face61788c37
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:11:24.863Z'
---

# Voice Wake (Từ khóa đánh thức toàn cục)

OpenClaw coi **từ khóa đánh thức là một danh sách toàn cục duy nhất** do **Gateway** sở hữu.

- **Không có từ khóa đánh thức tùy chỉnh cho từng node**.
- **Bất kỳ node/app UI nào cũng có thể chỉnh sửa** danh sách; các thay đổi được Gateway lưu trữ và phát sóng cho mọi người.
- Mỗi thiết bị vẫn giữ **công tắc bật/tắt Voice Wake** riêng của nó (UX cục bộ + quyền khác nhau).

## Lưu trữ (máy chủ Gateway)

Từ khóa đánh thức được lưu trữ trên máy gateway tại:

- `~/.openclaw/settings/voicewake.json`

Hình dạng:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## Protocol

### Methods

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` with params `{ triggers: string[] }` → `{ triggers: string[] }`

Notes:

- Triggers are normalized (trimmed, empties dropped). Empty lists fall back to defaults.
- Limits are enforced for safety (count/length caps).

### Events

- `voicewake.changed` payload `{ triggers: string[] }`

Who receives it:

- All WebSocket clients (macOS app, WebChat, etc.)
- All connected nodes (iOS/Android), and also on node connect as an initial “current state” push.

## Client behavior

### macOS app

- Uses the global list to gate `VoiceWakeRuntime` triggers.
- Editing “Trigger words” in Voice Wake settings calls `voicewake.set` and then relies on the broadcast to keep other clients in sync.

### iOS node

- Uses the global list for `VoiceWakeManager` trigger detection.
- Editing Wake Words in Settings calls `voicewake.set` (over the Gateway WS) and also keeps local wake-word detection responsive.

### Android node

- Exposes a Wake Words editor in Settings.
- Calls `voicewake.set` qua Gateway WS để các chỉnh sửa đồng bộ hóa ở mọi nơi.