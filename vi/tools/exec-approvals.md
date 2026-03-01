---
summary: 'Phê duyệt Exec, danh sách cho phép và các lời nhắc thoát sandbox'
read_when:
  - Cấu hình phê duyệt exec hoặc danh sách cho phép
  - Triển khai UX phê duyệt exec trong ứng dụng macOS
  - Xem xét các prompt thoát khỏi sandbox và những hàm ý của chúng
title: Phê duyệt Exec
x-i18n:
  source_path: tools\exec-approvals.md
  source_hash: ba5563de5adaf9e1fd59c73db00ff11bf4cf50ee5c4f4dcbece221964ca9af8b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:30:15.875Z'
---

# Phê duyệt Exec

Phê duyệt Exec là **ứng dụng đi kèm / guardrail máy chủ nút** để cho phép một agent sandbox chạy
các lệnh trên máy chủ thực (`gateway` hoặc `node`). Hãy coi nó như một khóa an toàn:
các lệnh chỉ được phép khi chính sách + danh sách cho phép + (tùy chọn) phê duyệt của người dùng đều đồng ý.
Phê duyệt Exec là **bổ sung** cho chính sách công cụ và gating nâng cao (trừ khi elevated được đặt thành `full`, điều này bỏ qua phê duyệt).
Chính sách hiệu quả là **nghiêm ngặt hơn** của `tools.exec.*` và giá trị mặc định phê duyệt; nếu một trường phê duyệt bị bỏ qua, giá trị `tools.exec` được sử dụng.

Nếu giao diện người dùng ứng dụng đi kèm **không khả dụng**, bất kỳ yêu cầu nào yêu cầu lời nhắc sẽ
được giải quyết bằng **ask fallback** (mặc định: từ chối).
## Nơi áp dụng

Phê duyệt thực thi được thực thi cục bộ trên máy chủ thực thi:

- **máy chủ gateway** → `openclaw` process trên máy gateway
- **máy chủ node** → node runner (ứng dụng đi kèm macOS hoặc máy chủ node không có giao diện)

Ghi chú mô hình tin cậy:

- Những người gọi được xác thực Gateway là các nhà điều hành đáng tin cậy cho Gateway đó.
- Các node được ghép nối mở rộng khả năng nhà điều hành đáng tin cậy đó lên máy chủ node.
- Phê duyệt thực thi giảm rủi ro thực thi vô tình, nhưng không phải là ranh giới xác thực cho mỗi người dùng.

Phân chia macOS:

- **dịch vụ máy chủ node** chuyển tiếp `system.run` đến **ứng dụng macOS** qua local IPC.
- **ứng dụng macOS** thực thi phê duyệt + thực thi lệnh trong ngữ cảnh giao diện người dùng.
## Cài đặt và lưu trữ

Các phê duyệt được lưu trữ trong một tệp JSON cục bộ trên máy chủ thực thi:

`~/.openclaw/exec-approvals.json`

