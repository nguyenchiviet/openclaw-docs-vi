---
title: Tôm hùm
summary: >-
  Thời gian chạy quy trình làm việc được gõ cho OpenClaw với các cổng phê duyệt
  có thể tiếp tục.
description: >-
  Typed workflow runtime for OpenClaw — composable pipelines with approval
  gates.
read_when:
  - Bạn muốn các quy trình làm việc đa bước xác định với các phê duyệt rõ ràng
  - Bạn cần tiếp tục một workflow mà không cần chạy lại các bước trước đó
x-i18n:
  source_path: tools\lobster.md
  source_hash: c6d7c06865f4646797044511822a24395deb92f50c41f72316bfc9b6273b7259
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:31:02.035Z'
---

# Lobster

Lobster là một workflow shell cho phép OpenClaw chạy các chuỗi công cụ đa bước như một hoạt động duy nhất, xác định với các điểm kiểm soát phê duyệt rõ ràng.
## Hook

Trợ lý của bạn có thể xây dựng các công cụ quản lý chính nó. Yêu cầu một quy trình làm việc, và 30 phút sau bạn sẽ có một CLI cộng với các pipeline chạy dưới dạng một lệnh gọi. Lobster là phần còn thiếu: các pipeline xác định, phê duyệt rõ ràng và trạng thái có thể tiếp tục.
## Tại sao

Ngày nay, các quy trình công việc phức tạp yêu cầu nhiều lệnh gọi công cụ qua lại. Mỗi lệnh gọi tốn token, và LLM phải điều phối từng bước. Lobster chuyển việc điều phối đó vào một runtime được gõ kiểu:

- **Một lệnh gọi thay vì nhiều**: OpenClaw chạy một lệnh gọi công cụ Lobster và nhận được một kết quả có cấu trúc.
- **Phê duyệt được tích hợp sẵn**: Các tác dụng phụ (gửi email, đăng bình luận) dừng quy trình công việc cho đến khi được phê duyệt rõ ràng.
- **Có thể tiếp tục**: Các quy trình công việc bị dừng trả về một token; phê duyệt và tiếp tục mà không cần chạy lại mọi thứ.
## Tại sao lại dùng DSL thay vì các chương trình thông thường?

Lobster được thiết kế cố ý để nhỏ gọn. Mục tiêu không phải là "một ngôn ngữ mới," mà là một đặc tả pipeline thân thiện với AI, có khả năng phê duyệt hạng nhất và token tiếp tục.

- **Phê duyệt/tiếp tục được tích hợp sẵn**: Một chương trình thông thường có thể nhắc người dùng, nhưng nó không thể _tạm dừng và tiếp tục_ với một token bền vững mà không cần bạn tự tạo runtime đó.
- **Tính xác định + khả năng kiểm toán**: Pipeline là dữ liệu, vì vậy chúng dễ dàng ghi nhật ký, so sánh, phát lại và xem xét.
- **Bề mặt bị giới hạn cho AI**: Một ngữ pháp nhỏ + piping JSON giảm các đường mã "sáng tạo" và làm cho xác thực trở nên thực tế.
- **Chính sách bảo mật được tích hợp sẵn**: Timeout, giới hạn đầu ra, kiểm tra sandbox và danh sách cho phép được thực thi bởi runtime, không phải từng script.
- **Vẫn có thể lập trình được**: Mỗi bước có thể gọi bất kỳ CLI hoặc script nào. Nếu bạn muốn JS/TS, hãy tạo các tệp `.lobster` từ mã.
## Cách hoạt động

OpenClaw khởi chạy `lobster` CLI cục bộ ở **chế độ công cụ** và phân tích một phong bì JSON từ stdout.
Nếu pipeline tạm dừng để chờ phê duyệt, công cụ sẽ trả về `resumeToken` để bạn có thể tiếp tục sau.
## Mẫu: CLI nhỏ + JSON pipes + phê duyệt

