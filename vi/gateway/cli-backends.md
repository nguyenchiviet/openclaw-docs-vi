---
summary: 'CLI backends: fallback chỉ văn bản thông qua các CLI AI cục bộ'
read_when:
  - >-
    Bạn muốn có một giải pháp dự phòng đáng tin cậy khi các nhà cung cấp API gặp
    sự cố
  - >-
    Bạn đang chạy Claude Code CLI hoặc các CLI AI cục bộ khác và muốn tái sử
    dụng chúng
  - >-
    Bạn cần một đường dẫn chỉ văn bản, không có công cụ nhưng vẫn hỗ trợ phiên
    làm việc và hình ảnh
title: CLI Backends
x-i18n:
  source_path: gateway\cli-backends.md
  source_hash: 8285f4829900bc810b1567264375fa92f3e25aebaee1bddaea4625a51a4e53d7
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:56:15.361Z'
---

# CLI backends (fallback runtime)

OpenClaw có thể chạy **các CLI AI cục bộ** như một **fallback chỉ văn bản** khi các nhà cung cấp API bị sự cố,
bị giới hạn tốc độ, hoặc tạm thời hoạt động không bình thường. Điều này được thiết kế một cách cẩn thận:

- **Các công cụ bị vô hiệu hóa** (không có lệnh gọi công cụ).
- **Văn bản vào → văn bản ra** (đáng tin cậy).
- **Các phiên được hỗ trợ** (vì vậy các lượt tiếp theo vẫn mạch lạc).
- **Hình ảnh có thể được chuyển qua** nếu CLI chấp nhận đường dẫn hình ảnh.

Điều này được thiết kế như một **lưới an toàn** chứ không phải là con đường chính. Sử dụng nó khi bạn
muốn các phản hồi văn bản "luôn hoạt động" mà không phụ thuộc vào các API bên ngoài.
## Bắt đầu nhanh thân thiện với người mới bắt đầu

Bạn có thể sử dụng Claude Code CLI **mà không cần bất kỳ cấu hình nào** (OpenClaw đi kèm với một cấu hình mặc định tích hợp):

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

Codex CLI also works out of the box:

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.3-codex
```

If your gateway runs under launchd/systemd and PATH is minimal, add just the
command path:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

Đó là tất cả. Không cần khóa, không cần cấu hình xác thực bổ sung ngoài chính CLI.
## Sử dụng nó làm giải pháp dự phòng

Thêm một backend CLI vào danh sách dự phòng của bạn để nó chỉ chạy khi các mô hình chính bị lỗi:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/opus-4.6", "claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/opus-4.6": {},
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

Notes:

- If you use `agents.defaults.models` (allowlist), you must include `claude-cli/...`.
- Nếu nhà cung cấp chính bị lỗi (xác thực, giới hạn tốc độ, hết thời gian chờ), OpenClaw sẽ
  thử backend CLI tiếp theo.
## Tổng quan cấu hình

Tất cả các backend CLI nằm dưới:

```
agents.defaults.cliBackends
```

Each entry is keyed by a **provider id** (e.g. `claude-cli`, `my-cli`).
The provider id becomes the left side of your model ref:

```
<provider>/<model>
```

### Example configuration

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-6": "opus",
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```
## Cách hoạt động

1. **Chọn một backend** dựa trên tiền tố nhà cung cấp (`claude-cli/...`).
2. **Xây dựng một system prompt** sử dụng cùng OpenClaw prompt + ngữ cảnh workspace.
3. **Thực thi CLI** với một session id (nếu được hỗ trợ) để lịch sử vẫn nhất quán.
4. **Phân tích đầu ra** (JSON hoặc văn bản thuần) và trả về văn bản cuối cùng.
5. **Lưu trữ session ids** cho mỗi backend, vì vậy các câu hỏi tiếp theo sẽ sử dụng lại cùng một CLI session.
## Phiên

- Nếu CLI hỗ trợ phiên, hãy đặt `sessionArg` (ví dụ: `--session-id`) hoặc
  `sessionArgs` (placeholder `{sessionId}`) khi ID cần được chèn
  vào nhiều cờ.
- Nếu CLI sử dụng **lệnh con resume** với các cờ khác nhau, hãy đặt
  `resumeArgs` (thay thế `args` khi tiếp tục) và tùy chọn `resumeOutput`
  (cho các bản resume không phải JSON).
- `sessionMode`:
  - `always`: luôn gửi một id phiên (UUID mới nếu không có được lưu trữ).
  - `existing`: chỉ gửi một id phiên nếu một cái đã được lưu trữ trước đó.
  - `none`: không bao giờ gửi một id phiên.
## Hình ảnh (pass-through)

Nếu CLI của bạn chấp nhận đường dẫn hình ảnh, hãy đặt `imageArg`:

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw will write base64 images to temp files. If `imageArg` is set, those
paths are passed as CLI args. If `imageArg` bị thiếu, OpenClaw sẽ nối thêm các
đường dẫn tệp vào lời nhắc (tiêm đường dẫn), điều này đủ cho các CLI tự động
tải các tệp cục bộ từ các đường dẫn thuần túy (hành vi Claude Code CLI).
## Đầu vào / Đầu ra

- `output: "json"` (mặc định) cố gắng phân tích JSON và trích xuất văn bản + id phiên.
- `output: "jsonl"` phân tích các luồng JSONL (Codex CLI `--json`) và trích xuất
  tin nhắn agent cuối cùng cộng với `thread_id` khi có.
- `output: "text"` coi stdout là phản hồi cuối cùng.

Chế độ đầu vào:

- `input: "arg"` (mặc định) chuyển lời nhắc dưới dạng đối số CLI cuối cùng.
- `input: "stdin"` gửi lời nhắc qua stdin.
- Nếu lời nhắc rất dài và `maxPromptArgChars` được đặt, stdin sẽ được sử dụng.
## Mặc định (tích hợp sẵn)

OpenClaw cung cấp mặc định cho `claude-cli`:

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--resume", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw cũng cung cấp mặc định cho `codex-cli`:

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

Chỉ ghi đè nếu cần thiết (phổ biến: đường dẫn `command` tuyệt đối).
## Hạn chế

- **Không có OpenClaw tools** (backend CLI không bao giờ nhận được các lệnh gọi công cụ). Một số CLI có thể vẫn chạy công cụ agent của riêng họ.
- **Không có truyền phát** (đầu ra CLI được thu thập rồi trả về).
- **Đầu ra có cấu trúc** phụ thuộc vào định dạng JSON của CLI.
- **Phiên Codex CLI** tiếp tục thông qua đầu ra văn bản (không có JSONL), ít có cấu trúc hơn lần chạy `--json` ban đầu. Các phiên OpenClaw vẫn hoạt động bình thường.
## Khắc phục sự cố

- **CLI không tìm thấy**: đặt `command` thành đường dẫn đầy đủ.
- **Tên mô hình sai**: sử dụng `modelAliases` để ánh xạ `provider/model` → mô hình CLI.
- **Không có tính liên tục của phiên**: đảm bảo `sessionArg` được đặt và `sessionMode` không phải là
  `none` (Codex CLI hiện không thể tiếp tục với đầu ra JSON).
- **Hình ảnh bị bỏ qua**: đặt `imageArg` (và xác minh CLI hỗ trợ đường dẫn tệp).