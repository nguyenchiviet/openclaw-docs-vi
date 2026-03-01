---
summary: >-
  Ghi nhật ký OpenClaw: tệp nhật ký chẩn đoán lăn cuộn + cờ bảo mật nhật ký
  thống nhất
read_when:
  - Ghi lại nhật ký macOS hoặc điều tra ghi nhật ký dữ liệu riêng tư
  - Gỡ lỗi các vấn đề về vòng đời phiên và kích hoạt bằng giọng nói
title: Ghi nhật ký macOS
x-i18n:
  source_path: platforms\mac\logging.md
  source_hash: c08d6bc012f8e8bb53353fe654713dede676b4e6127e49fd76e00c2510b9ab0b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:13.165Z'
---

# Ghi nhật ký (macOS)

## Tệp nhật ký chẩn đoán quay vòng (Ngăn Debug)

OpenClaw định tuyến nhật ký ứng dụng macOS thông qua swift-log (unified logging theo mặc định) và có thể ghi nhật ký tệp quay vòng cục bộ vào đĩa khi bạn cần một bản ghi bền vững.

- Mức chi tiết: **Ngăn Debug → Logs → App logging → Verbosity**
- Bật: **Ngăn Debug → Logs → App logging → "Write rolling diagnostics log (JSONL)"**
- Vị trí: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (quay vòng tự động; các tệp cũ được đặt hậu tố `.1`, `.2`, …)
- Xóa: **Ngăn Debug → Logs → App logging → "Clear"**

Ghi chú:

- Tính năng này **tắt theo mặc định**. Chỉ bật khi đang gỡ lỗi tích cực.
- Coi tệp là nhạy cảm; không chia sẻ mà không xem xét.

## Dữ liệu riêng tư unified logging trên macOS

Unified logging che giấu hầu hết các tải trọng trừ khi một subsystem chọn vào `privacy -off`. Theo bài viết của Peter về [logging privacy shenanigans](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025) trên macOS, điều này được kiểm soát bởi một plist trong `/Library/Preferences/Logging/Subsystems/` được khóa theo tên subsystem. Chỉ các mục nhập nhật ký mới mới nhận được cờ, vì vậy hãy bật nó trước khi tái tạo sự cố.

## Bật cho OpenClaw (`ai.openclaw`)

- Viết plist vào tệp tạm thời trước, sau đó cài đặt nó một cách nguyên tử dưới quyền root:

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

- No reboot is required; logd notices the file quickly, but only new log lines will include private payloads.
- View the richer output with the existing helper, e.g. `./scripts/clawlog.sh --category WebChat --last 5m`.

## Disable after debugging

- Remove the override: `sudo rm /Library/Preferences/Logging/Subsystems/ai.openclaw.plist`.
- Optionally run `sudo log config --reload` để buộc logd loại bỏ ghi đè ngay lập tức.
- Hãy nhớ rằng bề mặt này có thể bao gồm số điện thoại và nội dung tin nhắn; giữ plist tại chỗ chỉ khi bạn cần chi tiết bổ sung.