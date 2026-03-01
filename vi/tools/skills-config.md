---
summary: Lược đồ cấu hình Skills và các ví dụ
read_when:
  - Thêm hoặc sửa đổi cấu hình Skills
  - Điều chỉnh danh sách cho phép đi kèm hoặc hành vi cài đặt
title: Cấu hình Skills
x-i18n:
  source_path: tools\skills-config.md
  source_hash: 6f00565595d7ab01892e45e38152c2f81220db6b1c998b2fdc49ec1cf4d7dcf4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:31:54.596Z'
---

# Cấu hình Skills

Tất cả cấu hình liên quan đến Skills nằm dưới `skills` trong `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```
## Các trường

- `allowBundled`: danh sách cho phép tùy chọn cho các Skills **được đóng gói** chỉ. Khi được đặt, chỉ các Skills được đóng gói trong danh sách mới đủ điều kiện (các Skills được quản lý/workspace không bị ảnh hưởng).
- `load.extraDirs`: các thư mục Skills bổ sung để quét (ưu tiên thấp nhất).
- `load.watch`: theo dõi các thư mục Skills và làm mới ảnh chụp Skills (mặc định: true).
- `load.watchDebounceMs`: debounce cho các sự kiện trình theo dõi Skills tính bằng mili giây (mặc định: 250).
- `install.preferBrew`: ưu tiên trình cài đặt brew khi có sẵn (mặc định: true).
- `install.nodeManager`: tùy chọn ưu tiên trình cài đặt node (`npm` | `pnpm` | `yarn` | `bun`, mặc định: npm).
  Điều này chỉ ảnh hưởng đến **cài đặt Skills**; runtime Gateway vẫn phải là Node
  (Bun không được khuyến nghị cho WhatsApp/Telegram).
- `entries.<skillKey>`: ghi đè từng Skills.

Các trường từng Skills:

- `enabled`: đặt `false` để vô hiệu hóa một Skill ngay cả khi nó được đóng gói/cài đặt.
- `env`: biến môi trường được tiêm cho lần chạy agent (chỉ nếu chưa được đặt).
- `apiKey`: tiện ích tùy chọn cho các Skills khai báo một biến môi trường chính.
  Hỗ trợ chuỗi văn bản thuần túy hoặc đối tượng SecretRef (`{ source, provider, id }`).
## Ghi chú

- Các khóa dưới `entries` ánh xạ tới tên skill theo mặc định. Nếu một skill định nghĩa `metadata.openclaw.skillKey`, hãy sử dụng khóa đó thay thế.
- Các thay đổi đối với skills được áp dụng vào lượt agent tiếp theo khi trình giám sát được bật.

### Sandboxed skills + biến môi trường

Khi một phiên được **sandboxed**, các quy trình skill chạy bên trong Docker. Sandbox **không** kế thừa `process.env` của máy chủ.

Sử dụng một trong những cách sau:

- `agents.defaults.sandbox.docker.env` (hoặc `agents.list[].sandbox.docker.env` cho từng agent)
- bake env vào hình ảnh sandbox tùy chỉnh của bạn

`env` và `skills.entries.<skill>.env/apiKey` toàn cục chỉ áp dụng cho các lần chạy **host**.