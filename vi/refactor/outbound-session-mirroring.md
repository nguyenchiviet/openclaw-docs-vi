---
title: Tái cấu trúc Phản chiếu Phiên Gửi Đi (Issue
description: >-
  Track outbound session mirroring refactor notes, decisions, tests, and open
  items.
summary: Ghi chú tái cấu trúc để phản chiếu các lần gửi đi vào các phiên kênh đích
read_when:
  - Đang làm việc trên hành vi phản chiếu phiên/bản ghi gửi đi
  - Gỡ lỗi việc lấy derivation sessionKey cho các đường dẫn send/message tool
x-i18n:
  source_path: refactor\outbound-session-mirroring.md
  source_hash: 45e457bcea47dfbffc5f30a852914f33051b7d8d9ed27119be8005262eb22f70
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:18:07.330Z'
---

# Tái cấu trúc Mirroring Phiên Gửi Đi (Issue #1520)

## Trạng thái

- Đang tiến hành.
- Định tuyến kênh Core + plugin được cập nhật cho mirroring gửi đi.
- Gateway send hiện nay suy ra phiên đích khi sessionKey bị bỏ qua.
## Bối cảnh

Các lần gửi đi được sao chép vào phiên agent _hiện tại_ (khóa phiên công cụ) thay vì phiên kênh đích. Định tuyến đến sử dụng các khóa phiên kênh/ngang hàng, do đó các phản hồi đi lại đã hạ cánh vào phiên sai và các mục tiêu liên hệ lần đầu thường thiếu các mục nhập phiên.
## Mục tiêu

- Sao chép các tin nhắn gửi đi vào khóa phiên kênh đích.
- Tạo các mục nhập phiên khi gửi đi nếu chưa có.
- Giữ cho phạm vi chủ đề/topic phù hợp với các khóa phiên gửi đến.
- Bao gồm các kênh cốt lõi cộng với các tiện ích mở rộng được đi kèm.
## Tóm tắt Triển khai

- Trợ giúp định tuyến phiên gửi đi mới:
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` xây dựng sessionKey đích bằng cách sử dụng `buildAgentSessionKey` (dmScope + identityLinks).
  - `ensureOutboundSessionEntry` ghi `MsgContext` tối thiểu thông qua `recordSessionMetaFromInbound`.
- `runMessageAction` (send) lấy sessionKey đích và chuyển nó tới `executeSendAction` để sao chép.
- `message-tool` không còn sao chép trực tiếp; nó chỉ giải quyết agentId từ khóa phiên hiện tại.
- Đường dẫn gửi Plugin sao chép thông qua `appendAssistantMessageToSessionTranscript` bằng cách sử dụng sessionKey được lấy.
- Gateway send lấy khóa phiên đích khi không có khóa nào được cung cấp (agent mặc định) và đảm bảo một mục nhập phiên.
## Xử lý Luồng/Chủ đề

- Slack: replyTo/threadId -> `resolveThreadSessionKeys` (hậu tố).
- Discord: threadId/replyTo -> `resolveThreadSessionKeys` với `useSuffix=false` để khớp với inbound (thread channel id đã xác định phạm vi phiên).
- Telegram: topic IDs ánh xạ tới `chatId:topic:<id>` qua `buildTelegramGroupPeerId`.
## Các Tiện ích Mở rộng Được Hỗ trợ

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- Ghi chú:
  - Các mục tiêu Mattermost hiện loại bỏ `@` để định tuyến khóa phiên DM.
  - Zalo Personal sử dụng loại peer DM cho các mục tiêu 1:1 (chỉ nhóm khi `group:` có mặt).
  - Các mục tiêu nhóm BlueBubbles loại bỏ tiền tố `chat_*` để khớp với các khóa phiên inbound.
  - Slack auto-thread mirroring khớp các id kênh không phân biệt chữ hoa chữ thường.
  - Gateway send chuyển đổi các khóa phiên được cung cấp thành chữ thường trước khi mirroring.
## Quyết định

- **Phát sinh phiên của Gateway**: nếu `sessionKey` được cung cấp, hãy sử dụng nó. Nếu bị bỏ qua, hãy phát sinh một sessionKey từ target + agent mặc định và sao chép ở đó.
- **Tạo mục nhập phiên**: luôn sử dụng `recordSessionMetaFromInbound` với `Provider/From/To/ChatType/AccountId/Originating*` căn chỉnh theo các định dạng inbound.
- **Chuẩn hóa Target**: định tuyến outbound sử dụng các target đã được giải quyết (sau `resolveChannelTarget`) khi có sẵn.
- **Casing khóa phiên**: chuẩn hóa các khóa phiên thành chữ thường khi ghi và trong quá trình di chuyển.
## Các bài kiểm tra được thêm/cập nhật

- `src/infra/outbound/outbound.test.ts`
  - Khóa phiên luồng Slack.
  - Khóa chủ đề Telegram.
  - dmScope identityLinks với Discord.
- `src/agents/tools/message-tool.test.ts`
  - Lấy agentId từ khóa phiên (không có sessionKey được truyền qua).
- `src/gateway/server-methods/send.test.ts`
  - Lấy khóa phiên khi bị bỏ qua và tạo mục nhập phiên.
## Các mục mở / Theo dõi

- Plugin cuộc gọi thoại sử dụng các khóa `voice:<phone>` phiên tùy chỉnh. Ánh xạ đi ra không được chuẩn hóa ở đây; nếu công cụ tin nhắn nên hỗ trợ gửi cuộc gọi thoại, hãy thêm ánh xạ rõ ràng.
- Xác nhận xem có plugin bên ngoài nào sử dụng định dạng `From/To` không chuẩn hóa ngoài bộ đã đóng gói hay không.
## Các tệp được sửa đổi

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- Các bài kiểm tra trong:
  - `src/infra/outbound/outbound.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`