---
summary: 'Không gian làm việc của đặc vụ: vị trí, bố cục và chiến lược sao lưu'
read_when:
  - >-
    Bạn cần giải thích không gian làm việc của tác nhân hoặc cấu trúc tệp của
    nó.
  - Để sao lưu hoặc di chuyển không gian làm việc của agent
title: Không gian làm việc của nhân viên
x-i18n:
  source_path: concepts\agent-workspace.md
  source_hash: 44d5f78c26efd9cce70228910585da83224de4b4e45e9f612b4817113c9614c6
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:07:52.640Z'
---

# Không gian làm việc của agent

Không gian làm việc là "ngôi nhà" của agent. Đây là thư mục làm việc duy nhất được sử dụng cho các công cụ tệp và cho ngữ cảnh không gian làm việc. Hãy giữ nó riêng tư và coi nó như bộ nhớ.

Điều này tách biệt với `~/.openclaw/`, nơi lưu trữ config, credentials và sessions.

**Quan trọng:** không gian làm việc là **thư mục làm việc mặc định (cwd)**, không phải là một sandbox cứng. Các công cụ giải quyết các đường dẫn tương đối dựa trên không gian làm việc, nhưng các đường dẫn tuyệt đối vẫn có thể truy cập các vị trí khác trên máy chủ trừ khi sandboxing được bật. Nếu bạn cần cách ly, hãy sử dụng [`agents.defaults.sandbox`](/gateway/sandboxing) (và/hoặc config sandbox cho từng agent).
Khi sandboxing được bật và `workspaceAccess` không phải là `"rw"`, các công cụ hoạt động bên trong một sandbox workspace dưới `~/.openclaw/sandboxes`, chứ không phải không gian làm việc trên máy chủ của bạn.

## Vị trí mặc định

- Mặc định: `~/.openclaw/workspace`
- Nếu `OPENCLAW_PROFILE` được đặt và không phải `"default"`, mặc định sẽ là
  `~/.openclaw/workspace-<profile>`.
