---
summary: Quy trình onboarding lần đầu tiên cho OpenClaw (ứng dụng macOS)
read_when:
  - Thiết kế trợ lý onboarding cho macOS
  - Thiết lập xác thực hoặc danh tính
title: Onboarding (Ứng dụng macOS)
sidebarTitle: 'Onboarding: macOS App'
x-i18n:
  source_path: start\onboarding.md
  source_hash: a431c679b8e5e94d1d9bca23516cc726a8c3fd9530b92cbd8bc30dba9c984b8f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:24:54.771Z'
---

# Thiết lập ban đầu (Ứng dụng macOS)

Tài liệu này mô tả **hiện tại** luồng thiết lập ban đầu lần đầu chạy. Mục tiêu là trải nghiệm "ngày 0" mượt mà: chọn nơi Gateway chạy, kết nối xác thực, chạy trình hướng dẫn, và để agent tự khởi động.
Để xem tổng quan chung về các đường dẫn thiết lập ban đầu, hãy xem [Tổng quan Thiết lập ban đầu](/start/onboarding-overview).

<Steps>
<Step title="Phê duyệt cảnh báo macOS">
<Frame>
<img src="/assets/macos-onboarding/01-macos-warning.jpeg" alt="" />
</Frame>
</Step>
<Step title="Phê duyệt tìm mạng cục bộ">
<Frame>
<img src="/assets/macos-onboarding/02-local-networks.jpeg" alt="" />
</Frame>
</Step>
<Step title="Chào mừng và thông báo bảo mật">
<Frame caption="Đọc thông báo bảo mật được hiển thị và quyết định tương ứng">
<img src="/assets/macos-onboarding/03-security-notice.png" alt="" />
</Frame>

Mô hình tin cậy bảo mật:

- Theo mặc định, OpenClaw là một agent cá nhân: một ranh giới nhà điều hành đáng tin cậy.
- Các thiết lập chia sẻ/đa người dùng yêu cầu khóa (chia tách ranh giới tin cậy, giữ quyền truy cập công cụ tối thiểu, và tuân theo [Bảo mật](/gateway/security)).

</Step>
<Step title="Cục bộ vs Từ xa">
<Frame>
<img src="/assets/macos-onboarding/04-choose-gateway.png" alt="" />
</Frame>

**Gateway** chạy ở đâu?

- **Mac này (Chỉ cục bộ):** thiết lập ban đầu có thể cấu hình xác thực và ghi thông tin xác thực cục bộ.
- **Từ xa (qua SSH/Tailnet):** thiết lập ban đầu **không** cấu hình xác thực cục bộ; thông tin xác thực phải tồn tại trên máy chủ gateway.
- **Cấu hình sau:** bỏ qua thiết lập và để ứng dụng không được cấu hình.

<Tip>
**Mẹo xác thực Gateway:**

- Trình hướng dẫn hiện tạo một **token** ngay cả cho local loopback, vì vậy các máy khách WS cục bộ phải xác thực.
- Nếu bạn vô hiệu hóa xác thực, bất kỳ quy trình cục bộ nào cũng có thể kết nối; chỉ sử dụng điều đó trên các máy hoàn toàn đáng tin cậy.
- Sử dụng **token** để truy cập đa máy hoặc liên kết không phải local loopback.

</Tip>
</Step>
<Step title="Quyền hạn">
<Frame caption="Chọn quyền hạn bạn muốn cấp cho OpenClaw">
<img src="/assets/macos-onboarding/05-permissions.png" alt="" />
</Frame>

Thiết lập ban đầu yêu cầu quyền hạn TCC cần thiết cho:

- Tự động hóa (AppleScript)
- Thông báo
- Khả năng truy cập
- Ghi hình màn hình
- Micrô
- Nhận dạng giọng nói
- Camera
- Vị trí

</Step>
<Step title="CLI">
  <Info>Bước này là tùy chọn</Info>
  Ứng dụng có thể cài đặt `openclaw` CLI toàn cục thông qua npm/pnpm để các quy trình terminal và tác vụ launchd hoạt động ngay lập tức.
</Step>
<Step title="Thiết lập ban đầu Chat (phiên chuyên dụng)">
  Sau khi thiết lập, ứng dụng mở một phiên thiết lập ban đầu chuyên dụng để agent có thể giới thiệu bản thân và hướng dẫn các bước tiếp theo. Điều này giữ hướng dẫn lần chạy đầu tiên riêng biệt với cuộc trò chuyện bình thường của bạn. Xem [Bootstrapping](/start/bootstrapping) để biết những gì xảy ra trên máy chủ Gateway trong lần chạy agent đầu tiên.
</Step>
</Steps>