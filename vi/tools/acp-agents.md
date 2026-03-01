---
summary: >-
  Sử dụng các phiên chạy ACP cho Pi, Claude Code, Codex, OpenCode, Gemini CLI và
  các tác nhân điều khiển khác.
read_when:
  - Chạy bộ kiểm thử mã qua ACP
  - Thiết lập các phiên ACP ràng buộc theo luồng trên các kênh hỗ trợ luồng
  - Khắc phục sự cố ACP backend và kết nối plugin
  - Thực hiện các lệnh /acp từ trò chuyện
title: Các tác nhân ACP
x-i18n:
  source_path: tools\acp-agents.md
  source_hash: 38a9f35a7dd2642be3e7a1aeacdf183bc73d8e83128194b86def2f4ab276e845
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:18:43.113Z'
---

# agent ACP

Các phiên [Giao thức máy khách agent (https://agentclientprotocol.com/) cho phép OpenClaw chạy các bộ công cụ mã hóa bên ngoài (ví dụ: Pi, Claude Code, Codex, OpenCode và Gemini CLI) thông qua một plugin backend ACP.

Nếu bạn yêu cầu OpenClaw bằng ngôn ngữ tự nhiên "chạy cái này trong Codex" hoặc "khởi động Claude Code trong một luồng", OpenClaw sẽ định tuyến yêu cầu đó đến thời gian chạy ACP (chứ không phải thời gian chạy sub-agent gốc).
## Quy trình vận hành nhanh

Sử dụng cách này khi bạn muốn có một sổ tay hướng dẫn vận hành `/acp` thực tế:

1. Tạo một phiên:
   - `/acp spawn codex --mode persistent --thread auto`
2. Làm việc trong luồng đã liên kết (hoặc nhắm mục tiêu khóa phiên đó một cách rõ ràng).
3. Kiểm tra trạng thái thời gian chạy:
   - `/acp status`
4. Điều chỉnh các tùy chọn thời gian chạy khi cần:
   - `/acp model <provider/model>`
   - `/acp permissions <profile>`
   - `/acp timeout <seconds>`
5. Thúc đẩy một phiên hoạt động mà không thay thế ngữ cảnh:
   - `/acp steer tighten logging and continue`
6. Dừng công việc:
   - `/acp cancel` (dừng lượt hiện tại), hoặc
   - `/acp close` (đóng phiên + xóa liên kết)
## Bắt đầu nhanh cho người dùng

Ví dụ về các yêu cầu tự nhiên:

- "Bắt đầu một phiên Codex liên tục trong một luồng ở đây và giữ cho nó tập trung."
- "Chạy cái này như một phiên Claude Code ACP một lần và tóm tắt kết quả."
- "Sử dụng Gemini CLI cho tác vụ này trong một luồng, sau đó giữ các theo dõi trong cùng luồng đó."

Những gì OpenClaw nên làm:

1. Chọn `runtime: "acp"`.
2. Giải quyết mục tiêu harness được yêu cầu (`agentId`, ví dụ `codex`).
3. Nếu yêu cầu liên kết luồng và kênh hiện tại hỗ trợ, hãy liên kết phiên ACP với luồng.
4. Định tuyến các tin nhắn theo dõi trong luồng đến cùng phiên ACP đó cho đến khi không tập trung/đóng/hết hạn.
## ACP so với sub-agent

Sử dụng ACP khi bạn muốn một môi trường chạy harness bên ngoài. Sử dụng sub-agent khi bạn muốn các tác vụ được ủy quyền chạy nguyên bản trên OpenClaw.

| Lĩnh vực      | Phiên ACP                             | Chạy sub-agent                     |
| ------------- | ------------------------------------- | ---------------------------------- |
| Môi trường chạy | Plugin backend ACP (ví dụ acpx)       | Môi trường chạy sub-agent nguyên bản của OpenClaw |
| Khóa phiên    | `agent:<agentId>:acp:<uuid>`          | `agent:<agentId>:subagent:<uuid>`  |
| Lệnh chính    | `/acp ...`                            | `/subagents ...`                   |
| Công cụ tạo   | `sessions_spawn` với `runtime:"acp"` | `sessions_spawn` (môi trường chạy mặc định) |

Xem thêm [Sub-agent](/tools/subagents).

## Phiên ràng buộc theo luồng (không phụ thuộc kênh)

Khi tính năng ràng buộc luồng được bật cho một bộ điều hợp kênh, các phiên ACP có thể được ràng buộc với các luồng:

- OpenClaw ràng buộc một luồng với một phiên ACP mục tiêu.
- Các tin nhắn tiếp theo trong luồng đó sẽ được định tuyến đến phiên ACP đã ràng buộc.
- Đầu ra của ACP được gửi trở lại cùng một luồng.
- Hủy tập trung/đóng/lưu trữ/hết thời gian chờ không hoạt động hoặc hết hạn tuổi tối đa sẽ loại bỏ ràng buộc.

Hỗ trợ ràng buộc luồng là đặc thù của bộ điều hợp. Nếu bộ điều hợp kênh đang hoạt động không hỗ trợ ràng buộc luồng, OpenClaw sẽ trả về một thông báo rõ ràng về việc không được hỗ trợ/không khả dụng.

Các cờ tính năng bắt buộc cho ACP ràng buộc theo luồng:

- `acp.enabled=true`
- `acp.dispatch.enabled=true`
- Cờ tạo luồng ACP của bộ điều hợp kênh được bật (đặc thù của bộ điều hợp)
  - Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

### Các kênh hỗ trợ luồng

- Bất kỳ bộ điều hợp kênh nào hiển thị khả năng ràng buộc phiên/luồng.
- Hỗ trợ tích hợp hiện tại: Discord.
- Các kênh plugin có thể thêm hỗ trợ thông qua cùng một giao diện ràng buộc.
## Bắt đầu các phiên ACP (giao diện)

### Từ `sessions_spawn`

Sử dụng `runtime: "acp"` để bắt đầu một phiên ACP từ một lượt agent hoặc lệnh gọi công cụ.

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

Notes:

- `runtime` defaults to `subagent`, so set `runtime: "acp"__OC_I19N_0006__agentId` is omitted, OpenClaw uses `acp.defaultAgent` when configured.
- `mode: "session"` requires `thread: true` to keep a persistent bound conversation.

Interface details:

- `task` (required): initial prompt sent to the ACP session.
- `runtime` (required for ACP): must be `"acp"`.
- `agentId` (optional): ACP target harness id. Falls back to `acp.defaultAgent` if set.
- `thread` (optional, default `false`): request thread binding flow where supported.
- `mode` (optional): `run` (one-shot) or `session` (persistent).
  - default is `run`
- nếu `thread: true` và chế độ bị bỏ qua, OpenClaw có thể mặc định hành vi liên tục theo đường dẫn thời gian chạy
  - `mode: "session"` yêu cầu `thread: true`
- `cwd` (tùy chọn): thư mục làm việc thời gian chạy được yêu cầu (được xác thực bởi chính sách backend/thời gian chạy).
- `label` (tùy chọn): nhãn hướng tới người vận hành được sử dụng trong văn bản phiên/biểu ngữ.

### Từ lệnh `/acp`

Sử dụng `/acp spawn` để kiểm soát rõ ràng của người vận hành từ trò chuyện khi cần.

```text
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

Key flags:

- `--mode persistent|oneshot`
- `--thread auto|here|off`
- `--cwd <absolute-path>`
- `--label <name>`

Xem [Lệnh gạch chéo](/tools/slash-commands).

## Giải quyết mục tiêu phiên

Hầu hết các hành động `/acp` chấp nhận một mục tiêu phiên tùy chọn (`session-key`, `session-id`, hoặc `session-label`).

Thứ tự giải quyết:

1. Đối số mục tiêu rõ ràng (hoặc `--session` cho `/acp steer`)
   - thử khóa
   - sau đó là ID phiên có dạng UUID
   - sau đó là nhãn
2. Liên kết luồng hiện tại (nếu cuộc trò chuyện/luồng này được liên kết với một phiên ACP)
3. Dự phòng phiên của người yêu cầu hiện tại

Nếu không có mục tiêu nào được giải quyết, OpenClaw sẽ trả về một lỗi rõ ràng (`Unable to resolve session target: ...`).
## Các chế độ tạo luồng

`/acp spawn` hỗ trợ `--thread auto|here|off`.

| Chế độ   | Hành vi                                                                                            |
| ------ | --------------------------------------------------------------------------------------------------- |
| `auto` | Trong một luồng đang hoạt động: liên kết luồng đó. Ngoài một luồng: tạo/liên kết một luồng con khi được hỗ trợ. |
| `here` | Yêu cầu luồng đang hoạt động hiện tại; thất bại nếu không ở trong một luồng.                                                  |
| `off`  | Không liên kết. Phiên bắt đầu không liên kết.                                                                 |

Lưu ý:

- Trên các bề mặt không liên kết luồng, hành vi mặc định thực chất là `off`.
- Việc tạo luồng liên kết yêu cầu hỗ trợ chính sách kênh (đối với Discord: `channels.discord.threadBindings.spawnAcpSessions=true`).
## Điều khiển ACP

Nhóm lệnh có sẵn:

- __OC_I19N_0000__
- __OC_I19N_0001__
- __OC_I19N_0002__
- __OC_I19N_0003__
- __OC_I19N_0004__
- __OC_I19N_0005__
- __OC_I19N_0006__
- __OC_I19N_0007__
- __OC_I19N_0008__
- __OC_I19N_0009__
- __OC_I19N_0010__
- __OC_I19N_0011__
- __OC_I19N_0012__
- __OC_I19N_0013__
- __OC_I19N_0014__

__OC_I19N_0015__ hiển thị các tùy chọn thời gian chạy hiệu quả và, khi có sẵn, cả định danh phiên cấp thời gian chạy và cấp backend.

Một số điều khiển phụ thuộc vào khả năng của backend. Nếu một backend không hỗ trợ một điều khiển, OpenClaw sẽ trả về lỗi điều khiển không được hỗ trợ rõ ràng.
## Sổ tay lệnh ACP

| Lệnh              | Chức năng                                                 | Ví dụ                                                        |
| -------------------- | --------------------------------------------------------- | -------------------------------------------------------------- |
| `/acp spawn`         | Tạo phiên ACP; tùy chọn liên kết luồng.                  | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel`        | Hủy lượt đang thực hiện cho phiên mục tiêu.               | `/acp cancel agent:codex:acp:<uuid>`                           |
| `/acp steer`         | Gửi lệnh điều khiển đến phiên đang chạy.                  | `/acp steer --session support inbox prioritize failing tests`  |
| `/acp close`         | Đóng phiên và hủy liên kết các mục tiêu luồng.            | `/acp close`                                                   |
| `/acp status`        | Hiển thị backend, chế độ, trạng thái, tùy chọn thời gian chạy, khả năng. | `/acp status`                                                  |
| `/acp set-mode`      | Đặt chế độ thời gian chạy cho phiên mục tiêu.             | `/acp set-mode plan`                                           |
| `/acp set`           | Ghi tùy chọn cấu hình thời gian chạy chung.               | `/acp set model openai/gpt-5.2`                                |
| `/acp cwd`           | Đặt ghi đè thư mục làm việc thời gian chạy.               | `/acp cwd /Users/user/Projects/repo`                           |
| `/acp permissions`   | Đặt hồ sơ chính sách phê duyệt.                           | `/acp permissions strict`                                      |
| `/acp timeout`       | Đặt thời gian chờ thời gian chạy (giây).                  | `/acp timeout 120`                                             |
| `/acp model`         | Đặt ghi đè mô hình thời gian chạy.                        | `/acp model anthropic/claude-opus-4-5`                         |
| `/acp reset-options` | Xóa ghi đè tùy chọn thời gian chạy của phiên.             | `/acp reset-options`                                           |
| `/acp sessions`      | Liệt kê các phiên ACP gần đây từ kho lưu trữ.             | `/acp sessions`                                                |
| `/acp doctor`        | Tình trạng backend, khả năng, các bản sửa lỗi có thể thực hiện. | `/acp doctor`                                                  |
| `/acp install`       | In các bước cài đặt và kích hoạt xác định.                | `/acp install`                                                 |
## Ánh xạ tùy chọn thời gian chạy

`/acp` có các lệnh tiện ích và một bộ thiết lập chung.

Các thao tác tương đương:

- `/acp model <id>` ánh xạ tới khóa cấu hình thời gian chạy `model`.
- `/acp permissions <profile>` ánh xạ tới khóa cấu hình thời gian chạy `approval_policy`.
- `/acp timeout <seconds>` ánh xạ tới khóa cấu hình thời gian chạy `timeout`.
- `/acp cwd <path>` cập nhật trực tiếp ghi đè cwd thời gian chạy.
- `/acp set <key> <value>` là đường dẫn chung.
  - Trường hợp đặc biệt: `key=cwd` sử dụng đường dẫn ghi đè cwd.
- `/acp reset-options` xóa tất cả các ghi đè thời gian chạy cho phiên mục tiêu.
## hỗ trợ acpx harness (hiện tại)

Các bí danh harness tích hợp sẵn của acpx hiện tại:

- `pi`
- `claude`
- `codex`
- `opencode`
- `gemini`

Khi OpenClaw sử dụng backend acpx, hãy ưu tiên các giá trị này cho `agentId` trừ khi cấu hình acpx của bạn định nghĩa các bí danh agent tùy chỉnh.

Việc sử dụng CLI acpx trực tiếp cũng có thể nhắm mục tiêu các bộ điều hợp tùy ý thông qua `--agent <command>`, nhưng lối thoát thô đó là một tính năng của CLI acpx (không phải đường dẫn `agentId` thông thường của OpenClaw).
## Cấu hình bắt buộc

Đường cơ sở ACP cốt lõi:

```json5
{
  acp: {
    enabled: true,
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

Thread binding config is channel-adapter specific. Example for Discord:

```json5
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```
Nếu việc tạo ACP ràng buộc theo luồng không hoạt động, trước tiên hãy xác minh cờ tính năng bộ điều hợp:

- Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

Xem [Tài liệu cấu hình](/gateway/configuration-reference).

## Thiết lập plugin cho backend acpx

Cài đặt và bật plugin:

```bash
openclaw plugins install @openclaw/acpx
openclaw config set plugins.entries.acpx.enabled true
``__OC_I19N_0001__``bash
openclaw plugins install ./extensions/acpx
```

Then verify backend health:

```text
/acp doctor
```

### Pinned acpx install strategy (current behavior)

`@openclaw/acpx` now enforces a strict plugin-local pinning model:

1. The extension pins an exact acpx dependency in `extensions/acpx/package.json`.
2. Runtime command is fixed to the plugin-local binary (`extensions/acpx/node_modules/.bin/acpx`), not global `PATH`.
3. Plugin config does not expose `command` or `commandArgs`, so runtime command drift is blocked.
4. Startup registers the ACP backend immediately as not-ready.
5. A background ensure job verifies `acpx --version` so với phiên bản đã ghim.
6. Nếu thiếu/không khớp, nó sẽ chạy cài đặt plugin cục bộ (__OC_I19N_0000__) và xác minh lại trước khi hoạt động bình thường.

Lưu ý:

- Khởi động OpenClaw vẫn không bị chặn trong khi acpx ensure đang chạy.
- Nếu mạng/cài đặt thất bại, phần phụ trợ vẫn không khả dụng và __OC_I19N_0001__ báo cáo một bản sửa lỗi có thể thực hiện được.

Xem [Các plugin](__OC_I19N_0002__).
## Khắc phục sự cố

| Triệu chứng                                                             | Nguyên nhân có thể                             | Cách khắc phục                                             |
| :---------------------------------------------------------------------- | :--------------------------------------------- | :--------------------------------------------------------- |
| `ACP runtime backend is not configured`                                | Plugin backend bị thiếu hoặc bị vô hiệu hóa.   | Cài đặt và bật plugin backend, sau đó chạy `/acp doctor`. |
| `ACP is disabled by policy (acp.enabled=false)`                        | ACP bị vô hiệu hóa toàn cầu.                   | Đặt `acp.enabled=true`.                                    |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)`      | Gửi tin nhắn từ các luồng thông thường bị vô hiệu hóa. | Đặt `acp.dispatch.enabled=true`.                           |
| `ACP agent "<id>" is not allowed by policy`                            | Agent không có trong danh sách cho phép.       | Sử dụng `agentId` được phép hoặc cập nhật `acp.allowedAgents`.       |
| `Unable to resolve session target: ...`                                | Mã thông báo khóa/id/nhãn không hợp lệ.        | Chạy `/acp sessions`, sao chép chính xác khóa/nhãn, thử lại.          |
| `--thread here requires running /acp spawn inside an active ... thread` | `--thread here` được sử dụng bên ngoài ngữ cảnh luồng. | Di chuyển đến luồng đích hoặc sử dụng `--thread auto`/`off`.        |
| `Only <user-id> can rebind this thread.`                               | Người dùng khác sở hữu liên kết luồng.         | Liên kết lại với tư cách chủ sở hữu hoặc sử dụng một luồng khác.                 |
| `Thread bindings are unavailable for <channel>.`                       | Bộ điều hợp thiếu khả năng liên kết luồng.      | Sử dụng `--thread off` hoặc chuyển sang bộ điều hợp/kênh được hỗ trợ.   |
| Thiếu siêu dữ liệu ACP cho phiên đã liên kết                                 | Siêu dữ liệu phiên ACP cũ/đã xóa.             | Tạo lại bằng `/acp spawn`, sau đó liên kết lại/tập trung luồng.      |