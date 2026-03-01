---
summary: 'Terminal UI (TUI): kết nối đến Gateway từ bất kỳ máy nào'
read_when:
  - Bạn muốn một hướng dẫn thân thiện với người mới bắt đầu về TUI
  - 'Bạn cần danh sách đầy đủ các tính năng TUI, lệnh và phím tắt'
title: Giao diện người dùng văn bản
x-i18n:
  source_path: web\tui.md
  source_hash: 6ab8174870e4722d76af61915b9bb020dc6df1ddacc406e2f5a80416b6e7f904
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:35:05.852Z'
---

# TUI (Terminal UI)

## Bắt đầu nhanh

1. Khởi động Gateway.

```bash
openclaw gateway
```

2. Open the TUI.

```bash
openclaw tui
```

3. Type a message and press Enter.

Remote Gateway:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Use `--password` nếu Gateway của bạn sử dụng xác thực mật khẩu.
## Những gì bạn thấy

- Header: URL kết nối, agent hiện tại, phiên hiện tại.
- Chat log: tin nhắn người dùng, phản hồi của trợ lý, thông báo hệ thống, thẻ công cụ.
- Status line: trạng thái kết nối/chạy (đang kết nối, đang chạy, đang truyền phát, không hoạt động, lỗi).
- Footer: trạng thái kết nối + agent + phiên + mô hình + think/verbose/reasoning + số lượng token + deliver.
- Input: trình soạn thảo văn bản với tự động hoàn thành.
## Mô hình tư duy: agent + phiên

- Agent là các slug duy nhất (ví dụ: `main`, `research`). Gateway hiển thị danh sách.
- Phiên thuộc về agent hiện tại.
- Khóa phiên được lưu trữ dưới dạng `agent:<agentId>:<sessionKey>`.
  - Nếu bạn nhập `/session main`, TUI sẽ mở rộng nó thành `agent:<currentAgent>:main`.
  - Nếu bạn nhập `/session agent:other:main`, bạn chuyển sang phiên agent đó một cách rõ ràng.
- Phạm vi phiên:
  - `per-sender` (mặc định): mỗi agent có nhiều phiên.
  - `global`: TUI luôn sử dụng phiên `global` (bộ chọn có thể trống).
- Agent hiện tại + phiên luôn hiển thị trong chân trang.
## Gửi + giao hàng

- Tin nhắn được gửi đến Gateway; giao hàng cho các nhà cung cấp được tắt theo mặc định.
- Bật giao hàng:
  - `/deliver on`
  - hoặc bảng điều khiển Cài đặt
  - hoặc bắt đầu với `openclaw tui --deliver`
## Bộ chọn + lớp phủ

- Bộ chọn mô hình: liệt kê các mô hình có sẵn và đặt ghi đè phiên.
- Bộ chọn agent: chọn một agent khác.
- Bộ chọn phiên: chỉ hiển thị các phiên cho agent hiện tại.
- Cài đặt: bật tắt giao hàng, mở rộng đầu ra công cụ và khả năng hiển thị suy nghĩ.
## Phím tắt bàn phím

- Enter: gửi tin nhắn
- Esc: hủy bỏ lần chạy đang hoạt động
- Ctrl+C: xóa đầu vào (nhấn hai lần để thoát)
- Ctrl+D: thoát
- Ctrl+L: bộ chọn mô hình
- Ctrl+G: bộ chọn agent
- Ctrl+P: bộ chọn phiên
- Ctrl+O: bật/tắt mở rộng đầu ra công cụ
- Ctrl+T: bật/tắt hiển thị suy nghĩ (tải lại lịch sử)
## Lệnh gạch chéo

Cốt lõi:

- `/help`
- `/status`
- `/agent <id>` (hoặc `/agents`)
- `/session <key>` (hoặc `/sessions`)
- `/model <provider/model>` (hoặc `/models`)

Điều khiển phiên:

- `/think <off|minimal|low|medium|high>`
- `/verbose <on|full|off>`
- `/reasoning <on|off|stream>`
- `/usage <off|tokens|full>`
- `/elevated <on|off|ask|full>` (bí danh: `/elev`)
- `/activation <mention|always>`
- `/deliver <on|off>`

