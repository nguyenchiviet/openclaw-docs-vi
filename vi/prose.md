---
summary: 'OpenProse: các workflow .prose, lệnh slash và trạng thái trong OpenClaw'
read_when:
  - Bạn muốn chạy hoặc viết các quy trình công việc .prose
  - Bạn muốn bật plugin OpenProse
  - Bạn cần hiểu về lưu trữ trạng thái
title: OpenProse
x-i18n:
  source_path: prose.md
  source_hash: 53c161466d278e5f34759313eec600a26da7018bcd52ce68c5a15e5c769bcbe5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:26.800Z'
---

# OpenProse

OpenProse là một định dạng quy trình làm việc dựa trên markdown, có thể mang theo được, để điều phối các phiên làm việc với AI. Trong OpenClaw, nó được cung cấp dưới dạng một plugin cài đặt một gói Skills OpenProse cộng với một lệnh `/prose`. Các chương trình nằm trong các tệp `.prose` và có thể tạo ra nhiều agent phụ với luồng điều khiển rõ ràng.

Trang chính thức: [https://www.prose.md](https://www.prose.md)
## Những gì nó có thể làm

- Nghiên cứu đa agent + tổng hợp với song song rõ ràng.
- Quy trình làm việc an toàn phê duyệt có thể lặp lại (đánh giá mã, phân loại sự cố, đường ống nội dung).
- Các chương trình `.prose` có thể tái sử dụng mà bạn có thể chạy trên các agent runtime được hỗ trợ.
## Cài đặt + bật

Các plugin được đóng gói bị vô hiệu hóa theo mặc định. Bật OpenProse:

```bash
openclaw plugins enable open-prose
```

Restart the Gateway after enabling the plugin.

Dev/local checkout: `openclaw plugins install ./extensions/open-prose`

Tài liệu liên quan: [Plugins](/tools/plugin), [Plugin manifest](/plugins/manifest), [Skills](/tools/skills).
## Lệnh Slash

OpenProse đăng ký `/prose` như một lệnh kỹ năng có thể gọi được bởi người dùng. Nó định tuyến đến các hướng dẫn OpenProse VM và sử dụng các công cụ OpenClaw phía dưới.

Các lệnh phổ biến:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```
## Ví dụ: một tệp `.prose` đơn giản

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```
## Vị trí tệp

OpenProse lưu trữ trạng thái dưới `.prose/` trong không gian làm việc của bạn:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

User-level persistent agents live at:

```
~/.prose/agents/
```
## Chế độ trạng thái

OpenProse hỗ trợ nhiều backend trạng thái:

- **filesystem** (mặc định): `.prose/runs/...`
- **in-context**: tạm thời, cho các chương trình nhỏ
- **sqlite** (thử nghiệm): yêu cầu `sqlite3` binary
- **postgres** (thử nghiệm): yêu cầu `psql` và một chuỗi kết nối

Ghi chú:

- sqlite/postgres là tùy chọn và thử nghiệm.
- thông tin xác thực postgres chảy vào nhật ký subagent; sử dụng một DB chuyên dụng, có đặc quyền tối thiểu.
## Các chương trình từ xa

`/prose run <handle/slug>` được phân giải thành `https://p.prose.md/<handle>/<slug>`.
Các URL trực tiếp được tìm nạp nguyên trạng. Điều này sử dụng công cụ `web_fetch` (hoặc `exec` cho POST).
## Ánh xạ runtime OpenClaw

Các chương trình OpenProse ánh xạ tới các nguyên thủy OpenClaw:

| Khái niệm OpenProse       | Công cụ OpenClaw |
| ------------------------- | ---------------- |
| Spawn session / Task tool | `sessions_spawn` |
| File read/write           | `read` / `write` |
| Web fetch                 | `web_fetch`      |

Nếu danh sách cho phép công cụ của bạn chặn các công cụ này, các chương trình OpenProse sẽ thất bại. Xem [Cấu hình Skills](/tools/skills-config).
## Bảo mật + phê duyệt

Xử lý các tệp `.prose` như mã. Kiểm tra trước khi chạy. Sử dụng danh sách cho phép công cụ OpenClaw và cổng phê duyệt để kiểm soát các tác động phụ.

Đối với các quy trình làm việc xác định, được phê duyệt, hãy so sánh với [Lobster](/tools/lobster).