Ví dụ về lược đồ:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```
## Các công tắc chính sách

### Bảo mật (`exec.security`)

- **deny**: chặn tất cả các yêu cầu thực thi trên máy chủ.
- **allowlist**: chỉ cho phép các lệnh được phép.
- **full**: cho phép mọi thứ (tương đương với elevated).

### Hỏi (`exec.ask`)

- **off**: không bao giờ nhắc.
- **on-miss**: chỉ nhắc khi allowlist không khớp.
- **always**: nhắc trên mỗi lệnh.

### Fallback hỏi (`askFallback`)

Nếu cần nhắc nhưng không thể tiếp cận giao diện nào, fallback quyết định:

- **deny**: chặn.
- **allowlist**: chỉ cho phép nếu allowlist khớp.
- **full**: cho phép.
## Danh sách cho phép (mỗi agent)

Danh sách cho phép là **mỗi agent**. Nếu tồn tại nhiều agent, hãy chuyển đổi agent mà bạn đang chỉnh sửa trong ứng dụng macOS. Các mẫu là **khớp glob không phân biệt chữ hoa chữ thường**.
Các mẫu phải phân giải thành **đường dẫn nhị phân** (các mục chỉ chứa tên cơ sở bị bỏ qua).
Các mục `agents.default` cũ được di chuyển sang `agents.main` khi tải.

Ví dụ:

- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

Mỗi mục danh sách cho phép theo dõi:

- **id** UUID ổn định được sử dụng cho danh tính giao diện người dùng (tùy chọn)
- **last used** dấu thời gian
- **last used command** lệnh được sử dụng lần cuối
- **last resolved path** đường dẫn được phân giải lần cuối
## Tự động cho phép CLI của Skills

Khi **Tự động cho phép CLI của Skills** được bật, các tệp thực thi được tham chiếu bởi các Skills đã biết
được coi là được phép trên các node (node macOS hoặc máy chủ node không đầu). Điều này sử dụng
`skills.bins` qua Gateway RPC để tìm nạp danh sách bin của skill. Vô hiệu hóa tùy chọn này nếu bạn muốn danh sách cho phép thủ công nghiêm ngặt.

Ghi chú tin tưởng quan trọng:

- Đây là một **danh sách cho phép tiện lợi ngầm**, riêng biệt với các mục danh sách cho phép đường dẫn thủ công.
- Nó được dự định cho các môi trường nhà điều hành đáng tin cậy nơi Gateway và node nằm trong cùng một ranh giới tin tưởng.
- Nếu bạn yêu cầu tin tưởng rõ ràng nghiêm ngặt, hãy giữ `autoAllowSkills: false` và chỉ sử dụng các mục danh sách cho phép đường dẫn thủ công.
## Các thùng an toàn (chỉ stdin)

`tools.exec.safeBins` định nghĩa một danh sách nhỏ các tệp nhị phân **chỉ stdin** (ví dụ `jq`)
có thể chạy ở chế độ danh sách cho phép **mà không** cần các mục danh sách cho phép rõ ràng. Các thùng an toàn từ chối
các đối số tệp vị trí và token giống đường dẫn, vì vậy chúng chỉ có thể hoạt động trên luồng đến.
Hãy coi đây là một đường dẫn nhanh hẹp cho các bộ lọc luồng, không phải danh sách tin cậy chung.
**Không** thêm các tệp nhị phân trình thông dịch hoặc thời gian chạy (ví dụ `python3`, `node`, `ruby`, `bash`, `sh`, `zsh`) vào `safeBins`.
Nếu một lệnh có thể đánh giá mã, thực thi các lệnh con hoặc đọc tệp theo thiết kế, hãy ưu tiên các mục danh sách cho phép rõ ràng và giữ các lời nhắc phê duyệt được bật.
Các thùng an toàn tùy chỉnh phải định nghĩa một hồ sơ rõ ràng trong `tools.exec.safeBinProfiles.<bin>`.
Xác thực là xác định từ hình dạng argv chỉ (không có kiểm tra tồn tại hệ thống tệp máy chủ), điều này
ngăn chặn hành vi oracle tồn tại tệp từ sự khác biệt cho phép/từ chối.
Các tùy chọn hướng tệp bị từ chối cho các thùng an toàn mặc định (ví dụ `sort -o`, `sort --output`,
`sort --files0-from`, `sort --compress-program`, `sort --random-source`,
`sort --temporary-directory`/`-T`, `wc --files0-from`, `jq -f/--from-file`,
`grep -f/--file`).
Các thùng an toàn cũng thực thi chính sách cờ rõ ràng cho mỗi tệp nhị phân cho các tùy chọn phá vỡ hành vi
chỉ stdin (ví dụ `sort -o/--output/--compress-program` và các cờ đệ quy của grep).
Các tùy chọn dài được xác thực fail-closed ở chế độ safe-bin: các cờ không xác định và viết tắt không rõ ràng
bị từ chối.
Các cờ bị từ chối theo hồ sơ safe-bin:

<!-- SAFE_BIN_DENIED_FLAGS:START -->

- `grep`: `--dereference-recursive`, `--directories`, `--exclude-from`, `--file`, `--recursive`, `-R`, `-d`, `-f`, `-r`
- `jq`: `--argfile`, `--from-file`, `--library-path`, `--rawfile`, `--slurpfile`, `-L`, `-f`
- `sort`: `--compress-program`, `--files0-from`, `--output`, `--random-source`, `--temporary-directory`, `-T`, `-o`
- `wc`: `--files0-from`
<!-- SAFE_BIN_DENIED_FLAGS:END -->

Các thùng an toàn cũng buộc các token argv được coi là **văn bản theo nghĩa đen** tại thời gian thực thi (không có globbing
và không có mở rộng `$VARS`) cho các phân đoạn chỉ stdin, vì vậy các mẫu như `*` hoặc `$HOME/...` không thể
được sử dụng để buộc đọc tệp.
Các thùng an toàn cũng phải phân giải từ các thư mục tệp nhị phân đáng tin cậy (mặc định hệ thống cộng với tùy chọn
`tools.exec.safeBinTrustedDirs`). Các mục `PATH` không bao giờ được tự động tin cậy.
Các thư mục safe-bin đáng tin cậy mặc định được cố ý tối thiểu: `/bin`, `/usr/bin`.
Nếu tệp nhị phân safe-bin của bạn nằm trong các đường dẫn trình quản lý gói/người dùng (ví dụ
`/opt/homebrew/bin`, `/usr/local/bin`, `/opt/local/bin`, `/snap/bin`), hãy thêm chúng rõ ràng
vào `tools.exec.safeBinTrustedDirs`.
Chuỗi shell và chuyển hướng không được tự động cho phép ở chế độ danh sách cho phép.

Chuỗi shell (`&&`, `||`, `;`) được cho phép khi mỗi phân đoạn cấp cao nhất thỏa mãn danh sách cho phép
(bao gồm các thùng an toàn hoặc tự động cho phép skill). Các chuyển hướng vẫn không được hỗ trợ ở chế độ danh sách cho phép.
Thay thế lệnh (`$()` / backticks) bị từ chối trong quá trình phân tích danh sách cho phép, bao gồm bên trong
dấu ngoặc kép; sử dụng dấu ngoặc đơn nếu bạn cần văn bản `$()` theo nghĩa đen.
Trên các phê duyệt ứng dụng đi kèm macOS, văn bản shell thô chứa shell control hoặc cú pháp mở rộng
(`&&`, `||`, `;`, `|`, `` ` ``, `$`, `<`, `>`, `(`, `)`) is treated as an allowlist miss unless
the shell binary itself is allowlisted.
For shell wrappers (`bash|sh|zsh ... -c/-lc`), request-scoped env overrides are reduced to a
small explicit allowlist (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
For allow-always decisions in allowlist mode, known dispatch wrappers
(`env`, `nice`, `nohup`, `stdbuf`, `timeout`) persist inner executable paths instead of wrapper
paths. Shell multiplexers (`busybox`, `toybox`) are also unwrapped for shell applets (`sh`, `ash`,
etc.) so inner executables are persisted instead of multiplexer binaries. If a wrapper or
multiplexer cannot be safely unwrapped, no allowlist entry is persisted automatically.

Default safe bins: `jq`, `cut`, `uniq`, `head`, `tail`, `tr`, `wc`.

`grep` and `sort` are not in the default list. If you opt in, keep explicit allowlist entries for
their non-stdin workflows.
For `grep` in safe-bin mode, provide the pattern with `-e`/`--regexp`; dạng mẫu vị trí là
bị từ chối nên các toán hạng tệp không thể được đưa vào như các vị trí tích cực không rõ ràng.

### Safe bins so với allowlist

| Chủ đề            | `tools.exec.safeBins`                                  | Allowlist (`exec-approvals.json`)                            |
| ---------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| Mục tiêu             | Tự động cho phép các bộ lọc stdin hẹp                        | Tin tưởng rõ ràng các tệp thực thi cụ thể                        |
| Loại khớp       | Tên tệp thực thi + chính sách argv safe-bin                 | Mẫu glob đường dẫn tệp thực thi đã giải quyết                        |
| Phạm vi đối số   | Bị hạn chế bởi hồ sơ safe-bin và quy tắc literal-token | Chỉ khớp đường dẫn; các đối số khác là trách nhiệm của bạn |
| Ví dụ điển hình | `jq`, `head`, `tail`, `wc`                             | `python3`, `node`, `ffmpeg`, CLI tùy chỉnh                     |
| Sử dụng tốt nhất         | Các phép biến đổi văn bản rủi ro thấp trong đường ống                  | Bất kỳ công cụ nào có hành vi rộng hơn hoặc tác dụng phụ               |

Vị trí cấu hình:

- `safeBins` đến từ cấu hình (`tools.exec.safeBins` hoặc `agents.list[].tools.exec.safeBins` cho từng agent).
- `safeBinTrustedDirs` đến từ cấu hình (`tools.exec.safeBinTrustedDirs` hoặc `agents.list[].tools.exec.safeBinTrustedDirs` cho từng agent).
- `safeBinProfiles` đến từ cấu hình (`tools.exec.safeBinProfiles` hoặc `agents.list[].tools.exec.safeBinProfiles` cho từng agent). Các khóa hồ sơ cho từng agent ghi đè các khóa toàn cầu.
- các mục allowlist nằm trong `~/.openclaw/exec-approvals.json` cục bộ máy chủ dưới `agents.<id>.allowlist` (hoặc thông qua Control UI / `openclaw approvals allowlist ...`).
- `openclaw security audit` cảnh báo với `tools.exec.safe_bins_interpreter_unprofiled` khi các tệp interpreter/runtime xuất hiện trong `safeBins` mà không có hồ sơ rõ ràng.
- `openclaw doctor --fix` có thể tạo các mục `safeBinProfiles.<bin>` tùy chỉnh bị thiếu dưới dạng `{}` (xem xét và siết chặt sau đó). Các tệp interpreter/runtime không được tạo tự động.

Ví dụ hồ sơ tùy chỉnh:

```json5
{
  tools: {
    exec: {
      safeBins: ["jq", "myfilter"],
      safeBinProfiles: {
        myfilter: {
          minPositional: 0,
          maxPositional: 0,
          allowedValueFlags: ["-n", "--limit"],
          deniedFlags: ["-f", "--file", "-c", "--command"],
        },
      },
    },
  },
}
```
## Chỉnh sửa Control UI

Sử dụng thẻ **Control UI → Nodes → Exec approvals** để chỉnh sửa các giá trị mặc định, ghi đè cho từng agent và danh sách cho phép. Chọn một phạm vi (Defaults hoặc một agent), điều chỉnh chính sách, thêm/xóa các mẫu danh sách cho phép, sau đó **Save**. UI hiển thị siêu dữ liệu **last used** cho mỗi mẫu để bạn có thể giữ danh sách gọn gàng.

Bộ chọn mục tiêu chọn **Gateway** (phê duyệt cục bộ) hoặc một **Node**. Các Node phải quảng cáo `system.execApprovals.get/set` (ứng dụng macOS hoặc máy chủ node không đầu).
Nếu một node chưa quảng cáo exec approvals, hãy chỉnh sửa trực tiếp `~/.openclaw/exec-approvals.json` cục bộ của nó.

CLI: `openclaw approvals` hỗ trợ chỉnh sửa gateway hoặc node (xem [Approvals CLI](/cli/approvals)).
## Luồng phê duyệt

Khi cần nhắc nhở, gateway phát sóng `exec.approval.requested` đến các client của operator.
Control UI và ứng dụng macOS giải quyết nó thông qua `exec.approval.resolve`, sau đó gateway chuyển tiếp
yêu cầu được phê duyệt đến host node.

Khi cần phê duyệt, công cụ exec trả về ngay lập tức với một id phê duyệt. Sử dụng id đó để
liên kết các sự kiện hệ thống sau này (`Exec finished` / `Exec denied`). Nếu không có quyết định nào đến trước khi
hết thời gian chờ, yêu cầu được coi là hết thời gian phê duyệt và được hiển thị như một lý do từ chối.

Hộp thoại xác nhận bao gồm:

- lệnh + đối số
- cwd
- id agent
- đường dẫn tệp thực thi đã giải quyết
- metadata host + policy

Hành động:

- **Cho phép một lần** → chạy ngay bây giờ
- **Luôn cho phép** → thêm vào danh sách cho phép + chạy
- **Từ chối** → chặn
## Chuyển tiếp phê duyệt thực thi đến các kênh chat

Bạn có thể chuyển tiếp các lời nhắc phê duyệt thực thi đến bất kỳ kênh chat nào (bao gồm các kênh plugin) và phê duyệt chúng bằng `/approve`. Điều này sử dụng đường ống gửi đi bình thường.

Cấu hình:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

Reply in chat:

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

### macOS IPC flow

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Security notes:

- Unix socket mode `0600`, token stored in `exec-approvals.json`.
- Kiểm tra peer cùng UID.
- Challenge/response (nonce + mã thông báo HMAC + hash yêu cầu) + TTL ngắn.
## Sự kiện hệ thống

Vòng đời Exec được hiển thị dưới dạng các tin nhắn hệ thống:

- `Exec running` (chỉ khi lệnh vượt quá ngưỡng thông báo đang chạy)
- `Exec finished`
- `Exec denied`

Những tin nhắn này được đăng vào phiên của agent sau khi node báo cáo sự kiện.
Các phê duyệt exec Gateway-host phát ra các sự kiện vòng đời tương tự khi lệnh kết thúc (và tùy chọn khi chạy lâu hơn ngưỡng).
Các exec được gated phê duyệt sử dụng lại id phê duyệt làm `runId` trong các tin nhắn này để dễ dàng tương quan.
## Ý nghĩa

- **full** rất mạnh mẽ; ưu tiên danh sách cho phép khi có thể.
- **ask** giữ bạn trong vòng lặp trong khi vẫn cho phép phê duyệt nhanh.
- Danh sách cho phép theo agent ngăn chặn phê duyệt của một agent rò rỉ sang agent khác.
- Phê duyệt chỉ áp dụng cho các yêu cầu host exec từ **những người gửi được phép**. Những người gửi không được phép không thể phát hành `/exec`.
- `/exec security=full` là một tiện ích cấp phiên cho các nhà điều hành được phép và bỏ qua phê duyệt theo thiết kế.
  Để chặn cứng host exec, đặt bảo mật phê duyệt thành `deny` hoặc từ chối công cụ `exec` thông qua chính sách công cụ.

Liên quan:

- [Công cụ Exec](/tools/exec)
- [Chế độ nâng cao](/tools/elevated)
- [Skills](/tools/skills)