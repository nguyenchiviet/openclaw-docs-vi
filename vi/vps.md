---
summary: Trung tâm lưu trữ VPS cho OpenClaw (Oracle/Fly/Hetzner/GCP/exe.dev)
read_when:
  - Bạn muốn chạy Gateway trong cloud
  - Bạn cần một bản đồ nhanh về các hướng dẫn VPS/hosting
title: Lưu trữ VPS
x-i18n:
  source_path: vps.md
  source_hash: 481fffccbb57d7efb26a2143ca9b32eaf8ab8bd87b4bc64ba20c4c82158e8653
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:53.927Z'
---

# Lưu trữ VPS

Hub này liên kết đến các hướng dẫn VPS/lưu trữ được hỗ trợ và giải thích cách hoạt động của các triển khai đám mây ở mức cao.

## Chọn nhà cung cấp

- **Railway** (một cú nhấp + thiết lập trình duyệt): [Railway](/install/railway)
- **Northflank** (một cú nhấp + thiết lập trình duyệt): [Northflank](/install/northflank)
- **Oracle Cloud (Always Free)**: [Oracle](/platforms/oracle) — $0/tháng (Always Free, ARM; dung lượng/đăng ký có thể khó khăn)
- **Fly.io**: [Fly.io](/install/fly)
- **Hetzner (Docker)**: [Hetzner](/install/hetzner)
- **GCP (Compute Engine)**: [GCP](/install/gcp)
- **exe.dev** (VM + proxy HTTPS): [exe.dev](/install/exe-dev)
- **AWS (EC2/Lightsail/free tier)**: cũng hoạt động tốt. Hướng dẫn video:
  [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## Cách hoạt động của các thiết lập đám mây

- **Gateway chạy trên VPS** và sở hữu trạng thái + không gian làm việc.
- Bạn kết nối từ laptop/điện thoại của mình thông qua **Control UI** hoặc **Tailscale/SSH**.
- Coi VPS là nguồn sự thật và **sao lưu** trạng thái + không gian làm việc.
- Mặc định an toàn: giữ Gateway trên local loopback và truy cập nó thông qua SSH tunnel hoặc Tailscale Serve.
  Nếu bạn liên kết với `lan`/`tailnet`, yêu cầu `gateway.auth.token` hoặc `gateway.auth.password`.

Truy cập từ xa: [Gateway remote](/gateway/remote)  
Hub Platforms: [Platforms](/platforms)

## Agent công ty dùng chung trên VPS

Đây là một thiết lập hợp lệ khi các người dùng nằm trong một ranh giới tin cậy (ví dụ: một nhóm công ty), và agent chỉ dành cho kinh doanh.

- Giữ nó trên một runtime chuyên dụng (VPS/VM/container + OS user/accounts chuyên dụng).
- Không ký runtime đó vào các tài khoản Apple/Google cá nhân hoặc các hồ sơ trình duyệt/trình quản lý mật khẩu cá nhân.
- Nếu người dùng đối kháng với nhau, hãy chia theo gateway/host/OS user.

Chi tiết mô hình bảo mật: [Security](/gateway/security)

## Sử dụng nodes với VPS

Bạn có thể giữ Gateway trong đám mây và ghép nối **nodes** trên các thiết bị cục bộ của mình
(Mac/iOS/Android/headless). Nodes cung cấp khả năng màn hình/camera/canvas cục bộ và `system.run`
trong khi Gateway vẫn ở trong đám mây.

Tài liệu: [Nodes](/nodes), [Nodes CLI](/cli/nodes)