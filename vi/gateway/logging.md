---
summary: 'Bề mặt ghi nhật ký, nhật ký tệp, kiểu nhật ký WS và định dạng bảng điều khiển'
read_when:
  - Thay đổi đầu ra hoặc định dạng ghi nhật ký
  - Gỡ lỗi đầu ra CLI hoặc Gateway
title: Ghi nhật ký
x-i18n:
  source_path: gateway\logging.md
  source_hash: efb8eda5e77e3809369a8ff569fac110323a86b3945797093f20e9bc98f39b2e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:57:39.527Z'
---

# Ghi nhật ký

Để xem tổng quan dành cho người dùng (CLI + Control UI + cấu hình), xem [/logging](/logging).

OpenClaw có hai "bề mặt" ghi nhật ký:

- **Đầu ra bảng điều khiển** (những gì bạn thấy trong terminal / Debug UI).
- **Nhật ký tệp** (JSON lines) được viết bởi logger của Gateway.
## Trình ghi nhật ký dựa trên tệp

- Tệp nhật ký cuộn mặc định nằm dưới `/tmp/openclaw/` (một tệp mỗi ngày): `openclaw-YYYY-MM-DD.log`
  - Ngày sử dụng múi giờ địa phương của máy chủ gateway.
- Đường dẫn tệp nhật ký và mức độ có thể được cấu hình thông qua `~/.openclaw/openclaw.json`:
  - `logging.file`
  - `logging.level`

Định dạng tệp là một đối tượng JSON trên mỗi dòng.

Tab Nhật ký của Control UI theo dõi tệp này thông qua gateway (`logs.tail`).
CLI có thể làm tương tự:

```bash
openclaw logs --follow
```

**Verbose vs. log levels**

- **File logs** are controlled exclusively by `logging.level`.
- `--verbose` only affects **console verbosity** (and WS log style); it does **not**
  raise the file log level.
- To capture verbose-only details in file logs, set `logging.level` to `debug` or
  `trace`.
## Ghi lại bảng điều khiển

CLI ghi lại `console.log/info/warn/error/debug/trace` và ghi chúng vào tệp nhật ký,
đồng thời vẫn in ra stdout/stderr.

Bạn có thể điều chỉnh mức chi tiết bảng điều khiển độc lập thông qua:

- `logging.consoleLevel` (mặc định `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)
## Che giấu tóm tắt công cụ

Các tóm tắt công cụ chi tiết (ví dụ: `🛠️ Exec: ...`) có thể che giấu các token nhạy cảm trước khi chúng được ghi vào luồng console. Đây là **chỉ dành cho công cụ** và không thay đổi nhật ký tệp.

- `logging.redactSensitive`: `off` | `tools` (mặc định: `tools`)
- `logging.redactPatterns`: mảng các chuỗi regex (ghi đè các giá trị mặc định)
  - Sử dụng các chuỗi regex thô (tự động `gi`), hoặc `/pattern/flags` nếu bạn cần các cờ tùy chỉnh.
  - Các kết quả khớp được che giấu bằng cách giữ 6 ký tự đầu + 4 ký tự cuối (độ dài >= 18), nếu không `***`.
  - Các giá trị mặc định bao gồm các phép gán khóa phổ biến, cờ CLI, trường JSON, tiêu đề bearer, khối PEM và các tiền tố token phổ biến.
## Gateway WebSocket logs

Gateway in ra các bản ghi giao thức WebSocket ở hai chế độ:

- **Chế độ bình thường (không `--verbose`)**: chỉ in các kết quả RPC "thú vị":
  - lỗi (`ok=false`)
  - cuộc gọi chậm (ngưỡng mặc định: `>= 50ms`)
  - lỗi phân tích cú pháp
- **Chế độ chi tiết (`--verbose`)**: in tất cả lưu lượng yêu cầu/phản hồi WS.

### Kiểu bản ghi WS

`openclaw gateway` hỗ trợ chuyển đổi kiểu cho từng gateway:

- `--ws-log auto` (mặc định): chế độ bình thường được tối ưu hóa; chế độ chi tiết sử dụng đầu ra nhỏ gọn
- `--ws-log compact`: đầu ra nhỏ gọn (yêu cầu/phản hồi được ghép nối) khi chi tiết
- `--ws-log full`: đầu ra đầy đủ theo khung hình khi chi tiết
- `--compact`: bí danh cho `--ws-log compact`

Ví dụ:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```
## Định dạng console (ghi nhật ký hệ thống con)

Trình định dạng console **nhận biết TTY** và in các dòng có tiền tố nhất quán.
Các trình ghi nhật ký hệ thống con giữ cho đầu ra được nhóm và dễ quét.

Hành vi:

- **Tiền tố hệ thống con** trên mỗi dòng (ví dụ: `[gateway]`, `[canvas]`, `[tailscale]`)
- **Màu hệ thống con** (ổn định cho mỗi hệ thống con) cộng với màu mức độ
- **Màu khi đầu ra là TTY hoặc môi trường trông giống như terminal phong phú** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), tôn trọng `NO_COLOR`
- **Tiền tố hệ thống con rút gọn**: bỏ `gateway/` + `channels/` ở đầu, giữ 2 phân đoạn cuối cùng (ví dụ: `whatsapp/outbound`)
- **Trình ghi nhật ký con theo hệ thống con** (tiền tố tự động + trường có cấu trúc `{ subsystem }`)
- **`logRaw()`** cho đầu ra QR/UX (không tiền tố, không định dạng)
- **Kiểu console** (ví dụ: `pretty | compact | json`)
- **Mức độ ghi nhật ký console** riêng biệt với mức độ ghi nhật ký tệp (tệp giữ toàn bộ chi tiết khi `logging.level` được đặt thành `debug`/`trace`)
- **Nội dung tin nhắn WhatsApp** được ghi nhật ký ở `debug` (sử dụng `--verbose` để xem chúng)

Điều này giữ cho các tệp nhật ký hiện có ổn định trong khi làm cho đầu ra tương tác dễ quét.