---
summary: 'Lệnh Doctor: kiểm tra sức khỏe, di chuyển cấu hình và các bước sửa chữa'
read_when:
  - Thêm hoặc sửa đổi doctor migrations
  - Giới thiệu các thay đổi cấu hình đột phá
title: Bác sĩ
x-i18n:
  source_path: gateway\doctor.md
  source_hash: 71e340b45a41d03ce6e7ea01aaeea5166d99e23e9731fe66a1c9354735fccf1c
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:57:13.215Z'
---

# Doctor

`openclaw doctor` là công cụ sửa chữa + di chuyển cho OpenClaw. Nó sửa chữa cấu hình/trạng thái cũ, kiểm tra sức khỏe và cung cấp các bước sửa chữa có thể thực hiện được.
## Bắt đầu nhanh

```bash
openclaw doctor
```

### Headless / automation

```bash
openclaw doctor --yes
```

Accept defaults without prompting (including restart/service/sandbox repair steps when applicable).

```bash
openclaw doctor --repair
```

Apply recommended repairs without prompting (repairs + restarts where safe).

```bash
openclaw doctor --repair --force
```

Apply aggressive repairs too (overwrites custom supervisor configs).

```bash
openclaw doctor --non-interactive
```

Run without prompts and only apply safe migrations (config normalization + on-disk state moves). Skips restart/service/sandbox actions that require human confirmation.
Legacy state migrations run automatically when detected.

```bash
openclaw doctor --deep
```

Scan system services for extra gateway installs (launchd/systemd/schtasks).

If you want to review changes before writing, open the config file first:

```bash
cat ~/.openclaw/openclaw.json
```
## Chức năng (tóm tắt)

- Cập nhật trước khi chạy tùy chọn cho cài đặt git (chỉ ở chế độ tương tác).
- Kiểm tra tính mới của giao thức UI (xây dựng lại Control UI khi lược đồ giao thức mới hơn).
- Kiểm tra sức khỏe + lời nhắc khởi động lại.
- Tóm tắt trạng thái Skills (đủ điều kiện/thiếu/bị chặn).
- Chuẩn hóa cấu hình cho các giá trị cũ.
- Cảnh báo ghi đè nhà cung cấp OpenCode Zen (`models.providers.opencode`).
- Di chuyển trạng thái trên đĩa cũ (phiên/thư mục agent/xác thực WhatsApp).
- Kiểm tra tính toàn vẹn trạng thái và quyền (phiên, bản ghi, thư mục trạng thái).
- Kiểm tra quyền tệp cấu hình (chmod 600) khi chạy cục bộ.
- Sức khỏe xác thực mô hình: kiểm tra hết hạn OAuth, có thể làm mới các token sắp hết hạn, và báo cáo trạng thái cooldown/bị vô hiệu hóa của hồ sơ xác thực.
- Phát hiện thư mục không gian làm việc bổ sung (`~/openclaw`).
- Sửa chữa hình ảnh sandbox khi sandboxing được bật.
- Di chuyển dịch vụ cũ và phát hiện gateway bổ sung.
- Kiểm tra runtime Gateway (dịch vụ được cài đặt nhưng không chạy; nhãn launchd được lưu trong bộ nhớ đệm).
- Cảnh báo trạng thái kênh (được kiểm tra từ gateway đang chạy).
- Kiểm tra cấu hình Supervisor (launchd/systemd/schtasks) với tùy chọn sửa chữa.
- Kiểm tra các phương pháp hay nhất runtime Gateway (Node vs Bun, đường dẫn trình quản lý phiên bản).
- Chẩn đoán va chạm cổng Gateway (mặc định `18789`).
- Cảnh báo bảo mật cho các chính sách DM mở.
- Cảnh báo xác thực Gateway khi không có `gateway.auth.token` được đặt (chế độ cục bộ; cung cấp tạo token).
- Kiểm tra systemd linger trên Linux.
- Kiểm tra cài đặt nguồn (không khớp không gian làm việc pnpm, thiếu tài sản UI, thiếu nhị phân tsx).
- Ghi cấu hình được cập nhật + siêu dữ liệu trình hướng dẫn.
## Hành vi chi tiết và lý do

