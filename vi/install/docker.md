---
summary: Thiết lập và onboarding dựa trên Docker tùy chọn cho OpenClaw
read_when:
  - Bạn muốn một gateway được đóng gói trong container thay vì cài đặt cục bộ
  - Bạn đang xác thực luồng Docker
title: Docker
x-i18n:
  source_path: install\docker.md
  source_hash: 5de15981e456d8196dfd3eff010cb8bfcdec325edcbec2a9f54124221b5d0d3a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:07:53.539Z'
---

# Docker (tùy chọn)

Docker là **tùy chọn**. Sử dụng nó chỉ khi bạn muốn một Gateway được đóng gói trong container hoặc để xác thực quy trình Docker.
## Docker có phù hợp với tôi không?

- **Có**: bạn muốn một môi trường gateway bị cô lập, có thể loại bỏ hoặc chạy OpenClaw trên một máy chủ mà không cần cài đặt cục bộ.
- **Không**: bạn đang chạy trên máy của riêng mình và chỉ muốn vòng lặp phát triển nhanh nhất. Hãy sử dụng quy trình cài đặt bình thường thay thế.
- **Ghi chú về Sandboxing**: agent sandboxing cũng sử dụng Docker, nhưng nó **không** yêu cầu gateway đầy đủ chạy trong Docker. Xem [Sandboxing](/gateway/sandboxing).

Hướng dẫn này bao gồm:

- Gateway được đóng gói (OpenClaw đầy đủ trong Docker)
- Per-session Agent Sandbox (gateway trên máy chủ + công cụ agent được cô lập bằng Docker)

Chi tiết Sandboxing: [Sandboxing](/gateway/sandboxing)
## Yêu cầu

- Docker Desktop (hoặc Docker Engine) + Docker Compose v2
- Ít nhất 2 GB RAM để xây dựng hình ảnh (`pnpm install` có thể bị giết do hết bộ nhớ trên các máy chủ 1 GB với exit 137)
- Đủ dung lượng đĩa cho hình ảnh + nhật ký
## Gateway được đóng gói (Docker Compose)

### Bắt đầu nhanh (được khuyến nghị)

Từ thư mục gốc của repo:

```bash
./docker-setup.sh
```

This script:

- builds the gateway image
- runs the onboarding wizard
- prints optional provider setup hints
- starts the gateway via Docker Compose
- generates a gateway token and writes it to `.env`

Optional env vars:

- `OPENCLAW_DOCKER_APT_PACKAGES` — install extra apt packages during build
- `OPENCLAW_EXTRA_MOUNTS` — add extra host bind mounts
- `OPENCLAW_HOME_VOLUME` — persist `/home/node` in a named volume

After it finishes:

- Open `http://127.0.0.1:18789/` in your browser.
- Paste the token into the Control UI (Settings → token).
- Need the URL again? Run `docker compose run --rm openclaw-cli dashboard --no-open`.

It writes config/workspace on the host:

- `~/.openclaw/`
- `~/.openclaw/workspace`

Running on a VPS? See [Hetzner (Docker VPS)](/install/hetzner).

### Shell Helpers (optional)

For easier day-to-day Docker management, install `ClawDock`:

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**Add to your shell config (zsh):**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

Then use `clawdock-start`, `clawdock-stop`, `clawdock-dashboard`, etc. Run `clawdock-help` for all commands.

See [`ClawDock` Helper README](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) for details.

### Manual flow (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```
Lưu ý: chạy `docker compose ...` từ thư mục gốc của repo. Nếu bạn đã bật
`OPENCLAW_EXTRA_MOUNTS` hoặc `OPENCLAW_HOME_VOLUME`, tập lệnh thiết lập sẽ ghi
`docker-compose.extra.yml`; hãy đưa nó vào khi chạy Compose ở nơi khác:

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Control UI token + pairing (Docker)

If you see “unauthorized” or “disconnected (1008): pairing required”, fetch a
fresh dashboard link and approve the browser device:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

More detail: [Dashboard](/web/dashboard), [Devices](/cli/devices).

### Extra mounts (optional)

