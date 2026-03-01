---
title: Danh sách kiểm tra phát hành
summary: Danh sách kiểm tra phát hành từng bước cho npm + macOS app
read_when:
  - Phát hành phiên bản npm mới
  - Phát hành phiên bản ứng dụng macOS mới
  - Xác minh metadata trước khi xuất bản
x-i18n:
  source_path: reference\RELEASING.md
  source_hash: 48f3db2c96d02622efcf3b947856e3e02e230ad0cd15830d7928a3a93bd8aa35
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:17:04.989Z'
---

# Danh sách kiểm tra phát hành (npm + macOS)

Sử dụng `pnpm` (Node 22+) từ thư mục gốc của kho lưu trữ. Giữ cây làm việc sạch sẽ trước khi gắn thẻ/xuất bản.

## Kích hoạt bởi người vận hành

Khi người vận hành nói “release”, hãy thực hiện ngay các bước kiểm tra trước khi phát hành này (không hỏi thêm trừ khi bị chặn):

- Đọc tài liệu này và `docs/platforms/mac/release.md`.
- Tải biến môi trường từ `~/.profile` và xác nhận `SPARKLE_PRIVATE_KEY_FILE` + các biến App Store Connect đã được đặt (SPARKLE_PRIVATE_KEY_FILE nên nằm trong `~/.profile`).
- Sử dụng khóa Sparkle từ `~/Library/CloudStorage/Dropbox/Backup/Sparkle` nếu cần.

1. **Phiên bản & siêu dữ liệu**

- [ ] Tăng phiên bản `package.json` (ví dụ: `2026.1.29`).
- [ ] Chạy `pnpm plugins:sync` để đồng bộ hóa phiên bản gói tiện ích mở rộng + nhật ký thay đổi.
- [ ] Cập nhật chuỗi CLI/phiên bản trong [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) và tác nhân người dùng Baileys trong [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts).
- [ ] Xác nhận siêu dữ liệu gói (tên, mô tả, kho lưu trữ, từ khóa, giấy phép) và bản đồ `bin` trỏ đến [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) cho `openclaw`.
- [ ] Nếu các phụ thuộc thay đổi, hãy chạy `pnpm install` để `pnpm-lock.yaml` được cập nhật.

2. **Bản dựng & tạo phẩm**

- [ ] Nếu đầu vào A2UI thay đổi, hãy chạy `pnpm canvas:a2ui:bundle` và commit bất kỳ [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) nào đã cập nhật.
- [ ] `pnpm run build` (tạo lại `dist/`).
- [ ] Xác minh gói npm `files` bao gồm tất cả các thư mục `dist/*` cần thiết (đặc biệt là `dist/node-host/**` và `dist/acp/**` cho headless node + ACP CLI).
- [ ] Xác nhận `dist/build-info.json` tồn tại và bao gồm băm `commit` dự kiến (biểu ngữ CLI sử dụng điều này cho các cài đặt npm).
- [ ] Tùy chọn: `npm pack --pack-destination /tmp` sau khi xây dựng; kiểm tra nội dung tarball và giữ nó tiện dụng cho bản phát hành GitHub (**không** commit nó).

3. **Nhật ký thay đổi & tài liệu**

- [ ] Cập nhật `CHANGELOG.md` với các điểm nổi bật dành cho người dùng (tạo tệp nếu thiếu); giữ các mục theo thứ tự giảm dần nghiêm ngặt theo phiên bản.
- [ ] Đảm bảo các ví dụ/cờ README khớp với hành vi CLI hiện tại (đặc biệt là các lệnh hoặc tùy chọn mới).

