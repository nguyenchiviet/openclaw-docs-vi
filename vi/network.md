---
summary: 'Trung tâm mạng: các bề mặt gateway, ghép nối, khám phá và bảo mật'
read_when:
  - Bạn cần kiến trúc mạng + tổng quan bảo mật
  - Bạn đang gỡ lỗi truy cập cục bộ so với truy cập tailnet hoặc ghép nối
  - Bạn muốn danh sách chính thức của các tài liệu về mạng
title: Mạng
x-i18n:
  source_path: network.md
  source_hash: 6a0d5080db73de4c21d9bf376059f6c4a26ab129c8280ce6b1f54fa9ace48beb
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:10:01.766Z'
---

# Trung tâm mạng

Trung tâm này liên kết các tài liệu cốt lõi về cách OpenClaw kết nối, ghép nối và bảo mật các thiết bị trên localhost, LAN và tailnet.

## Mô hình cốt lõi

- [Kiến trúc Gateway](/concepts/architecture)
- [Giao thức Gateway](/gateway/protocol)
- [Sổ tay chạy Gateway](/gateway)
- [Bề mặt web + chế độ liên kết](/web)

## Ghép nối + nhận dạng

- [Tổng quan ghép nối](/channels/pairing)
- [Ghép nối node do Gateway sở hữu](/gateway/pairing)
- [CLI Thiết bị](/cli/devices)
- [CLI Ghép nối](/cli/pairing)

Tin cậy cục bộ:

- Các kết nối cục bộ (loopback hoặc địa chỉ tailnet của chính host Gateway) có thể được tự động phê duyệt để ghép nối nhằm giữ trải nghiệm UX trên cùng một host mượt mà.
- Các client tailnet/LAN không cục bộ vẫn yêu cầu phê duyệt ghép nối rõ ràng.

## Khám phá + giao thức truyền tải

- [Khám phá & giao thức truyền tải](/gateway/discovery)
- [Bonjour / mDNS](/gateway/bonjour)
- [Truy cập từ xa](/gateway/remote)
- [Tailscale](/gateway/tailscale)

## Node + giao thức truyền tải

- [Tổng quan Node](/nodes)
- [Giao thức Bridge](/gateway/bridge-protocol)
- [Sổ tay chạy Node: iOS](/platforms/ios)
- [Sổ tay chạy Node: Android](/platforms/android)

## Bảo mật

- [Tổng quan bảo mật](/gateway/security)
- [Tham chiếu cấu hình Gateway](/gateway/configuration)
- [Khắc phục sự cố](/gateway/troubleshooting)
- [Doctor](/gateway/doctor)