If you want to mount additional host directories into the containers, set
`OPENCLAW_EXTRA_MOUNTS` before running `docker-setup.sh`. This accepts a
comma-separated list of Docker bind mounts and applies them to both
`openclaw-gateway` and `openclaw-cli` by generating `docker-compose.extra.yml`.

Example:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes:

- Paths must be shared with Docker Desktop on macOS/Windows.
- Each entry must be `source:target[:options]` with no spaces, tabs, or newlines.
- If you edit `OPENCLAW_EXTRA_MOUNTS`, rerun `docker-setup.sh` to regenerate the
  extra compose file.
- `docker-compose.extra.yml` is generated. Don’t hand-edit it.

### Persist the entire container home (optional)

If you want `/home/node` to persist across container recreation, set a named
volume via `OPENCLAW_HOME_VOLUME`. This creates a Docker volume and mounts it at
`/home/node`, while keeping the standard config/workspace bind mounts. Use a
named volume here (not a bind path); for bind mounts, use
`OPENCLAW_EXTRA_MOUNTS`.

Example:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Bạn có thể kết hợp điều này với các mount bổ sung:
```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes:

- Named volumes must match `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
- If you change `OPENCLAW_HOME_VOLUME`, rerun `docker-setup.sh` to regenerate the
  extra compose file.
- The named volume persists until removed with `docker volume rm <name>`.

### Install extra apt packages (optional)

If you need system packages inside the image (for example, build tools or media
libraries), set `OPENCLAW_DOCKER_APT_PACKAGES` before running `docker-setup.sh`.
This installs the packages during the image build, so they persist even if the
container is deleted.

Example:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notes:

- This accepts a space-separated list of apt package names.
- If you change `OPENCLAW_DOCKER_APT_PACKAGES`, rerun `docker-setup.sh` to rebuild
  the image.

### Power-user / full-featured container (opt-in)

The default Docker image is **security-first** and runs as the non-root `node`
user. This keeps the attack surface small, but it means:

- no system package installs at runtime
- no Homebrew by default
- no bundled Chromium/Playwright browsers

If you want a more full-featured container, use these opt-in knobs:

1. **Persist `/home/node`** so browser downloads and tool caches survive:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2. **Bake system deps into the image** (repeatable + persistent):

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3. **Install Playwright browsers without `npx`** (tránh xung đột ghi đè npm):
```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

If you need Playwright to install system deps, rebuild the image with
`OPENCLAW_DOCKER_APT_PACKAGES` instead of using `--with-deps` at runtime.

4. **Persist Playwright browser downloads**:

- Set `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` in
  `docker-compose.yml`.
- Ensure `/home/node` persists via `OPENCLAW_HOME_VOLUME`, or mount
  `/home/node/.cache/ms-playwright` via `OPENCLAW_EXTRA_MOUNTS`.

### Permissions + EACCES

The image runs as `node` (uid 1000). If you see permission errors on
`/home/node/.openclaw`, make sure your host bind mounts are owned by uid 1000.

Example (Linux host):

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

If you choose to run as root for convenience, you accept the security tradeoff.

### Faster rebuilds (recommended)

To speed up rebuilds, order your Dockerfile so dependency layers are cached.
This avoids re-running `pnpm install` unless lockfiles change:

```dockerfile
FROM node:22-bookworm

# Install Bun (required for build scripts)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Cache dependencies unless package metadata changes
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```
### Thiết lập kênh (tùy chọn)

Sử dụng container CLI để cấu hình các kênh, sau đó khởi động lại Gateway nếu cần.

WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (bot token):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (bot token):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Docs: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord)

### OpenAI Codex OAuth (headless Docker)

If you pick OpenAI Codex OAuth in the wizard, it opens a browser URL and tries
to capture a callback on `http://127.0.0.1:1455/auth/callback`. In Docker or
headless setups that callback can show a browser error. Copy the full redirect
URL you land on and paste it back into the wizard to finish auth.

### Health check

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### E2E smoke test (Docker)

```bash
scripts/e2e/onboard-docker.sh
```

### QR import smoke test (Docker)

```bash
pnpm test:docker:qr
```

### Notes

