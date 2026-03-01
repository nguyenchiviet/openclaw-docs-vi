---
summary: Ghi chú giao thức RPC cho trình hướng dẫn onboarding và schema cấu hình
read_when: Changing onboarding wizard steps or config schema endpoints
title: Giao thức Onboarding và Cấu hình
x-i18n:
  source_path: experiments\onboarding-config-protocol.md
  source_hash: 55163b3ee029c02476800cb616a054e5adfe97dae5bb72f2763dce0079851e06
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:53.642Z'
---

# Thiết lập ban đầu + Giao thức Cấu hình

Mục đích: các bề mặt thiết lập ban đầu + cấu hình được chia sẻ trên CLI, ứng dụng macOS và Web UI.

## Thành phần

- Trình hướng dẫn (phiên được chia sẻ + lời nhắc + trạng thái thiết lập ban đầu).
- Thiết lập ban đầu CLI sử dụng cùng luồng trình hướng dẫn như các ứng dụng UI.
- Gateway RPC hiển thị các điểm cuối trình hướng dẫn + lược đồ cấu hình.
- Thiết lập ban đầu macOS sử dụng mô hình bước trình hướng dẫn.
- Web UI hiển thị các biểu mẫu cấu hình từ JSON Schema + gợi ý UI.

## Gateway RPC

- `wizard.start` params: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` params: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` params: `{ sessionId }`
- `wizard.status` params: `{ sessionId }`
- `config.schema` params: `{}`

Phản hồi (hình dạng)

- Trình hướng dẫn: `{ sessionId, done, step?, status?, error? }`
- Lược đồ cấu hình: `{ schema, uiHints, version, generatedAt }`

## Gợi ý UI

- `uiHints` được khóa theo đường dẫn; siêu dữ liệu tùy chọn (nhãn/trợ giúp/nhóm/thứ tự/nâng cao/nhạy cảm/trình giữ chỗ).
- Các trường nhạy cảm được hiển thị dưới dạng đầu vào mật khẩu; không có lớp biên tập lại.
- Các nút lược đồ không được hỗ trợ quay lại trình chỉnh sửa JSON thô.

## Ghi chú

- Tài liệu này là nơi duy nhất để theo dõi các tái cấu trúc giao thức cho thiết lập ban đầu/cấu hình.