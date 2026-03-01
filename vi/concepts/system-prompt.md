---
summary: Những gì chứa trong system prompt của OpenClaw và cách nó được lắp ráp
read_when:
  - >-
    Chỉnh sửa văn bản system prompt, danh sách công cụ hoặc các phần thời
    gian/nhịp tim
  - Thay đổi hành vi khởi động không gian làm việc hoặc tiêm Skills
title: Lời nhắc hệ thống
x-i18n:
  source_path: concepts\system-prompt.md
  source_hash: 5a302ce29ad1df958c0bebc135987a7348b233c4a98d3ef2a58088488fd3e01f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:14.714Z'
---

# System Prompt

OpenClaw xây dựng một system prompt tùy chỉnh cho mỗi lần chạy agent. Prompt này **thuộc sở hữu của OpenClaw** và không sử dụng default prompt của pi-coding-agent.

Prompt được OpenClaw lắp ráp và tiêm vào mỗi lần chạy agent.
## Cấu trúc

Prompt được thiết kế cố ý để compact và sử dụng các phần cố định:

- **Tooling**: danh sách công cụ hiện tại + mô tả ngắn.
- **Safety**: nhắc nhở guardrail ngắn để tránh hành vi tìm kiếm quyền lực hoặc vượt qua giám sát.
- **Skills** (khi có sẵn): cho mô hình biết cách tải hướng dẫn skill theo yêu cầu.
- **OpenClaw Self-Update**: cách chạy `config.apply` và `update.run`.
- **Workspace**: thư mục làm việc (`agents.defaults.workspace`).
- **Documentation**: đường dẫn cục bộ đến tài liệu OpenClaw (repo hoặc gói npm) và khi nào nên đọc.
- **Workspace Files (injected)**: cho biết các tệp bootstrap được bao gồm bên dưới.
- **Sandbox** (khi được bật): cho biết runtime sandboxed, đường dẫn sandbox, và liệu có sẵn elevated exec hay không.
- **Current Date & Time**: thời gian cục bộ của người dùng, múi giờ và định dạng thời gian.
- **Reply Tags**: cú pháp thẻ trả lời tùy chọn cho các nhà cung cấp được hỗ trợ.
- **Heartbeats**: prompt heartbeat và hành vi ack.
- **Runtime**: host, OS, node, mô hình, repo root (khi được phát hiện), mức thinking (một dòng).
- **Reasoning**: mức độ hiển thị hiện tại + gợi ý chuyển đổi /reasoning.

Các guardrail an toàn trong system prompt mang tính chất tư vấn. Chúng hướng dẫn hành vi mô hình nhưng không thực thi chính sách. Sử dụng chính sách công cụ, phê duyệt exec, sandboxing và allowlist kênh để thực thi cứng; các nhà điều hành có thể vô hiệu hóa những điều này theo thiết kế.
## Chế độ Prompt

OpenClaw có thể hiển thị các system prompt nhỏ hơn cho các sub-agent. Runtime đặt một
`promptMode` cho mỗi lần chạy (không phải cấu hình dành cho người dùng):

- `full` (mặc định): bao gồm tất cả các phần ở trên.
- `minimal`: được sử dụng cho sub-agent; bỏ qua **Skills**, **Memory Recall**, **OpenClaw
  Self-Update**, **Model Aliases**, **User Identity**, **Reply Tags**,
  **Messaging**, **Silent Replies**, và **Heartbeats**. Tooling, **Safety**,
  Workspace, Sandbox, Current Date & Time (khi biết), Runtime, và injected
  context vẫn có sẵn.
- `none`: chỉ trả về dòng nhận dạng cơ bản.

Khi `promptMode=minimal`, các prompt được inject thêm được gắn nhãn là **Subagent
Context** thay vì **Group Chat Context**.
## Tiêm bootstrap không gian làm việc

Các tệp bootstrap được cắt ngắn và nối thêm dưới **Project Context** để mô hình thấy bối cảnh danh tính và hồ sơ mà không cần đọc rõ ràng:

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (chỉ trên các không gian làm việc hoàn toàn mới)
- `MEMORY.md` và/hoặc `memory.md` (khi có trong không gian làm việc; có thể tiêm một hoặc cả hai)