4. **Xác thực**
- [ ] `pnpm build`
- [ ] `pnpm check`
- [ ] `pnpm test` (hoặc `pnpm test:coverage` nếu bạn cần đầu ra độ bao phủ)
- [ ] `pnpm release:check` (xác minh nội dung gói npm)
- [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (kiểm tra nhanh cài đặt Docker, đường dẫn nhanh; bắt buộc trước khi phát hành)
  - Nếu bản phát hành npm trước đó được biết là bị lỗi, hãy đặt `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` hoặc `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` cho bước cài đặt trước.
- [ ] (Tùy chọn) Kiểm tra nhanh trình cài đặt đầy đủ (thêm độ bao phủ không phải root + CLI): `pnpm test:install:smoke`
- [ ] (Tùy chọn) E2E trình cài đặt (Docker, chạy `curl -fsSL https://openclaw.ai/install.sh | bash`, thiết lập ban đầu, sau đó chạy các lệnh gọi công cụ thực tế):
  - `pnpm test:install:e2e:openai` (yêu cầu `OPENAI_API_KEY`)
  - `pnpm test:install:e2e:anthropic` (yêu cầu `ANTHROPIC_API_KEY`)
  - `pnpm test:install:e2e` (yêu cầu cả hai khóa; chạy cả hai nhà cung cấp)
- [ ] (Tùy chọn) Kiểm tra nhanh Gateway web nếu các thay đổi của bạn ảnh hưởng đến đường dẫn gửi/nhận.

5. **Ứng dụng macOS (Sparkle)**

- [ ] Xây dựng + ký ứng dụng macOS, sau đó nén lại để phân phối.
- [ ] Tạo appcast Sparkle (ghi chú HTML qua [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) và cập nhật `appcast.xml`.
- [ ] Giữ tệp zip ứng dụng (và tệp zip dSYM tùy chọn) sẵn sàng để đính kèm vào bản phát hành GitHub.
- [ ] Làm theo [hướng dẫn phát hành macOS](/platforms/mac/release) để biết các lệnh chính xác và biến môi trường cần thiết.
  - `APP_BUILD` phải là số + đơn điệu (không có `-beta`) để Sparkle so sánh các phiên bản một cách chính xác.
  - Nếu công chứng, hãy sử dụng hồ sơ chuỗi khóa `openclaw-notary` được tạo từ các biến môi trường API App Store Connect (xem [phát hành macOS](/platforms/mac/release)).

6. **Xuất bản (npm)**

- [ ] Xác nhận trạng thái git sạch; commit và push khi cần.
- [ ] `npm login` (xác minh 2FA) nếu cần.
- [ ] `npm publish --access public` (sử dụng `--tag beta` cho các bản phát hành trước).
- [ ] Xác minh registry: `npm view openclaw version`, `npm view openclaw dist-tags`, và `npx -y openclaw@X.Y.Z --version` (hoặc `--help`).
### Khắc phục sự cố (ghi chú từ bản phát hành 2.0.0-beta2)

- **npm pack/publish bị treo hoặc tạo ra tarball quá lớn**: gói ứng dụng macOS trong `dist/OpenClaw.app` (và các tệp zip phát hành) bị đưa vào gói. Khắc phục bằng cách đưa nội dung xuất bản vào danh sách trắng thông qua `package.json` `files` (bao gồm các thư mục con dist, docs, skills; loại trừ các gói ứng dụng). Xác nhận với `npm pack --dry-run` rằng `dist/OpenClaw.app` không được liệt kê.
- **Vòng lặp web xác thực npm cho dist-tags**: sử dụng xác thực cũ để nhận lời nhắc OTP:
  - `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
- **Xác minh `npx` thất bại với `ECOMPROMISED: Lock compromised`**: thử lại với bộ nhớ đệm mới:
  - `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
- **Thẻ cần được định vị lại sau khi sửa lỗi muộn**: cập nhật và đẩy thẻ bắt buộc, sau đó đảm bảo các tài sản phát hành trên GitHub vẫn khớp:
  - `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **Phát hành GitHub + appcast**

- [ ] Gắn thẻ và đẩy: `git tag vX.Y.Z && git push origin vX.Y.Z` (hoặc `git push --tags`).
- [ ] Tạo/làm mới bản phát hành GitHub cho `vX.Y.Z` với **tiêu đề `openclaw X.Y.Z`** (không chỉ thẻ); nội dung phải bao gồm phần changelog **đầy đủ** cho phiên bản đó (Điểm nổi bật + Thay đổi + Sửa lỗi), nội tuyến (không có liên kết trần), và **không được lặp lại tiêu đề trong nội dung**.
- [ ] Đính kèm các tạo phẩm: tarball `npm pack` (tùy chọn), `OpenClaw-X.Y.Z.zip`, và `OpenClaw-X.Y.Z.dSYM.zip` (nếu được tạo).
- [ ] Commit `appcast.xml` đã cập nhật và đẩy nó (Sparkle lấy dữ liệu từ main).
- [ ] Từ một thư mục tạm thời sạch (không có `package.json`), chạy `npx -y openclaw@X.Y.Z send --help` để xác nhận các điểm vào cài đặt/CLI hoạt động.
- [ ] Thông báo/chia sẻ ghi chú phát hành.
## Phạm vi xuất bản plugin (npm)

Chúng tôi chỉ xuất bản **các plugin npm hiện có** trong phạm vi `@openclaw/*`.
Các plugin đi kèm không có trên npm vẫn chỉ là **cây thư mục trên đĩa** (vẫn được vận chuyển trong `extensions/**`).

Quy trình để tạo danh sách:

1. `npm search @openclaw --json` và ghi lại tên gói.
2. So sánh với tên `extensions/*/package.json`.
3. Chỉ xuất bản **phần giao nhau** (đã có trên npm).

Danh sách plugin npm hiện tại (cập nhật khi cần):

- @openclaw/bluebubbles
- @openclaw/diagnostics-otel
- @openclaw/discord
- @openclaw/feishu
- @openclaw/lobster
- @openclaw/matrix
- @openclaw/msteams
- @openclaw/nextcloud-talk
- @openclaw/nostr
- @openclaw/voice-call
- @openclaw/zalo
- @openclaw/zalouser

Ghi chú phát hành cũng phải đề cập đến **các plugin đi kèm tùy chọn mới** mà **không được bật theo mặc định** (ví dụ: `tlon`).