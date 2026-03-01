---
summary: >-
  Trình hướng dẫn onboarding CLI: thiết lập có hướng dẫn cho gateway, workspace,
  channels và skills
read_when:
  - Chạy hoặc cấu hình trình hướng dẫn onboarding
  - Thiết lập một máy tính mới
title: Trợ lý Onboarding (CLI)
sidebarTitle: 'Onboarding: CLI'
x-i18n:
  source_path: start\wizard.md
  source_hash: d8d13df2f8eff5d1f13dd22196983295dae892764d310e7bf1af5adf5f1529f8
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:25:48.116Z'
---

# Trình hướng dẫn thiết lập ban đầu (CLI)

Trình hướng dẫn thiết lập ban đầu là cách **được khuyến nghị** để thiết lập OpenClaw trên macOS,
Linux, hoặc Windows (qua WSL2; được khuyến nghị mạnh mẽ).
Nó cấu hình một Gateway cục bộ hoặc kết nối Gateway từ xa, cộng với các kênh, Skills,
và các giá trị mặc định không gian làm việc trong một luồng hướng dẫn.

```bash
openclaw onboard
```

<Info>
Fastest first chat: open the Control UI (no channel setup needed). Run
`openclaw dashboard` and chat in the browser. Docs: [Dashboard](/web/dashboard).
</Info>

To reconfigure later:

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` does not imply non-interactive mode. For scripts, use `--non-interactive`.
</Note>

<Tip>
Recommended: set up a Brave Search API key so the agent can use `web_search`
(`web_fetch` works without a key). Easiest path: `openclaw configure --section web`
which stores `tools.web.search.apiKey`. Tài liệu: [Công cụ web](/tools/web).
</Tip>
## Bắt đầu nhanh vs Nâng cao

Trình hướng dẫn bắt đầu với **Bắt đầu nhanh** (mặc định) hoặc **Nâng cao** (kiểm soát toàn bộ).

<Tabs>
  <Tab title="Bắt đầu nhanh (mặc định)">
    - Gateway cục bộ (local loopback)
    - Workspace mặc định (hoặc workspace hiện có)
    - Cổng Gateway **18789**
    - Xác thực Gateway **Token** (tự động tạo, ngay cả trên local loopback)
    - Mặc định cách ly tin nhắn riêng: thiết lập ban đầu cục bộ ghi `session.dmScope: "per-channel-peer"` khi chưa được đặt. Chi tiết: [Tham khảo thiết lập ban đầu CLI](/start/wizard-cli-reference#outputs-and-internals)
    - Tiếp xúc Tailscale **Tắt**
    - Telegram + WhatsApp tin nhắn riêng mặc định là **danh sách cho phép** (bạn sẽ được nhắc nhập số điện thoại)
  </Tab>
  <Tab title="Nâng cao (kiểm soát toàn bộ)">
    - Hiển thị từng bước (chế độ, workspace, gateway, kênh, daemon, Skills).
  </Tab>
</Tabs>
## Trình hướng dẫn cấu hình những gì

**Chế độ cục bộ (mặc định)** hướng dẫn bạn qua các bước sau:

1. **Mô hình/Xác thực** — Khóa API Anthropic (được khuyến nghị), OpenAI, hoặc Nhà cung cấp tùy chỉnh
   (tương thích OpenAI, tương thích Anthropic, hoặc Tự động phát hiện không xác định). Chọn một mô hình mặc định.
   Đối với các lần chạy không tương tác, `--secret-input-mode ref` lưu trữ các tham chiếu được hỗ trợ bởi env trong các hồ sơ xác thực thay vì các giá trị khóa API dạng văn bản thuần.
   Ở chế độ `ref` không tương tác, biến môi trường nhà cung cấp phải được đặt; truyền các cờ khóa nội tuyến mà không có biến môi trường đó sẽ thất bại nhanh chóng.
   Trong các lần chạy tương tác, chọn chế độ tham chiếu bí mật cho phép bạn trỏ đến một biến môi trường hoặc một tham chiếu nhà cung cấp được cấu hình (`file` hoặc `exec`), với xác thực preflight nhanh trước khi lưu.
2. **Không gian làm việc** — Vị trí cho các tệp agent (mặc định `~/.openclaw/workspace`). Tạo các tệp bootstrap.
3. **Gateway** — Cổng, địa chỉ liên kết, chế độ xác thực, phơi bày Tailscale.
4. **Kênh** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles, hoặc iMessage.
5. **Daemon** — Cài đặt LaunchAgent (macOS) hoặc đơn vị người dùng systemd (Linux/WSL2).
6. **Kiểm tra sức khỏe** — Khởi động Gateway và xác minh nó đang chạy.
7. **Skills** — Cài đặt các Skills được khuyến nghị và các phụ thuộc tùy chọn.

<Note>
Chạy lại trình hướng dẫn **không** xóa bất cứ điều gì trừ khi bạn rõ ràng chọn **Đặt lại** (hoặc truyền `--reset`).
CLI `--reset` mặc định là cấu hình, thông tin xác thực và phiên; sử dụng `--reset-scope full` để bao gồm không gian làm việc.
Nếu cấu hình không hợp lệ hoặc chứa các khóa cũ, trình hướng dẫn yêu cầu bạn chạy `openclaw doctor` trước.
</Note>

**Chế độ từ xa** chỉ cấu hình máy khách cục bộ để kết nối với Gateway ở nơi khác.
Nó **không** cài đặt hoặc thay đổi bất cứ điều gì trên máy chủ từ xa.
## Thêm một agent khác

Sử dụng `openclaw agents add <name>` để tạo một agent riêng biệt với không gian làm việc,
phiên và hồ sơ xác thực của riêng nó. Chạy mà không có `--workspace` sẽ khởi chạy trình hướng dẫn.

Những gì nó đặt:

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

Ghi chú:

- Không gian làm việc mặc định tuân theo `~/.openclaw/workspace-<agentId>`.
- Thêm `bindings` để định tuyến các tin nhắn đến (trình hướng dẫn có thể làm điều này).
- Các cờ không tương tác: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.
## Tham chiếu đầy đủ

Để xem các hướng dẫn chi tiết từng bước, kịch bản không tương tác, thiết lập Signal,
RPC API và danh sách đầy đủ các trường cấu hình mà trình hướng dẫn ghi, hãy xem
[Tham chiếu Trình hướng dẫn](/reference/wizard).
## Tài liệu liên quan

- Tham chiếu lệnh CLI: [`openclaw onboard`](/cli/onboard)
- Tổng quan thiết lập ban đầu: [Tổng quan Thiết lập ban đầu](/start/onboarding-overview)
- Thiết lập ban đầu ứng dụng macOS: [Thiết lập ban đầu](/start/onboarding)
- Nghi thức chạy lần đầu của Agent: [Khởi động Agent](/start/bootstrapping)