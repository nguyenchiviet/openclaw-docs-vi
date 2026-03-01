---
summary: 'Context window + compaction: cách OpenClaw giữ các phiên dưới giới hạn mô hình'
read_when:
  - Bạn muốn hiểu về auto-compaction và /compact
  - Bạn đang gỡ lỗi các phiên dài vượt quá giới hạn ngữ cảnh
title: Nén dữ liệu
x-i18n:
  source_path: concepts\compaction.md
  source_hash: 88b89bd640e878819a17b0db10b0bf77da01856b9b0435a2dce4a88fef7f387b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:38:51.527Z'
---

# Cửa sổ Ngữ cảnh & Nén dữ liệu

Mỗi mô hình có một **cửa sổ ngữ cảnh** (số token tối đa nó có thể nhìn thấy). Các cuộc trò chuyện chạy lâu dài tích lũy tin nhắn và kết quả công cụ; khi cửa sổ bị chặt chẽ, OpenClaw **nén** lịch sử cũ để ở trong giới hạn.
## Compaction là gì

Compaction **tóm tắt các cuộc trò chuyện cũ hơn** thành một mục tóm tắt gọn gàng và giữ nguyên các tin nhắn gần đây. Bản tóm tắt được lưu trữ trong lịch sử phiên, vì vậy các yêu cầu trong tương lai sử dụng:

- Bản tóm tắt compaction
- Các tin nhắn gần đây sau điểm compaction

Compaction **được lưu trữ** trong lịch sử JSONL của phiên.
## Cấu hình

Sử dụng cài đặt `agents.defaults.compaction` trong `openclaw.json` của bạn để cấu hình hành vi nén (chế độ, mã thông báo mục tiêu, v.v.).
Tóm tắt nén bảo toàn các định danh không trong suốt theo mặc định (`identifierPolicy: "strict"`). Bạn có thể ghi đè điều này bằng `identifierPolicy: "off"` hoặc cung cấp văn bản tùy chỉnh bằng `identifierPolicy: "custom"` và `identifierInstructions`.
## Nén tự động (bật theo mặc định)

Khi một phiên tiến gần hoặc vượt quá cửa sổ ngữ cảnh của mô hình, OpenClaw kích hoạt nén tự động và có thể thử lại yêu cầu ban đầu bằng cách sử dụng ngữ cảnh đã nén.

Bạn sẽ thấy:

- `🧹 Auto-compaction complete` ở chế độ chi tiết
- `/status` hiển thị `🧹 Compactions: <count>`

Trước khi nén, OpenClaw có thể chạy một lượt **xóa bộ nhớ im lặng** để lưu trữ các ghi chú bền vững vào đĩa. Xem [Bộ nhớ](/concepts/memory) để biết chi tiết và cấu hình.
## Nén dữ liệu thủ công

Sử dụng `/compact` (tùy chọn kèm theo hướng dẫn) để buộc thực hiện một lần nén:

```
/compact Focus on decisions and open questions
```
## Nguồn cửa sổ ngữ cảnh

Cửa sổ ngữ cảnh là dành riêng cho mô hình. OpenClaw sử dụng định nghĩa mô hình từ danh mục nhà cung cấp được cấu hình để xác định giới hạn.
## Nén dữ liệu so với cắt tỉa

- **Nén dữ liệu**: tóm tắt và **lưu trữ** trong JSONL.
- **Cắt tỉa phiên**: cắt bỏ các **kết quả công cụ** cũ, **trong bộ nhớ**, cho mỗi yêu cầu.

Xem [/concepts/session-pruning](/concepts/session-pruning) để biết chi tiết về cắt tỉa.
## Nén dữ liệu phía máy chủ OpenAI

OpenClaw cũng hỗ trợ các gợi ý nén Responses phía máy chủ OpenAI cho các mô hình OpenAI trực tiếp tương thích. Điều này tách biệt với nén OpenClaw cục bộ và có thể chạy song song với nó.

- Nén cục bộ: OpenClaw tóm tắt và lưu trữ vào JSONL phiên.
- Nén phía máy chủ: OpenAI nén ngữ cảnh phía nhà cung cấp khi `store` + `context_management` được bật.

Xem [nhà cung cấp OpenAI](/providers/openai) để biết các tham số mô hình và ghi đè.
## Mẹo

- Sử dụng `/compact` khi các phiên cảm thấy cũ hoặc ngữ cảnh bị quá tải.
- Các kết quả công cụ lớn đã được cắt ngắn; cắt tỉa có thể giảm thêm sự tích tụ kết quả công cụ.
- Nếu bạn cần một bảng tính mới, `/new` hoặc `/reset` bắt đầu một id phiên mới.