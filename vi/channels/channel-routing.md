---
summary: >-
  Quy tắc định tuyến theo từng kênh (WhatsApp, Telegram, Discord, Slack) và ngữ
  cảnh chia sẻ
read_when:
  - Thay đổi định tuyến kênh hoặc hành vi hộp thư đến
title: Định tuyến Kênh
x-i18n:
  source_path: channels\channel-routing.md
  source_hash: 05c34d5f0046921dafa639904527b9fe1476d1121da410411a104999dd94e217
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:14:57.083Z'
---

# Kênh & định tuyến

OpenClaw định tuyến các phản hồi **trở lại kênh nơi tin nhắn được gửi đến**. Mô hình không chọn kênh; việc định tuyến là xác định và được kiểm soát bởi cấu hình máy chủ.
## Thuật ngữ chính

- **Channel**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
- **AccountId**: phiên bản tài khoản theo kênh (khi được hỗ trợ).
- **AgentId**: không gian làm việc cô lập + kho lưu trữ phiên ("bộ não").
- **SessionKey**: khóa bucket được sử dụng để lưu trữ ngữ cảnh và kiểm soát đồng thời.
## Hình dạng khóa phiên (ví dụ)

Tin nhắn riêng được gộp vào phiên **chính** của agent:

- `agent:<agentId>:<mainKey>` (mặc định: `agent:main:main`)

Nhóm và kênh vẫn được cách ly theo từng kênh:

- Nhóm: `agent:<agentId>:<channel>:group:<id>`
- Kênh/phòng: `agent:<agentId>:<channel>:channel:<id>`

Chuỗi tin nhắn:

- Chuỗi tin nhắn Slack/Discord thêm `:thread:<threadId>` vào khóa cơ sở.
- Chủ đề diễn đàn Telegram nhúng `:topic:<topicId>` vào khóa nhóm.

Ví dụ:

- `agent:main:telegram:group:-1001234567890:topic:42`
- `agent:main:discord:channel:123456:thread:987654`
## Quy tắc định tuyến (cách chọn agent)

Định tuyến chọn **một agent** cho mỗi tin nhắn đến:

1. **Khớp peer chính xác** (`bindings` với `peer.kind` + `peer.id`).
2. **Khớp peer cha** (kế thừa luồng).
3. **Khớp guild + vai trò** (Discord) qua `guildId` + `roles`.
4. **Khớp guild** (Discord) qua `guildId`.
5. **Khớp team** (Slack) qua `teamId`.
6. **Khớp tài khoản** (`accountId` trên kênh).
7. **Khớp kênh** (bất kỳ tài khoản nào trên kênh đó, `accountId: "*"`).
8. **Agent mặc định** (`agents.list[].default`, nếu không có thì mục đầu tiên trong danh sách, dự phòng là `main`).

Khi một ràng buộc bao gồm nhiều trường khớp (`peer`, `guildId`, `teamId`, `roles`), **tất cả các trường được cung cấp phải khớp** để ràng buộc đó được áp dụng.

Agent được khớp xác định workspace và kho lưu trữ phiên nào được sử dụng.
## Nhóm phát sóng (chạy nhiều agent)

Nhóm phát sóng cho phép bạn chạy **nhiều agent** cho cùng một peer **khi OpenClaw thường sẽ trả lời** (ví dụ: trong nhóm WhatsApp, sau khi được đề cập/kích hoạt cổng).

Cấu hình:

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

Xem: [Nhóm phát sóng](/channels/broadcast-groups).
## Tổng quan cấu hình

- `agents.list`: định nghĩa agent có tên (workspace, mô hình, v.v.).
- `bindings`: ánh xạ các kênh/tài khoản/peer đến với agent.

Ví dụ:

```json5
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```
## Lưu trữ phiên

Các kho lưu trữ phiên nằm trong thư mục trạng thái (mặc định `~/.openclaw`):

- `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Bản ghi JSONL nằm cùng với kho lưu trữ

Bạn có thể ghi đè đường dẫn kho lưu trữ thông qua `session.store` và `{agentId}` templating.
## Hành vi WebChat

WebChat kết nối với **agent được chọn** và mặc định sử dụng phiên chính của agent. Vì vậy, WebChat cho phép bạn xem ngữ cảnh đa kênh của agent đó tại một nơi.
## Ngữ cảnh trả lời

Các phản hồi đến bao gồm:

- `ReplyToId`, `ReplyToBody`, và `ReplyToSender` khi có sẵn.
- Ngữ cảnh được trích dẫn được thêm vào `Body` dưới dạng khối `[Replying to ...]`.

Điều này nhất quán trên tất cả các kênh.