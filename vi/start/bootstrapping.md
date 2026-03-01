---
summary: Nghi thức khởi động Agent để khởi tạo các tệp không gian làm việc và danh tính
read_when:
  - Hiểu điều gì xảy ra trong lần chạy agent đầu tiên
  - Giải thích nơi các tệp bootstrapping được lưu trữ
  - Gỡ lỗi thiết lập danh tính onboarding
title: Khởi động Agent
sidebarTitle: Bootstrapping
x-i18n:
  source_path: start\bootstrapping.md
  source_hash: 4a08b5102f25c6c4bcdbbdd44384252a9e537b245a7b070c4961a72b4c6c6601
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:24:14.923Z'
---

# Khởi động Agent

Khởi động là **nghi thức chạy lần đầu** chuẩn bị không gian làm việc của agent và
thu thập chi tiết nhận dạng. Nó xảy ra sau thiết lập ban đầu, khi agent khởi động
lần đầu tiên.

## Khởi động làm gì

Khi chạy agent lần đầu, OpenClaw khởi động không gian làm việc (mặc định
`~/.openclaw/workspace`):

- Khởi tạo `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
- Chạy nghi thức hỏi đáp ngắn (một câu hỏi tại một thời điểm).
- Ghi nhận dạng + tùy chọn vào `IDENTITY.md`, `USER.md`, `SOUL.md`.
- Xóa `BOOTSTRAP.md` khi hoàn thành để nó chỉ chạy một lần.

## Nơi nó chạy

Khởi động luôn chạy trên **máy chủ gateway**. Nếu ứng dụng macOS kết nối với
một Gateway từ xa, không gian làm việc và các tệp khởi động sẽ nằm trên máy từ xa đó.

<Note>
Khi Gateway chạy trên một máy khác, hãy chỉnh sửa các tệp không gian làm việc trên
máy chủ gateway (ví dụ: `user@gateway-host:~/.openclaw/workspace`).
</Note>

## Tài liệu liên quan

- Thiết lập ban đầu ứng dụng macOS: [Thiết lập ban đầu](/start/onboarding)
- Bố cục không gian làm việc: [Không gian làm việc của agent](/concepts/agent-workspace)