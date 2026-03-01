---
summary: Tổng quan về các tùy chọn và quy trình onboarding của OpenClaw
read_when:
  - Chọn một đường dẫn onboarding
  - Thiết lập một môi trường mới
title: Tổng Quan Về Onboarding
sidebarTitle: Onboarding Overview
x-i18n:
  source_path: start\onboarding-overview.md
  source_hash: 64540138b717f4a4c1201868220d755a21b16fa330c558c33beb426cfa4504d0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:24:47.743Z'
---

# Thiết lập ban đầu

OpenClaw hỗ trợ nhiều đường dẫn thiết lập ban đầu tùy thuộc vào nơi Gateway chạy
và cách bạn muốn cấu hình các nhà cung cấp.

## Chọn đường dẫn thiết lập ban đầu của bạn

- **Trình hướng dẫn CLI** cho macOS, Linux và Windows (qua WSL2).
- **Ứng dụng macOS** để thiết lập hướng dẫn lần đầu trên Apple silicon hoặc Intel Macs.

## Trình hướng dẫn thiết lập ban đầu CLI

Chạy trình hướng dẫn trong terminal:

```bash
openclaw onboard
```

Use the CLI wizard when you want full control of the Gateway, workspace,
channels, and skills. Docs:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## Thiết lập ban đầu ứng dụng macOS

Sử dụng ứng dụng OpenClaw khi bạn muốn thiết lập hướng dẫn đầy đủ trên macOS. Tài liệu:

- [Thiết lập ban đầu (/start/onboarding)

## Nhà cung cấp tùy chỉnh

Nếu bạn cần một endpoint không được liệt kê, bao gồm các nhà cung cấp được lưu trữ
công khai các API tương thích OpenAI hoặc Anthropic, hãy chọn **Nhà cung cấp tùy chỉnh** trong
trình hướng dẫn CLI. Bạn sẽ được yêu cầu:

- Chọn tương thích OpenAI, tương thích Anthropic, hoặc **Không xác định** (tự động phát hiện).
- Nhập URL cơ sở và khóa API (nếu nhà cung cấp yêu cầu).
- Cung cấp ID mô hình và bí danh tùy chọn.
- Chọn ID Endpoint để nhiều endpoint tùy chỉnh có thể cùng tồn tại.

Để biết các bước chi tiết, hãy làm theo tài liệu thiết lập ban đầu CLI ở trên.