---
summary: 'Khám phá: cấu hình mô hình, hồ sơ xác thực và hành vi dự phòng'
read_when:
  - Khám phá các ý tưởng về lựa chọn mô hình trong tương lai + hồ sơ xác thực
title: Khám Phá Cấu Hình Mô Hình
x-i18n:
  source_path: experiments\proposals\model-config.md
  source_hash: 48623233d80f874c0ae853b51f888599cf8b50ae6fbfe47f6d7b0216bae9500b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:55:09.398Z'
---

# Cấu hình Mô hình (Khám phá)

Tài liệu này ghi lại **các ý tưởng** cho cấu hình mô hình trong tương lai. Đây không phải là thông số kỹ thuật chính thức. Để biết hành vi hiện tại, xem:

- [Models](/concepts/models)
- [Model failover](/concepts/model-failover)
- [OAuth + profiles](/concepts/oauth)

## Động lực

Các nhà điều hành muốn:

- Nhiều hồ sơ xác thực cho mỗi nhà cung cấp (cá nhân vs công việc).
- Lựa chọn `/model` đơn giản với các fallback có thể dự đoán được.
- Tách biệt rõ ràng giữa các mô hình văn bản và các mô hình có khả năng xử lý hình ảnh.

## Hướng đi có thể (cấp cao)

- Giữ lựa chọn mô hình đơn giản: `provider/model` với các bí danh tùy chọn.
- Cho phép các nhà cung cấp có nhiều hồ sơ xác thực, với một thứ tự rõ ràng.
- Sử dụng danh sách fallback toàn cục để tất cả các phiên failover một cách nhất quán.
- Chỉ ghi đè định tuyến hình ảnh khi được cấu hình rõ ràng.

## Các câu hỏi mở

- Xoay vòng hồ sơ có nên theo nhà cung cấp hay theo mô hình?
- Giao diện người dùng nên hiển thị lựa chọn hồ sơ cho một phiên như thế nào?
- Đường dẫn di chuyển an toàn nhất từ các khóa cấu hình kế thừa là gì?