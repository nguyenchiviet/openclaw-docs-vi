---
summary: 'OAuth trong OpenClaw: trao đổi token, lưu trữ và các mô hình đa tài khoản'
read_when:
  - Bạn muốn hiểu OpenClaw OAuth từ đầu đến cuối
  - Bạn gặp phải các vấn đề vô hiệu hóa token / đăng xuất.
  - Bạn muốn setup-token hoặc luồng xác thực OAuth.
  - Bạn muốn nhiều tài khoản hoặc định tuyến hồ sơ
title: OAuth
x-i18n:
  source_path: concepts\oauth.md
  source_hash: e265b1115f9e510d43fa0d7bccbba68e576efdba82455bc31e22e24d81d62ccd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:07:47.186Z'
---

# OAuth

OpenClaw hỗ trợ “xác thực đăng ký” qua OAuth cho các nhà cung cấp có hỗ trợ (đáng chú ý là **OpenAI Codex (ChatGPT OAuth)**). Đối với các gói đăng ký Anthropic, hãy sử dụng luồng **setup-token**. Trang này giải thích:

- cách **trao đổi token** OAuth hoạt động (PKCE)
- nơi các token được **lưu trữ** (và tại sao)
- cách xử lý **nhiều tài khoản** (profiles + ghi đè theo phiên)

OpenClaw cũng hỗ trợ các **plugin nhà cung cấp** đi kèm với luồng OAuth hoặc API‑key riêng của chúng. Chạy chúng qua:

``__OC_I19N_0000__``

## Bộ chứa token (lý do tồn tại)

Các nhà cung cấp OAuth thường tạo một **refresh token mới** trong quá trình đăng nhập/làm mới. Một số nhà cung cấp (hoặc ứng dụng khách OAuth) có thể vô hiệu hóa các refresh token cũ hơn khi một token mới được cấp cho cùng một người dùng/ứng dụng.

Triệu chứng thực tế:

- bạn đăng nhập qua OpenClaw _và_ qua Claude Code / Codex CLI → một trong số chúng ngẫu nhiên bị “đăng xuất” sau đó

Để giảm thiểu điều đó, OpenClaw coi `auth-profiles.json` là một **bộ chứa token**:

- thời gian chạy đọc thông tin xác thực từ **một nơi duy nhất**
- chúng ta có thể giữ nhiều hồ sơ và định tuyến chúng một cách xác định

## Lưu trữ (nơi lưu trữ token)

Các bí mật được lưu trữ **theo từng agent**:

- Hồ sơ xác thực (OAuth + khóa API + tham chiếu cấp giá trị tùy chọn): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- Tệp tương thích cũ: `~/.openclaw/agents/<agentId>/agent/auth.json`
  (các mục `api_key` tĩnh sẽ bị xóa khi được phát hiện)

Tệp chỉ nhập cũ (vẫn được hỗ trợ, nhưng không phải kho lưu trữ chính):

- `~/.openclaw/credentials/oauth.json` (được nhập vào `auth-profiles.json` khi sử dụng lần đầu)

Tất cả những điều trên cũng tuân thủ `$OPENCLAW_STATE_DIR` (ghi đè thư mục trạng thái). Tham khảo đầy đủ: [/gateway/configuration](/gateway/configuration#auth-storage-oauth--api-keys)

Để biết các tham chiếu bí mật tĩnh và hành vi kích hoạt ảnh chụp nhanh thời gian chạy, hãy xem [Quản lý bí mật](/gateway/secrets).
## Thiết lập token Anthropic (xác thực đăng ký)

Chạy `claude setup-token` trên bất kỳ máy nào, sau đó dán vào OpenClaw:

```bash
openclaw models auth setup-token --provider anthropic
```

If you generated the token elsewhere, paste it manually:

```bash
openclaw models auth paste-token --provider anthropic
```

Verify:

```bash
openclaw models status
```
## Trao đổi OAuth (cách thức đăng nhập hoạt động)

Các luồng đăng nhập tương tác của OpenClaw được triển khai trong `@mariozechner/pi-ai` và được tích hợp vào các trình hướng dẫn/lệnh.

### Thiết lập token Anthropic (Claude Pro/Max)

Dạng luồng:

1. chạy `claude setup-token`
2. dán token vào OpenClaw
3. lưu trữ dưới dạng hồ sơ xác thực token (không làm mới)

Đường dẫn trình hướng dẫn là `openclaw onboard` → lựa chọn xác thực `setup-token` (Anthropic).

### OpenAI Codex (ChatGPT OAuth)

Dạng luồng (PKCE):

1. tạo trình xác minh/thử thách PKCE + `state` ngẫu nhiên
2. mở `https://auth.openai.com/oauth/authorize?...`
3. cố gắng bắt callback trên `http://127.0.0.1:1455/auth/callback`
4. nếu callback không thể liên kết (hoặc bạn đang ở xa/không có giao diện), dán URL/mã chuyển hướng
5. trao đổi tại `https://auth.openai.com/oauth/token`
6. trích xuất `accountId` từ access token và lưu trữ `{ access, refresh, expires, accountId }`

Đường dẫn trình hướng dẫn là `openclaw onboard` → lựa chọn xác thực `openai-codex`.
## Làm mới + hết hạn

Các hồ sơ lưu trữ một dấu thời gian `expires`.

Khi chạy:

- nếu `expires` là trong tương lai → sử dụng mã truy cập đã lưu trữ
- nếu đã hết hạn → làm mới (dưới khóa tệp) và ghi đè thông tin xác thực đã lưu trữ

Quy trình làm mới là tự động; bạn thường không cần quản lý mã thông báo theo cách thủ công.
## Nhiều tài khoản (hồ sơ) + định tuyến

Hai mô hình:

### 1) Ưu tiên: các agent riêng biệt

Nếu bạn muốn “cá nhân” và “công việc” không bao giờ tương tác với nhau, hãy sử dụng các agent biệt lập (phiên + thông tin xác thực + không gian làm việc riêng biệt):

```bash
openclaw agents add work
openclaw agents add personal
```

Then configure auth per-agent (wizard) and route chats to the right agent.

### 2) Advanced: multiple profiles in one agent

`auth-profiles.json` supports multiple profile IDs for the same provider.

Pick which profile is used:

- globally via config ordering (`auth.order`)
- per-session via `/model ...@<profileId>`

Example (session override):

- `/model Opus@anthropic:work`

Cách xem các ID hồ sơ hiện có:

- `openclaw channels list --json` (hiển thị `auth[]`)

Tài liệu liên quan:

- [/concepts/model-failover](/concepts/model-failover) (quy tắc xoay vòng + thời gian chờ)
- [/tools/slash-commands](/tools/slash-commands) (giao diện lệnh)