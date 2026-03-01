---
summary: Các tác vụ LLM chỉ dùng JSON cho workflows (công cụ plugin tùy chọn)
read_when:
  - Bạn muốn một bước LLM chỉ JSON bên trong các quy trình làm việc
  - Bạn cần đầu ra LLM được xác thực theo schema để tự động hóa
title: Nhiệm vụ LLM
x-i18n:
  source_path: tools\llm-task.md
  source_hash: b7aa78f179cb0f6361084bf6d0b895856f116d7077669c5ef995b92959211001
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:30:36.101Z'
---

# Tác vụ LLM

`llm-task` là một **công cụ plugin tùy chọn** chạy tác vụ LLM chỉ JSON và
trả về kết quả có cấu trúc (tùy chọn được xác thực dựa trên JSON Schema).

Điều này lý tưởng cho các công cụ quy trình làm việc như Lobster: bạn có thể thêm một bước LLM duy nhất
mà không cần viết mã OpenClaw tùy chỉnh cho mỗi quy trình làm việc.
## Bật plugin

1. Bật plugin:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Allowlist the tool (it is registered with `optional: true`):

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```
## Cấu hình (tùy chọn)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.3-codex"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` is an allowlist of `provider/model` chuỗi. Nếu được đặt, bất kỳ yêu cầu nào ngoài danh sách sẽ bị từ chối.
## Tham số công cụ

- `prompt` (string, required)
- `input` (any, optional)
- `schema` (object, optional JSON Schema)
- `provider` (string, optional)
- `model` (string, optional)
- `authProfileId` (string, optional)
- `temperature` (number, optional)
- `maxTokens` (number, optional)
- `timeoutMs` (number, optional)
## Kết quả

Trả về `details.json` chứa JSON được phân tích cú pháp (và xác thực so với
`schema` khi được cung cấp).
## Ví dụ: Bước quy trình Lobster

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```
## Ghi chú về bảo mật

- Công cụ này **chỉ hỗ trợ JSON** và hướng dẫn mô hình chỉ xuất JSON (không có code fences, không có bình luận).
- Không có công cụ nào được tiếp xúc với mô hình trong lần chạy này.
- Coi đầu ra là không đáng tin cậy trừ khi bạn xác thực bằng `schema`.
- Đặt phê duyệt trước bất kỳ bước nào có tác dụng phụ (gửi, đăng, thực thi).