- Gateway bind defaults to `lan` for container use.
- Dockerfile CMD uses `--allow-unconfigured`; mounted config with `gateway.mode` not `local` will still start. Override CMD to enforce the guard.
- The gateway container is the source of truth for sessions (`~/.openclaw/agents/<agentId>/sessions/`).
## Agent Sandbox (host gateway + Docker tools)

Deep dive: [Sandboxing](/gateway/sandboxing)

### Nó làm gì

Khi `agents.defaults.sandbox` được bật, **các phiên không phải phiên chính** chạy các công cụ bên trong một container Docker. Gateway vẫn ở trên máy chủ của bạn, nhưng việc thực thi công cụ được cách ly:

- scope: `"agent"` theo mặc định (một container + workspace cho mỗi agent)
- scope: `"session"` để cách ly từng phiên
- thư mục workspace cho mỗi scope được gắn tại `/workspace`
- truy cập workspace agent tùy chọn (`agents.defaults.sandbox.workspaceAccess`)
- chính sách cho phép/từ chối công cụ (từ chối thắng)
- phương tiện truyền vào được sao chép vào workspace sandbox hoạt động (`media/inbound/*`) để các công cụ có thể đọc nó (với `workspaceAccess: "rw"`, điều này sẽ nằm trong workspace agent)

Cảnh báo: `scope: "shared"` vô hiệu hóa cách ly giữa các phiên. Tất cả các phiên chia sẻ một container và một workspace.

### Hồ sơ sandbox cho mỗi agent (multi-agent)

Nếu bạn sử dụng định tuyến multi-agent, mỗi agent có thể ghi đè các cài đặt sandbox + công cụ:
`agents.list[].sandbox` và `agents.list[].tools` (cộng với `agents.list[].tools.sandbox.tools`). Điều này cho phép bạn chạy các mức truy cập hỗn hợp trong một gateway:

- Truy cập đầy đủ (agent cá nhân)
- Công cụ chỉ đọc + workspace chỉ đọc (agent gia đình/công việc)
- Không có công cụ hệ thống tệp/shell (agent công khai)

Xem [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) để xem ví dụ, thứ tự ưu tiên và khắc phục sự cố.

### Hành vi mặc định

- Image: `openclaw-sandbox:bookworm-slim`
- Một container cho mỗi agent
- Truy cập workspace agent: `workspaceAccess: "none"` (mặc định) sử dụng `~/.openclaw/sandboxes`
  - `"ro"` giữ workspace sandbox tại `/workspace` và gắn workspace agent chỉ đọc tại `/agent` (vô hiệu hóa `write`/`edit`/`apply_patch`)
  - `"rw"` gắn workspace agent đọc/ghi tại `/workspace`
- Tự động dọn dẹp: nhàn rỗi > 24h HOẶC tuổi > 7 ngày
- Mạng: `none` theo mặc định (chọn tham gia rõ ràng nếu bạn cần egress)
  - `host` bị chặn.
  - `container:<id>` bị chặn theo mặc định (rủi ro namespace-join).
- Cho phép mặc định: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- Từ chối mặc định: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### Bật sandboxing

Nếu bạn dự định cài đặt các gói trong `setupCommand`, lưu ý:

- `docker.network` mặc định là `"none"` (không egress).
- `docker.network: "host"` bị chặn.
- `docker.network: "container:<id>"` bị chặn theo mặc định.
- Ghi đè break-glass: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.
- `readOnlyRoot: true` chặn cài đặt gói.
- `user` phải là root cho `apt-get` (bỏ qua `user` hoặc đặt `user: "0:0"`).
  OpenClaw tự động tạo lại các container khi `setupCommand` (hoặc cấu hình docker) thay đổi
  trừ khi container được **sử dụng gần đây** (trong ~5 phút). Các container nóng
  ghi lại một cảnh báo với lệnh `openclaw sandbox recreate ...` chính xác.
