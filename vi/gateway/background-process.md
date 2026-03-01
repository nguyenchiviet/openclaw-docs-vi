---
summary: Thực thi nền và quản lý tiến trình
read_when:
  - Thêm hoặc sửa đổi hành vi thực thi nền
  - Gỡ lỗi các tác vụ exec chạy lâu dài
title: Công cụ Thực thi Nền và Xử lý
x-i18n:
  source_path: gateway\background-process.md
  source_hash: ca7b62675af8628e5a691570410e380c65b453d1e39103a6cb8178df593f1033
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:56:09.319Z'
---

# Công cụ Exec nền + Process

OpenClaw chạy các lệnh shell thông qua công cụ `exec` và giữ các tác vụ chạy lâu dài trong bộ nhớ. Công cụ `process` quản lý các phiên nền đó.
## exec tool

Các tham số chính:

- `command` (bắt buộc)
- `yieldMs` (mặc định 10000): tự động chạy nền sau độ trễ này
- `background` (bool): chạy nền ngay lập tức
- `timeout` (giây, mặc định 1800): kết thúc quy trình sau thời gian chờ này
- `elevated` (bool): chạy trên máy chủ nếu chế độ nâng cao được bật/cho phép
- Cần TTY thực? Đặt `pty: true`.
- `workdir`, `env`

Hành vi:

- Các lần chạy ở chế độ nền trả về kết quả trực tiếp.
- Khi chạy nền (rõ ràng hoặc hết thời gian chờ), công cụ trả về `status: "running"` + `sessionId` và một phần đuôi ngắn.
- Kết quả được giữ trong bộ nhớ cho đến khi phiên được kiểm tra hoặc xóa.
- Nếu công cụ `process` không được phép, `exec` chạy đồng bộ và bỏ qua `yieldMs`/`background`.
## Kết nối tiến trình con

Khi tạo các tiến trình con chạy lâu dài bên ngoài các công cụ exec/process (ví dụ: CLI respawns hoặc gateway helpers), hãy đính kèm trình hỗ trợ kết nối tiến trình con để các tín hiệu kết thúc được chuyển tiếp và các listener được tách rời khi thoát/lỗi. Điều này tránh các tiến trình mồ côi trên systemd và giữ cho hành vi tắt máy nhất quán trên các nền tảng.

Ghi đè biến môi trường:

- `PI_BASH_YIELD_MS`: yield mặc định (ms)
- `PI_BASH_MAX_OUTPUT_CHARS`: giới hạn đầu ra trong bộ nhớ (ký tự)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: giới hạn stdout/stderr đang chờ xử lý trên mỗi luồng (ký tự)
- `PI_BASH_JOB_TTL_MS`: TTL cho các phiên đã hoàn thành (ms, giới hạn từ 1m–3h)

Cấu hình (ưu tiên):

- `tools.exec.backgroundMs` (mặc định 10000)
- `tools.exec.timeoutSec` (mặc định 1800)
- `tools.exec.cleanupMs` (mặc định 1800000)
- `tools.exec.notifyOnExit` (mặc định true): xếp hàng một sự kiện hệ thống + yêu cầu nhịp tim khi một exec được chạy nền thoát.
- `tools.exec.notifyOnExitEmptySuccess` (mặc định false): khi true, cũng xếp hàng các sự kiện hoàn thành cho các lần chạy backgrounded thành công không tạo ra đầu ra.
## công cụ process

Hành động:

- `list`: phiên đang chạy + đã hoàn thành
- `poll`: lấy đầu ra mới cho một phiên (cũng báo cáo trạng thái thoát)
- `log`: đọc đầu ra tổng hợp (hỗ trợ `offset` + `limit`)
- `write`: gửi stdin (`data`, `eof` tùy chọn)
- `kill`: kết thúc một phiên chạy nền
- `clear`: xóa một phiên đã hoàn thành khỏi bộ nhớ
- `remove`: kết thúc nếu đang chạy, nếu không xóa nếu đã hoàn thành

Ghi chú:

- Chỉ các phiên chạy nền được liệt kê/lưu trữ trong bộ nhớ.
- Các phiên bị mất khi khởi động lại process (không có tính bền vững trên đĩa).
- Nhật ký phiên chỉ được lưu vào lịch sử trò chuyện nếu bạn chạy `process poll/log` và kết quả công cụ được ghi lại.
- `process` được phạm vi cho mỗi agent; nó chỉ thấy các phiên được bắt đầu bởi agent đó.
- `process list` bao gồm một `name` dẫn xuất (động từ lệnh + mục tiêu) để quét nhanh.
- `process log` sử dụng `offset`/`limit` dựa trên dòng.
- Khi cả `offset` và `limit` bị bỏ qua, nó trả về 200 dòng cuối cùng và bao gồm gợi ý phân trang.
- Khi `offset` được cung cấp và `limit` bị bỏ qua, nó trả về từ `offset` đến cuối (không giới hạn ở 200).
## Ví dụ

Chạy một tác vụ dài và kiểm tra sau:

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

Start immediately in background:

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

Send stdin:

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```