Tất cả các tệp này được **tiêm vào cửa sổ ngữ cảnh** trên mỗi lượt, điều này
có nghĩa là chúng tiêu thụ token. Giữ chúng ngắn gọn — đặc biệt là `MEMORY.md`, có thể
phát triển theo thời gian và dẫn đến mức sử dụng ngữ cảnh cao hơn dự kiến và nén
thường xuyên hơn.

> **Lưu ý:** Các tệp `memory/*.md` hàng ngày **không** được tiêm tự động. Chúng
> được truy cập theo yêu cầu thông qua các công cụ `memory_search` và `memory_get`, vì vậy chúng
> không tính vào cửa sổ ngữ cảnh trừ khi mô hình rõ ràng đọc chúng.

Các tệp lớn được cắt ngắn bằng một dấu hiệu. Kích thước tối đa trên mỗi tệp được kiểm soát bởi
`agents.defaults.bootstrapMaxChars` (mặc định: 20000). Tổng nội dung bootstrap được tiêm
trên các tệp được giới hạn bởi `agents.defaults.bootstrapTotalMaxChars`
(mặc định: 150000). Các tệp bị thiếu tiêm một dấu hiệu tệp bị thiếu ngắn.

Các phiên sub-agent chỉ tiêm `AGENTS.md` và `TOOLS.md` (các tệp bootstrap khác
được lọc ra để giữ ngữ cảnh sub-agent nhỏ).

Các hook nội bộ có thể chặn bước này thông qua `agent:bootstrap` để thay đổi hoặc thay thế
các tệp bootstrap được tiêm (ví dụ: hoán đổi `SOUL.md` cho một nhân vật thay thế).

Để kiểm tra mỗi tệp được tiêm đóng góp bao nhiêu (thô vs được tiêm, cắt ngắn, cộng với chi phí lược đồ công cụ), sử dụng `/context list` hoặc `/context detail`. Xem [Ngữ cảnh](/concepts/context).
## Xử lý thời gian

Lời nhắc hệ thống bao gồm một phần **Ngày & Giờ hiện tại** chuyên dụng khi biết múi giờ của người dùng. Để giữ cho bộ nhớ đệm lời nhắc ổn định, nó hiện chỉ bao gồm **múi giờ** (không có đồng hồ động hoặc định dạng thời gian).

Sử dụng `session_status` khi agent cần thời gian hiện tại; thẻ trạng thái bao gồm một dòng dấu thời gian.

Cấu hình với:

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Xem [Ngày & Giờ](/date-time) để biết chi tiết hành vi đầy đủ.
## Skills

Khi các Skills phù hợp tồn tại, OpenClaw sẽ chèn một danh sách **các Skills có sẵn** nhỏ gọn
(`formatSkillsForPrompt`) bao gồm **đường dẫn tệp** cho mỗi Skill. Prompt hướng dẫn mô hình sử dụng `read` để tải SKILL.md tại vị trí được liệt kê (workspace, managed, hoặc bundled). Nếu không có Skills phù hợp, phần Skills sẽ bị bỏ qua.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Điều này giữ cho prompt cơ bản nhỏ gọn trong khi vẫn cho phép sử dụng Skill có mục tiêu.
## Tài liệu

Khi có sẵn, lời nhắc hệ thống bao gồm phần **Tài liệu** trỏ đến
thư mục tài liệu OpenClaw cục bộ (hoặc `docs/` trong không gian làm việc repo hoặc tài liệu gói npm được đóng gói) và cũng ghi chú gương công khai, repo nguồn, Discord cộng đồng, và
ClawHub ([https://clawhub.com](https://clawhub.com)) để khám phá Skills. Lời nhắc hướng dẫn mô hình tham khảo tài liệu cục bộ trước
cho hành vi OpenClaw, lệnh, cấu hình hoặc kiến trúc, và chạy
`openclaw status` chính nó khi có thể (chỉ yêu cầu người dùng khi nó không có quyền truy cập).