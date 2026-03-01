---
summary: Các bước ký cho bản dựng gỡ lỗi macOS được tạo bởi các tập lệnh đóng gói
read_when:
  - Xây dựng hoặc ký các bản dựng gỡ lỗi Mac
title: Ký trên macOS
x-i18n:
  source_path: platforms\mac\signing.md
  source_hash: 403b92f9a0ecdb7cb42ec097c684b7a696be3696d6eece747314a4dc90d8797e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:33.305Z'
---

# Ký macOS (bản dựng gỡ lỗi)

Ứng dụng này thường được xây dựng từ [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), hiện tại:

- đặt định danh gói gỡ lỗi ổn định: `ai.openclaw.mac.debug`
- ghi Info.plist với định danh gói đó (ghi đè qua `BUNDLE_ID=...`)
- gọi [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) để ký tệp nhị phân chính và gói ứng dụng để macOS coi mỗi lần xây dựng lại là cùng một gói đã ký và giữ lại quyền TCC (thông báo, khả năng truy cập, ghi hình màn hình, micrô, chuyển đổi giọng nói). Để có quyền ổn định, hãy sử dụng định danh ký thực; ad-hoc là tùy chọn và dễ vỡ (xem [quyền macOS](/platforms/mac/permissions)).
- sử dụng `CODESIGN_TIMESTAMP=auto` theo mặc định; nó cho phép dấu thời gian đáng tin cậy cho chữ ký Developer ID. Đặt `CODESIGN_TIMESTAMP=off` để bỏ qua dấu thời gian (bản dựng gỡ lỗi ngoại tuyến).
- tiêm siêu dữ liệu xây dựng vào Info.plist: `OpenClawBuildTimestamp` (UTC) và `OpenClawGitCommit` (hash ngắn) để ngăn About có thể hiển thị bản dựng, git và kênh gỡ lỗi/phát hành.
- **Đóng gói yêu cầu Node 22+**: tập lệnh chạy bản dựng TS và bản dựng Control UI.
- đọc `SIGN_IDENTITY` từ môi trường. Thêm `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (hoặc chứng chỉ Developer ID Application của bạn) vào shell rc của bạn để luôn ký bằng chứng chỉ của bạn. Ký ad-hoc yêu cầu tùy chọn rõ ràng qua `ALLOW_ADHOC_SIGNING=1` hoặc `SIGN_IDENTITY="-"` (không được khuyến nghị để kiểm tra quyền).
- chạy kiểm toán Team ID sau khi ký và thất bại nếu bất kỳ Mach-O nào bên trong gói ứng dụng được ký bởi một Team ID khác. Đặt `SKIP_TEAM_ID_CHECK=1` để bỏ qua.

## Cách sử dụng

```bash
# from repo root
scripts/package-mac-app.sh               # auto-selects identity; errors if none found
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # real cert
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (permissions will not stick)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # explicit ad-hoc (same caveat)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # dev-only Sparkle Team ID mismatch workaround
```

### Ad-hoc Signing Note

When signing with `SIGN_IDENTITY="-"` (ad-hoc), the script automatically disables the **Hardened Runtime** (`--options runtime`). This is necessary to prevent crashes when the app attempts to load embedded frameworks (like Sparkle) that do not share the same Team ID. Ad-hoc signatures also break TCC permission persistence; see [macOS permissions](/platforms/mac/permissions) for recovery steps.

## Build metadata for About

`package-mac-app.sh` stamps the bundle with:

- `OpenClawBuildTimestamp`: ISO8601 UTC at package time
- `OpenClawGitCommit`: short git hash (or `unknown` if unavailable)

The About tab reads these keys to show version, build date, git commit, and whether it’s a debug build (via `#if DEBUG`). Run the packager to refresh these values after code changes.

## Why

TCC permissions are tied to the bundle identifier _and_ code signature. Unsigned debug builds with changing UUIDs were causing macOS to forget grants after each rebuild. Signing the binaries (ad‑hoc by default) and keeping a fixed bundle id/path (`dist/OpenClaw.app`) bảo tồn các cấp phát giữa các bản dựng, phù hợp với cách tiếp cận VibeTunnel.