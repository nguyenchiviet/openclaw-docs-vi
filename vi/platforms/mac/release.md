---
summary: 'Danh sách kiểm tra phát hành OpenClaw cho macOS (Sparkle feed, đóng gói, ký)'
read_when:
  - Cắt hoặc xác thực bản phát hành OpenClaw macOS
  - Cập nhật tài nguyên appcast hoặc feed của Sparkle
title: Phát hành macOS
x-i18n:
  source_path: platforms\mac\release.md
  source_hash: 2fea8b53a228e681c59efe142c042d210167f94102ca9df5ff1042d6ebcb4a7d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:20.102Z'
---

# Phát hành OpenClaw macOS (Sparkle)

Ứng dụng này hiện được cung cấp với các bản cập nhật tự động Sparkle. Các bản dựng phát hành phải được ký bằng Developer ID, nén thành zip và xuất bản với một mục appcast được ký.
## Điều kiện tiên quyết

- Chứng chỉ Developer ID Application được cài đặt (ví dụ: `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Đường dẫn khóa riêng Sparkle được đặt trong môi trường dưới dạng `SPARKLE_PRIVATE_KEY_FILE` (đường dẫn đến khóa riêng ed25519 Sparkle của bạn; khóa công khai được nhúng trong Info.plist). Nếu nó bị thiếu, hãy kiểm tra `~/.profile`.
- Thông tin xác thực Notary (hồ sơ keychain hoặc khóa API) cho `xcrun notarytool` nếu bạn muốn phân phối DMG/zip an toàn với Gatekeeper.
  - Chúng tôi sử dụng hồ sơ Keychain có tên `openclaw-notary`, được tạo từ biến môi trường khóa API App Store Connect trong hồ sơ shell của bạn:
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- Các phụ thuộc `pnpm` được cài đặt (`pnpm install --config.node-linker=hoisted`).
- Các công cụ Sparkle được tìm nạp tự động thông qua SwiftPM tại `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, v.v.).
## Xây dựng & đóng gói

Ghi chú:

- `APP_BUILD` ánh xạ tới `CFBundleVersion`/`sparkle:version`; giữ nó ở dạng số + đơn điệu (không `-beta`), hoặc Sparkle sẽ so sánh nó bằng nhau.
- Mặc định là kiến trúc hiện tại (`$(uname -m)`). Đối với bản phát hành/bản dựng phổ quát, đặt `BUILD_ARCHS="arm64 x86_64"` (hoặc `BUILD_ARCHS=all`).
- Sử dụng `scripts/package-mac-dist.sh` cho các tạo phẩm phát hành (zip + DMG + notarization). Sử dụng `scripts/package-mac-app.sh` cho đóng gói cục bộ/phát triển.

```bash
# From repo root; set release IDs so Sparkle feed is enabled.
# APP_BUILD must be numeric + monotonic for Sparkle compare.
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.2.27 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip for distribution (includes resource forks for Sparkle delta support)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.2.27.zip

# Optional: also build a styled DMG for humans (drag to /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.2.27.dmg

# Recommended: build + notarize/staple zip + DMG
# First, create a keychain profile once:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.2.27 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Optional: ship dSYM alongside the release
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.2.27.dSYM.zip
```
## Mục nhập Appcast

Sử dụng trình tạo ghi chú phát hành để Sparkle hiển thị các ghi chú HTML được định dạng:

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.2.27.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Generates HTML release notes from `CHANGELOG.md` (via [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) and embeds them in the appcast entry.
Commit the updated `appcast.xml` cùng với các tài sản phát hành (zip + dSYM) khi xuất bản.
## Xuất bản & xác minh

- Tải lên `OpenClaw-2026.2.27.zip` (và `OpenClaw-2026.2.27.dSYM.zip`) lên bản phát hành GitHub cho thẻ `v2026.2.27`.
- Đảm bảo URL appcast thô khớp với bản phát hành được nướng: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- Kiểm tra sanity:
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` trả về 200.
  - `curl -I <enclosure url>` trả về 200 sau khi tải lên tài sản.
  - Trên bản dựng công khai trước đó, chạy "Kiểm tra cập nhật…" từ tab About và xác minh Sparkle cài đặt bản dựng mới một cách sạch sẽ.

Định nghĩa hoàn thành: ứng dụng đã ký + appcast được xuất bản, luồng cập nhật hoạt động từ phiên bản đã cài đặt cũ hơn, và tài sản phát hành được đính kèm vào bản phát hành GitHub.