Vòng đời phiên:

- `/new` hoặc `/reset` (đặt lại phiên)
- `/abort` (hủy bỏ lần chạy đang hoạt động)
- `/settings`
- `/exit`

Các lệnh gạch chéo Gateway khác (ví dụ: `/context`) được chuyển tiếp đến Gateway và hiển thị dưới dạng đầu ra hệ thống. Xem [Lệnh gạch chéo](/tools/slash-commands).
## Lệnh shell cục bộ

- Thêm tiền tố `!` vào một dòng để chạy lệnh shell cục bộ trên máy chủ TUI.
- TUI nhắc một lần mỗi phiên để cho phép thực thi cục bộ; từ chối sẽ giữ `!` bị vô hiệu hóa trong suốt phiên.
- Các lệnh chạy trong một shell mới, không tương tác trong thư mục làm việc TUI (không có `cd`/env liên tục).
- Một `!` đơn lẻ được gửi dưới dạng tin nhắn bình thường; các khoảng trắng ở đầu không kích hoạt thực thi cục bộ.
## Kết quả công cụ

- Các lệnh gọi công cụ hiển thị dưới dạng thẻ với đối số + kết quả.
- Ctrl+O chuyển đổi giữa chế độ thu gọn/mở rộng.
- Khi các công cụ chạy, các bản cập nhật một phần được truyền phát vào cùng một thẻ.
## Lịch sử + truyền phát

- Khi kết nối, TUI tải lịch sử mới nhất (mặc định 200 tin nhắn).
- Các phản hồi truyền phát cập nhật tại chỗ cho đến khi hoàn tất.
- TUI cũng lắng nghe các sự kiện công cụ agent để có các thẻ công cụ phong phú hơn.
## Chi tiết kết nối

- TUI đăng ký với Gateway dưới dạng `mode: "tui"`.
- Kết nối lại hiển thị thông báo hệ thống; các khoảng trống sự kiện được hiển thị trong nhật ký.
## Tùy chọn

- `--url <url>`: URL WebSocket của Gateway (mặc định từ cấu hình hoặc `ws://127.0.0.1:<port>`)
- `--token <token>`: Token của Gateway (nếu cần)
- `--password <password>`: Mật khẩu của Gateway (nếu cần)
- `--session <key>`: Khóa phiên (mặc định: `main`, hoặc `global` khi phạm vi là toàn cục)
- `--deliver`: Gửi phản hồi của trợ lý đến nhà cung cấp (mặc định tắt)
- `--thinking <level>`: Ghi đè mức độ suy nghĩ cho các lần gửi
- `--timeout-ms <ms>`: Thời gian chờ của agent tính bằng ms (mặc định `agents.defaults.timeoutSeconds`)

Lưu ý: khi bạn đặt `--url`, TUI không quay lại cấu hình hoặc thông tin xác thực môi trường.
Chuyển `--token` hoặc `--password` một cách rõ ràng. Thiếu thông tin xác thực rõ ràng là một lỗi.
## Khắc phục sự cố

Không có đầu ra sau khi gửi tin nhắn:

- Chạy `/status` trong TUI để xác nhận Gateway được kết nối và ở trạng thái nhàn rỗi/bận.
- Kiểm tra nhật ký Gateway: `openclaw logs --follow`.
- Xác nhận agent có thể chạy: `openclaw status` và `openclaw models status`.
- Nếu bạn mong đợi tin nhắn trong kênh trò chuyện, hãy bật giao hàng (`/deliver on` hoặc `--deliver`).
- `--history-limit <n>`: Các mục lịch sử cần tải (mặc định 200)
## Khắc phục sự cố kết nối

- `disconnected`: đảm bảo Gateway đang chạy và `--url/--token/--password` của bạn là chính xác.
- Không có agent trong bộ chọn: kiểm tra `openclaw agents list` và cấu hình định tuyến của bạn.
- Bộ chọn phiên trống: bạn có thể đang ở phạm vi toàn cầu hoặc chưa có phiên nào.