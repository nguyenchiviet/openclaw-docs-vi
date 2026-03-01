---
summary: Yêu cầu về quyền hạn macOS (TCC) và ký số
read_when:
  - Gỡ lỗi các lời nhắc quyền macOS bị thiếu hoặc bị treo
  - Đóng gói hoặc ký ứng dụng macOS
  - Thay đổi ID gói hoặc đường dẫn cài đặt ứng dụng
title: Quyền truy cập trên macOS
x-i18n:
  source_path: platforms\mac\permissions.md
  source_hash: 250065b964c98c307a075ab9e23bf798f9d247f27befe2e5f271ffef1f497def
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:12.666Z'
---

# Quyền macOS (TCC)

Các cấp quyền macOS rất dễ bị ảnh hưởng. TCC liên kết một cấp quyền với chữ ký mã, định danh gói và đường dẫn trên đĩa của ứng dụng. Nếu bất kỳ điều nào trong số đó thay đổi, macOS sẽ coi ứng dụng là mới và có thể xóa hoặc ẩn các lời nhắc.

## Yêu cầu để có quyền ổn định

- Cùng đường dẫn: chạy ứng dụng từ một vị trí cố định (đối với OpenClaw, `dist/OpenClaw.app`).
- Cùng định danh gói: thay đổi ID gói sẽ tạo một danh tính quyền mới.
- Ứng dụng được ký: các bản dựng không được ký hoặc được ký ad-hoc không duy trì quyền.
- Chữ ký nhất quán: sử dụng chứng chỉ Apple Development hoặc Developer ID thực để chữ ký vẫn ổn định trên các lần xây dựng lại.

Các chữ ký ad-hoc tạo ra một danh tính mới mỗi lần xây dựng. macOS sẽ quên các cấp quyền trước đó và các lời nhắc có thể biến mất hoàn toàn cho đến khi các mục cũ được xóa.

## Danh sách kiểm tra khôi phục khi các lời nhắc biến mất

1. Thoát ứng dụng.
2. Xóa mục ứng dụng trong System Settings -> Privacy & Security.
3. Khởi chạy lại ứng dụng từ cùng đường dẫn và cấp lại quyền.
4. Nếu lời nhắc vẫn không xuất hiện, hãy đặt lại các mục TCC bằng `tccutil` và thử lại.
5. Một số quyền chỉ xuất hiện lại sau khi khởi động lại macOS hoàn toàn.

Ví dụ về đặt lại (thay thế ID gói khi cần):

```bash
sudo tccutil reset Accessibility ai.openclaw.mac
sudo tccutil reset ScreenCapture ai.openclaw.mac
sudo tccutil reset AppleEvents
```

## Files and folders permissions (Desktop/Documents/Downloads)

macOS may also gate Desktop, Documents, and Downloads for terminal/background processes. If file reads or directory listings hang, grant access to the same process context that performs file operations (for example Terminal/iTerm, LaunchAgent-launched app, or SSH process).

Workaround: move files into the OpenClaw workspace (`~/.openclaw/workspace`) nếu bạn muốn tránh các cấp quyền theo từng thư mục.

Nếu bạn đang kiểm tra quyền, luôn ký bằng chứng chỉ thực. Các bản dựng ad-hoc chỉ chấp nhận được cho các lần chạy cục bộ nhanh chóng nơi quyền không quan trọng.