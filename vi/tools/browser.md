---
summary: Dịch vụ điều khiển trình duyệt tích hợp + lệnh hành động
read_when:
  - Thêm tự động hóa trình duyệt được điều khiển bởi agent
  - Gỡ lỗi tại sao OpenClaw đang can thiệp vào Chrome của riêng bạn
  - Triển khai cài đặt trình duyệt + vòng đời trong ứng dụng macOS
title: Trình duyệt (do OpenClaw quản lý)
x-i18n:
  source_path: tools\browser.md
  source_hash: 67d94d2bd57d729e22a1bbdbb0fcb08221e680f6d69410b73344819f0f341332
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:29:03.940Z'
---

# Trình duyệt (do OpenClaw quản lý)

OpenClaw có thể chạy một **hồ sơ Chrome/Brave/Edge/Chromium chuyên dụng** mà agent kiểm soát.
Nó được cách ly khỏi trình duyệt cá nhân của bạn và được quản lý thông qua một dịch vụ điều khiển cục bộ nhỏ bên trong Gateway (chỉ local loopback).

Chế độ xem cho người mới bắt đầu:

- Hãy coi nó như một **trình duyệt riêng biệt, chỉ dành cho agent**.
- Hồ sơ `openclaw` **không** ảnh hưởng đến hồ sơ trình duyệt cá nhân của bạn.
- Agent có thể **mở tab, đọc trang, nhấp chuột và gõ** trong một làn đường an toàn.
- Hồ sơ `chrome` mặc định sử dụng **trình duyệt Chromium mặc định của hệ thống** thông qua relay tiện ích mở rộng; chuyển sang `openclaw` để sử dụng trình duyệt được quản lý cách ly.
## Những gì bạn nhận được

- Một hồ sơ trình duyệt riêng biệt có tên **openclaw** (mặc định có dấu nhấn màu cam).
- Kiểm soát tab xác định (liệt kê/mở/tập trung/đóng).
- Các hành động của agent (nhấp/gõ/kéo/chọn), ảnh chụp, ảnh chụp màn hình, PDF.
- Hỗ trợ nhiều hồ sơ tùy chọn (`openclaw`, `work`, `remote`, ...).

Trình duyệt này **không phải** là trình duyệt hàng ngày của bạn. Đây là một bề mặt an toàn, cô lập để tự động hóa agent và xác minh.
## Bắt đầu nhanh

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Nếu bạn nhận được "Browser disabled", hãy bật nó trong cấu hình (xem bên dưới) và khởi động lại Gateway.
## Hồ sơ: `openclaw` vs `chrome`

- `openclaw`: trình duyệt được quản lý, cô lập (không cần tiện ích mở rộng).
- `chrome`: tiếp sóng tiện ích mở rộng đến **trình duyệt hệ thống** của bạn (yêu cầu tiện ích mở rộng OpenClaw được gắn vào một tab).

Đặt `browser.defaultProfile: "openclaw"` nếu bạn muốn chế độ được quản lý theo mặc định.
## Cấu hình

Cài đặt trình duyệt nằm trong `~/.openclaw/openclaw.json`.

