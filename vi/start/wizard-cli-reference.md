---
summary: >-
  Tài liệu tham khảo hoàn chỉnh cho quy trình onboarding CLI, thiết lập
  auth/model, outputs và internals
read_when:
  - Bạn cần hành vi chi tiết cho quá trình onboard của OpenClaw
  - Bạn đang gỡ lỗi kết quả onboarding hoặc tích hợp các client onboarding
title: Tài liệu Tham khảo Onboarding CLI
sidebarTitle: CLI reference
x-i18n:
  source_path: start\wizard-cli-reference.md
  source_hash: c1e9bb7c5f1c49ea0a45b2d5a9d736480b834472e963dd6cace531820733d824
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:26:09.258Z'
---

# Tham chiếu Thiết lập ban đầu CLI

Trang này là tham chiếu đầy đủ cho `openclaw onboard`.
Để xem hướng dẫn ngắn, hãy xem [Trình hướng dẫn Thiết lập ban đầu (/start/wizard).
## Trình hướng dẫn làm gì

Chế độ cục bộ (mặc định) hướng dẫn bạn qua:

- Thiết lập mô hình và xác thực (OpenAI Code subscription OAuth, khóa API Anthropic hoặc token thiết lập, cộng với các tùy chọn MiniMax, GLM, Moonshot và AI Gateway)
- Vị trí không gian làm việc và tệp bootstrap
- Cài đặt Gateway (cổng, liên kết, xác thực, Tailscale)
- Kênh và nhà cung cấp (Telegram, WhatsApp, Discord, Google Chat, plugin Mattermost, Signal)
- Cài đặt daemon (LaunchAgent hoặc systemd user unit)
- Kiểm tra sức khỏe
- Thiết lập Skills

Chế độ từ xa cấu hình máy này để kết nối với gateway ở nơi khác.
Nó không cài đặt hoặc sửa đổi bất cứ điều gì trên máy chủ từ xa.
## Chi tiết luồng cục bộ

<Steps>
  <Step title="Phát hiện cấu hình hiện có">
    - Nếu `~/.openclaw/openclaw.json` tồn tại, chọn Keep, Modify, hoặc Reset.
    - Chạy lại trình hướng dẫn không xóa bất cứ điều gì trừ khi bạn rõ ràng chọn Reset (hoặc truyền `--reset`).
    - CLI `--reset` mặc định là `config+creds+sessions`; sử dụng `--reset-scope full` để cũng xóa workspace.
    - Nếu cấu hình không hợp lệ hoặc chứa các khóa cũ, trình hướng dẫn dừng lại và yêu cầu bạn chạy `openclaw doctor` trước khi tiếp tục.
    - Reset sử dụng `trash` và cung cấp các phạm vi:
      - Config only
      - Config + credentials + sessions
      - Full reset (cũng xóa workspace)
  </Step>
  <Step title="Mô hình và xác thực">
    - Ma trận tùy chọn đầy đủ nằm trong [Auth and model options](#auth-and-model-options).
  </Step>
  <Step title="Workspace">
    - Mặc định `~/.openclaw/workspace` (có thể cấu hình).
    - Tạo các tệp workspace cần thiết cho nghi thức bootstrap lần đầu chạy.
    - Bố cục workspace: [Agent workspace](/concepts/agent-workspace).
  </Step>
  <Step title="Gateway">
    - Nhắc nhở cổng, bind, chế độ xác thực và tiếp xúc Tailscale.
    - Khuyến nghị: giữ xác thực token được bật ngay cả đối với local loopback để các máy khách WS cục bộ phải xác thực.
    - Chỉ vô hiệu hóa xác thực nếu bạn hoàn toàn tin tưởng mọi quy trình cục bộ.
    - Các bind không phải loopback vẫn yêu cầu xác thực.
  </Step>
  <Step title="Kênh">
    - [WhatsApp](/channels/whatsapp): đăng nhập QR tùy chọn
    - [Telegram](/channels/telegram): bot token
    - [Discord](/channels/discord): bot token
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience
    - [Mattermost](/channels/mattermost) plugin: bot token + base URL
    - [Signal](/channels/signal): cài đặt `signal-cli` tùy chọn + cấu hình tài khoản
    - [BlueBubbles](/channels/bluebubbles): được khuyến nghị cho iMessage; server URL + password + webhook
    - [iMessage](/channels/imessage): CLI `imsg` cũ + truy cập DB
    - Bảo mật tin nhắn riêng: mặc định là pairing. Tin nhắn riêng đầu tiên gửi một mã; phê duyệt qua
      `openclaw pairing approve <channel> <code>` hoặc sử dụng allowlists.
  </Step>
  <Step title="Cài đặt daemon">
    - macOS: LaunchAgent
      - Yêu cầu phiên người dùng đã đăng nhập; đối với headless, sử dụng LaunchDaemon tùy chỉnh (không được cung cấp).
    - Linux và Windows qua WSL2: systemd user unit
      - Trình hướng dẫn cố gắng `loginctl enable-linger <user>` để gateway vẫn hoạt động sau khi đăng xuất.
      - Có thể nhắc nhở sudo (ghi `/var/lib/systemd/linger`); nó cố gắng không có sudo trước tiên.
    - Lựa chọn runtime: Node (được khuyến nghị; bắt buộc cho WhatsApp và Telegram). Bun không được khuyến nghị.
  </Step>
  <Step title="Kiểm tra sức khỏe">
    - Bắt đầu gateway (nếu cần) và chạy `openclaw health`.
    - `openclaw status --deep` thêm các bộ kiểm tra sức khỏe gateway vào đầu ra trạng thái.
  </Step>
  <Step title="Skills">
    - Đọc các Skills có sẵn và kiểm tra yêu cầu.
    - Cho phép bạn chọn node manager: npm hoặc pnpm (bun không được khuyến nghị).
    - Cài đặt các phụ thuộc tùy chọn (một số sử dụng Homebrew trên macOS).
  </Step>
  <Step title="Hoàn thành">
    - Tóm tắt và các bước tiếp theo, bao gồm các tùy chọn ứng dụng iOS, Android và macOS.
  </Step>
</Steps>
<Note>
Nếu không phát hiện được GUI, trình hướng dẫn sẽ in hướng dẫn port-forward SSH cho Control UI thay vì mở trình duyệt.
Nếu thiếu tài sản Control UI, trình hướng dẫn sẽ cố gắng xây dựng chúng; giải pháp dự phòng là `pnpm ui:build` (tự động cài đặt các phụ thuộc UI).
</Note>
## Chi tiết chế độ Remote

Chế độ Remote cấu hình máy này để kết nối đến một gateway ở nơi khác.

<Info>
Chế độ Remote không cài đặt hoặc sửa đổi bất kỳ thứ gì trên máy chủ từ xa.
</Info>

Những gì bạn đặt:

- URL gateway từ xa (`ws://...`)
- Token nếu yêu cầu xác thực gateway từ xa (được khuyến nghị)

<Note>
- Nếu gateway chỉ là local loopback, hãy sử dụng SSH tunneling hoặc một tailnet.
- Gợi ý khám phá:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>
## Xác thực và tùy chọn mô hình

<AccordionGroup>
  <Accordion title="Khóa API Anthropic (được khuyến nghị)">
    Sử dụng `ANTHROPIC_API_KEY` nếu có hoặc yêu cầu nhập khóa, sau đó lưu nó để sử dụng daemon.
  </Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: kiểm tra mục Keychain "Claude Code-credentials"
    - Linux và Windows: sử dụng lại `~/.claude/.credentials.json` nếu có

    Trên macOS, chọn "Always Allow" để các lần khởi động launchd không bị chặn.

  </Accordion>
  <Accordion title="Mã thông báo Anthropic (setup-token paste)">
    Chạy `claude setup-token` trên bất kỳ máy nào, sau đó dán mã thông báo.
    Bạn có thể đặt tên cho nó; để trống sẽ sử dụng mặc định.
  </Accordion>
  <Accordion title="Đăng ký OpenAI Code (sử dụng lại Codex CLI)">
    Nếu `~/.codex/auth.json` tồn tại, trình hướng dẫn có thể sử dụng lại nó.
  </Accordion>
  <Accordion title="Đăng ký OpenAI Code (OAuth)">
    Luồng trình duyệt; dán `code#state`.

    Đặt `agents.defaults.model` thành `openai-codex/gpt-5.3-codex` khi mô hình chưa được đặt hoặc `openai/*`.

  </Accordion>
  <Accordion title="Khóa API OpenAI">
    Sử dụng `OPENAI_API_KEY` nếu có hoặc yêu cầu nhập khóa, sau đó lưu trữ thông tin xác thực trong hồ sơ xác thực.

    Đặt `agents.defaults.model` thành `openai/gpt-5.1-codex` khi mô hình chưa được đặt, `openai/*` hoặc `openai-codex/*`.

  </Accordion>
  <Accordion title="Khóa API xAI (Grok)">
    Yêu cầu nhập `XAI_API_KEY` và cấu hình xAI làm nhà cung cấp mô hình.
  </Accordion>
  <Accordion title="OpenCode Zen">
    Yêu cầu nhập `OPENCODE_API_KEY` (hoặc `OPENCODE_ZEN_API_KEY`).
    URL thiết lập: [opencode.ai/auth](https://opencode.ai/auth).
  </Accordion>
  <Accordion title="Khóa API (chung)">
    Lưu trữ khóa cho bạn.
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    Yêu cầu nhập `AI_GATEWAY_API_KEY`.
    Chi tiết thêm: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    Yêu cầu nhập ID tài khoản, ID gateway và `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    Chi tiết thêm: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  </Accordion>
  <Accordion title="MiniMax M2.1">
    Cấu hình được viết tự động.
    Chi tiết thêm: [MiniMax](/providers/minimax).
  </Accordion>
  <Accordion title="Synthetic (tương thích Anthropic)">
    Yêu cầu nhập `SYNTHETIC_API_KEY`.
    Chi tiết thêm: [Synthetic](/providers/synthetic).
  </Accordion>
  <Accordion title="Moonshot và Kimi Coding">
    Cấu hình Moonshot (Kimi K2) và Kimi Coding được viết tự động.
  </Accordion>
</AccordionGroup>
Chi tiết hơn: [Moonshot AI (/providers/moonshot).
  </Accordion>
  <Accordion title="Nhà cung cấp tùy chỉnh">
    Hoạt động với các điểm cuối tương thích OpenAI và tương thích Anthropic.

    Thiết lập ban đầu tương tác hỗ trợ các lựa chọn lưu trữ khóa API giống như các luồng khóa API nhà cung cấp khác:
    - **Dán khóa API ngay bây giờ** (văn bản thuần)
    - **Sử dụng tham chiếu bí mật** (tham chiếu env hoặc tham chiếu nhà cung cấp được cấu hình, với xác thực trước chuyến bay)

    Cờ không tương tác:
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (tùy chọn; quay lại `CUSTOM_API_KEY`)
    - `--custom-provider-id` (tùy chọn)
    - `--custom-compatibility <openai|anthropic>` (tùy chọn; mặc định `openai`)

  </Accordion>
  <Accordion title="Bỏ qua">
    Để xác thực không được cấu hình.
  </Accordion>
</AccordionGroup>

Hành vi mô hình:

- Chọn mô hình mặc định từ các tùy chọn được phát hiện hoặc nhập nhà cung cấp và mô hình theo cách thủ công.
- Trình hướng dẫn chạy kiểm tra mô hình và cảnh báo nếu mô hình được cấu hình không xác định hoặc thiếu xác thực.

Đường dẫn thông tin xác thực và hồ sơ:

- Thông tin xác thực OAuth: `~/.openclaw/credentials/oauth.json`
- Hồ sơ xác thực (khóa API + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

Chế độ lưu trữ khóa API:

- Hành vi thiết lập ban đầu mặc định duy trì các khóa API dưới dạng giá trị văn bản thuần trong hồ sơ xác thực.
- `--secret-input-mode ref` cho phép chế độ tham chiếu thay vì lưu trữ khóa văn bản thuần.
  Trong thiết lập ban đầu tương tác, bạn có thể chọn một trong hai:
  - tham chiếu biến môi trường (ví dụ `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`)
  - tham chiếu nhà cung cấp được cấu hình (`file` hoặc `exec`) với bí danh nhà cung cấp + id
- Chế độ tham chiếu tương tác chạy xác thực trước chuyến bay nhanh trước khi lưu.
  - Tham chiếu Env: xác thực tên biến + giá trị không trống trong môi trường thiết lập ban đầu hiện tại.
  - Tham chiếu nhà cung cấp: xác thực cấu hình nhà cung cấp và giải quyết id được yêu cầu.
  - Nếu xác thực trước chuyến bay không thành công, thiết lập ban đầu sẽ hiển thị lỗi và cho phép bạn thử lại.
- Ở chế độ không tương tác, `--secret-input-mode ref` chỉ được hỗ trợ bởi env.
  - Đặt biến môi trường nhà cung cấp trong môi trường quy trình thiết lập ban đầu.
  - Các cờ khóa nội tuyến (ví dụ `--openai-api-key`) yêu cầu biến env đó được đặt; nếu không, thiết lập ban đầu sẽ thất bại nhanh.
  - Đối với các nhà cung cấp tùy chỉnh, chế độ `ref` không tương tác lưu trữ `models.providers.<id>.apiKey` dưới dạng `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.
  - Trong trường hợp nhà cung cấp tùy chỉnh đó, `--custom-api-key` yêu cầu `CUSTOM_API_KEY` được đặt; nếu không, thiết lập ban đầu sẽ thất bại nhanh.
- Các thiết lập văn bản thuần hiện có tiếp tục hoạt động không thay đổi.

<Note>
Mẹo không đầu và máy chủ: hoàn thành OAuth trên máy có trình duyệt, sau đó sao chép
`~/.openclaw/credentials/oauth.json` (hoặc `$OPENCLAW_STATE_DIR/credentials/oauth.json`)
đến máy chủ Gateway.
</Note>
## Đầu ra và nội bộ

Các trường điển hình trong `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (nếu chọn Minimax)
- `gateway.*` (mode, bind, auth, tailscale)
- `session.dmScope` (thiết lập ban đầu cục bộ mặc định này thành `per-channel-peer` khi không được đặt; các giá trị rõ ràng hiện có được bảo toàn)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Danh sách cho phép kênh (Slack, Discord, Matrix, Microsoft Teams) khi bạn chọn tham gia trong các lời nhắc (tên được phân giải thành ID khi có thể)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` ghi `agents.list[]` và `bindings` tùy chọn.

Thông tin xác thực WhatsApp được đặt dưới `~/.openclaw/credentials/whatsapp/<accountId>/`.
Các phiên được lưu trữ dưới `~/.openclaw/agents/<agentId>/sessions/`.

<Note>
Một số kênh được cung cấp dưới dạng plugin. Khi được chọn trong quá trình thiết lập ban đầu, trình hướng dẫn
sẽ nhắc cài đặt plugin (npm hoặc đường dẫn cục bộ) trước khi cấu hình kênh.
</Note>

Gateway wizard RPC:

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

Các ứng dụng khách (ứng dụng macOS và Control UI) có thể hiển thị các bước mà không cần triển khai lại logic thiết lập ban đầu.

Hành vi thiết lập Signal:

- Tải xuống tài sản phát hành thích hợp
- Lưu trữ nó dưới `~/.openclaw/tools/signal-cli/<version>/`
- Ghi `channels.signal.cliPath` trong cấu hình
- Bản dựng JVM yêu cầu Java 21
- Bản dựng gốc được sử dụng khi có sẵn
- Windows sử dụng WSL2 và tuân theo luồng signal-cli Linux bên trong WSL
## Tài liệu liên quan

- Trung tâm thiết lập ban đầu: [Trình hướng dẫn thiết lập ban đầu (/start/wizard)
- Tự động hóa và tập lệnh: [Tự động hóa CLI](/start/wizard-cli-automation)
- Tham chiếu lệnh: [`openclaw onboard`](/cli/onboard)