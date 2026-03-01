---
summary: Theo dõi sử dụng và yêu cầu thông tin xác thực
read_when:
  - Bạn đang kết nối các bề mặt sử dụng/hạn ngạch của nhà cung cấp
  - Bạn cần giải thích hành vi theo dõi sử dụng hoặc yêu cầu xác thực
title: Theo dõi Sử dụng
x-i18n:
  source_path: concepts\usage-tracking.md
  source_hash: 6f6ed2a70329b2a6206c327aa749a84fbfe979762caca5f0e7fb556f91631cbb
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:25.733Z'
---

# Theo dõi sử dụng

## Nó là gì

- Kéo dữ liệu sử dụng/hạn ngạch nhà cung cấp trực tiếp từ các endpoint sử dụng của họ.
- Không có chi phí ước tính; chỉ các cửa sổ được nhà cung cấp báo cáo.

## Nơi nó xuất hiện

- `/status` trong các cuộc trò chuyện: thẻ trạng thái giàu emoji với các token phiên + chi phí ước tính (chỉ khóa API). Sử dụng nhà cung cấp hiển thị cho **nhà cung cấp mô hình hiện tại** khi có sẵn.
- `/usage off|tokens|full` trong các cuộc trò chuyện: chân trang sử dụng cho mỗi phản hồi (OAuth chỉ hiển thị token).
- `/usage cost` trong các cuộc trò chuyện: tóm tắt chi phí cục bộ được tổng hợp từ nhật ký phiên OpenClaw.
- CLI: `openclaw status --usage` in ra bảng phân tích đầy đủ theo nhà cung cấp.
- CLI: `openclaw channels list` in ra cùng một ảnh chụp sử dụng cùng với cấu hình nhà cung cấp (sử dụng `--no-usage` để bỏ qua).
- Thanh menu macOS: phần "Usage" dưới Context (chỉ nếu có sẵn).

## Nhà cung cấp + thông tin xác thực

- **Anthropic (Claude)**: Token OAuth trong hồ sơ xác thực.
- **GitHub Copilot**: Token OAuth trong hồ sơ xác thực.
- **Gemini CLI**: Token OAuth trong hồ sơ xác thực.
- **Antigravity**: Token OAuth trong hồ sơ xác thực.
- **OpenAI Codex**: Token OAuth trong hồ sơ xác thực (accountId được sử dụng khi có).
- **MiniMax**: Khóa API (khóa gói mã hóa; `MINIMAX_CODE_PLAN_KEY` hoặc `MINIMAX_API_KEY`); sử dụng cửa sổ gói mã hóa 5 giờ.
- **z.ai**: Khóa API qua env/config/auth store.

Sử dụng bị ẩn nếu không có thông tin xác thực OAuth/API phù hợp.