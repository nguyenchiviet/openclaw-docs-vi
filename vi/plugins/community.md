---
summary: 'Các plugin cộng đồng: thanh chất lượng, yêu cầu lưu trữ và đường dẫn gửi PR'
read_when:
  - Bạn muốn xuất bản một plugin OpenClaw của bên thứ ba
  - Bạn muốn đề xuất một plugin để liệt kê tài liệu
title: Plugin cộng đồng
x-i18n:
  source_path: plugins\community.md
  source_hash: 5203b7a6973ed0551fe87d17cda2f6f6e35ce520fe7f8078b1840a445ed67642
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:45.378Z'
---

# Plugin cộng đồng

Trang này theo dõi các **plugin được duy trì bởi cộng đồng** chất lượng cao cho OpenClaw.

Chúng tôi chấp nhận PR thêm plugin cộng đồng tại đây khi chúng đáp ứng tiêu chuẩn chất lượng.

## Bắt buộc để liệt kê

- Gói plugin được xuất bản trên npmjs (có thể cài đặt qua `openclaw plugins install <npm-spec>`).
- Mã nguồn được lưu trữ trên GitHub (kho lưu trữ công khai).
- Kho lưu trữ bao gồm tài liệu thiết lập/sử dụng và trình theo dõi vấn đề.
- Plugin có tín hiệu bảo trì rõ ràng (người duy trì tích cực, cập nhật gần đây hoặc xử lý vấn đề phản hồi nhanh).

## Cách gửi

Mở PR thêm plugin của bạn vào trang này với:

- Tên plugin
- Tên gói npm
- URL kho lưu trữ GitHub
- Mô tả một dòng
- Lệnh cài đặt

## Tiêu chuẩn đánh giá

Chúng tôi ưa thích các plugin hữu ích, được ghi chép đầy đủ và an toàn để vận hành.
Các wrapper nỗ lực thấp, quyền sở hữu không rõ ràng hoặc các gói không được duy trì có thể bị từ chối.

## Định dạng ứng cử

Sử dụng định dạng này khi thêm mục:

- **Tên Plugin** — mô tả ngắn
  npm: `@scope/package`
  repo: `https://github.com/org/repo`
  install: `openclaw plugins install @scope/package`

## Plugin được liệt kê

- **WeChat** — Kết nối OpenClaw với các tài khoản cá nhân WeChat qua WeChatPadPro (giao thức iPad). Hỗ trợ trao đổi văn bản, hình ảnh và tệp với các cuộc trò chuyện được kích hoạt bằng từ khóa.
  npm: `@icesword760/openclaw-wechat`
  repo: `https://github.com/icesword0760/openclaw-wechat`
  install: `openclaw plugins install @icesword760/openclaw-wechat`