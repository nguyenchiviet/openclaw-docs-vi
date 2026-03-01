---
title: Node.js
summary: >-
  Cài đặt và cấu hình Node.js cho OpenClaw — yêu cầu phiên bản, tùy chọn cài đặt
  và khắc phục sự cố PATH
read_when:
  - Bạn cần cài đặt Node.js trước khi cài đặt OpenClaw.
  - Bạn đã cài đặt OpenClaw nhưng `openclaw` không tìm thấy lệnh.
  - npm install -g thất bại do vấn đề về quyền hoặc PATH
x-i18n:
  source_path: install\node.md
  source_hash: f848d6473a1830904f7e75bb211161cfb22ac7a4de6623835cad1444a18f0579
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:15:36.453Z'
---

# Node.js

OpenClaw yêu cầu **Node 22 trở lên**. [Tập lệnh cài đặt](/install#install-methods) sẽ tự động phát hiện và cài đặt Node — trang này dành cho trường hợp bạn muốn tự thiết lập Node và đảm bảo mọi thứ được kết nối đúng cách (phiên bản, PATH, cài đặt toàn cục).

## Kiểm tra phiên bản của bạn

Nếu phiên bản Node.js của bạn là ```bash
node -v
```

If this prints `v22.x.x` trở lên, bạn đã sẵn sàng. Nếu Node chưa được cài đặt hoặc phiên bản quá cũ, hãy chọn một phương pháp cài đặt bên dưới.

## Cài đặt Node

<Tabs>
  <Tab title="macOS">
    **Homebrew** (khuyên dùng):

    ```bash
    brew install node
    ```

    Or download the macOS installer from [nodejs.org](https://nodejs.org/).

  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian:**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL:**

    ```bash
    sudo dnf install nodejs
    ```

    Hoặc sử dụng trình quản lý phiên bản (xem bên dưới).

  </Tab>
  <Tab title="Windows">
    **winget** (khuyên dùng):

    ``__OC_I19N_0000__`__OC_I19N_0001__`__OC_I19N_0002__`__OC_I19N_0003__`__OC_I19N_0004__``
<Warning>
  Đảm bảo trình quản lý phiên bản của bạn được khởi tạo trong tệp khởi động shell của bạn (`~/.zshrc` hoặc `~/.bashrc`). Nếu không, `openclaw` có thể không được tìm thấy trong các phiên terminal mới vì PATH sẽ không bao gồm thư mục bin của Node.
  </Warning>
</Accordion>
## Khắc phục sự cố

### `openclaw: command not found`

Điều này hầu như luôn có nghĩa là thư mục bin toàn cục của npm không có trong PATH của bạn.

<Steps>
  <Step title="Tìm tiền tố npm toàn cục của bạn">
    ```bash
    npm prefix -g
    ```
  </Step>
  <Step title="Check if it's on your PATH">
    ```bash
    echo "$PATH"
    ```

    Look for `<npm-prefix>/bin` (macOS/Linux) or `<npm-prefix>` (Windows) in the output.

  </Step>
  <Step title="Add it to your shell startup file">
    <Tabs>
      <Tab title="macOS / Linux">
        Add to `~/.zshrc` or `~/.bashrc`:

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        Then open a new terminal (or run `rehash` in zsh / `hash -r` trong bash).
  </Step>
</Steps>
      </Tab>
      <Tab title="Windows">
        Thêm đầu ra của `npm prefix -g` vào biến môi trường PATH của hệ thống thông qua Cài đặt → Hệ thống → Biến môi trường.
      </Tab>
    </Tabs>

  </Step>
</Steps>

### Lỗi quyền trên `npm install -g` (Linux)

Nếu bạn thấy lỗi `EACCES`, hãy chuyển tiền tố toàn cục của npm sang một thư mục mà người dùng có quyền ghi:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Add the `export PATH=...` line to your `~/.bashrc` or `~/.zshrc` để làm cho nó vĩnh viễn.