Xây dựng các lệnh nhỏ gọn nói JSON, sau đó kết nối chúng thành một lệnh gọi Lobster duy nhất. (Tên lệnh ví dụ dưới đây — thay thế bằng lệnh của riêng bạn.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

If the pipeline requests approval, resume with the token:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI triggers the workflow; Lobster executes the steps. Approval gates keep side effects explicit and auditable.

Example: map input items into tool calls:

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```
## Các bước LLM chỉ JSON (llm-task)

Đối với các quy trình làm việc cần một **bước LLM có cấu trúc**, hãy bật công cụ plugin tùy chọn
`llm-task` và gọi nó từ Lobster. Điều này giữ cho quy trình làm việc có tính xác định trong khi vẫn cho phép bạn phân loại/tóm tắt/soạn thảo với một mô hình.

Bật công cụ:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
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

Use it in a pipeline:

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
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

Xem [LLM Task](/tools/llm-task) để biết chi tiết và các tùy chọn cấu hình.
## Tệp quy trình làm việc (.lobster)

Lobster có thể chạy các tệp quy trình làm việc YAML/JSON với các trường `name`, `args`, `steps`, `env`, `condition`, và `approval`. Trong các lệnh gọi công cụ OpenClaw, đặt `pipeline` thành đường dẫn tệp.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Notes:

- `stdin: $step.stdout` and `stdin: $step.json` pass a prior step’s output.
- `condition` (or `when`) can gate steps on `$step.approved`.
## Cài đặt Lobster

Cài đặt Lobster CLI trên **cùng một máy chủ** chạy OpenClaw Gateway (xem [kho lưu trữ Lobster](https://github.com/openclaw/lobster)), và đảm bảo `lobster` nằm trên `PATH`.
## Bật công cụ

Lobster là một công cụ plugin **tùy chọn** (không được bật theo mặc định).

Được khuyến nghị (bổ sung, an toàn):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Or per-agent:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Avoid using `tools.allow: ["lobster"]` unless you intend to run in restrictive allowlist mode.

Note: allowlists are opt-in for optional plugins. If your allowlist only names
plugin tools (like `lobster`), OpenClaw giữ các công cụ cốt lõi được bật. Để hạn chế các công cụ cốt lõi, hãy bao gồm các công cụ cốt lõi hoặc nhóm bạn muốn trong danh sách cho phép.
## Ví dụ: Phân loại email

Không có Lobster:

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

With Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Returns a JSON envelope (truncated):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

User approves → resume:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Một quy trình. Xác định. An toàn.
## Tham số công cụ

### `run`

Chạy một pipeline ở chế độ công cụ.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Run a workflow file with args:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### `resume`

Continue a halted workflow after approval.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### Optional inputs

- `cwd`: Relative working directory for the pipeline (must stay within the current process working directory).
- `timeoutMs`: Kill the subprocess if it exceeds this duration (default: 20000).
- `maxStdoutBytes`: Kill the subprocess if stdout exceeds this size (default: 512000).
- `argsJson`: JSON string passed to `lobster run --args-json` (chỉ các tệp workflow).
## Bao bọc đầu ra

Lobster trả về một bao bọc JSON với một trong ba trạng thái:

- `ok` → hoàn thành thành công
- `needs_approval` → tạm dừng; `requiresApproval.resumeToken` được yêu cầu để tiếp tục
- `cancelled` → bị từ chối hoặc hủy một cách rõ ràng

Công cụ hiển thị bao bọc ở cả `content` (JSON đẹp) và `details` (đối tượng thô).
## Phê duyệt

Nếu `requiresApproval` có mặt, hãy kiểm tra lời nhắc và quyết định:

- `approve: true` → tiếp tục và thực hiện các tác dụng phụ
- `approve: false` → hủy và hoàn tất quy trình làm việc

Sử dụng `approve --preview-from-stdin --limit N` để đính kèm bản xem trước JSON vào các yêu cầu phê duyệt mà không cần glue jq/heredoc tùy chỉnh. Các token tiếp tục hiện đã được nén gọn: Lobster lưu trữ trạng thái tiếp tục quy trình làm việc trong thư mục trạng thái của nó và trả lại một khóa token nhỏ.
## OpenProse

OpenProse kết hợp tốt với Lobster: sử dụng `/prose` để điều phối chuẩn bị đa agent, sau đó chạy pipeline Lobster để phê duyệt xác định. Nếu chương trình Prose cần Lobster, cho phép công cụ `lobster` cho các agent phụ thông qua `tools.subagents.tools`. Xem [OpenProse](/prose).
## Bảo mật

- **Chỉ subprocess cục bộ** — không có lệnh gọi mạng từ plugin.
- **Không có bí mật** — Lobster không quản lý OAuth; nó gọi các công cụ OpenClaw thực hiện việc đó.
- **Nhận thức sandbox** — bị vô hiệu hóa khi ngữ cảnh công cụ được sandboxed.
- **Được gia cố** — tên tệp thực thi cố định (`lobster`) trên `PATH`; các hết thời gian chờ và giới hạn đầu ra được thực thi.
## Khắc phục sự cố

- **`lobster subprocess timed out`** → tăng `timeoutMs`, hoặc chia nhỏ một pipeline dài.
- **`lobster output exceeded maxStdoutBytes`** → nâng cao `maxStdoutBytes` hoặc giảm kích thước đầu ra.
- **`lobster returned invalid JSON`** → đảm bảo pipeline chạy ở chế độ công cụ và chỉ in JSON.
- **`lobster failed (code …)`** → chạy cùng một pipeline trong terminal để kiểm tra stderr.
## Tìm hiểu thêm

- [Plugins](/tools/plugin)
- [Plugin tool authoring](/plugins/agent-tools)
## Nghiên cứu trường hợp: quy trình làm việc của cộng đồng

Một ví dụ công khai: CLI "second brain" + Lobster pipelines quản lý ba kho Markdown (cá nhân, đối tác, chia sẻ). CLI phát ra JSON cho thống kê, danh sách inbox và quét lỗi thời; Lobster kết nối các lệnh đó thành quy trình làm việc như `weekly-review`, `inbox-triage`, `memory-consolidation`, và `shared-task-sync`, mỗi cái có cổng phê duyệt. AI xử lý phán đoán (phân loại) khi có sẵn và quay lại các quy tắc xác định khi không.

- Luồng: [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
- Kho: [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)