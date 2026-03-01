---
summary: >-
  Tài liệu tham khảo đầy đủ cho trình hướng dẫn khởi tạo CLI: mọi bước, cờ và
  trường cấu hình
read_when:
  - Tra cứu một bước wizard hoặc flag cụ thể
  - Tự động hóa quy trình giới thiệu với chế độ không tương tác
  - Gỡ lỗi hành vi trình hướng dẫn
title: Tài liệu tham khảo Trình hướng dẫn khởi tạo
sidebarTitle: Wizard Reference
x-i18n:
  source_path: reference\wizard.md
  source_hash: c83d54afd2e790ee3b0cd5e69b906fe374b46679a74c1eca4e961107961f4ad6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:17:16.920Z'
---

# Tham chiếu Trình hướng dẫn Thiết lập Ban đầu

Đây là tài liệu tham chiếu đầy đủ cho trình hướng dẫn CLI `openclaw onboard`.
Để có cái nhìn tổng quan, hãy xem [Trình hướng dẫn Thiết lập Ban đầu](/start/wizard).

## Chi tiết luồng (chế độ cục bộ)

<Steps>
  <Step title="Phát hiện cấu hình hiện có">
    - Nếu `~/.openclaw/openclaw.json` tồn tại, chọn **Giữ / Sửa đổi / Đặt lại**.
    - Chạy lại trình hướng dẫn **không** xóa bất cứ thứ gì trừ khi bạn chọn **Đặt lại** một cách rõ ràng
      (hoặc truyền `--reset`).
    - CLI `--reset` mặc định là `config+creds+sessions`; sử dụng `--reset-scope full`
      để cũng xóa không gian làm việc.
    - Nếu cấu hình không hợp lệ hoặc chứa các khóa cũ, trình hướng dẫn sẽ dừng và yêu cầu
      bạn chạy `openclaw doctor` trước khi tiếp tục.
    - Đặt lại sử dụng `trash` (không bao giờ `rm`) và cung cấp các phạm vi:
      - Chỉ cấu hình
      - Cấu hình + thông tin đăng nhập + phiên
      - Đặt lại hoàn toàn (cũng xóa không gian làm việc)
  </Step>
  <Step title="Mô hình/Xác thực">
    - **Khóa API Anthropic (khuyên dùng)**: sử dụng `ANTHROPIC_API_KEY` nếu có hoặc nhắc nhập khóa, sau đó lưu khóa đó để daemon sử dụng.
    - **Anthropic OAuth (Claude Code CLI)**: trên macOS, trình hướng dẫn kiểm tra mục Keychain "Claude Code-credentials" (chọn "Luôn cho phép" để các lần khởi động launchd không bị chặn); trên Linux/Windows, nó sử dụng lại `~/.claude/.credentials.json` nếu có.
    - **Mã thông báo Anthropic (dán setup-token)**: chạy `claude setup-token` trên bất kỳ máy nào, sau đó dán mã thông báo (bạn có thể đặt tên cho nó; để trống = mặc định).
    - **Đăng ký OpenAI Code (Codex) (Codex CLI)**: nếu `~/.codex/auth.json` tồn tại, trình hướng dẫn có thể sử dụng lại nó.
    - **Đăng ký OpenAI Code (Codex) (OAuth)**: luồng trình duyệt; dán `code#state`.
      - Đặt `agents.defaults.model` thành `openai-codex/gpt-5.2` khi mô hình chưa được đặt hoặc `openai/*`.
    - **Khóa API OpenAI**: sử dụng `OPENAI_API_KEY` nếu có hoặc nhắc nhập khóa, sau đó lưu trữ khóa đó trong hồ sơ xác thực.
    - **Khóa API xAI (Grok)**: nhắc nhập `XAI_API_KEY` và cấu hình xAI làm nhà cung cấp mô hình.
    - **OpenCode Zen (proxy đa mô hình)**: nhắc nhập `OPENCODE_API_KEY` (hoặc `OPENCODE_ZEN_API_KEY`, lấy tại https://opencode.ai/auth).
    - **Khóa API**: lưu trữ khóa cho bạn.
    - **Vercel AI Gateway (proxy đa mô hình)**: nhắc nhập `AI_GATEWAY_API_KEY`.
    - Chi tiết hơn: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: nhắc nhập ID tài khoản, ID Gateway và `CLOUDFLARE_AI_GATEWAY_API_KEY`.
</Steps>
- Xem chi tiết: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    -   **MiniMax M2.1**: cấu hình được tự động ghi.
    -   Xem chi tiết: [MiniMax](/providers/minimax)
    -   **Synthetic (tương thích Anthropic)**: nhắc nhở cho `SYNTHETIC_API_KEY`.
    -   Xem chi tiết: [Synthetic](/providers/synthetic)
    -   **Moonshot (Kimi K2)**: cấu hình được tự động ghi.
    -   **Kimi Coding**: cấu hình được tự động ghi.
    -   Xem chi tiết: [Moonshot AI (/providers/moonshot)
    -   **Bỏ qua**: chưa có xác thực nào được cấu hình.
    -   Chọn một mô hình mặc định từ các tùy chọn được phát hiện (hoặc nhập nhà cung cấp/mô hình thủ công).
    -   Trình hướng dẫn chạy kiểm tra mô hình và cảnh báo nếu mô hình được cấu hình không xác định hoặc thiếu xác thực.
    -   Chế độ lưu trữ khóa API mặc định là các giá trị hồ sơ xác thực dạng văn bản thuần túy. Sử dụng `--secret-input-mode ref` để lưu trữ các tham chiếu được hỗ trợ bởi biến môi trường thay thế (ví dụ `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
    -   Thông tin xác thực OAuth nằm trong `~/.openclaw/credentials/oauth.json`; hồ sơ xác thực nằm trong `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (khóa API + OAuth).
    -   Xem chi tiết: [/concepts/oauth](/concepts/oauth)
    <Note>
    Mẹo cho máy chủ/không giao diện: hoàn tất OAuth trên một máy có trình duyệt, sau đó sao chép
    `~/.openclaw/credentials/oauth.json` (hoặc `$OPENCLAW_STATE_DIR/credentials/oauth.json`) vào
    máy chủ gateway.
    </Note>
  </Step>
  <Step title="Không gian làm việc">
    -   Mặc định `~/.openclaw/workspace` (có thể cấu hình).
    -   Khởi tạo các tệp không gian làm việc cần thiết cho quy trình khởi động agent.
    -   Bố cục không gian làm việc đầy đủ + hướng dẫn sao lưu: [Không gian làm việc của agent](/concepts/agent-workspace)
  </Step>
  <Step title="Gateway">
    -   Cổng, liên kết, chế độ xác thực, hiển thị Tailscale.
    -   Khuyến nghị xác thực: giữ **Token** ngay cả đối với local loopback để các máy khách WS cục bộ phải xác thực.
    -   Chỉ tắt xác thực nếu bạn hoàn toàn tin tưởng mọi tiến trình cục bộ.
    -   Các liên kết không phải local loopback vẫn yêu cầu xác thực.
  </Step>
  <Step title="Kênh">
    - [WhatsApp](/channels/whatsapp): đăng nhập QR tùy chọn.
    - [Telegram](/channels/telegram): mã thông báo bot.
    - [Discord](/channels/discord): mã thông báo bot.
    - [Google Chat](/channels/googlechat): JSON tài khoản dịch vụ + đối tượng webhook.
    - [Mattermost](/channels/mattermost) (plugin): mã thông báo bot + URL cơ sở.
    - [Signal](/channels/signal): cài đặt `signal-cli` tùy chọn + cấu hình tài khoản.
    - [BlueBubbles](/channels/bluebubbles): **được khuyến nghị cho iMessage**; URL máy chủ + mật khẩu + webhook.
    - [iMessage](/channels/imessage): đường dẫn CLI `imsg` cũ + truy cập DB.
    - Bảo mật tin nhắn riêng: mặc định là ghép nối. Tin nhắn riêng đầu tiên gửi một mã; phê duyệt qua `openclaw pairing approve <channel> <code>` hoặc sử dụng danh sách cho phép.
  </Step>
  <Step title="Cài đặt Daemon">
    - macOS: LaunchAgent
      - Yêu cầu phiên người dùng đã đăng nhập; đối với chế độ không đầu (headless), sử dụng LaunchDaemon tùy chỉnh (không được cung cấp).
    - Linux (và Windows qua WSL2): đơn vị người dùng systemd
      - Trình hướng dẫn cố gắng bật tính năng lingering qua `loginctl enable-linger <user>` để Gateway vẫn hoạt động sau khi đăng xuất.
      - Có thể yêu cầu sudo (ghi `/var/lib/systemd/linger`); nó thử không có sudo trước.
    - **Lựa chọn thời gian chạy:** Node (được khuyến nghị; bắt buộc đối với WhatsApp/Telegram). Bun **không được khuyến nghị**.
  </Step>
  <Step title="Kiểm tra tình trạng">
    - Khởi động Gateway (nếu cần) và chạy `openclaw health`.
    - Mẹo: `openclaw status --deep` thêm các thăm dò tình trạng Gateway vào đầu ra trạng thái (yêu cầu một Gateway có thể truy cập được).
  </Step>
  <Step title="Skills (được khuyến nghị)">
    - Đọc các Skills có sẵn và kiểm tra các yêu cầu.
    - Cho phép bạn chọn trình quản lý node: **npm / pnpm** (bun không được khuyến nghị).
    - Cài đặt các phụ thuộc tùy chọn (một số sử dụng Homebrew trên macOS).
  </Step>
  <Step title="Hoàn tất">
- Tóm tắt + các bước tiếp theo, bao gồm các ứng dụng iOS/Android/macOS để có thêm tính năng.
  </Step>
</Steps>

<Note>
Nếu không phát hiện thấy GUI, trình hướng dẫn sẽ in hướng dẫn chuyển tiếp cổng SSH cho Giao diện người dùng điều khiển thay vì mở trình duyệt.
Nếu các tài nguyên của Giao diện người dùng điều khiển bị thiếu, trình hướng dẫn sẽ cố gắng xây dựng chúng; phương án dự phòng là `pnpm ui:build` (tự động cài đặt các phụ thuộc UI).
</Note>
## Chế độ không tương tác

Sử dụng `--non-interactive` để tự động hóa hoặc tạo script cho quá trình thiết lập ban đầu:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Add `--json` for a machine‑readable summary.

<Note>
`--json` does **not** imply non-interactive mode. Use `--non-interactive` (and `--workspace`) for scripts.
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Ví dụ Z.AI">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Ví dụ Moonshot">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### Thêm agent (không tương tác)

``__OC_I19N_0000__``

## RPC trình hướng dẫn Gateway

Gateway hiển thị luồng trình hướng dẫn qua RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Các máy khách (ứng dụng macOS, Giao diện người dùng điều khiển) có thể hiển thị các bước mà không cần triển khai lại logic thiết lập ban đầu.

## Thiết lập Signal (signal-cli)

Trình hướng dẫn có thể cài đặt __OC_I19N_0000__ từ các bản phát hành trên GitHub:

- Tải xuống tài sản phát hành phù hợp.
- Lưu trữ nó dưới __OC_I19N_0001__.
- Ghi __OC_I19N_0002__ vào cấu hình của bạn.

Lưu ý:

- Các bản dựng JVM yêu cầu **Java 21**.
- Các bản dựng gốc được sử dụng khi có sẵn.
- Windows sử dụng WSL2; quá trình cài đặt signal-cli tuân theo luồng Linux bên trong WSL.
## Những gì trình hướng dẫn ghi lại

Các trường điển hình trong `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (nếu chọn Minimax)
- `gateway.*` (chế độ, liên kết, xác thực, tailscale)
- `session.dmScope` (chi tiết hành vi: [Tham khảo thiết lập ban đầu CLI](/start/wizard-cli-reference#outputs-and-internals))
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Danh sách kênh được phép (Slack/Discord/Matrix/Microsoft Teams) khi bạn chọn tham gia trong quá trình nhắc nhở (tên sẽ được giải quyết thành ID nếu có thể).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` ghi `agents.list[]` và `bindings` tùy chọn.

Thông tin đăng nhập WhatsApp nằm dưới `~/.openclaw/credentials/whatsapp/<accountId>/`.
Các phiên được lưu trữ dưới `~/.openclaw/agents/<agentId>/sessions/`.

Một số kênh được cung cấp dưới dạng plugin. Khi bạn chọn một kênh trong quá trình thiết lập ban đầu, trình hướng dẫn sẽ nhắc bạn cài đặt nó (npm hoặc một đường dẫn cục bộ) trước khi có thể cấu hình.

## Tài liệu liên quan

- Tổng quan trình hướng dẫn: [Trình hướng dẫn thiết lập ban đầu](/start/wizard)
- Thiết lập ban đầu ứng dụng macOS: [Thiết lập ban đầu](/start/onboarding)
- Tham chiếu cấu hình: [Cấu hình Gateway](/gateway/configuration)
- Nhà cung cấp: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (cũ)
- Skills: [Skills](/tools/skills), [Cấu hình Skills](/tools/skills-config)