---
summary: 'Chrome extension: để OpenClaw điều khiển tab Chrome hiện tại của bạn'
read_when:
  - Bạn muốn agent điều khiển một tab Chrome hiện có (nút thanh công cụ)
  - Bạn cần Gateway từ xa + tự động hóa trình duyệt cục bộ qua Tailscale
  - >-
    Bạn muốn hiểu rõ những hàm ý bảo mật của việc chiếm quyền điều khiển trình
    duyệt
title: Tiện ích mở rộng Chrome
x-i18n:
  source_path: tools\chrome-extension.md
  source_hash: 38fb2cadcf02887d6b51e3e96fe1ec6cf603dc88a86a59e22cff522f91581483
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:29:44.360Z'
---

# Tiện ích mở rộng Chrome (browser relay)

Tiện ích mở rộng Chrome của OpenClaw cho phép agent điều khiển các **tab Chrome hiện có** của bạn (cửa sổ Chrome thông thường của bạn) thay vì khởi chạy một hồ sơ Chrome được quản lý bởi openclaw riêng biệt.

Đính kèm/tách rời xảy ra thông qua một **nút thanh công cụ Chrome duy nhất**.
## Nó là gì (khái niệm)

Có ba phần:

- **Dịch vụ điều khiển trình duyệt** (Gateway hoặc node): API mà agent/tool gọi (thông qua Gateway)
- **Máy chủ relay cục bộ** (loopback CDP): kết nối giữa máy chủ điều khiển và tiện ích (`http://127.0.0.1:18792` theo mặc định)
- **Tiện ích Chrome MV3**: gắn vào tab hoạt động bằng `chrome.debugger` và chuyển tiếp các thông báo CDP đến relay

OpenClaw sau đó điều khiển tab được gắn thông qua bề mặt công cụ `browser` bình thường (chọn hồ sơ phù hợp).
## Cài đặt / tải (chưa đóng gói)

1. Cài đặt tiện ích mở rộng vào một đường dẫn cục bộ ổn định:

```bash
openclaw browser extension install
```

2. Print the installed extension directory path:

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

- Bật "Chế độ nhà phát triển"
- "Tải tiện ích chưa đóng gói" → chọn thư mục được in ở trên

4. Ghim tiện ích mở rộng.
## Cập nhật (không có bước xây dựng)

Tiện ích mở rộng được đi kèm trong bản phát hành OpenClaw (gói npm) dưới dạng các tệp tĩnh. Không có bước "xây dựng" riêng biệt.

Sau khi nâng cấp OpenClaw:

- Chạy lại `openclaw browser extension install` để làm mới các tệp được cài đặt trong thư mục trạng thái OpenClaw của bạn.
- Chrome → `chrome://extensions` → nhấp vào "Reload" trên tiện ích mở rộng.
## Sử dụng (đặt token gateway một lần)

OpenClaw được cung cấp với một hồ sơ trình duyệt tích hợp có tên `chrome` nhắm mục tiêu đến relay tiện ích mở rộng trên cổng mặc định.

Trước khi kết nối lần đầu, hãy mở Tùy chọn tiện ích mở rộng và đặt:

- `Port` (mặc định `18792`)
- `Gateway token` (phải khớp với `gateway.auth.token` / `OPENCLAW_GATEWAY_TOKEN`)

Sử dụng nó:

- CLI: `openclaw browser --browser-profile chrome tabs`
- Công cụ agent: `browser` với `profile="chrome"`

Nếu bạn muốn một tên khác hoặc một cổng relay khác, hãy tạo hồ sơ của riêng bạn:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### Custom Gateway ports

If you're using a custom gateway port, the extension relay port is automatically derived:

**Extension Relay Port = Gateway Port + 3**

Example: if `gateway.port: 19001`, then:

- Extension relay port: `19004` (gateway + 3)

Cấu hình tiện ích mở rộng để sử dụng cổng relay dẫn xuất trong trang Tùy chọn tiện ích mở rộng.
## Gắn / tách (nút thanh công cụ)

- Mở tab mà bạn muốn OpenClaw điều khiển.
- Nhấp vào biểu tượng tiện ích.
  - Badge hiển thị `ON` khi được gắn.
- Nhấp lại để tách.
## Tab nào mà nó kiểm soát?

- Nó **không** tự động kiểm soát "bất kỳ tab nào bạn đang xem".
- Nó kiểm soát **chỉ những tab(s) bạn đã gắn kèm một cách rõ ràng** bằng cách nhấp vào nút thanh công cụ.
- Để chuyển đổi: mở tab khác và nhấp vào biểu tượng tiện ích mở rộng ở đó.
## Badge + lỗi thường gặp

- `ON`: đã kết nối; OpenClaw có thể điều khiển tab đó.
- `…`: đang kết nối đến relay cục bộ.
- `!`: relay không thể tiếp cận/xác thực (phổ biến nhất: relay server không chạy, hoặc gateway token bị thiếu/sai).

