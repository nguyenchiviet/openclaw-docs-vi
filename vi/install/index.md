---
summary: >-
  Cài đặt OpenClaw — script cài đặt, npm/pnpm, từ mã nguồn, Docker, và nhiều hơn
  nữa
read_when:
  - >-
    Bạn cần một phương pháp cài đặt khác ngoài hướng dẫn quickstart Getting
    Started
  - Bạn muốn triển khai đến một nền tảng đám mây
  - 'Bạn cần cập nhật, di chuyển hoặc gỡ cài đặt'
title: Cài đặt
x-i18n:
  source_path: install\index.md
  source_hash: 1641f9077d83c2883bef5c51605e278f80be3747b8d7d43552484f4008fa0b7d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:08:16.875Z'
---

# Cài đặt

Đã làm theo [Bắt đầu](/start/getting-started)? Bạn đã sẵn sàng — trang này dành cho các phương pháp cài đặt thay thế, hướng dẫn dành riêng cho từng nền tảng và bảo trì.
## Yêu cầu hệ thống

- **[Node 22+](/install/node)** (tập lệnh [cài đặt](#install-methods) sẽ cài đặt nó nếu bị thiếu)
- macOS, Linux, hoặc Windows
- `pnpm` chỉ khi bạn xây dựng từ nguồn

<Note>
Trên Windows, chúng tôi khuyên bạn nên chạy OpenClaw dưới [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).
</Note>
## Phương pháp cài đặt

<Tip>
**Trình cài đặt** là cách được khuyến nghị để cài đặt OpenClaw. Nó xử lý phát hiện Node, cài đặt và thiết lập ban đầu trong một bước.
</Tip>

<Warning>
Đối với VPS/máy chủ đám mây, tránh các hình ảnh marketplace "1-click" của bên thứ ba nếu có thể. Ưu tiên hình ảnh OS cơ sở sạch (ví dụ Ubuntu LTS), sau đó cài đặt OpenClaw tự mình bằng trình cài đặt.
</Warning>

<AccordionGroup>
  <Accordion title="Trình cài đặt" icon="rocket" defaultOpen>
    Tải xuống CLI, cài đặt nó toàn cục qua npm và khởi chạy trình hướng dẫn thiết lập ban đầu.

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    That's it — the script handles Node detection, installation, and onboarding.

    To skip onboarding and just install the binary:

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
        ```
      </Tab>
    </Tabs>

    For all flags, env vars, and CI/automation options, see [Installer internals](/install/installer).

  </Accordion>

  <Accordion title="npm / pnpm" icon="package">
    If you already have Node 22+ and prefer to manage the install yourself:

    <Tabs>
      <Tab title="npm">
        ```bash
        npm install -g openclaw@latest
        openclaw onboard --install-daemon
        ```

        <Accordion title="sharp build errors?">
          If you have libvips installed globally (common on macOS via Homebrew) and `sharp` không thành công, buộc sử dụng các tệp nhị phân được xây dựng sẵn:
```bash
          SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
          ```

          If you see `#: Vui lòng thêm node-gyp vào các phụ thuộc của bạn`, either install build tooling (macOS: Xcode CLT + `npm install -g node-gyp`) or use the env var above.
        </Accordion>
      </Tab>
      <Tab title="pnpm">
        ```bash
        pnpm add -g openclaw@latest
        pnpm approve-builds -g        # approve openclaw, node-llama-cpp, sharp, etc.
        openclaw onboard --install-daemon
        ```

        <Note>
        pnpm requires explicit approval for packages with build scripts. After the first install shows the "Ignored build scripts" warning, run `pnpm approve-builds -g` and select the listed packages.
        </Note>
      </Tab>
    </Tabs>

  </Accordion>

  <Accordion title="From source" icon="github">
    For contributors or anyone who wants to run from a local checkout.

    <Steps>
      <Step title="Clone and build">
        Clone the [OpenClaw repo](https://github.com/openclaw/openclaw) and build:

        ```bash
        git clone https://github.com/openclaw/openclaw.git
        cd openclaw
        pnpm install
        pnpm ui:build
        pnpm build
        ```
      </Step>
      <Step title="Link the CLI">
        Make the `openclaw` command available globally:

        ```bash
        pnpm link --global
        ```

        Alternatively, skip the link and run commands via `pnpm openclaw ...` from inside the repo.
      </Step>
      <Step title="Run onboarding">
        ```bash
        openclaw onboard --install-daemon
        ```
      </Step>
    </Steps>

    Để tìm hiểu thêm về quy trình phát triển, xem [Thiết lập](/start/setup).

  </Accordion>
</AccordionGroup>
## Các phương pháp cài đặt khác

<CardGroup cols={2}>
  <Card title="Docker" href="/install/docker" icon="container">
    Triển khai được đóng gói hoặc không có giao diện.
  </Card>
  <Card title="Podman" href="/install/podman" icon="container">
    Container không cần quyền root: chạy `setup-podman.sh` một lần, sau đó chạy tập lệnh khởi động.
  </Card>
  <Card title="Nix" href="/install/nix" icon="snowflake">
    Cài đặt khai báo thông qua Nix.
  </Card>
  <Card title="Ansible" href="/install/ansible" icon="server">
    Cấp phát fleet tự động.
  </Card>
  <Card title="Bun" href="/install/bun" icon="zap">
    Sử dụng chỉ CLI thông qua runtime Bun.
  </Card>
</CardGroup>
## Sau khi cài đặt

Xác minh mọi thứ đang hoạt động:

```bash
openclaw doctor         # check for config issues
openclaw status         # gateway status
openclaw dashboard      # open the browser UI
```

If you need custom runtime paths, use:

- `OPENCLAW_HOME` for home-directory based internal paths
- `OPENCLAW_STATE_DIR` for mutable state location
- `OPENCLAW_CONFIG_PATH` cho vị trí tệp cấu hình

Xem [Biến môi trường](/help/environment) để biết thứ tự ưu tiên và chi tiết đầy đủ.
## Khắc phục sự cố: `openclaw` không tìm thấy

<Accordion title="Chẩn đoán và sửa PATH">
  Chẩn đoán nhanh:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

If `$(npm prefix -g)/bin` (macOS/Linux) or `$(npm prefix -g)` (Windows) is **not** in your `$PATH`, your shell can't find global npm binaries (including `openclaw`).

Fix — add it to your shell startup file (`~/.zshrc` or `~/.bashrc`):

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

On Windows, add the output of `npm prefix -g` to your PATH.

Then open a new terminal (or `rehash` in zsh / `hash -r` trong bash).
</Accordion>
## Cập nhật / gỡ cài đặt

<CardGroup cols={3}>
  <Card title="Cập nhật" href="/install/updating" icon="refresh-cw">
    Giữ OpenClaw luôn cập nhật.
  </Card>
  <Card title="Di chuyển" href="/install/migrating" icon="arrow-right">
    Chuyển sang máy mới.
  </Card>
  <Card title="Gỡ cài đặt" href="/install/uninstall" icon="trash-2">
    Gỡ cài đặt OpenClaw hoàn toàn.
  </Card>
</CardGroup>