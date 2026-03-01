---
summary: Sử dụng API thống nhất của Qianfan để truy cập nhiều mô hình trong OpenClaw
read_when:
  - Bạn muốn một khóa API duy nhất cho nhiều LLM
  - Bạn cần hướng dẫn thiết lập Baidu Qianfan
title: Qianfan
x-i18n:
  source_path: providers\qianfan.md
  source_hash: 2ca710b422f190b65d23db51a3219f0abd67074fb385251efeca6eae095d02e0
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:16:47.059Z'
---

# Hướng dẫn Nhà cung cấp Qianfan

Qianfan là nền tảng MaaS của Baidu, cung cấp một **API thống nhất** định tuyến các yêu cầu đến nhiều mô hình phía sau một điểm cuối duy nhất và khóa API. Nó tương thích với OpenAI, vì vậy hầu hết các SDK OpenAI hoạt động bằng cách chuyển đổi URL cơ sở.

## Điều kiện tiên quyết

1. Tài khoản Baidu Cloud với quyền truy cập API Qianfan
2. Khóa API từ bảng điều khiển Qianfan
3. OpenClaw được cài đặt trên hệ thống của bạn

## Lấy khóa API của bạn

1. Truy cập [Bảng điều khiển Qianfan](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2. Tạo ứng dụng mới hoặc chọn ứng dụng hiện có
3. Tạo khóa API (định dạng: `bce-v3/ALTAK-...`)
4. Sao chép khóa API để sử dụng với OpenClaw

## Thiết lập CLI

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## Tài liệu liên quan

- [Cấu hình OpenClaw](/gateway/configuration)
- [Nhà cung cấp Mô hình](/concepts/model-providers)
- [Thiết lập Agent](/concepts/agent)
- [Tài liệu API Qianfan](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)