### 0) Cập nhật tùy chọn (cài đặt git)

Nếu đây là một checkout git và doctor đang chạy tương tác, nó sẽ đề nghị
cập nhật (fetch/rebase/build) trước khi chạy doctor.

### 1) Chuẩn hóa cấu hình

Nếu cấu hình chứa các hình dạng giá trị cũ (ví dụ `messages.ackReaction`
mà không có ghi đè dành riêng cho kênh), doctor sẽ chuẩn hóa chúng thành
lược đồ hiện tại.

### 2) Di chuyển khóa cấu hình cũ

Khi cấu hình chứa các khóa không dùng nữa, các lệnh khác từ chối chạy và yêu cầu
bạn chạy `openclaw doctor`.

Doctor sẽ:

- Giải thích những khóa cũ nào được tìm thấy.
- Hiển thị quá trình di chuyển mà nó đã áp dụng.
- Viết lại `~/.openclaw/openclaw.json` với lược đồ được cập nhật.

Gateway cũng tự động chạy các quá trình di chuyển doctor khi khởi động khi nó phát hiện
định dạng cấu hình cũ, vì vậy các cấu hình cũ được sửa chữa mà không cần can thiệp thủ công.

Các quá trình di chuyển hiện tại:

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → `bindings` cấp cao nhất
- `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.media.audio.models`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- Đối với các kênh có tên `accounts` nhưng thiếu `accounts.default`, hãy di chuyển các giá trị kênh đơn tài khoản cấp cao nhất có phạm vi tài khoản vào `channels.<channel>.accounts.default` khi có
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
- `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

### 2b) Ghi đè nhà cung cấp OpenCode Zen

Nếu bạn đã thêm `models.providers.opencode` (hoặc `opencode-zen`) theo cách thủ công, nó
sẽ ghi đè danh mục OpenCode Zen tích hợp từ `@mariozechner/pi-ai`. Điều đó có thể
buộc mọi mô hình vào một API duy nhất hoặc loại bỏ chi phí. Doctor sẽ cảnh báo để bạn có thể
xóa ghi đè và khôi phục định tuyến API theo mô hình + chi phí.

### 3) Di chuyển trạng thái cũ (bố cục đĩa)

Doctor có thể di chuyển các bố cục cũ hơn trên đĩa vào cấu trúc hiện tại:

- Kho lưu trữ phiên + bản ghi:
  - từ `~/.openclaw/sessions/` đến `~/.openclaw/agents/<agentId>/sessions/`
- Thư mục agent:
- từ `~/.openclaw/agent/` đến `~/.openclaw/agents/<agentId>/agent/`
- Trạng thái xác thực WhatsApp (Baileys):
  - từ `~/.openclaw/credentials/*.json` cũ (ngoại trừ `oauth.json`)
  - đến `~/.openclaw/credentials/whatsapp/<accountId>/...` (id tài khoản mặc định: `default`)

Những quá trình di chuyển này là nỗ lực tốt nhất và idempotent; doctor sẽ phát hành cảnh báo khi nó để lại bất kỳ thư mục cũ nào làm bản sao lưu. Gateway/CLI cũng tự động di chuyển các phiên cũ + thư mục agent khi khởi động để lịch sử/xác thực/mô hình được đặt trong đường dẫn cho từng agent mà không cần chạy doctor thủ công. Xác thực WhatsApp được cố ý chỉ di chuyển qua `openclaw doctor`.

### 4) Kiểm tra tính toàn vẹn trạng thái (lưu trữ phiên, định tuyến và an toàn)

Thư mục trạng thái là thân kinh hoạt động. Nếu nó biến mất, bạn sẽ mất các phiên, thông tin xác thực, nhật ký và cấu hình (trừ khi bạn có bản sao lưu ở nơi khác).

Kiểm tra của Doctor:

- **Thư mục trạng thái bị thiếu**: cảnh báo về mất trạng thái thảm họa, nhắc nhở tạo lại thư mục và nhắc bạn rằng nó không thể khôi phục dữ liệu bị thiếu.
- **Quyền thư mục trạng thái**: xác minh khả năng ghi; cung cấp để sửa quyền (và phát hành gợi ý `chown` khi phát hiện không khớp chủ sở hữu/nhóm).
- **Thư mục phiên bị thiếu**: `sessions/` và thư mục lưu trữ phiên được yêu cầu để lưu trữ lịch sử và tránh các sự cố `ENOENT`.
- **Không khớp bản ghi**: cảnh báo khi các mục phiên gần đây có tệp bản ghi bị thiếu.
- **Phiên chính "1-line JSONL"**: đánh dấu khi bản ghi chính chỉ có một dòng (lịch sử không tích lũy).
- **Nhiều thư mục trạng thái**: cảnh báo khi nhiều thư mục `~/.openclaw` tồn tại trên các thư mục chính hoặc khi `OPENCLAW_STATE_DIR` trỏ đến nơi khác (lịch sử có thể chia tách giữa các bản cài đặt).
- **Nhắc nhở chế độ từ xa**: nếu `gateway.mode=remote`, doctor nhắc bạn chạy nó trên máy chủ từ xa (trạng thái nằm ở đó).
- **Quyền tệp cấu hình**: cảnh báo nếu `~/.openclaw/openclaw.json` có thể đọc được bởi nhóm/công khai và cung cấp để siết chặt thành `600`.

### 5) Sức khỏe xác thực mô hình (hết hạn OAuth)

Doctor kiểm tra các hồ sơ OAuth trong kho lưu trữ xác thực, cảnh báo khi các token sắp hết hạn/đã hết hạn và có thể làm mới chúng khi an toàn. Nếu hồ sơ Anthropic Claude Code bị cũ, nó gợi ý chạy `claude setup-token` (hoặc dán một token thiết lập). Các lời nhắc làm mới chỉ xuất hiện khi chạy tương tác (TTY); `--non-interactive` bỏ qua các nỗ lực làm mới.

Doctor cũng báo cáo các hồ sơ xác thực tạm thời không thể sử dụng do:

- thời gian chờ ngắn (giới hạn tốc độ/hết thời gian chờ/lỗi xác thực)
- vô hiệu hóa lâu hơn (lỗi thanh toán/tín dụng)

### 6) Xác thực mô hình Hooks

Nếu `hooks.gmail.model` được đặt, doctor xác thực tham chiếu mô hình dựa trên danh mục và danh sách cho phép và cảnh báo khi nó không thể phân giải hoặc bị cấm.

### 7) Sửa chữa hình ảnh Sandbox

Khi sandboxing được bật, doctor kiểm tra các hình ảnh Docker và cung cấp để xây dựng hoặc chuyển sang tên cũ nếu hình ảnh hiện tại bị thiếu.

### 8) Gợi ý di chuyển dịch vụ Gateway và dọn dẹp
Bác sĩ phát hiện các dịch vụ gateway cũ (launchd/systemd/schtasks) và
đề xuất xóa chúng cũng như cài đặt dịch vụ OpenClaw bằng cổng gateway hiện tại. Nó cũng có thể quét các dịch vụ giống gateway bổ sung và in gợi ý dọn dẹp.
Các dịch vụ gateway OpenClaw được đặt tên theo hồ sơ được coi là hạng nhất và không được đánh dấu là "bổ sung."

### 9) Cảnh báo bảo mật

Bác sĩ phát hành cảnh báo khi một nhà cung cấp mở cho tin nhắn riêng mà không có danh sách cho phép, hoặc
khi một chính sách được cấu hình theo cách nguy hiểm.

### 10) systemd linger (Linux)

Nếu chạy như một dịch vụ người dùng systemd, bác sĩ đảm bảo lingering được bật để gateway
vẫn hoạt động sau khi đăng xuất.

### 11) Trạng thái Skills

Bác sĩ in tóm tắt nhanh về các Skills đủ điều kiện/thiếu/bị chặn cho không gian làm việc hiện tại.

### 12) Kiểm tra xác thực Gateway (token cục bộ)

Bác sĩ cảnh báo khi `gateway.auth` bị thiếu trên gateway cục bộ và đề xuất
tạo token. Sử dụng `openclaw doctor --generate-gateway-token` để buộc tạo token
trong tự động hóa.

### 13) Kiểm tra sức khỏe Gateway + khởi động lại

Bác sĩ chạy kiểm tra sức khỏe và đề xuất khởi động lại gateway khi nó trông
không khỏe.

### 14) Cảnh báo trạng thái kênh

Nếu gateway khỏe mạnh, bác sĩ chạy kiểm tra trạng thái kênh và báo cáo
cảnh báo với các sửa chữa được đề xuất.

### 15) Kiểm tra cấu hình Supervisor + sửa chữa

Bác sĩ kiểm tra cấu hình supervisor được cài đặt (launchd/systemd/schtasks) để tìm
các giá trị mặc định bị thiếu hoặc lỗi thời (ví dụ: phụ thuộc network-online của systemd và
độ trễ khởi động lại). Khi nó tìm thấy sự không khớp, nó khuyến nghị cập nhật và có thể
viết lại tệp dịch vụ/tác vụ thành các giá trị mặc định hiện tại.

Ghi chú:

- `openclaw doctor` nhắc trước khi viết lại cấu hình supervisor.
- `openclaw doctor --yes` chấp nhận các lời nhắc sửa chữa mặc định.
- `openclaw doctor --repair` áp dụng các sửa chữa được đề xuất mà không có lời nhắc.
- `openclaw doctor --repair --force` ghi đè các cấu hình supervisor tùy chỉnh.
- Bạn luôn có thể buộc viết lại đầy đủ thông qua `openclaw gateway install --force`.

### 16) Chẩn đoán runtime + cổng Gateway

Bác sĩ kiểm tra runtime dịch vụ (PID, trạng thái thoát cuối cùng) và cảnh báo khi
dịch vụ được cài đặt nhưng không thực sự chạy. Nó cũng kiểm tra xung đột cổng
trên cổng gateway (mặc định `18789`) và báo cáo các nguyên nhân có khả năng (gateway đã
chạy, SSH tunnel).
### 17) Các phương pháp hay nhất cho runtime của Gateway

Doctor cảnh báo khi dịch vụ Gateway chạy trên Bun hoặc đường dẫn Node được quản lý phiên bản
(`nvm`, `fnm`, `volta`, `asdf`, v.v.). Các kênh WhatsApp + Telegram yêu cầu Node,
và đường dẫn trình quản lý phiên bản có thể bị hỏng sau khi nâng cấp vì dịch vụ không
tải shell init của bạn. Doctor đề xuất di chuyển đến cài đặt Node hệ thống khi
có sẵn (Homebrew/apt/choco).

### 18) Ghi config + siêu dữ liệu trình hướng dẫn

Doctor duy trì bất kỳ thay đổi config nào và ghi dấu siêu dữ liệu trình hướng dẫn để ghi lại
lần chạy doctor.

### 19) Mẹo không gian làm việc (hệ thống sao lưu + bộ nhớ)

Doctor gợi ý một hệ thống bộ nhớ không gian làm việc khi bị thiếu và in mẹo sao lưu
nếu không gian làm việc chưa được đưa vào git.

Xem [/concepts/agent-workspace](/concepts/agent-workspace) để có hướng dẫn đầy đủ về
cấu trúc không gian làm việc và sao lưu git (khuyến nghị GitHub hoặc GitLab riêng tư).