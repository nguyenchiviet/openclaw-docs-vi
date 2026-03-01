---
summary: >-
  Cách thức hoạt động của các script cài đặt (install.sh, install-cli.sh,
  install.ps1), các cờ lệnh và tự động hóa
read_when:
  - Bạn muốn tìm hiểu `openclaw.ai/install.sh`
  - Bạn muốn tự động hóa các bản cài đặt (CI / headless)
  - Bạn muốn cài đặt từ một bản checkout trên GitHub.
title: Nội bộ Trình cài đặt
x-i18n:
  source_path: install\installer.md
  source_hash: 65aae56b011f7a32ec52b67632c4b3e0c1bb1462fb1d1596c3c29b516246a4fd
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:16:17.895Z'
---

# Cấu trúc bên trong trình cài đặt

OpenClaw cung cấp ba tập lệnh cài đặt, được phục vụ từ `openclaw.ai`.

| Tập lệnh                           | Nền tảng             | Chức năng                                                                                    |
| :--------------------------------- | :------------------- | :------------------------------------------------------------------------------------------- |
| [`install.sh`](#installsh)         | macOS / Linux / WSL  | Cài đặt Node nếu cần, cài đặt OpenClaw qua npm (mặc định) hoặc git, và có thể chạy thiết lập ban đầu. |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL  | Cài đặt Node + OpenClaw vào một tiền tố cục bộ (__OC_I19N_0003__). Không yêu cầu quyền root.              |
| [`install.ps1`](#installps1)       | Windows (PowerShell) | Cài đặt Node nếu cần, cài đặt OpenClaw qua npm (mặc định) hoặc git, và có thể chạy thiết lập ban đầu. |
## Các lệnh nhanh

<Tabs>
  <Tab title="install.sh">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```

    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --help
    ```

  </Tab>
  <Tab title="install-cli.sh">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash
    ```

    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --help
    ```

  </Tab>
  <Tab title="install.ps1">
    ```powershell
    iwr -useb https://openclaw.ai/install.ps1 | iex
    ```

    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -Tag beta -NoOnboard -DryRun
    ```
  </Tab>
</Tabs>
  </Tab>
</Tabs>

<Note>
Nếu cài đặt thành công nhưng không tìm thấy __OC_I19N_0000__ trong một terminal mới, hãy xem [khắc phục sự cố Node.js](__OC_I19N_0001__).
</Note>

---

## install.sh

<Tip>
Được khuyến nghị cho hầu hết các cài đặt tương tác trên macOS/Linux/WSL.
</Tip>

### Luồng (install.sh)

<Steps>
  <Step title="Phát hiện hệ điều hành">
    Hỗ trợ macOS và Linux (bao gồm WSL). Nếu phát hiện macOS, sẽ cài đặt Homebrew nếu chưa có.
  </Step>
  <Step title="Đảm bảo Node.js 22+">
    Kiểm tra phiên bản Node và cài đặt Node 22 nếu cần (Homebrew trên macOS, script thiết lập NodeSource trên Linux apt/dnf/yum).
  </Step>
  <Step title="Đảm bảo Git">
    Cài đặt Git nếu chưa có.
  </Step>
  <Step title="Cài đặt OpenClaw">
    - Phương pháp `npm` (mặc định): cài đặt npm toàn cục
    - Phương pháp `git`: sao chép/cập nhật kho lưu trữ, cài đặt các phụ thuộc bằng pnpm, xây dựng, sau đó cài đặt trình bao bọc tại `~/.local/bin/openclaw`
  </Step>
  <Step title="Các tác vụ sau cài đặt">
    - Chạy `openclaw doctor --non-interactive` khi nâng cấp và cài đặt git (nỗ lực tốt nhất)
    - Cố gắng thiết lập ban đầu khi thích hợp (TTY khả dụng, thiết lập ban đầu không bị tắt và kiểm tra bootstrap/cấu hình thành công)
    - Mặc định `SHARP_IGNORE_GLOBAL_LIBVIPS=1`
  </Step>
</Steps>

### Phát hiện kiểm xuất mã nguồn
Nếu chạy bên trong một bản sao OpenClaw (`package.json` + `pnpm-workspace.yaml`), script sẽ cung cấp:

- sử dụng bản sao (`git`), hoặc
- sử dụng cài đặt toàn cục (`npm`)

Nếu không có TTY và không có phương thức cài đặt nào được thiết lập, nó sẽ mặc định là `npm` và đưa ra cảnh báo.

Script thoát với mã `2` nếu lựa chọn phương thức không hợp lệ hoặc giá trị `--install-method` không hợp lệ.

### Ví dụ (install.sh)

<Tabs>
  <Tab title="Mặc định">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```
  </Tab>
  <Tab title="Skip onboarding">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --no-onboard
    ```
  </Tab>
  <Tab title="Git install">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
    ```
  </Tab>
  <Tab title="Dry run">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --dry-run
    ```
  </Tab>
</Tabs>
```vietnamese
  </Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Tham chiếu cờ lệnh">

| Cờ lệnh                         | Mô tả                                                      |
| ------------------------------- | ---------------------------------------------------------- |
| `--install-method npm\|git`     | Chọn phương thức cài đặt (mặc định: `npm`). Bí danh: `--method`  |
| `--npm`                         | Phím tắt cho phương thức npm                               |
| `--git`                         | Phím tắt cho phương thức git. Bí danh: `--github`                 |
| `--version <version\|dist-tag>` | Phiên bản npm hoặc dist-tag (mặc định: `latest`)                |
| `--beta`                        | Sử dụng dist-tag beta nếu có, nếu không thì quay lại `latest`  |
| `--git-dir <path>`              | Thư mục checkout (mặc định: `~/openclaw`). Bí danh: `--dir` |
| `--no-git-update`               | Bỏ qua `git pull` cho checkout hiện có                      |
| `--no-prompt`                   | Tắt lời nhắc                                               |
| `--no-onboard`                  | Bỏ qua thiết lập ban đầu                                            |
| `--onboard`                     | Bật thiết lập ban đầu                                          |
| `--dry-run`                     | In các hành động mà không áp dụng thay đổi                     |
| `--verbose`                     | Bật đầu ra gỡ lỗi (`set -x`, nhật ký cấp thông báo của npm)      |
| `--help`                        | Hiển thị cách sử dụng (`-h`)                                          |

  </Accordion>

  <Accordion title="Tham chiếu biến môi trường">

| Biến                                        | Mô tả                                       |
| ------------------------------------------- | --------------------------------------------- |
| `OPENCLAW_INSTALL_METHOD=git\|npm`          | Phương thức cài đặt                                |
| `OPENCLAW_VERSION=latest\|next\|<semver>`   | Phiên bản npm hoặc dist-tag                       |
```
| `OPENCLAW_BETA=0\|1`                        | Sử dụng bản beta nếu có                         |
| `OPENCLAW_GIT_DIR=<path>`                   | Thư mục thanh toán                            |
| __OC_I19N_0002__                  | Bật/tắt cập nhật git                            |
| `OPENCLAW_NO_PROMPT=1`                      | Tắt lời nhắc                               |
| `OPENCLAW_NO_ONBOARD=1`                     | Bỏ qua thiết lập ban đầu                               |
| `OPENCLAW_DRY_RUN=1`                        | Chế độ chạy thử                                  |
| `OPENCLAW_VERBOSE=1`                        | Chế độ gỡ lỗi                                    |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Mức độ ghi nhật ký npm                                 |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1`          | Kiểm soát hành vi sharp/libvips (mặc định: `1`) |

  </Accordion>
</AccordionGroup>

---
## install-cli.sh

<Info>
Được thiết kế cho các môi trường mà bạn muốn mọi thứ nằm dưới một tiền tố cục bộ (mặc định `~/.openclaw`) và không có phụ thuộc Node hệ thống.
</Info>

### Luồng (install-cli.sh)

<Steps>
  <Step title="Cài đặt môi trường chạy Node cục bộ">
    Tải xuống gói Node tarball (mặc định `22.22.0`) vào `<prefix>/tools/node-v<version>` và xác minh SHA-256.
  </Step>
  <Step title="Đảm bảo có Git">
    Nếu thiếu Git, sẽ cố gắng cài đặt qua apt/dnf/yum trên Linux hoặc Homebrew trên macOS.
  </Step>
  <Step title="Cài đặt OpenClaw dưới tiền tố">
    Cài đặt bằng npm sử dụng `--prefix <prefix>`, sau đó ghi trình bao bọc vào `<prefix>/bin/openclaw`.
  </Step>
</Steps>

### Ví dụ (install-cli.sh)

<Tabs>
  <Tab title="Mặc định">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash
    ```
  </Tab>
  <Tab title="Custom prefix + version">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --prefix /opt/openclaw --version latest
    ```
  </Tab>
</Tabs>
  </Tab>
  <Tab title="Đầu ra JSON tự động hóa">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
    ```
  </Tab>
  <Tab title="Run onboarding">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --onboard
    ```
  </Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Flag                   | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| `--prefix <path>`      | Install prefix (default: `~/.openclaw`)                                         |
| `--version <ver>`      | OpenClaw version or dist-tag (default: `latest`)                                |
| `--node-version <ver>` | Node version (default: `22.22.0`)                                               |
| `--json`               | Emit NDJSON events                                                              |
| `--onboard`            | Run `openclaw onboard` after install                                            |
| `--no-onboard`         | Skip onboarding (default)                                                       |
| `--set-npm-prefix`     | On Linux, force npm prefix to `~/.npm-global` if current prefix is not writable |
| `--help`               | Show usage (`-h`)                                                               |

  </Accordion>

  <Accordion title="Tham chiếu biến môi trường">
| Biến                                         | Mô tả                                                                             |
| ------------------------------------------- | --------------------------------------------------------------------------------- |
| `OPENCLAW_PREFIX=<path>`                    | Tiền tố cài đặt                                                                   |
| `OPENCLAW_VERSION=<ver>`                    | Phiên bản OpenClaw hoặc thẻ phân phối                                             |
| `OPENCLAW_NODE_VERSION=<ver>`               | Phiên bản Node                                                                    |
| `OPENCLAW_NO_ONBOARD=1`                     | Bỏ qua thiết lập ban đầu                                                          |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Mức độ ghi nhật ký của npm                                                        |
| `OPENCLAW_GIT_DIR=<path>`                   | Đường dẫn tra cứu dọn dẹp cũ (được sử dụng khi xóa bản kiểm tra submodule `Peekaboo` cũ) |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1`          | Kiểm soát hành vi của sharp/libvips (mặc định: `1`)                                     |

  </Accordion>
</AccordionGroup>

---

## install.ps1

### Luồng (install.ps1)

<Steps>
  <Step title="Đảm bảo môi trường PowerShell + Windows">
    Yêu cầu PowerShell 5+ trở lên.
  </Step>
  <Step title="Đảm bảo Node.js 22+">
    Nếu thiếu, sẽ cố gắng cài đặt qua winget, sau đó là Chocolatey, rồi đến Scoop.
  </Step>
  <Step title="Cài đặt OpenClaw">
    - Phương pháp `npm` (mặc định): cài đặt npm toàn cục sử dụng `-Tag` đã chọn
    - Phương pháp `git`: sao chép/cập nhật kho lưu trữ, cài đặt/xây dựng bằng pnpm và cài đặt trình bao bọc tại `%USERPROFILE%\.local\bin\openclaw.cmd`
  </Step>
  <Step title="Các tác vụ sau cài đặt">
    Thêm thư mục bin cần thiết vào PATH của người dùng khi có thể, sau đó chạy `openclaw doctor --non-interactive` khi nâng cấp và cài đặt git (nỗ lực tốt nhất).
  </Step>
</Steps>

### Ví dụ (install.ps1)

<Tabs>
  <Tab title="Mặc định">
    ```powershell
    iwr -useb https://openclaw.ai/install.ps1 | iex
    ```
  </Tab>
  <Tab title="Git install">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -InstallMethod git
    ```
  </Tab>
</Tabs>
  </Tab>
  <Tab title="Thư mục git tùy chỉnh">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -InstallMethod git -GitDir "C:\openclaw"
    ```
  </Tab>
  <Tab title="Dry run">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -DryRun
    ```
  </Tab>
  <Tab title="Debug trace">
    ```powershell
    # install.ps1 has no dedicated -Verbose flag yet.
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```
  </Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Flag                      | Description                                            |
| ------------------------- | ------------------------------------------------------ |
| `-InstallMethod npm\|git` | Install method (default: `npm`)                        |
| `-Tag <tag>`              | npm dist-tag (default: `latest`)                       |
| `-GitDir <path>`          | Checkout directory (default: `%USERPROFILE%\openclaw`) |
| `-NoOnboard`              | Bỏ qua thiết lập ban đầu                                        |
| `-NoGitUpdate`            | Bỏ qua `git pull`                                        |
| `-DryRun`                 | Chỉ in các hành động                                     |

  </Accordion>

  <Accordion title="Tham chiếu biến môi trường">

| Biến                           | Mô tả        |
| ---------------------------------- | ------------------ |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Phương thức cài đặt     |
| `OPENCLAW_GIT_DIR=<path>`          | Thư mục kiểm tra |
| `OPENCLAW_NO_ONBOARD=1`            | Bỏ qua thiết lập ban đầu    |
| `OPENCLAW_GIT_UPDATE=0`            | Tắt git pull   |
| `OPENCLAW_DRY_RUN=1`               | Chế độ chạy thử       |

  </Accordion>
</AccordionGroup>

<Note>
Nếu `-InstallMethod git` được sử dụng và Git bị thiếu, tập lệnh sẽ thoát và in liên kết Git for Windows.
</Note>

---

## CI và tự động hóa

Sử dụng các cờ/biến môi trường không tương tác để chạy dự đoán được.

<Tabs>
  <Tab title="install.sh (npm không tương tác)">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --no-prompt --no-onboard
    ```
  </Tab>
  <Tab title="install.sh (non-interactive git)">
    ```bash
    OPENCLAW_INSTALL_METHOD=git OPENCLAW_NO_PROMPT=1 \
      curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```
  </Tab>
  <Tab title="install-cli.sh (JSON)">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
    ```
  </Tab>
  <Tab title="install.ps1 (skip onboarding)">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    ```
  </Tab>
</Tabs>

---

## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="Tại sao Git lại được yêu cầu?">
    Git được yêu cầu cho phương pháp cài đặt `git`. Đối với các cài đặt `npm`, Git vẫn được kiểm tra/cài đặt để tránh các lỗi `spawn git ENOENT` khi các phần phụ thuộc sử dụng URL git.
  </Accordion>

  <Accordion title="Tại sao npm gặp lỗi EACCES trên Linux?">
    Một số thiết lập Linux trỏ tiền tố toàn cục của npm đến các đường dẫn thuộc sở hữu của root. `install.sh` có thể chuyển tiền tố sang `~/.npm-global` và thêm các xuất PATH vào các tệp rc của shell (khi các tệp đó tồn tại).
  </Accordion>

  <Accordion title="Các vấn đề về sharp/libvips">
    Các script mặc định `SHARP_IGNORE_GLOBAL_LIBVIPS=1` để tránh sharp xây dựng dựa trên libvips của hệ thống. Để ghi đè:

    ```bash
    SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```

  </Accordion>

  <Accordion title='Windows: "npm error spawn git / ENOENT"'>
    Install Git for Windows, reopen PowerShell, rerun installer.
  </Accordion>

  <Accordion title='Windows: "openclaw is not recognized"'>
    Run `npm config get prefix`, append `\bin`, add that directory to user PATH, then reopen PowerShell.
  </Accordion>

  <Accordion title="Windows: how to get verbose installer output">
    `install.ps1` does not currently expose a `-Verbose` switch.
</AccordionGroup>
Sử dụng tính năng theo dõi PowerShell để chẩn đoán cấp độ script:

    ``__OC_I19N_0000__``

  </Accordion>

  <Accordion title="không tìm thấy openclaw sau khi cài đặt">
    Thường là vấn đề về PATH. Xem [khắc phục sự cố Node.js](__OC_I19N_0001__).
  </Accordion>
</AccordionGroup>