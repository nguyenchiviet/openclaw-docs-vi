---
summary: Luồng ứng dụng macOS để điều khiển một OpenClaw gateway từ xa qua SSH
read_when:
  - Thiết lập hoặc gỡ lỗi điều khiển Mac từ xa
title: Điều Khiển Từ Xa
x-i18n:
  source_path: platforms\mac\remote.md
  source_hash: 4bb945460f613e02cc26008d19400b1439ff1208fe93a1e4c956865ad0286fe1
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:49.539Z'
---

# OpenClaw từ xa (macOS ⇄ máy chủ từ xa)

Luồng này cho phép ứng dụng macOS hoạt động như một điều khiển từ xa hoàn toàn cho một Gateway OpenClaw chạy trên máy chủ khác (máy tính để bàn/máy chủ). Đây là tính năng **Remote over SSH** (chạy từ xa) của ứng dụng. Tất cả các tính năng—kiểm tra sức khỏe, chuyển tiếp Voice Wake và Web Chat—sử dụng lại cùng một cấu hình SSH từ xa từ _Settings → General_.
## Chế độ

- **Local (máy Mac này)**: Mọi thứ chạy trên laptop. Không liên quan đến SSH.
- **Remote over SSH (mặc định)**: Các lệnh OpenClaw được thực thi trên máy chủ từ xa. Ứng dụng Mac mở kết nối SSH với `-o BatchMode` cộng với danh tính/khóa được chọn của bạn và một port-forward cục bộ.
- **Remote direct (ws/wss)**: Không có SSH tunnel. Ứng dụng Mac kết nối trực tiếp đến URL gateway (ví dụ: qua Tailscale Serve hoặc một reverse proxy HTTPS công khai).
## Giao thức truyền tải từ xa

Chế độ từ xa hỗ trợ hai giao thức truyền tải:

- **SSH tunnel** (mặc định): Sử dụng `ssh -N -L ...` để chuyển tiếp cổng gateway đến localhost. Gateway sẽ thấy IP của node là `127.0.0.1` vì tunnel là local loopback.
- **Direct (ws/wss)**: Kết nối trực tiếp đến URL gateway. Gateway thấy IP máy khách thực tế.
## Điều kiện tiên quyết trên máy chủ từ xa

1. Cài đặt Node + pnpm và xây dựng/cài đặt OpenClaw CLI (`pnpm install && pnpm build && pnpm link --global`).
2. Đảm bảo `openclaw` nằm trên PATH cho các shell không tương tác (tạo symlink vào `/usr/local/bin` hoặc `/opt/homebrew/bin` nếu cần).
3. Mở SSH với xác thực khóa. Chúng tôi khuyên dùng **Tailscale** IPs để có khả năng tiếp cận ổn định ngoài LAN.
## Thiết lập ứng dụng macOS

1. Mở _Cài đặt → Chung_.
2. Dưới **OpenClaw runs**, chọn **Remote over SSH** và đặt:
   - **Transport**: **SSH tunnel** hoặc **Direct (ws/wss)**.
   - **SSH target**: `user@host` (tùy chọn `:port`).
     - Nếu gateway nằm trên cùng LAN và quảng cáo Bonjour, chọn nó từ danh sách được phát hiện để tự động điền vào trường này.
   - **Gateway URL** (Direct only): `wss://gateway.example.ts.net` (hoặc `ws://...` cho local/LAN).
   - **Identity file** (advanced): đường dẫn đến khóa của bạn.
   - **Project root** (advanced): đường dẫn checkout từ xa được sử dụng cho các lệnh.
   - **CLI path** (advanced): đường dẫn tùy chọn đến điểm vào `openclaw` có thể chạy được/nhị phân (tự động điền khi được quảng cáo).
3. Nhấn **Test remote**. Thành công cho biết remote `openclaw status --json` chạy chính xác. Lỗi thường có nghĩa là vấn đề PATH/CLI; exit 127 có nghĩa là CLI không được tìm thấy từ xa.
4. Kiểm tra sức khỏe và Web Chat sẽ chạy qua SSH tunnel này một cách tự động.
## Web Chat

- **SSH tunnel**: Web Chat kết nối với Gateway thông qua cổng điều khiển WebSocket được chuyển tiếp (mặc định 18789).
- **Direct (ws/wss)**: Web Chat kết nối trực tiếp đến URL Gateway được cấu hình.
- Không còn máy chủ WebChat HTTP riêng biệt nữa.
## Quyền hạn

- Máy chủ từ xa cần các phê duyệt TCC giống như máy cục bộ (Automation, Accessibility, Screen Recording, Microphone, Speech Recognition, Notifications). Chạy thiết lập ban đầu trên máy đó để cấp chúng một lần.
- Các node quảng cáo trạng thái quyền hạn của chúng thông qua `node.list` / `node.describe` để các agent biết những gì có sẵn.
## Ghi chú bảo mật

- Ưu tiên liên kết local loopback trên máy chủ từ xa và kết nối qua SSH hoặc Tailscale.
- SSH tunneling sử dụng kiểm tra khóa máy chủ nghiêm ngặt; hãy tin tưởng khóa máy chủ trước để nó tồn tại trong `~/.ssh/known_hosts`.
- Nếu bạn liên kết Gateway với giao diện không phải loopback, yêu cầu xác thực token/mật khẩu.
- Xem [Bảo mật](/gateway/security) và [Tailscale](/gateway/tailscale).
## Luồng đăng nhập WhatsApp (từ xa)

- Chạy `openclaw channels login --verbose` **trên máy chủ từ xa**. Quét mã QR bằng WhatsApp trên điện thoại của bạn.
- Chạy lại đăng nhập trên máy chủ đó nếu xác thực hết hạn. Kiểm tra sức khỏe sẽ phát hiện các vấn đề về liên kết.
## Khắc phục sự cố

- **exit 127 / not found**: `openclaw` không nằm trên PATH cho các shell không phải login. Thêm nó vào `/etc/paths`, shell rc của bạn, hoặc tạo symlink vào `/usr/local/bin`/`/opt/homebrew/bin`.
- **Health probe failed**: kiểm tra khả năng tiếp cận SSH, PATH, và xác nhận Baileys đã đăng nhập (`openclaw status --json`).
- **Web Chat stuck**: xác nhận gateway đang chạy trên máy chủ từ xa và cổng được chuyển tiếp khớp với cổng WS của gateway; giao diện người dùng yêu cầu kết nối WS lành mạnh.
- **Node IP shows 127.0.0.1**: dự kiến với SSH tunnel. Chuyển **Transport** sang **Direct (ws/wss)** nếu bạn muốn gateway nhìn thấy IP máy khách thực.
- **Voice Wake**: các cụm từ kích hoạt được chuyển tiếp tự động ở chế độ từ xa; không cần forwarder riêng biệt.
## Âm thanh thông báo

Chọn âm thanh cho mỗi thông báo từ các script có `openclaw` và `node.invoke`, ví dụ:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

Không còn công tắc "âm thanh mặc định" toàn cục trong ứng dụng nữa; những người gọi chọn một âm thanh (hoặc không có) cho mỗi yêu cầu.