```json5
{
  browser: {
    enabled: true, // default: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // default trusted-network mode
      // allowPrivateNetwork: true, // legacy alias
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    remoteCdpTimeoutMs: 1500, // remote CDP HTTP timeout (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // remote CDP WebSocket handshake timeout (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

Notes:

- The browser control service binds to loopback on a port derived from `gateway.port`
  (default: `18791`, which is gateway + 2). The relay uses the next port (`18792`).
- If you override the Gateway port (`gateway.port` or `OPENCLAW_GATEWAY_PORT`),
  the derived browser ports shift to stay in the same “family”.
- `cdpUrl` defaults to the relay port when unset.
- `remoteCdpTimeoutMs` applies to remote (non-loopback) CDP reachability checks.
- `remoteCdpHandshakeTimeoutMs` applies to remote CDP WebSocket reachability checks.
- Browser navigation/open-tab is SSRF-guarded before navigation and best-effort re-checked on final `http(s)` URL after navigation.
- `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` defaults to `true` (trusted-network model). Set it to `false` for strict public-only browsing.
- `browser.ssrfPolicy.allowPrivateNetwork` remains supported as a legacy alias for compatibility.
- `attachOnly: true` means “never launch a local browser; only attach if it is already running.”
- `color` + per-profile `color` tint the browser UI so you can see which profile is active.
- Default profile is `chrome` (extension relay). Use `defaultProfile: "openclaw"` for the managed browser.
- Auto-detect order: system default browser if Chromium-based; otherwise Chrome → Brave → Edge → Chromium → Chrome Canary.
- Local `openclaw` profiles auto-assign `cdpPort`/`cdpUrl` — chỉ đặt những cái này cho CDP từ xa.
## Sử dụng Brave (hoặc trình duyệt dựa trên Chromium khác)

Nếu trình duyệt **mặc định của hệ thống** của bạn dựa trên Chromium (Chrome/Brave/Edge/v.v.),
OpenClaw sẽ sử dụng nó tự động. Đặt `browser.executablePath` để ghi đè
tự động phát hiện:

Ví dụ CLI:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```
## Điều khiển cục bộ so với điều khiển từ xa

- **Điều khiển cục bộ (mặc định):** Gateway khởi động dịch vụ điều khiển local loopback và có thể khởi chạy trình duyệt cục bộ.
- **Điều khiển từ xa (node host):** chạy node host trên máy có trình duyệt; Gateway chuyển tiếp các hành động trình duyệt đến nó.
- **Remote CDP:** đặt `browser.profiles.<name>.cdpUrl` (hoặc `browser.cdpUrl`) để
  kết nối với trình duyệt dựa trên Chromium từ xa. Trong trường hợp này, OpenClaw sẽ không khởi chạy trình duyệt cục bộ.

URL CDP từ xa có thể bao gồm xác thực:

- Mã thông báo truy vấn (ví dụ: `https://provider.example?token=<token>`)
- HTTP Basic auth (ví dụ: `https://user:pass@provider.example`)

OpenClaw bảo tồn xác thực khi gọi các điểm cuối `/json/*` và khi kết nối
đến CDP WebSocket. Ưu tiên sử dụng biến môi trường hoặc trình quản lý bí mật cho
các mã thông báo thay vì cam kết chúng vào các tệp cấu hình.
## Node browser proxy (zero-config default)

Nếu bạn chạy một **node host** trên máy có trình duyệt của bạn, OpenClaw có thể
tự động định tuyến các lệnh gọi công cụ trình duyệt đến node đó mà không cần cấu hình trình duyệt bổ sung.
Đây là đường dẫn mặc định cho các gateway từ xa.

Ghi chú:

- Node host hiển thị máy chủ điều khiển trình duyệt cục bộ của nó thông qua một **lệnh proxy**.
- Các hồ sơ đến từ cấu hình `browser.profiles` của chính node (giống như cục bộ).
- Vô hiệu hóa nếu bạn không muốn:
  - Trên node: `nodeHost.browserProxy.enabled=false`
  - Trên gateway: `gateway.nodes.browser.mode="off"`
## Browserless (CDP từ xa được lưu trữ)

