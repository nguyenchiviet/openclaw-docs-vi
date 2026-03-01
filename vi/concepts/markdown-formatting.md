---
summary: Đường ống định dạng Markdown cho các kênh gửi đi
read_when:
  - >-
    Bạn đang thay đổi định dạng markdown hoặc phân chia nội dung cho các kênh
    gửi đi
  - Bạn đang thêm một bộ định dạng kênh mới hoặc ánh xạ kiểu
  - Bạn đang gỡ lỗi các vấn đề về định dạng trên các kênh
title: Định dạng Markdown
x-i18n:
  source_path: concepts\markdown-formatting.md
  source_hash: f9cbf9b744f9a218860730f29435bcad02d3db80b1847fed5f17c063c97d4820
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:39:35.913Z'
---

# Định dạng Markdown

OpenClaw định dạng Markdown gửi đi bằng cách chuyển đổi nó thành một biểu diễn trung gian chung (IR) trước khi hiển thị đầu ra dành riêng cho kênh. IR giữ nguyên văn bản nguồn trong khi mang theo các khoảng kiểu/liên kết để chunking và rendering có thể giữ tính nhất quán trên các kênh.
## Mục tiêu

- **Tính nhất quán:** một bước phân tích cú pháp, nhiều trình kết xuất.
- **Phân chia an toàn:** chia nhỏ văn bản trước khi kết xuất để định dạng nội tuyến không bao giờ bị gãy trên các khối.
- **Phù hợp với kênh:** ánh xạ cùng một IR tới Slack mrkdwn, Telegram HTML và Signal style ranges mà không cần phân tích cú pháp Markdown lại.
## Pipeline

1. **Phân tích Markdown -> IR**
   - IR là văn bản thuần túy cộng với các khoảng kiểu (bold/italic/strike/code/spoiler) và khoảng liên kết.
   - Offsets là các đơn vị mã UTF-16 để các khoảng kiểu Signal căn chỉnh với API của nó.
   - Bảng chỉ được phân tích khi một kênh chọn vào chuyển đổi bảng.
2. **Chia IR (định dạng trước)**
   - Chia nhỏ xảy ra trên văn bản IR trước khi hiển thị.
   - Định dạng nội tuyến không chia tách giữa các khối; các khoảng được cắt lát theo từng khối.
3. **Hiển thị theo kênh**
   - **Slack:** mrkdwn tokens (bold/italic/strike/code), liên kết dưới dạng `<url|label>`.
   - **Telegram:** HTML tags (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   - **Signal:** văn bản thuần túy + `text-style` ranges; liên kết trở thành `label (url)` khi nhãn khác.
## Ví dụ IR

Input Markdown:

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (schematic):

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```
## Nơi nó được sử dụng

- Các adapter gửi đi của Slack, Telegram và Signal hiển thị từ IR.
- Các kênh khác (WhatsApp, iMessage, MS Teams, Discord) vẫn sử dụng văn bản thuần túy hoặc
  các quy tắc định dạng riêng của chúng, với chuyển đổi bảng Markdown được áp dụng trước
  khi chunking được bật.
## Xử lý bảng

Các bảng Markdown không được hỗ trợ nhất quán trên các ứng dụng chat. Sử dụng
`markdown.tables` để kiểm soát chuyển đổi cho mỗi kênh (và cho mỗi tài khoản).

- `code`: hiển thị bảng dưới dạng khối mã (mặc định cho hầu hết các kênh).
- `bullets`: chuyển đổi mỗi hàng thành các điểm dấu đầu dòng (mặc định cho Signal + WhatsApp).
- `off`: vô hiệu hóa phân tích bảng và chuyển đổi; văn bản bảng thô được truyền qua.

Các khóa cấu hình:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```
## Quy tắc chia nhỏ

- Giới hạn chia nhỏ đến từ các bộ điều hợp kênh/cấu hình và được áp dụng cho văn bản IR.
- Các khối mã được bảo toàn dưới dạng một khối duy nhất với dòng mới ở cuối để các kênh hiển thị chúng một cách chính xác.
- Các tiền tố danh sách và tiền tố trích dẫn khối là một phần của văn bản IR, vì vậy chia nhỏ không chia tách giữa tiền tố.
- Các kiểu nội tuyến (in đậm/in nghiêng/gạch ngang/mã nội tuyến/spoiler) không bao giờ được chia tách giữa các khối; trình kết xuất mở lại các kiểu bên trong mỗi khối.

Nếu bạn cần thêm thông tin về hành vi chia nhỏ trên các kênh, hãy xem [Truyền phát + chia nhỏ](/concepts/streaming).
## Chính sách liên kết

- **Slack:** `[label](url)` -> `<url|label>`; các URL trần giữ nguyên. Autolink
  bị vô hiệu hóa trong quá trình phân tích cú pháp để tránh liên kết kép.
- **Telegram:** `[label](url)` -> `<a href="url">label</a>` (chế độ phân tích HTML).
- **Signal:** `[label](url)` -> `label (url)` trừ khi nhãn khớp với URL.
## Spoiler

Các dấu spoiler (`||spoiler||`) chỉ được phân tích cú pháp cho Signal, nơi chúng ánh xạ tới các phạm vi kiểu SPOILER. Các kênh khác coi chúng là văn bản thuần túy.
## Cách thêm hoặc cập nhật trình định dạng kênh

1. **Parse once:** sử dụng helper `markdownToIR(...)` được chia sẻ với các tùy chọn phù hợp với kênh (autolink, heading style, blockquote prefix).
2. **Render:** triển khai renderer với `renderMarkdownWithMarkers(...)` và bản đồ style marker (hoặc Signal style ranges).
3. **Chunk:** gọi `chunkMarkdownIR(...)` trước khi render; render từng chunk.
4. **Wire adapter:** cập nhật channel outbound adapter để sử dụng chunker và renderer mới.
5. **Test:** thêm hoặc cập nhật các bài kiểm tra định dạng và bài kiểm tra outbound delivery nếu kênh sử dụng chunking.
## Những điều cần lưu ý

- Các token dấu ngoặc nhọn Slack (`<@U123>`, `<#C123>`, `<https://...>`) phải được
  bảo toàn; thoát HTML thô một cách an toàn.
- HTML Telegram yêu cầu thoát văn bản bên ngoài các thẻ để tránh markup bị hỏng.
- Các phạm vi kiểu Signal phụ thuộc vào độ lệch UTF-16; không sử dụng độ lệch điểm mã.
- Bảo toàn các dòng mới ở cuối các khối mã được bao quanh để các dấu đóng nằm trên
  dòng riêng của chúng.