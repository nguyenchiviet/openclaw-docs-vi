---
summary: Cài đặt OpenClaw và chạy cuộc trò chuyện đầu tiên của bạn trong vài phút.
read_when:
  - Thiết lập lần đầu từ đầu
  - Bạn muốn con đường nhanh nhất để có một ứng dụng chat hoạt động
title: Bắt Đầu
x-i18n:
  source_path: start\getting-started.md
  source_hash: 4ec86bd0345cc7a70236e566da2ccb9ff17764cc5a7c3b23eab8d5d558251520
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:24:23.700Z'
---

# Bắt đầu

Mục tiêu: từ không có gì đến cuộc trò chuyện đầu tiên hoạt động với thiết lập tối thiểu.

<Info>
Trò chuyện nhanh nhất: mở Control UI (không cần thiết lập kênh). Chạy `openclaw dashboard`
và trò chuyện trong trình duyệt, hoặc mở `http://127.0.0.1:18789/` trên
<Tooltip headline="Gateway host" tip="Máy chạy dịch vụ gateway OpenClaw.">gateway host</Tooltip>.
Tài liệu: [Dashboard](/web/dashboard) và [Control UI](/web/control-ui).
</Info>
## Điều kiện tiên quyết

- Node 22 hoặc mới hơn

<Tip>
Kiểm tra phiên bản Node của bạn với `node --version` nếu bạn không chắc chắn.
</Tip>
## Bắt đầu nhanh (CLI)

<Steps>
  <Step title="Cài đặt OpenClaw (được khuyến nghị)">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
        <img
  src="/assets/install-script.svg"
  alt="Install Script Process"
  className="rounded-lg"
/>
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    <Note>
    Other install methods and requirements: [Install](/install).
    </Note>

  </Step>
  <Step title="Run the onboarding wizard">
    ```bash
    openclaw onboard --install-daemon
    ```

    The wizard configures auth, gateway settings, and optional channels.
    See [Onboarding Wizard](/start/wizard) for details.

  </Step>
  <Step title="Check the Gateway">
    If you installed the service, it should already be running:

    ```bash
    openclaw gateway status
    ```

  </Step>
  <Step title="Open the Control UI">
    ```bash
    openclaw dashboard
    ```
  </Step>
</Steps>

<Check>
Nếu Control UI tải được, Gateway của bạn đã sẵn sàng sử dụng.
</Check>
## Các kiểm tra tùy chọn và tính năng bổ sung

<AccordionGroup>
  <Accordion title="Chạy Gateway ở chế độ nền trước">
    Hữu ích cho các bài kiểm tra nhanh hoặc khắc phục sự cố.

    ```bash
    openclaw gateway --port 18789
    ```

  </Accordion>
  <Accordion title="Send a test message">
    Requires a configured channel.

    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```

  </Accordion>
</AccordionGroup>
## Các biến môi trường hữu ích

Nếu bạn chạy OpenClaw như một tài khoản dịch vụ hoặc muốn các vị trí cấu hình/trạng thái tùy chỉnh:

- `OPENCLAW_HOME` đặt thư mục chính được sử dụng để phân giải đường dẫn nội bộ.
- `OPENCLAW_STATE_DIR` ghi đè thư mục trạng thái.
- `OPENCLAW_CONFIG_PATH` ghi đè đường dẫn tệp cấu hình.

Tham khảo đầy đủ biến môi trường: [Biến môi trường](/help/environment).
## Tìm hiểu sâu hơn

<Columns>
  <Card title="Trình hướng dẫn Thiết lập ban đầu (chi tiết)" href="/start/wizard">
    Tham chiếu trình hướng dẫn CLI đầy đủ và các tùy chọn nâng cao.
  </Card>
  <Card title="Thiết lập ban đầu ứng dụng macOS" href="/start/onboarding">
    Luồng chạy lần đầu tiên cho ứng dụng macOS.
  </Card>
</Columns>
## Những gì bạn sẽ có

- Một Gateway đang chạy
- Xác thực được cấu hình
- Truy cập Control UI hoặc một kênh được kết nối
## Bước tiếp theo

- Tin nhắn riêng an toàn và phê duyệt: [Pairing](/channels/pairing)
- Kết nối thêm kênh: [Channels](/channels)
- Quy trình nâng cao và từ nguồn: [Setup](/start/setup)