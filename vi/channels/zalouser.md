---
summary: 'Hỗ trợ tài khoản cá nhân Zalo qua zca-cli (đăng nhập QR), khả năng và cấu hình'
read_when:
  - Thiết lập Zalo Personal cho OpenClaw
  - Gỡ lỗi luồng đăng nhập hoặc tin nhắn Zalo Personal
title: Zalo Personal
x-i18n:
  source_path: channels\zalouser.md
  source_hash: ede847ebe62722568f8d24d0257986abaad539ecef96183ab0e83bbe6e6dc078
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:34:14.489Z'
---

# Zalo Personal (không chính thức)

Trạng thái: thử nghiệm. Tích hợp này tự động hóa một **tài khoản Zalo cá nhân** thông qua `zca-cli`.

> **Cảnh báo:** Đây là tích hợp không chính thức và có thể dẫn đến tạm khóa/cấm tài khoản. Sử dụng tại rủi ro của bạn.
## Plugin bắt buộc

Zalo Personal được cung cấp dưới dạng plugin và không được đi kèm với bản cài đặt cốt lõi.

- Cài đặt qua CLI: `openclaw plugins install @openclaw/zalouser`
- Hoặc từ một bản sao nguồn: `openclaw plugins install ./extensions/zalouser`
- Chi tiết: [Plugins](/tools/plugin)
## Điều kiện tiên quyết: zca-cli

Máy Gateway phải có tệp nhị phân `zca` có sẵn trong `PATH`.

- Xác minh: `zca --version`
- Nếu thiếu, cài đặt zca-cli (xem `extensions/zalouser/README.md` hoặc tài liệu zca-cli chính thức).
## Bắt đầu nhanh (người mới bắt đầu)

1. Cài đặt plugin (xem ở trên).
2. Đăng nhập (QR, trên máy Gateway):
   - `openclaw channels login --channel zalouser`
   - Quét mã QR trong terminal bằng ứng dụng di động Zalo.
3. Bật kênh:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4. Khởi động lại Gateway (hoặc hoàn thành thiết lập ban đầu).
5. Truy cập tin nhắn riêng mặc định là ghép nối; phê duyệt mã ghép nối khi liên hệ lần đầu.
## Nó là gì

- Sử dụng `zca listen` để nhận tin nhắn đến.
- Sử dụng `zca msg ...` để gửi trả lời (văn bản/phương tiện/liên kết).
- Được thiết kế cho các trường hợp sử dụng "tài khoản cá nhân" khi Zalo Bot API không khả dụng.
## Đặt tên

Channel id là `zalouser` để làm rõ ràng rằng điều này tự động hóa một **tài khoản người dùng Zalo cá nhân** (không chính thức). Chúng tôi giữ `zalo` dành riêng cho một tích hợp API Zalo chính thức tiềm năng trong tương lai.
## Tìm ID (thư mục)

Sử dụng CLI thư mục để khám phá các peer/nhóm và ID của chúng:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```
## Giới hạn

- Văn bản gửi đi được chia thành các khối ~2000 ký tự (giới hạn của ứng dụng khách Zalo).
- Truyền phát theo khối bị chặn theo mặc định.
## Kiểm soát truy cập (Tin nhắn riêng)

`channels.zalouser.dmPolicy` hỗ trợ: `pairing | allowlist | open | disabled` (mặc định: `pairing`).
`channels.zalouser.allowFrom` chấp nhận ID hoặc tên người dùng. Trình hướng dẫn phân giải tên thành ID thông qua `zca friend find` khi có sẵn.

Phê duyệt thông qua:

- `openclaw pairing list zalouser`
- `openclaw pairing approve zalouser <code>`
## Truy cập nhóm (tùy chọn)

- Mặc định: `channels.zalouser.groupPolicy = "open"` (các nhóm được phép). Sử dụng `channels.defaults.groupPolicy` để ghi đè mặc định khi không được đặt.
- Hạn chế danh sách cho phép với:
  - `channels.zalouser.groupPolicy = "allowlist"`
  - `channels.zalouser.groups` (các khóa là ID hoặc tên nhóm)
- Chặn tất cả các nhóm: `channels.zalouser.groupPolicy = "disabled"`.
- Trình hướng dẫn cấu hình có thể nhắc nhở danh sách cho phép nhóm.
- Khi khởi động, OpenClaw phân giải tên nhóm/người dùng trong danh sách cho phép thành ID và ghi lại ánh xạ; các mục không được phân giải được giữ nguyên như đã nhập.

Ví dụ:

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```
## Nhiều tài khoản

Các tài khoản ánh xạ tới các hồ sơ zca. Ví dụ:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```
## Khắc phục sự cố

**`zca` không tìm thấy:**

- Cài đặt zca-cli và đảm bảo nó nằm trên `PATH` cho quy trình Gateway.

**Đăng nhập không được lưu:**

- `openclaw channels status --probe`
- Đăng nhập lại: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`