Nếu bạn thấy `!`:

- Đảm bảo Gateway đang chạy cục bộ (thiết lập mặc định), hoặc chạy một node host trên máy này nếu Gateway chạy ở nơi khác.
- Mở trang Options của extension; nó xác thực khả năng tiếp cận relay + xác thực gateway-token.
## Gateway từ xa (sử dụng một node host)

### Gateway cục bộ (cùng máy với Chrome) — thường **không cần bước bổ sung**

Nếu Gateway chạy trên cùng máy với Chrome, nó sẽ khởi động dịch vụ điều khiển trình duyệt trên local loopback
và tự động khởi động máy chủ relay. Extension nói chuyện với relay cục bộ; các lệnh CLI/tool đi đến Gateway.

### Gateway từ xa (Gateway chạy ở nơi khác) — **chạy một node host**

Nếu Gateway của bạn chạy trên một máy khác, hãy khởi động một node host trên máy chạy Chrome.
Gateway sẽ chuyển tiếp các hành động trình duyệt đến node đó; extension + relay vẫn ở cục bộ trên máy trình duyệt.

Nếu nhiều node được kết nối, hãy ghim một node với `gateway.nodes.browser.node` hoặc đặt `gateway.nodes.browser.mode`.
## Sandboxing (tool containers)

Nếu phiên agent của bạn được sandboxing (`agents.defaults.sandbox.mode != "off"`), công cụ `browser` có thể bị hạn chế:

- Theo mặc định, các phiên được sandboxing thường nhắm đến **trình duyệt sandbox** (`target="sandbox"`), không phải Chrome trên máy chủ của bạn.
- Tiếp sóng phần mở rộng Chrome yêu cầu kiểm soát **máy chủ điều khiển trình duyệt** trên máy chủ.

Tùy chọn:

- Dễ nhất: sử dụng phần mở rộng từ một phiên/agent **không được sandboxing**.
- Hoặc cho phép kiểm soát trình duyệt máy chủ cho các phiên được sandboxing:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Then ensure the tool isn’t denied by tool policy, and (if needed) call `browser` with `target="host"`.

Debugging: `openclaw sandbox explain`
## Mẹo truy cập từ xa

- Giữ Gateway và node host trên cùng một tailnet; tránh để lộ các cổng relay cho LAN hoặc Internet công cộng.
- Ghép nối các node một cách có chủ đích; vô hiệu hóa định tuyến proxy trình duyệt nếu bạn không muốn điều khiển từ xa (`gateway.nodes.browser.mode="off"`).
## Cách "extension path" hoạt động

`openclaw browser extension path` in ra **thư mục được cài đặt** trên đĩa chứa các tệp extension.

CLI cố ý **không** in ra đường dẫn `node_modules`. Luôn chạy `openclaw browser extension install` trước để sao chép extension đến một vị trí ổn định trong thư mục trạng thái OpenClaw của bạn.

Nếu bạn di chuyển hoặc xóa thư mục cài đặt đó, Chrome sẽ đánh dấu extension là bị hỏng cho đến khi bạn tải lại nó từ một đường dẫn hợp lệ.
## Hàm ý bảo mật (hãy đọc phần này)

Đây là một tính năng mạnh mẽ và rủi ro. Hãy coi nó như việc cấp cho mô hình "quyền truy cập trực tiếp vào trình duyệt của bạn".

- Tiện ích mở rộng sử dụng API trình gỡ lỗi của Chrome (`chrome.debugger`). Khi được kết nối, mô hình có thể:
  - nhấp chuột/gõ/điều hướng trong tab đó
  - đọc nội dung trang
  - truy cập bất kỳ thứ gì mà phiên đăng nhập của tab có thể truy cập
- **Điều này không được cách ly** như hồ sơ được quản lý riêng của openclaw.
  - Nếu bạn kết nối với hồ sơ/tab sử dụng hàng ngày của mình, bạn đang cấp quyền truy cập vào trạng thái tài khoản đó.

Khuyến nghị:

- Ưu tiên một hồ sơ Chrome riêng (tách biệt khỏi duyệt web cá nhân của bạn) để sử dụng relay tiện ích mở rộng.
- Giữ Gateway và bất kỳ node host nào chỉ trong tailnet; dựa vào xác thực Gateway + ghép nối node.
- Tránh phơi bày các cổng relay qua LAN (`0.0.0.0`) và tránh Funnel (công khai).
- Relay chặn các nguồn gốc không phải tiện ích mở rộng và yêu cầu xác thực gateway-token cho cả (`/cdp`) và (`/extension`).

Liên quan:

- Tổng quan công cụ trình duyệt: [Browser](/tools/browser)
- Kiểm toán bảo mật: [Security](/gateway/security)
- Thiết lập Tailscale: [Tailscale](/gateway/tailscale)