- Ghi đè trong `~/.openclaw/openclaw.json`:

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`, `openclaw configure`, or `openclaw setup` will create the
workspace and seed the bootstrap files if they are missing.

If you already manage the workspace files yourself, you can disable bootstrap
file creation:

```json5
{ agent: { skipBootstrap: true } }
```

## Thư mục không gian làm việc bổ sung

Các cài đặt cũ hơn có thể đã tạo `~/openclaw`. Việc giữ nhiều thư mục không gian làm việc có thể gây ra sự nhầm lẫn về xác thực hoặc trôi trạng thái, vì chỉ một không gian làm việc hoạt động tại một thời điểm.

**Khuyến nghị:** chỉ giữ một không gian làm việc hoạt động. Nếu bạn không còn sử dụng các thư mục bổ sung, hãy lưu trữ hoặc chuyển chúng vào Thùng rác (ví dụ `trash ~/openclaw`). Nếu bạn cố ý giữ nhiều không gian làm việc, hãy đảm bảo `agents.defaults.workspace` trỏ đến không gian làm việc đang hoạt động.

`openclaw doctor` cảnh báo khi phát hiện các thư mục không gian làm việc bổ sung.
## Sơ đồ tệp không gian làm việc (ý nghĩa của từng tệp)

Đây là các tệp tiêu chuẩn mà OpenClaw mong đợi trong không gian làm việc:

- `AGENTS.md`
  - Hướng dẫn vận hành cho agent và cách nó nên sử dụng bộ nhớ.
  - Được tải khi bắt đầu mỗi phiên.
  - Nơi tốt để đặt các quy tắc, ưu tiên và chi tiết về "cách ứng xử".

- `SOUL.md`
  - Tính cách, giọng điệu và giới hạn.
  - Được tải mỗi phiên.

- `USER.md`
  - Người dùng là ai và cách xưng hô với họ.
  - Được tải mỗi phiên.

- `IDENTITY.md`
  - Tên, phong cách và biểu tượng cảm xúc của agent.
  - Được tạo/cập nhật trong quá trình khởi động.

- `TOOLS.md`
  - Ghi chú về các công cụ và quy ước cục bộ của bạn.
  - Không kiểm soát tính khả dụng của công cụ; nó chỉ là hướng dẫn.

- `HEARTBEAT.md`
  - Danh sách kiểm tra nhỏ tùy chọn cho các lần chạy heartbeat.
  - Giữ ngắn gọn để tránh lãng phí token.

- `BOOT.md`
  - Danh sách kiểm tra khởi động tùy chọn được thực thi khi Gateway khởi động lại nếu các hook nội bộ được bật.
  - Giữ ngắn gọn; sử dụng công cụ tin nhắn để gửi đi.

- `BOOTSTRAP.md`
  - Nghi thức chạy lần đầu một lần.
  - Chỉ được tạo cho một không gian làm việc hoàn toàn mới.
  - Xóa nó sau khi nghi thức hoàn tất.

- `memory/YYYY-MM-DD.md`
  - Nhật ký bộ nhớ hàng ngày (một tệp mỗi ngày).
  - Nên đọc hôm nay + hôm qua khi bắt đầu phiên.

- `MEMORY.md` (tùy chọn)
  - Bộ nhớ dài hạn được quản lý.
  - Chỉ tải trong phiên chính, riêng tư (không phải ngữ cảnh chia sẻ/nhóm).

Xem [Bộ nhớ](/concepts/memory) để biết quy trình làm việc và xóa bộ nhớ tự động.

- `skills/` (tùy chọn)
  - Skills dành riêng cho không gian làm việc.
  - Ghi đè các Skills được quản lý/đóng gói khi tên bị trùng.

- `canvas/` (tùy chọn)
  - Các tệp giao diện người dùng Canvas để hiển thị node (ví dụ `canvas/index.html`).

Nếu bất kỳ tệp bootstrap nào bị thiếu, OpenClaw sẽ chèn một dấu hiệu "tệp bị thiếu" vào
phiên và tiếp tục. Các tệp bootstrap lớn sẽ bị cắt bớt khi được chèn;
điều chỉnh giới hạn bằng `agents.defaults.bootstrapMaxChars` (mặc định: 20000) và
`agents.defaults.bootstrapTotalMaxChars` (mặc định: 150000).
`openclaw setup` có thể tạo lại các giá trị mặc định bị thiếu mà không ghi đè lên các tệp hiện có
tệp.

## Những gì KHÔNG có trong không gian làm việc

Những mục này nằm dưới `~/.openclaw/` và KHÔNG nên được cam kết vào kho lưu trữ không gian làm việc:

- `~/.openclaw/openclaw.json` (cấu hình)
- `~/.openclaw/credentials/` (mã thông báo OAuth, khóa API)
- `~/.openclaw/agents/<agentId>/sessions/` (bản ghi phiên + siêu dữ liệu)
- __OC_I19N_0004__ (Skills được quản lý)

Nếu bạn cần di chuyển các phiên hoặc cấu hình, hãy sao chép chúng riêng biệt và giữ chúng
ngoài hệ thống kiểm soát phiên bản.

## Sao lưu Git (được khuyến nghị, riêng tư)

Coi không gian làm việc là bộ nhớ riêng tư. Đặt nó vào một kho lưu trữ git **riêng tư** để nó được
sao lưu và có thể khôi phục.

Thực hiện các bước này trên máy mà Gateway đang chạy (đó là nơi không gian làm việc
tồn tại).

### 1) Khởi tạo kho lưu trữ

Nếu git được cài đặt, các không gian làm việc hoàn toàn mới sẽ được khởi tạo tự động. Nếu không gian
làm việc này chưa phải là một kho lưu trữ, hãy chạy:

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) Add a private remote (beginner-friendly options)

Option A: GitHub web UI

1. Create a new **private** repository on GitHub.
2. Do not initialize with a README (avoids merge conflicts).
3. Copy the HTTPS remote URL.
4. Add the remote and push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```
Tùy chọn B: GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Option C: GitLab web UI

1. Create a new **private** repository on GitLab.
2. Do not initialize with a README (avoids merge conflicts).
3. Copy the HTTPS remote URL.
4. Add the remote and push:

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) Ongoing updates

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## Không cam kết bí mật

Ngay cả trong một kho lưu trữ riêng tư, hãy tránh lưu trữ bí mật trong không gian làm việc:

- Khóa API, mã thông báo OAuth, mật khẩu hoặc thông tin xác thực riêng tư.
- Bất cứ thứ gì thuộc `~/.openclaw/`.
- Các bản sao thô của cuộc trò chuyện hoặc tệp đính kèm nhạy cảm.

Nếu bạn phải lưu trữ các tham chiếu nhạy cảm, hãy sử dụng trình giữ chỗ và giữ bí mật thực sự ở nơi khác (trình quản lý mật khẩu, biến môi trường hoặc `~/.openclaw/`).

Gợi ý khởi đầu `.gitignore`:

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## Di chuyển không gian làm việc sang máy mới

1. Sao chép kho lưu trữ vào đường dẫn mong muốn (mặc định `~/.openclaw/workspace`).
2. Đặt `agents.defaults.workspace` thành đường dẫn đó trong `~/.openclaw/openclaw.json`.
3. Chạy `openclaw setup --workspace <path>` để tạo bất kỳ tệp nào bị thiếu.
4. Nếu bạn cần các phiên, hãy sao chép `~/.openclaw/agents/<agentId>/sessions/` từ máy cũ một cách riêng biệt.

## Ghi chú nâng cao

- Định tuyến đa agent có thể sử dụng các không gian làm việc khác nhau cho mỗi agent. Xem
  [Định tuyến kênh](/channels/channel-routing) để biết cấu hình định tuyến.
- Nếu `agents.defaults.sandbox` được bật, các phiên không phải phiên chính có thể sử dụng các không gian làm việc sandbox theo phiên dưới `agents.defaults.sandbox.workspaceRoot`.
