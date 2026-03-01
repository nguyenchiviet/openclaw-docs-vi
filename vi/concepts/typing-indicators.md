---
summary: Khi OpenClaw hiển thị chỉ báo đang gõ và cách điều chỉnh chúng
read_when:
  - Thay đổi hành vi hoặc cài đặt mặc định của chỉ báo đang gõ
title: Chỉ báo Gõ
x-i18n:
  source_path: concepts\typing-indicators.md
  source_hash: 8ee82d02829c4ff58462be8bf5bb52f23f519aeda816c2fd8a583e7a317a2e98
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:51:49.186Z'
---

# Chỉ báo đang gõ

Chỉ báo đang gõ được gửi đến kênh trò chuyện trong khi một lần chạy đang hoạt động. Sử dụng
`agents.defaults.typingMode` để kiểm soát **khi nào** gõ bắt đầu và `typingIntervalSeconds`
để kiểm soát **tần suất** làm mới.
## Mặc định

Khi `agents.defaults.typingMode` **không được đặt**, OpenClaw giữ lại hành vi cũ:

- **Trò chuyện trực tiếp**: gõ bắt đầu ngay khi vòng lặp mô hình bắt đầu.
- **Trò chuyện nhóm có đề cập**: gõ bắt đầu ngay lập tức.
- **Trò chuyện nhóm không có đề cập**: gõ chỉ bắt đầu khi văn bản tin nhắn bắt đầu truyền phát.
- **Chạy nhịp tim**: gõ bị vô hiệu hóa.
## Chế độ

Đặt `agents.defaults.typingMode` thành một trong các giá trị sau:

- `never` — không có chỉ báo đang nhập, bao giờ hết.
- `instant` — bắt đầu nhập **ngay khi vòng lặp mô hình bắt đầu**, ngay cả khi lần chạy sau này chỉ trả về token trả lời im lặng.
- `thinking` — bắt đầu nhập trên **delta lý luận đầu tiên** (yêu cầu `reasoningLevel: "stream"` cho lần chạy).
- `message` — bắt đầu nhập trên **delta văn bản không im lặng đầu tiên** (bỏ qua token im lặng `NO_REPLY`).

Thứ tự "kích hoạt sớm như thế nào":
`never` → `message` → `thinking` → `instant`
## Cấu hình

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6,
  },
}
```

You can override mode or cadence per session:

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4,
  },
}
```
## Ghi chú

- Chế độ `message` sẽ không hiển thị trạng thái đang gõ cho các phản hồi chỉ im lặng (ví dụ: token `NO_REPLY`
  được sử dụng để loại bỏ đầu ra).
- `thinking` chỉ kích hoạt nếu lần chạy truyền phát lý luận (`reasoningLevel: "stream"`).
  Nếu mô hình không phát ra các delta lý luận, trạng thái đang gõ sẽ không bắt đầu.
- Nhịp tim không bao giờ hiển thị trạng thái đang gõ, bất kể chế độ nào.
- `typingIntervalSeconds` kiểm soát **tần suất làm mới**, không phải thời gian bắt đầu.
  Giá trị mặc định là 6 giây.