---
summary: >-
  Ngữ cảnh: những gì mô hình nhìn thấy, cách nó được xây dựng, và cách kiểm tra
  nó
read_when:
  - Bạn muốn hiểu "context" có nghĩa gì trong OpenClaw
  - Bạn đang gỡ lỗi để tìm hiểu tại sao mô hình "biết" điều gì đó (hoặc quên nó)
  - 'Bạn muốn giảm chi phí ngữ cảnh (/context, /status, /compact)'
title: >-
  Tôi hiểu. Tôi sẽ:


  1. Dịch văn bản tiếng Anh sang tiếng Việt

  2. Chỉ xuất bản văn bản đã dịch, không có nội dung khác

  3. Giữ nguyên tên sản phẩm: OpenClaw, Pi, Gateway, WhatsApp, Telegram,
  Discord, iMessage, Slack, Signal, Tailscale, CLI, Skills

  4. Sử dụng tiếng Việt tự nhiên và phù hợp cho tài liệu kỹ thuật


  Tôi đã sẵn sàng. Vui lòng cung cấp văn bản tiếng Anh cần dịch.
x-i18n:
  source_path: concepts\context.md
  source_hash: 704b0d69b3a7f38a4a510e21c84b0ddf4b14c1a1aaa0d8d0dc6632e30897cca4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:39:04.558Z'
---

# Ngữ cảnh

"Ngữ cảnh" là **mọi thứ OpenClaw gửi đến mô hình cho một lần chạy**. Nó bị giới hạn bởi **cửa sổ ngữ cảnh** của mô hình (giới hạn token).

Mô hình tư duy cho người mới bắt đầu:

- **System prompt** (được xây dựng bởi OpenClaw): quy tắc, công cụ, danh sách Skills, thời gian/thời gian chạy và các tệp workspace được tiêm.
- **Lịch sử cuộc trò chuyện**: tin nhắn của bạn + tin nhắn của trợ lý cho phiên này.
- **Lệnh gọi công cụ/kết quả + tệp đính kèm**: đầu ra lệnh, đọc tệp, hình ảnh/âm thanh, v.v.

Ngữ cảnh _không giống với_ "bộ nhớ": bộ nhớ có thể được lưu trữ trên đĩa và tải lại sau; ngữ cảnh là những gì bên trong cửa sổ hiện tại của mô hình.
## Bắt đầu nhanh (kiểm tra ngữ cảnh)

- `/status` → chế độ xem nhanh "cửa sổ của tôi đầy bao nhiêu?" + cài đặt phiên.
- `/context list` → những gì được tiêm vào + kích thước gần đúng (mỗi tệp + tổng cộng).
- `/context detail` → phân tích sâu hơn: kích thước lược đồ mỗi tệp, mỗi công cụ, kích thước mục nhập mỗi Skill và kích thước lời nhắc hệ thống.
- `/usage tokens` → thêm chân trang sử dụng mỗi lần trả lời vào các câu trả lời bình thường.
- `/compact` → tóm tắt lịch sử cũ hơn thành một mục nhập nhỏ gọn để giải phóng không gian cửa sổ.

Xem thêm: [Lệnh gạch chéo](/tools/slash-commands), [Sử dụng token & chi phí](/reference/token-use), [Nén dữ liệu](/concepts/compaction).
## Ví dụ về kết quả

Các giá trị khác nhau tùy theo mô hình, nhà cung cấp, chính sách công cụ và những gì có trong không gian làm việc của bạn.

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/chi tiết ngữ cảnh`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```
## Những gì tính vào cửa sổ ngữ cảnh

Mọi thứ mô hình nhận được đều tính, bao gồm:

- System prompt (tất cả các phần).
- Lịch sử cuộc trò chuyện.
- Lệnh gọi công cụ + kết quả công cụ.
- Tệp đính kèm/bản ghi chép (hình ảnh/âm thanh/tệp).
- Tóm tắt nén và các tạo tác cắt tỉa.
- "Wrapper" nhà cung cấp hoặc tiêu đề ẩn (không hiển thị, vẫn được tính).
## Cách OpenClaw xây dựng system prompt

System prompt là **do OpenClaw sở hữu** và được xây dựng lại mỗi lần chạy. Nó bao gồm:

