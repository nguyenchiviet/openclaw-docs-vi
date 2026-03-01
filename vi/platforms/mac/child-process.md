---
summary: Vòng đời Gateway trên macOS (launchd)
read_when:
  - Tích hợp ứng dụng Mac với vòng đời Gateway
title: Vòng đời Gateway
x-i18n:
  source_path: platforms\mac\child-process.md
  source_hash: 73e7eb64ef432c3bfc81b949a5cc2a344c64f2310b794228609aae1da817ec41
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:12:54.220Z'
---

# Vòng đời Gateway trên macOS

Ứng dụng macOS **quản lý Gateway thông qua launchd** theo mặc định và không tạo
Gateway như một tiến trình con. Trước tiên, nó cố gắng kết nối với một Gateway
đang chạy trên cổng được cấu hình; nếu không thể truy cập được, nó bật dịch vụ
launchd thông qua CLI `openclaw` bên ngoài (không có runtime nhúng). Điều này
cung cấp cho bạn khởi động tự động đáng tin cậy khi đăng nhập và khởi động lại
khi gặp sự cố.

Chế độ tiến trình con (Gateway được tạo trực tiếp bởi ứng dụng) **không được sử dụng**
ngày nay. Nếu bạn cần kết nối chặt chẽ hơn với giao diện người dùng, hãy chạy Gateway
theo cách thủ công trong một terminal.
## Hành vi mặc định (launchd)

- Ứng dụng cài đặt một LaunchAgent cho mỗi người dùng có nhãn `ai.openclaw.gateway`
  (hoặc `ai.openclaw.<profile>` khi sử dụng `--profile`/`OPENCLAW_PROFILE`; `com.openclaw.*` cũ được hỗ trợ).
- Khi bật chế độ Local, ứng dụng đảm bảo LaunchAgent được tải và
  khởi động Gateway nếu cần.
- Nhật ký được ghi vào đường dẫn nhật ký launchd gateway (hiển thị trong Cài đặt Gỡ lỗi).

Các lệnh phổ biến:

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Replace the label with `ai.openclaw.<profile>` khi chạy một hồ sơ được đặt tên.
## Bản dựng dev không ký

`scripts/restart-mac.sh --no-sign` dành cho các bản dựng cục bộ nhanh khi bạn không có
các khóa ký. Để ngăn launchd trỏ đến một tệp nhị phân relay không ký, nó:

- Ghi `~/.openclaw/disable-launchagent`.

Các lần chạy có ký của `scripts/restart-mac.sh` xóa ghi đè này nếu có
dấu hiệu hiện diện. Để đặt lại thủ công:

```bash
rm ~/.openclaw/disable-launchagent
```
## Chế độ chỉ đính kèm

Để buộc ứng dụng macOS **không bao giờ cài đặt hoặc quản lý launchd**, hãy khởi chạy nó với
`--attach-only` (hoặc `--no-launchd`). Điều này đặt `~/.openclaw/disable-launchagent`,
vì vậy ứng dụng chỉ đính kèm vào một Gateway đang chạy. Bạn có thể chuyển đổi cùng một
hành vi trong Debug Settings.
## Chế độ từ xa

Chế độ từ xa không bao giờ khởi động một Gateway cục bộ. Ứng dụng sử dụng một đường hầm SSH đến máy chủ từ xa và kết nối qua đường hầm đó.
## Tại sao chúng tôi ưa thích launchd

- Tự động khởi động khi đăng nhập.
- Ngữ nghĩa khởi động lại/KeepAlive tích hợp sẵn.
- Nhật ký và giám sát có thể dự đoán được.

Nếu chế độ tiến trình con thực sự cần thiết lại, nó nên được ghi lại như một
chế độ riêng biệt, rõ ràng chỉ dành cho nhà phát triển.