`json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
`
Các nút cứng hóa nằm dưới `agents.defaults.sandbox.docker`:
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`,
`dangerouslyAllowContainerNamespaceJoin` (chỉ dành cho trường hợp khẩn cấp).

Multi-agent: ghi đè `agents.defaults.sandbox.{docker,browser,prune}.*` cho mỗi agent thông qua `agents.list[].sandbox.{docker,browser,prune}.*`
(bị bỏ qua khi `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` là `"shared"`).

### Xây dựng hình ảnh sandbox mặc định

```bash
scripts/sandbox-setup.sh
```

This builds `openclaw-sandbox:bookworm-slim` using `Dockerfile.sandbox`.

### Sandbox common image (optional)

If you want a sandbox image with common build tooling (Node, Go, Rust, etc.), build the common image:

```bash
scripts/sandbox-common-setup.sh
```

This builds `openclaw-sandbox-common:bookworm-slim`. To use it:

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } },
    },
  },
}
```

### Sandbox browser image

To run the browser tool inside the sandbox, build the browser image:

```bash
scripts/sandbox-browser-setup.sh
```

This builds `openclaw-sandbox-browser:bookworm-slim` using
`Dockerfile.sandbox-browser`. The container runs Chromium with CDP enabled and
an optional noVNC observer (headful via Xvfb).

Notes:

- Headful (Xvfb) reduces bot blocking vs headless.
- Headless can still be used by setting `agents.defaults.sandbox.browser.headless=true`.
- No full desktop environment (GNOME) is needed; Xvfb provides the display.
- Browser containers default to a dedicated Docker network (`openclaw-sandbox-browser`) instead of global `bridge`.
- Optional `agents.defaults.sandbox.browser.cdpSourceRange` restricts container-edge CDP ingress by CIDR (for example `172.21.0.1/32``).
- Quyền truy cập noVNC observer được bảo vệ bằng mật khẩu theo mặc định; OpenClaw cung cấp URL token observer có thời hạn ngắn thay vì chia sẻ mật khẩu thô trong URL.

Sử dụng cấu hình:
```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true },
      },
    },
  },
}
```

Custom browser image:

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } },
    },
  },
}
```

When enabled, the agent receives:

- a sandbox browser control URL (for the `browser` tool)
- a noVNC URL (if enabled and headless=false)

Remember: if you use an allowlist for tools, add `browser` (and remove it from
deny) or the tool remains blocked.
Prune rules (`agents.defaults.sandbox.prune`) apply to browser containers too.

### Custom sandbox image

Build your own image and point config to it:

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } },
    },
  },
}
```

### Tool policy (allow/deny)

- `deny` wins over `allow`.
- If `allow` is empty: all tools (except deny) are available.
- If `allow` is non-empty: only tools in `allow` có sẵn (trừ deny).

### Chiến lược loại bỏ

Hai tùy chọn:
- `prune.idleHours`: xóa các container không được sử dụng trong X giờ (0 = tắt)
- `prune.maxAgeDays`: xóa các container cũ hơn X ngày (0 = tắt)

Ví dụ:

- Giữ các phiên đang hoạt động nhưng giới hạn thời gian tồn tại:
  `idleHours: 24`, `maxAgeDays: 7`
- Không bao giờ xóa:
  `idleHours: 0`, `maxAgeDays: 0`

### Ghi chú bảo mật

- Tường lửa cứng chỉ áp dụng cho **công cụ** (exec/read/write/edit/apply_patch).
- Các công cụ chỉ dành cho host như browser/camera/canvas bị chặn theo mặc định.
- Cho phép `browser` trong sandbox **phá vỡ cách ly** (browser chạy trên host).
## Khắc phục sự cố

- Hình ảnh bị thiếu: xây dựng với [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) hoặc đặt `agents.defaults.sandbox.docker.image`.
- Container không chạy: nó sẽ tự động tạo theo phiên khi cần.
- Lỗi quyền trong sandbox: đặt `docker.user` thành UID:GID phù hợp với quyền sở hữu không gian làm việc được gắn kết (hoặc chown thư mục không gian làm việc).
- Công cụ tùy chỉnh không được tìm thấy: OpenClaw chạy các lệnh với `sh -lc` (login shell), lệnh này sẽ tìm nạp `/etc/profile` và có thể đặt lại PATH. Đặt `docker.env.PATH` để thêm các đường dẫn công cụ tùy chỉnh của bạn (ví dụ: `/custom/bin:/usr/local/share/npm-global/bin`), hoặc thêm một script dưới `/etc/profile.d/` trong Dockerfile của bạn.