- Danh sách công cụ + mô tả ngắn.
- Danh sách Skills (chỉ siêu dữ liệu; xem bên dưới).
- Vị trí không gian làm việc.
- Thời gian (UTC + thời gian người dùng được chuyển đổi nếu được cấu hình).
- Siêu dữ liệu runtime (host/OS/mô hình/thinking).
- Các tệp bootstrap không gian làm việc được tiêm dưới **Project Context**.

Chi tiết đầy đủ: [System Prompt](/concepts/system-prompt).
## Các tệp workspace được tiêm (Ngữ cảnh dự án)

Theo mặc định, OpenClaw tiêm một bộ tệp workspace cố định (nếu có):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (chỉ lần chạy đầu tiên)

Các tệp lớn được cắt ngắn theo từng tệp bằng `agents.defaults.bootstrapMaxChars` (mặc định `20000` ký tự). OpenClaw cũng thực thi giới hạn tiêm bootstrap tổng thể trên các tệp với `agents.defaults.bootstrapTotalMaxChars` (mặc định `150000` ký tự). `/context` hiển thị **kích thước thô vs được tiêm** và liệu có xảy ra cắt ngắn hay không.
## Skills: những gì được tiêm vào so với tải theo yêu cầu

System prompt bao gồm một **danh sách skills** nhỏ gọn (tên + mô tả + vị trí). Danh sách này có chi phí thực tế.

Hướng dẫn Skill _không_ được bao gồm theo mặc định. Mô hình dự kiến sẽ `read` **chỉ khi cần thiết** của skill `SKILL.md`.
## Tools: có hai chi phí

Tools ảnh hưởng đến context theo hai cách:

1. **Văn bản danh sách Tool** trong system prompt (những gì bạn thấy là "Tooling").
2. **Các schema Tool** (JSON). Chúng được gửi đến mô hình để nó có thể gọi các tool. Chúng tính vào context ngay cả khi bạn không thấy chúng dưới dạng văn bản thuần túy.

`/context detail` phân tích các schema tool lớn nhất để bạn có thể thấy những gì chiếm ưu thế.
## Lệnh, chỉ thị và "phím tắt nội tuyến"

Các lệnh gạch chéo được xử lý bởi Gateway. Có một vài hành vi khác nhau:

- **Lệnh độc lập**: một tin nhắn chỉ `/...` chạy dưới dạng lệnh.
- **Chỉ thị**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` được loại bỏ trước khi mô hình nhìn thấy tin nhắn.
  - Các tin nhắn chỉ chứa chỉ thị duy trì cài đặt phiên.
  - Các chỉ thị nội tuyến trong một tin nhắn bình thường hoạt động như các gợi ý cho từng tin nhắn.
- **Phím tắt nội tuyến** (chỉ những người gửi được phép): các `/...` token nhất định bên trong một tin nhắn bình thường có thể chạy ngay lập tức (ví dụ: "hey /status"), và được loại bỏ trước khi mô hình nhìn thấy phần còn lại của văn bản.

Chi tiết: [Lệnh gạch chéo](/tools/slash-commands).
## Phiên, nén dữ liệu và cắt tỉa (những gì được lưu giữ)

Những gì được lưu giữ qua các tin nhắn phụ thuộc vào cơ chế:

- **Lịch sử bình thường** được lưu giữ trong bản ghi phiên cho đến khi được nén/cắt tỉa theo chính sách.
- **Nén dữ liệu** lưu giữ một bản tóm tắt vào bản ghi phiên và giữ nguyên các tin nhắn gần đây.
- **Cắt tỉa** loại bỏ các kết quả công cụ cũ khỏi prompt _trong bộ nhớ_ cho một lần chạy, nhưng không viết lại bản ghi phiên.

Tài liệu: [Phiên](/concepts/session), [Nén dữ liệu](/concepts/compaction), [Cắt tỉa phiên](/concepts/session-pruning).
## Những gì `/context` thực sự báo cáo

`/context` ưu tiên báo cáo lời nhắc hệ thống **run-built** mới nhất khi có sẵn:

- `System prompt (run)` = được thu thập từ lần chạy nhúng cuối cùng (có khả năng công cụ) và được lưu trữ trong kho phiên.
- `System prompt (estimate)` = được tính toán ngay lập tức khi không có báo cáo chạy (hoặc khi chạy qua backend CLI không tạo báo cáo).

Dù bằng cách nào, nó báo cáo kích thước và những người đóng góp hàng đầu; nó **không** đổ ra toàn bộ lời nhắc hệ thống hoặc lược đồ công cụ.