[Browserless](https://browserless.io) là một dịch vụ Chromium được lưu trữ công khai, cung cấp các điểm cuối CDP qua HTTPS. Bạn có thể trỏ một hồ sơ trình duyệt OpenClaw tới điểm cuối vùng Browserless và xác thực bằng khóa API của bạn.

Ví dụ:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

Notes:

- Replace `<BROWSERLESS_API_KEY>` bằng mã thông báo Browserless thực của bạn.
- Chọn điểm cuối vùng phù hợp với tài khoản Browserless của bạn (xem tài liệu của họ).
## Bảo mật

Các ý tưởng chính:

- Điều khiển trình duyệt chỉ hoạt động trên local loopback; quyền truy cập được chuyển qua xác thực của Gateway hoặc ghép nối node.
- Nếu điều khiển trình duyệt được bật và không có xác thực nào được cấu hình, OpenClaw sẽ tự động tạo `gateway.auth.token` khi khởi động và lưu nó vào cấu hình.
- Giữ Gateway và bất kỳ máy chủ node nào trên mạng riêng (Tailscale); tránh tiếp xúc công khai.
- Coi các URL/token CDP từ xa là bí mật; ưu tiên sử dụng biến môi trường hoặc trình quản lý bí mật.

Mẹo CDP từ xa:

- Ưu tiên các điểm cuối HTTPS và token có thời gian sống ngắn nếu có thể.
- Tránh nhúng các token có thời gian sống dài trực tiếp trong các tệp cấu hình.
## Hồ sơ (trình duyệt đa)

OpenClaw hỗ trợ nhiều hồ sơ được đặt tên (cấu hình định tuyến). Hồ sơ có thể là:

- **openclaw-managed**: một phiên bản trình duyệt dựa trên Chromium chuyên dụng với thư mục dữ liệu người dùng riêng + cổng CDP
- **remote**: một URL CDP rõ ràng (trình duyệt dựa trên Chromium chạy ở nơi khác)
- **extension relay**: các tab Chrome hiện có của bạn thông qua relay cục bộ + tiện ích Chrome

Mặc định:

- Hồ sơ `openclaw` được tạo tự động nếu bị thiếu.
- Hồ sơ `chrome` được tích hợp sẵn cho extension relay Chrome (trỏ đến `http://127.0.0.1:18792` theo mặc định).
- Các cổng CDP cục bộ được cấp phát từ **18800–18899** theo mặc định.
- Xóa một hồ sơ sẽ di chuyển thư mục dữ liệu cục bộ của nó vào Thùng rác.

Tất cả các điểm cuối kiểm soát chấp nhận `?profile=<name>`; CLI sử dụng `--browser-profile`.
## Relay tiện ích Chrome (sử dụng Chrome hiện có của bạn)

OpenClaw cũng có thể điều khiển **các tab Chrome hiện có của bạn** (không cần một instance Chrome "openclaw" riêng biệt) thông qua một relay CDP cục bộ + một tiện ích Chrome.

Hướng dẫn đầy đủ: [Chrome extension](/tools/chrome-extension)

Luồng:

- Gateway chạy cục bộ (cùng máy) hoặc một node host chạy trên máy trình duyệt.
- Một **relay server** cục bộ lắng nghe tại một loopback `cdpUrl` (mặc định: `http://127.0.0.1:18792`).
- Bạn nhấp vào biểu tượng tiện ích **OpenClaw Browser Relay** trên một tab để gắn kết (nó không tự động gắn kết).
- Agent điều khiển tab đó thông qua công cụ `browser` bình thường, bằng cách chọn profile phù hợp.

Nếu Gateway chạy ở nơi khác, hãy chạy một node host trên máy trình duyệt để Gateway có thể proxy các hành động trình duyệt.

### Phiên sandboxed

Nếu phiên agent được sandboxed, công cụ `browser` có thể mặc định là `target="sandbox"` (trình duyệt sandbox).
Tiếp quản relay tiện ích Chrome yêu cầu kiểm soát trình duyệt host, vì vậy hãy:

- chạy phiên không được sandboxed, hoặc
- đặt `agents.defaults.sandbox.browser.allowHostControl: true` và sử dụng `target="host"` khi gọi công cụ.

### Thiết lập

1. Tải tiện ích (dev/unpacked):

```bash
openclaw browser extension install
```

- Chrome → `chrome://extensions` → enable “Developer mode”
- “Load unpacked” → select the directory printed by `openclaw browser extension path`
- Pin the extension, then click it on the tab you want to control (badge shows `ON`).

2. Use it:

- CLI: `openclaw browser --browser-profile chrome tabs`
- Agent tool: `browser` with `profile="chrome"`

Optional: if you want a different name or relay port, create your own profile:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Ghi chú:

- Chế độ này dựa vào Playwright-on-CDP cho hầu hết các hoạt động (ảnh chụp/ảnh nhanh/hành động).
- Tách rời bằng cách nhấp vào biểu tượng tiện ích một lần nữa.
## Đảm bảo cách ly

- **Thư mục dữ liệu người dùng riêng**: không bao giờ chạm vào hồ sơ trình duyệt cá nhân của bạn.
- **Cổng riêng**: tránh `9222` để ngăn chặn xung đột với quy trình phát triển.
- **Kiểm soát tab xác định**: nhắm mục tiêu các tab theo `targetId`, không phải "tab cuối cùng".
## Lựa chọn trình duyệt

Khi khởi chạy cục bộ, OpenClaw chọn trình duyệt đầu tiên có sẵn:

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

Bạn có thể ghi đè bằng `browser.executablePath`.

Các nền tảng:

- macOS: kiểm tra `/Applications` và `~/Applications`.
- Linux: tìm kiếm `google-chrome`, `brave`, `microsoft-edge`, `chromium`, v.v.
- Windows: kiểm tra các vị trí cài đặt phổ biến.
## Control API (tùy chọn)

Đối với các tích hợp cục bộ, Gateway cung cấp một API HTTP loopback nhỏ:

- Status/start/stop: `GET /`, `POST /start`, `POST /stop`
- Tabs: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
- Snapshot/screenshot: `GET /snapshot`, `POST /screenshot`
- Actions: `POST /navigate`, `POST /act`
- Hooks: `POST /hooks/file-chooser`, `POST /hooks/dialog`
- Downloads: `POST /download`, `POST /wait/download`
- Debugging: `GET /console`, `POST /pdf`
- Debugging: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
- Network: `POST /response/body`
- State: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
- State: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
- Settings: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Tất cả các endpoint chấp nhận `?profile=<name>`.

Nếu xác thực gateway được cấu hình, các tuyến HTTP trình duyệt cũng yêu cầu xác thực:

- `Authorization: Bearer <gateway token>`
- `x-openclaw-password: <gateway password>` hoặc HTTP Basic auth với mật khẩu đó

### Yêu cầu Playwright

Một số tính năng (navigate/act/AI snapshot/role snapshot, chụp ảnh phần tử, PDF) yêu cầu
Playwright. Nếu Playwright chưa được cài đặt, các endpoint đó sẽ trả về lỗi 501
rõ ràng. ARIA snapshots và ảnh chụp cơ bản vẫn hoạt động cho Chrome do openclaw quản lý.
Đối với trình điều khiển relay tiện ích mở rộng Chrome, ARIA snapshots và ảnh chụp yêu cầu Playwright.

Nếu bạn thấy `Playwright is not available in this gateway build`, hãy cài đặt gói Playwright đầy đủ
(không phải `playwright-core`) và khởi động lại gateway, hoặc cài đặt lại
OpenClaw với hỗ trợ trình duyệt.

#### Cài đặt Playwright trong Docker

Nếu Gateway của bạn chạy trong Docker, tránh `npx playwright` (xung đột ghi đè npm).
Sử dụng CLI đi kèm thay thế:

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

To persist browser downloads, set `PLAYWRIGHT_BROWSERS_PATH` (for example,
`/home/node/.cache/ms-playwright`) and make sure `/home/node` is persisted via
`OPENCLAW_HOME_VOLUME` hoặc bind mount. Xem [Docker](/install/docker).
## Cách hoạt động (nội bộ)

Luồng cấp cao:

- Một **máy chủ điều khiển** nhỏ chấp nhận các yêu cầu HTTP.
- Nó kết nối với các trình duyệt dựa trên Chromium (Chrome/Brave/Edge/Chromium) thông qua **CDP**.
- Đối với các hành động nâng cao (click/type/snapshot/PDF), nó sử dụng **Playwright** trên
  CDP.
- Khi Playwright không có sẵn, chỉ các hoạt động không phụ thuộc Playwright mới khả dụng.

Thiết kế này giữ cho agent ở trên một giao diện ổn định, xác định được trong khi cho phép
bạn hoán đổi các trình duyệt và hồ sơ cục bộ/từ xa.
## Tham khảo nhanh CLI

Tất cả các lệnh chấp nhận `--browser-profile <name>` để nhắm mục tiêu một hồ sơ cụ thể.
Tất cả các lệnh cũng chấp nhận `--json` để xuất dữ liệu có thể đọc bằng máy (tải trọng ổn định).

Cơ bản:

- `openclaw browser status`
- `openclaw browser start`
- `openclaw browser stop`
- `openclaw browser tabs`
- `openclaw browser tab`
- `openclaw browser tab new`
- `openclaw browser tab select 2`
- `openclaw browser tab close 2`
- `openclaw browser open https://example.com`
- `openclaw browser focus abcd1234`
- `openclaw browser close abcd1234`

Kiểm tra:

- `openclaw browser screenshot`
- `openclaw browser screenshot --full-page`
- `openclaw browser screenshot --ref 12`
- `openclaw browser screenshot --ref e12`
- `openclaw browser snapshot`
- `openclaw browser snapshot --format aria --limit 200`
- `openclaw browser snapshot --interactive --compact --depth 6`
- `openclaw browser snapshot --efficient`
- `openclaw browser snapshot --labels`
- `openclaw browser snapshot --selector "#main" --interactive`
- `openclaw browser snapshot --frame "iframe#main" --interactive`
- `openclaw browser console --level error`
- `openclaw browser errors --clear`
- `openclaw browser requests --filter api --clear`
- `openclaw browser pdf`
- `openclaw browser responsebody "**/api" --max-chars 5000`

Hành động:

- `openclaw browser navigate https://example.com`
- `openclaw browser resize 1280 720`
- `openclaw browser click 12 --double`
- `openclaw browser click e12 --double`
- `openclaw browser type 23 "hello" --submit`
- `openclaw browser press Enter`
- `openclaw browser hover 44`
- `openclaw browser scrollintoview e12`
- `openclaw browser drag 10 11`
- `openclaw browser select 9 OptionA OptionB`
- `openclaw browser download e12 report.pdf`
- `openclaw browser waitfordownload report.pdf`
- `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
- `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
- `openclaw browser dialog --accept`
- `openclaw browser wait --text "Done"`
- `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
- `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
- `openclaw browser highlight e12`
- `openclaw browser trace start`
- `openclaw browser trace stop`

Trạng thái:

- `openclaw browser cookies`
- `openclaw browser cookies set session abc123 --url "https://example.com"`
- `openclaw browser cookies clear`
- `openclaw browser storage local get`
- `openclaw browser storage local set theme dark`
- `openclaw browser storage session clear`
- `openclaw browser set offline on`
- `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
- `openclaw browser set credentials user pass`
- `openclaw browser set credentials --clear`
- `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
- `openclaw browser set geo --clear`
- `openclaw browser set media dark`
- `openclaw browser set timezone America/New_York`
- `openclaw browser set locale en-US`
- `openclaw browser set device "iPhone 14"`

Ghi chú:

- `upload` và `dialog` là các lệnh gọi **chuẩn bị**; chạy chúng trước khi nhấp/nhấn
  kích hoạt bộ chọn/hộp thoại.
- Đường dẫn tải xuống và đầu ra theo dõi bị giới hạn trong các thư mục tạm thời của OpenClaw:
  - traces: `/tmp/openclaw` (dự phòng: `${os.tmpdir()}/openclaw`)
  - downloads: `/tmp/openclaw/downloads` (dự phòng: `${os.tmpdir()}/openclaw/downloads`)
- Đường dẫn tải lên bị giới hạn trong thư mục tạm thời tải lên của OpenClaw:
  - uploads: `/tmp/openclaw/uploads` (dự phòng: `${os.tmpdir()}/openclaw/uploads`)
- `upload` cũng có thể đặt các đầu vào tệp trực tiếp thông qua `--input-ref` hoặc `--element`.
- `snapshot`:
  - `--format ai` (mặc định khi Playwright được cài đặt): trả về ảnh chụp AI với các tham chiếu số (`aria-ref="<n>"`).
  - `--format aria`: trả về cây khả năng truy cập (không có tham chiếu; chỉ kiểm tra).
  - `--efficient` (hoặc `--mode efficient`): cài đặt trước ảnh chụp vai trò nhỏ gọn (tương tác + nhỏ gọn + độ sâu + maxChars thấp hơn).
  - Mặc định cấu hình (chỉ công cụ/CLI): đặt `browser.snapshotDefaults.mode: "efficient"` để sử dụng ảnh chụp hiệu quả khi người gọi không chuyển chế độ (xem [Cấu hình Gateway](/gateway/configuration#browser-openclaw-managed-browser)).
  - Tùy chọn ảnh chụp vai trò (`--interactive`, `--compact`, `--depth`, `--selector`) buộc ảnh chụp dựa trên vai trò với các tham chiếu như `ref=e12`.
  - `--frame "<iframe selector>"` giới hạn ảnh chụp vai trò trong một iframe (kết hợp với các tham chiếu vai trò như `e12`).
  - `--interactive` xuất ra danh sách phẳng, dễ chọn các phần tử tương tác (tốt nhất để điều khiển các hành động).
  - `--labels` thêm ảnh chụp chỉ chứa khung nhìn với nhãn tham chiếu được phủ lên (in `MEDIA:<path>`).
- `click`/`type`/v.v. yêu cầu một `ref` từ `snapshot` (tham chiếu số `12` hoặc vai trò `e12`).
  Bộ chọn CSS không được hỗ trợ có chủ ý cho các hành động.
## Snapshots và refs

OpenClaw hỗ trợ hai kiểu "snapshot":

- **AI snapshot (numeric refs)**: `openclaw browser snapshot` (mặc định; `--format ai`)
  - Output: một snapshot văn bản bao gồm numeric refs.
  - Actions: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  - Nội bộ, ref được phân giải thông qua `aria-ref` của Playwright.

- **Role snapshot (role refs như `e12`)**: `openclaw browser snapshot --interactive` (hoặc `--compact`, `--depth`, `--selector`, `--frame`)
  - Output: một danh sách/cây dựa trên role với `[ref=e12]` (và `[nth=1]` tùy chọn).
  - Actions: `openclaw browser click e12`, `openclaw browser highlight e12`.
  - Nội bộ, ref được phân giải thông qua `getByRole(...)` (cộng với `nth()` cho các bản sao).
  - Thêm `--labels` để bao gồm ảnh chụp viewport với các nhãn `e12` được phủ lên trên.

Hành vi của Ref:

- Refs **không ổn định qua các lần điều hướng**; nếu có lỗi xảy ra, chạy lại `snapshot` và sử dụng một ref mới.
- Nếu role snapshot được lấy với `--frame`, role refs được giới hạn trong iframe đó cho đến khi có role snapshot tiếp theo.
## Các tính năng chờ đợi nâng cao

Bạn có thể chờ đợi nhiều hơn chỉ thời gian/văn bản:

- Chờ đợi URL (hỗ trợ glob bởi Playwright):
  - `openclaw browser wait --url "**/dash"`
- Chờ đợi trạng thái tải:
  - `openclaw browser wait --load networkidle`
- Chờ đợi một vị từ JS:
  - `openclaw browser wait --fn "window.ready===true"`
- Chờ đợi một bộ chọn trở nên hiển thị:
  - `openclaw browser wait "#main"`

Những điều này có thể được kết hợp:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```
## Gỡ lỗi quy trình làm việc

Khi một hành động không thành công (ví dụ: "không hiển thị", "vi phạm chế độ strict", "bị che phủ"):

1. `openclaw browser snapshot --interactive`
2. Sử dụng `click <ref>` / `type <ref>` (ưu tiên tham chiếu vai trò ở chế độ tương tác)
3. Nếu vẫn không thành công: `openclaw browser highlight <ref>` để xem Playwright đang nhắm mục tiêu gì
4. Nếu trang hoạt động kỳ lạ:
   - `openclaw browser errors --clear`
   - `openclaw browser requests --filter api --clear`
5. Để gỡ lỗi sâu: ghi lại một dấu vết:
   - `openclaw browser trace start`
   - tái tạo vấn đề
   - `openclaw browser trace stop` (in ra `TRACE:<path>`)
## Đầu ra JSON

`--json` dành cho scripting và công cụ có cấu trúc.

Ví dụ:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Role snapshots in JSON include `refs` plus a small `stats` block (lines/chars/refs/interactive) để các công cụ có thể suy luận về kích thước và mật độ payload.
## Các công tắc trạng thái và môi trường

Những công tắc này rất hữu ích cho các quy trình "làm cho trang web hoạt động như X":

- Cookies: `cookies`, `cookies set`, `cookies clear`
- Storage: `storage local|session get|set|clear`
- Offline: `set offline on|off`
- Headers: `set headers --headers-json '{"X-Debug":"1"}'` (phiên bản cũ `set headers --json '{"X-Debug":"1"}'` vẫn được hỗ trợ)
- HTTP basic auth: `set credentials user pass` (hoặc `--clear`)
- Geolocation: `set geo <lat> <lon> --origin "https://example.com"` (hoặc `--clear`)
- Media: `set media dark|light|no-preference|none`
- Timezone / locale: `set timezone ...`, `set locale ...`
- Device / viewport:
  - `set device "iPhone 14"` (Playwright device presets)
  - `set viewport 1280 720`
## Bảo mật & quyền riêng tư

- Hồ sơ trình duyệt openclaw có thể chứa các phiên đã đăng nhập; hãy coi nó là nhạy cảm.
- `browser act kind=evaluate` / `openclaw browser evaluate` và `wait --fn`
  thực thi JavaScript tùy ý trong ngữ cảnh trang. Prompt injection có thể điều hướng điều này. Vô hiệu hóa nó bằng `browser.evaluateEnabled=false` nếu bạn không cần nó.
- Để đăng nhập và ghi chú chống bot (X/Twitter, v.v.), xem [Đăng nhập trình duyệt + đăng bài X/Twitter](/tools/browser-login).
- Giữ máy chủ Gateway/node ở chế độ riêng tư (local loopback hoặc chỉ tailnet).
- Các điểm cuối CDP từ xa rất mạnh; hãy đường hầm và bảo vệ chúng.

Ví dụ chế độ nghiêm ngặt (chặn các đích riêng/nội bộ theo mặc định):

```json5
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // optional exact allow
    },
  },
}
```
## Khắc phục sự cố

Để giải quyết các vấn đề cụ thể của Linux (đặc biệt là snap Chromium), xem
[Khắc phục sự cố trình duyệt](/tools/browser-linux-troubleshooting).
## Agent tools + how control works

The agent gets **one tool** for browser automation:

- `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

How it maps:

- `browser snapshot` returns a stable UI tree (AI or ARIA).
- `browser act` uses the snapshot `ref` IDs to click/type/drag/select.
- `browser screenshot` captures pixels (full page or element).
- `browser` accepts:
  - `profile` to choose a named browser profile (openclaw, chrome, or remote CDP).
  - `target` (`sandbox` | `host` | `node`) to select where the browser lives.
  - In sandboxed sessions, `target: "host"` requires `agents.defaults.sandbox.browser.allowHostControl=true`.
  - If `target` is omitted: sandboxed sessions default to `sandbox`, non-sandbox sessions default to `host`.
  - If a browser-capable node is connected, the tool may auto-route to it unless you pin `target="host"` or `target="node"`.

This keeps the agent deterministic and avoids brittle selectors.