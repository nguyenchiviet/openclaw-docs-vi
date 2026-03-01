---
summary: Đăng nhập thủ công cho tự động hóa trình duyệt + đăng bài X/Twitter
read_when:
  - Bạn cần đăng nhập vào các trang web để tự động hóa trình duyệt
  - Bạn muốn đăng cập nhật lên X/Twitter
title: Đăng nhập Trình duyệt
x-i18n:
  source_path: tools\browser-login.md
  source_hash: c30faa9da6c6ef70ab8ee7dd9835572d8b16efd3ac3b99c2f55d25f798564ee9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:27:56.783Z'
---

# Đăng nhập trình duyệt + đăng bài X/Twitter

## Đăng nhập thủ công (được khuyến nghị)

Khi một trang web yêu cầu đăng nhập, **hãy đăng nhập thủ công** trong hồ sơ trình duyệt **chính** (trình duyệt openclaw).

**Không** cung cấp thông tin xác thực của bạn cho mô hình. Các lần đăng nhập tự động thường kích hoạt các biện pháp phòng chống bot và có thể khóa tài khoản.

Quay lại tài liệu trình duyệt chính: [Browser](/tools/browser).
## Hồ sơ Chrome nào được sử dụng?

OpenClaw kiểm soát một **hồ sơ Chrome chuyên dụng** (được đặt tên là `openclaw`, giao diện tinted cam). Điều này tách biệt với hồ sơ trình duyệt hàng ngày của bạn.

Hai cách dễ dàng để truy cập nó:

1. **Yêu cầu agent mở trình duyệt** và sau đó tự đăng nhập.
2. **Mở nó qua CLI**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

If you have multiple profiles, pass `--browser-profile <name>` (the default is `openclaw`).
## X/Twitter: luồng được khuyến nghị

- **Đọc/tìm kiếm/luồng:** sử dụng **trình duyệt máy chủ** (đăng nhập thủ công).
- **Đăng cập nhật:** sử dụng **trình duyệt máy chủ** (đăng nhập thủ công).
## Sandboxing + truy cập trình duyệt máy chủ

Các phiên trình duyệt trong sandbox **có khả năng cao hơn** kích hoạt phát hiện bot. Đối với X/Twitter (và các trang web nghiêm ngặt khác), hãy ưu tiên trình duyệt **máy chủ**.

Nếu agent được sandboxing, công cụ trình duyệt mặc định sử dụng sandbox. Để cho phép kiểm soát máy chủ:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Then target the host browser:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Hoặc vô hiệu hóa sandboxing cho agent đăng bài cập nhật.