---
summary: 'Skills: quản lý workspace, quy tắc gating, và kết nối config/env'
read_when:
  - Thêm hoặc sửa đổi kỹ năng
  - Thay đổi quy tắc gating hoặc tải skill
title: Skills
x-i18n:
  source_path: tools\skills.md
  source_hash: 55511e1bd2438998137c1d50da73098a9b9afaf57b0e99e4f49c9a72a316349b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:33:35.822Z'
---

# Skills (OpenClaw)

OpenClaw sử dụng các thư mục kỹ năng tương thích với **[AgentSkills](https://agentskills.io)**để dạy agent cách sử dụng các công cụ. Mỗi kỹ năng là một thư mục chứa `SKILL.md` với frontmatter YAML và hướng dẫn. OpenClaw tải **các kỹ năng được đóng gói** cộng với các ghi đè cục bộ tùy chọn, và lọc chúng tại thời điểm tải dựa trên môi trường, cấu hình và sự hiện diện của tệp nhị phân.
## Vị trí và thứ tự ưu tiên

Skills được tải từ **ba** nơi:

1. **Skills được đóng gói**: được gửi kèm với bản cài đặt (gói npm hoặc OpenClaw.app)
2. **Skills được quản lý/cục bộ**: `~/.openclaw/skills`
3. **Skills không gian làm việc**: `<workspace>/skills`

Nếu tên Skill xung đột, thứ tự ưu tiên là:

`<workspace>/skills` (cao nhất) → `~/.openclaw/skills` → bundled skills (thấp nhất)

Ngoài ra, bạn có thể cấu hình các thư mục Skill bổ sung (thứ tự ưu tiên thấp nhất) thông qua
`skills.load.extraDirs` trong `~/.openclaw/openclaw.json`.
## Các Skills riêng cho từng agent so với Skills dùng chung

Trong các thiết lập **multi-agent**, mỗi agent có không gian làm việc riêng của nó. Điều đó có nghĩa là:

- **Các Skills riêng cho từng agent** nằm trong `<workspace>/skills` chỉ cho agent đó.
- **Các Skills dùng chung** nằm trong `~/.openclaw/skills` (được quản lý/cục bộ) và hiển thị cho
  **tất cả các agent** trên cùng một máy.
- **Các thư mục dùng chung** cũng có thể được thêm vào thông qua `skills.load.extraDirs` (mức độ ưu tiên thấp nhất) nếu bạn muốn một gói Skills chung được sử dụng bởi nhiều agent.

Nếu cùng một tên Skills tồn tại ở nhiều hơn một vị trí, mức độ ưu tiên thông thường sẽ áp dụng: workspace thắng, sau đó là managed/local, sau đó là bundled.
## Plugin + Skills

Plugin có thể cung cấp các Skills của riêng chúng bằng cách liệt kê các thư mục `skills` trong
`openclaw.plugin.json` (các đường dẫn tương đối với thư mục gốc của plugin). Các Skills của plugin được tải
khi plugin được bật và tham gia vào các quy tắc ưu tiên Skills thông thường.
Bạn có thể kiểm soát chúng thông qua `metadata.openclaw.requires.config` trên mục cấu hình của plugin.
Xem [Plugins](/tools/plugin) để biết về khám phá/cấu hình và [Tools](/tools) để biết về bề mặt công cụ mà các Skills đó dạy.
## ClawHub (cài đặt + đồng bộ)

ClawHub là kho đăng ký Skills công khai cho OpenClaw. Duyệt tại
[https://clawhub.com](https://clawhub.com). Sử dụng nó để khám phá, cài đặt, cập nhật và sao lưu Skills.
Hướng dẫn đầy đủ: [ClawHub](/tools/clawhub).

Các quy trình phổ biến:

- Cài đặt một Skill vào không gian làm việc của bạn:
  - `clawhub install <skill-slug>`
- Cập nhật tất cả các Skills đã cài đặt:
  - `clawhub update --all`
- Đồng bộ (quét + xuất bản cập nhật):
  - `clawhub sync --all`

Theo mặc định, `clawhub` cài đặt vào `./skills` trong thư mục làm việc hiện tại của bạn
(hoặc quay lại không gian làm việc OpenClaw được cấu hình). OpenClaw nhận
nó dưới dạng `<workspace>/skills` trong phiên tiếp theo.
## Ghi chú bảo mật

- Coi các Skills của bên thứ ba là **mã không đáng tin cậy**. Đọc chúng trước khi bật.
- Ưu tiên chạy trong sandbox cho các đầu vào không đáng tin cậy và các công cụ rủi ro. Xem [Sandboxing](/gateway/sandboxing).
- `skills.entries.*.env` và `skills.entries.*.apiKey` tiêm các bí mật vào quá trình **máy chủ**
  cho lượt agent đó (không phải sandbox). Giữ bí mật ra khỏi các lời nhắc và nhật ký.
- Để biết mô hình mối đe dọa rộng hơn và danh sách kiểm tra, xem [Security](/gateway/security).
## Định dạng (AgentSkills + Pi-compatible)

`SKILL.md` phải bao gồm ít nhất:

```markdown
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

Notes:

- We follow the AgentSkills spec for layout/intent.
- The parser used by the embedded agent supports **single-line** frontmatter keys only.
- `metadata` should be a **single-line JSON object**.
- Use `{baseDir}` in instructions to reference the skill folder path.
- Optional frontmatter keys:
  - `homepage` — URL surfaced as “Website” in the macOS Skills UI (also supported via `metadata.openclaw.homepage`).
  - `user-invocable` — `true|false` (default: `true`). When `true`, the skill is exposed as a user slash command.
  - `disable-model-invocation` — `true|false` (default: `false`). When `true`, the skill is excluded from the model prompt (still available via user invocation).
  - `command-dispatch` — `tool` (optional). When set to `tool`, the slash command bypasses the model and dispatches directly to a tool.
  - `command-tool` — tool name to invoke when `command-dispatch: tool` is set.
  - `command-arg-mode` — `raw` (default). For tool dispatch, forwards the raw args string to the tool (no core parsing).

    The tool is invoked with params:
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.
## Gating (bộ lọc thời gian tải)

OpenClaw **lọc Skills tại thời gian tải** sử dụng `metadata` (JSON một dòng):

```markdown
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

Fields under `metadata.openclaw`:

- `always: true` — always include the skill (skip other gates).
- `emoji` — optional emoji used by the macOS Skills UI.
- `homepage` — optional URL shown as “Website” in the macOS Skills UI.
- `os` — optional list of platforms (`darwin`, `linux`, `win32`). If set, the skill is only eligible on those OSes.
- `requires.bins` — list; each must exist on `PATH`.
- `requires.anyBins` — list; at least one must exist on `PATH`.
- `requires.env` — list; env var must exist **or** be provided in config.
- `requires.config` — list of `openclaw.json` paths that must be truthy.
- `primaryEnv` — env var name associated with `skills.entries.<name>.apiKey`.
- `install` — optional array of installer specs used by the macOS Skills UI (brew/node/go/uv/download).

Note on sandboxing:

- `requires.bins` is checked on the **host** at skill load time.
- If an agent is sandboxed, the binary must also exist **inside the container**.
  Install it via `agents.defaults.sandbox.docker.setupCommand` (or a custom image).
  `setupCommand` runs once after the container is created.
  Package installs also require network egress, a writable root FS, and a root user in the sandbox.
  Example: the `summarize` skill (`skills/summarize/SKILL.md`) needs the `summarize` CLI
  in the sandbox container to run there.

Installer example:

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```
Ghi chú:

- Nếu có nhiều trình cài đặt được liệt kê, gateway chọn **một** tùy chọn ưu tiên duy nhất (brew khi có sẵn, nếu không thì node).
- Nếu tất cả các trình cài đặt là `download`, OpenClaw liệt kê từng mục để bạn có thể xem các tạo phẩm có sẵn.
- Thông số kỹ thuật trình cài đặt có thể bao gồm `os: ["darwin"|"linux"|"win32"]` để lọc tùy chọn theo nền tảng.
- Cài đặt Node tuân theo `skills.install.nodeManager` trong `openclaw.json` (mặc định: npm; tùy chọn: npm/pnpm/yarn/bun).
  Điều này chỉ ảnh hưởng đến **cài đặt skill**; runtime Gateway vẫn phải là Node
  (Bun không được khuyến nghị cho WhatsApp/Telegram).
- Cài đặt Go: nếu `go` bị thiếu và `brew` có sẵn, gateway cài đặt Go qua Homebrew trước và đặt `GOBIN` thành `bin` của Homebrew khi có thể.
- Cài đặt tải xuống: `url` (bắt buộc), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (mặc định: tự động khi phát hiện lưu trữ), `stripComponents`, `targetDir` (mặc định: `~/.openclaw/tools/<skillKey>`).

Nếu không có `metadata.openclaw`, skill luôn đủ điều kiện (trừ khi
bị vô hiệu hóa trong cấu hình hoặc bị chặn bởi `skills.allowBundled` cho các skill được đóng gói).
## Ghi đè cấu hình (`~/.openclaw/openclaw.json`)

Các Skills được đóng gói/quản lý có thể được bật tắt và cung cấp các giá trị env:

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

Note: if the skill name contains hyphens, quote the key (JSON5 allows quoted keys).

Config keys match the **skill name** by default. If a skill defines
`metadata.openclaw.skillKey`, use that key under `skills.entries`.

Rules:

- `enabled: false` disables the skill even if it’s bundled/installed.
- `env`: injected **only if** the variable isn’t already set in the process.
- `apiKey`: convenience for skills that declare `metadata.openclaw.primaryEnv`.
  Supports plaintext string or SecretRef object (`{ source, provider, id }`).
- `config`: optional bag for custom per-skill fields; custom keys must live here.
- `allowBundled`: danh sách cho phép tùy chọn cho các Skills **được đóng gói** chỉ. Nếu được đặt, chỉ các Skills được đóng gói trong danh sách mới đủ điều kiện (các Skills được quản lý/workspace không bị ảnh hưởng).
## Tiêm biến môi trường (cho mỗi lần chạy agent)

Khi một lần chạy agent bắt đầu, OpenClaw:

1. Đọc siêu dữ liệu skill.
2. Áp dụng bất kỳ `skills.entries.<key>.env` hoặc `skills.entries.<key>.apiKey` cho
   `process.env`.
3. Xây dựng system prompt với các skills **đủ điều kiện**.
4. Khôi phục lại môi trường ban đầu sau khi lần chạy kết thúc.

Điều này **được giới hạn trong lần chạy agent**, không phải môi trường shell toàn cục.
## Ảnh chụp phiên (hiệu suất)

OpenClaw chụp ảnh các Skills đủ điều kiện **khi một phiên bắt đầu** và sử dụng lại danh sách đó cho các lượt tiếp theo trong cùng một phiên. Các thay đổi đối với Skills hoặc cấu hình sẽ có hiệu lực trong phiên mới tiếp theo.

Skills cũng có thể làm mới giữa phiên khi bộ giám sát Skills được bật hoặc khi một node từ xa đủ điều kiện mới xuất hiện (xem bên dưới). Hãy coi đây là một **hot reload**: danh sách làm mới sẽ được áp dụng trong lượt agent tiếp theo.
## Các node macOS từ xa (Gateway Linux)

Nếu Gateway đang chạy trên Linux nhưng một **node macOS** được kết nối **với `system.run` được phép** (bảo mật phê duyệt Exec không được đặt thành `deny`), OpenClaw có thể coi các Skills chỉ dành cho macOS là đủ điều kiện khi các tệp nhị phân cần thiết có mặt trên node đó. Agent nên thực thi các Skills đó thông qua công cụ `nodes` (thường là `nodes.run`).

Điều này dựa vào node báo cáo hỗ trợ lệnh của nó và trên một bộ thăm dò bin thông qua `system.run`. Nếu node macOS ngắt kết nối sau đó, các Skills vẫn hiển thị; các lần gọi có thể thất bại cho đến khi node kết nối lại.
## Skills watcher (tự động làm mới)

Theo mặc định, OpenClaw theo dõi các thư mục Skills và cập nhật ảnh chụp Skills khi `SKILL.md` thay đổi. Cấu hình điều này trong `skills.load`:

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```
## Tác động token (danh sách Skills)

Khi Skills đủ điều kiện, OpenClaw sẽ chèn một danh sách XML nhỏ gọn của các Skills có sẵn vào system prompt (thông qua `formatSkillsForPrompt` trong `pi-coding-agent`). Chi phí là xác định:

- **Overhead cơ bản (chỉ khi ≥1 skill):** 195 ký tự.
- **Mỗi skill:** 97 ký tự + độ dài của các giá trị `<name>`, `<description>` và `<location>` được escape XML.

Công thức (ký tự):

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Notes:

- XML escaping expands `& < > " '` into entities (`&amp;`, `&lt;`, v.v.), làm tăng độ dài.
- Số lượng token khác nhau tùy theo tokenizer của mô hình. Ước tính theo kiểu OpenAI là khoảng ~4 ký tự/token, vì vậy **97 ký tự ≈ 24 token** mỗi skill cộng với độ dài trường thực tế của bạn.
## Vòng đời Skills được quản lý

OpenClaw đi kèm với một bộ Skills cơ bản được gọi là **bundled skills** như một phần của quá trình cài đặt (npm package hoặc OpenClaw.app). `~/.openclaw/skills` tồn tại để ghi đè cục bộ (ví dụ: ghim/vá một Skill mà không thay đổi bản bundled). Workspace skills được sở hữu bởi người dùng và ghi đè cả hai khi có xung đột tên.
## Tham chiếu cấu hình

Xem [Cấu hình Skills](/tools/skills-config) để biết toàn bộ lược đồ cấu hình.
## Tìm kiếm thêm Skills?

Duyệt [https://clawhub.com](https://clawhub.com).

---