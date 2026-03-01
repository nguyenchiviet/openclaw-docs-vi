---
summary: Kiến trúc liên kết phiên độc lập với kênh và phạm vi giao hàng lần 1
read_when:
  - Tái cấu trúc định tuyến phiên và ràng buộc độc lập với kênh
  - >-
    Điều tra việc phát hành phiên bản trùng lặp, lỗi thời hoặc bị thiếu trên các
    kênh
owner: onutc
status: in-progress
last_updated: '2026-02-21'
title: Kế hoạch Liên kết Phiên không phụ thuộc Kênh
x-i18n:
  source_path: experiments\plans\session-binding-channel-agnostic.md
  source_hash: 9bf79098053da3d7a28ec4f7fe8289b5c39462c23b5ca1420a83ffb02191c7cd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:55:29.237Z'
---

# Kế hoạch Liên kết Phiên Không Phụ Thuộc Kênh

## Tổng quan

Tài liệu này định nghĩa mô hình liên kết phiên không phụ thuộc kênh dài hạn và phạm vi cụ thể cho lần lặp triển khai tiếp theo.

Mục tiêu:

- làm cho định tuyến phiên liên kết subagent trở thành một khả năng cốt lõi
- giữ hành vi cụ thể của kênh trong các adapter
- tránh các hồi quy trong hành vi Discord bình thường
## Tại sao điều này tồn tại

Hành vi hiện tại trộn lẫn:

- chính sách nội dung hoàn thành
- chính sách định tuyến đích
- chi tiết cụ thể của Discord

Điều này gây ra các trường hợp đặc biệt như:

- gửi trùng lặp chính và luồng dưới các lần chạy đồng thời
- sử dụng token cũ trên các trình quản lý ràng buộc được sử dụng lại
- thiếu kế toán hoạt động cho các lần gửi webhook
## Phạm vi Iteration 1

Iteration này được giới hạn một cách có chủ đích.

### 1. Thêm các giao diện cốt lõi không phụ thuộc vào kênh

Thêm các loại cốt lõi và giao diện dịch vụ cho các ràng buộc và định tuyến.

Các loại cốt lõi được đề xuất:

```ts
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

Core service contract:

```ts
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2. Thêm một bộ định tuyến giao hàng cốt lõi cho các hoàn thành subagent

Thêm một đường dẫn phân giải đích duy nhất cho các sự kiện hoàn thành.
Hợp đồng Router:

```ts
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

For this iteration:

- only `task_completion` is routed through this new path
- existing paths for other event kinds remain as-is

### 3. Keep Discord as adapter

Discord remains the first adapter implementation.

Adapter responsibilities:

- create/reuse thread conversations
- send bound messages via webhook or channel send
- validate thread state (archived/deleted)
- map adapter metadata (webhook identity, thread ids)

### 4. Fix currently known correctness issues

Required in this iteration:

- refresh token usage when reusing existing thread binding manager
- record outbound activity for webhook based Discord sends
- stop implicit main channel fallback when a bound thread destination is selected for session mode completion

### 5. Preserve current runtime safety defaults

No behavior change for users with thread bound spawn disabled.

Defaults stay:

- `channels.discord.threadBindings.spawnSubagentSessions = false`

Kết quả:

- người dùng Discord thông thường giữ nguyên hành vi hiện tại
- đường dẫn cốt lõi mới chỉ ảnh hưởng đến định tuyến hoàn thành phiên được ràng buộc nơi được bật
## Không có trong lần lặp 1

Được hoãn lại một cách rõ ràng:

- Các mục tiêu liên kết ACP (`targetKind: "acp"`)
- các bộ điều hợp kênh mới ngoài Discord
- thay thế toàn cầu tất cả các đường dẫn giao hàng (`spawn_ack`, tương lai `subagent_message`)
- các thay đổi ở cấp độ giao thức
- thiết kế lại di chuyển/phiên bản lưu trữ cho tất cả tính bền vững liên kết

Ghi chú về ACP:

- thiết kế giao diện giữ chỗ cho ACP
- triển khai ACP chưa bắt đầu trong lần lặp này
## Bất biến định tuyến

Những bất biến này là bắt buộc cho lần lặp 1.

- lựa chọn đích đến và tạo nội dung là các bước riêng biệt
- nếu hoàn thành chế độ phiên phân giải thành một đích đến được liên kết hoạt động, việc gửi phải nhắm mục tiêu đến đích đến đó
- không có định tuyến lại ẩn từ đích đến được liên kết đến kênh chính
- hành vi dự phòng phải rõ ràng và có thể quan sát được
## Tương thích và triển khai

Mục tiêu tương thích:

- không có hồi quy cho người dùng với thread bound spawning tắt
- không thay đổi các kênh không phải Discord trong lần lặp này

Triển khai:

1. Đặt các giao diện và router phía sau các cổng tính năng hiện tại.
2. Định tuyến các lần gửi hoàn thành chế độ Discord bound thông qua router.
3. Giữ đường dẫn kế thừa cho các luồng không bound.
4. Xác minh bằng các bài kiểm tra được nhắm mục tiêu và nhật ký runtime canary.
## Các bài kiểm tra cần thiết trong lần lặp 1

Yêu cầu về độ phủ unit và integration:

- quay vòng token manager sử dụng token mới nhất sau khi sử dụng lại manager
- webhook gửi cập nhật dấu thời gian hoạt động kênh
- hai phiên bound hoạt động trong cùng một kênh requester không trùng lặp đến kênh chính
- hoàn thành cho chế độ phiên bound chạy phân giải đến đích thread chỉ
- cờ spawn bị vô hiệu hóa giữ nguyên hành vi cũ không thay đổi
## Các tệp triển khai được đề xuất

Core:

- `src/infra/outbound/session-binding-service.ts` (mới)
- `src/infra/outbound/bound-delivery-router.ts` (mới)
- `src/agents/subagent-announce.ts` (tích hợp phân giải điểm đến hoàn thành)

Discord adapter và runtime:

- `src/discord/monitor/thread-bindings.manager.ts`
- `src/discord/monitor/reply-delivery.ts`
- `src/discord/send.outbound.ts`

Tests:

- `src/discord/monitor/provider*.test.ts`
- `src/discord/monitor/reply-delivery.test.ts`
- `src/agents/subagent-announce.format.test.ts`
## Tiêu chí hoàn thành cho lần lặp 1

- các giao diện cốt lõi tồn tại và được kết nối cho định tuyến hoàn thành
- các bản sửa lỗi về tính chính xác ở trên được hợp nhất với các bài kiểm tra
- không có phân phối hoàn thành trùng lặp giữa main và thread trong các lần chạy ở chế độ phiên bị ràng buộc
- không có thay đổi hành vi cho các triển khai spawn bị ràng buộc bị vô hiệu hóa
- ACP vẫn bị hoãn lại một cách rõ ràng