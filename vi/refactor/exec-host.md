---
summary: >-
  Kế hoạch tái cấu trúc: định tuyến máy chủ thực thi, phê duyệt nút và trình
  chạy không giao diện
read_when:
  - Thiết kế định tuyến exec host hoặc phê duyệt exec
  - Triển khai node runner + UI IPC
  - Thêm chế độ bảo mật exec host và lệnh slash
title: Tái cấu trúc Exec Host
x-i18n:
  source_path: refactor\exec-host.md
  source_hash: 53a9059cbeb1f3f1dbb48c2b5345f88ca92372654fef26f8481e651609e45e3a
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:18:31.942Z'
---

# Kế hoạch tái cấu trúc Exec host

## Mục tiêu

- Thêm `exec.host` + `exec.security` để định tuyến thực thi trên **sandbox**, **gateway**, và **node**.
- Giữ mặc định **an toàn**: không thực thi liên máy chủ trừ khi được bật rõ ràng.
- Chia thực thi thành một **dịch vụ runner không giao diện** với UI tùy chọn (ứng dụng macOS) thông qua IPC cục bộ.
- Cung cấp **chính sách cho từng agent**, danh sách cho phép, chế độ hỏi, và ràng buộc node.
- Hỗ trợ **các chế độ hỏi** hoạt động _với_ hoặc _không có_ danh sách cho phép.
- Đa nền tảng: Unix socket + xác thực token (tương thích macOS/Linux/Windows).
## Các mục không phải là mục tiêu

- Không hỗ trợ di chuyển danh sách cho phép cũ hoặc hỗ trợ lược đồ cũ.
- Không hỗ trợ PTY/truyền phát cho thực thi node (chỉ đầu ra tổng hợp).
- Không có lớp mạng mới ngoài Bridge + Gateway hiện có.
## Quyết định (khóa)

- **Khóa cấu hình:** `exec.host` + `exec.security` (cho phép ghi đè cho từng agent).
- **Nâng cao quyền:** giữ `/elevated` làm bí danh cho quyền truy cập đầy đủ của Gateway.
- **Mặc định Ask:** `on-miss`.
- **Kho lưu trữ phê duyệt:** `~/.openclaw/exec-approvals.json` (JSON, không di chuyển kế thừa).
- **Runner:** dịch vụ hệ thống không giao diện; ứng dụng UI lưu trữ Unix socket cho phê duyệt.
- **Nhận dạng node:** sử dụng `nodeId` hiện có.
- **Xác thực Unix socket:** Unix socket + token (đa nền tảng); chia tách sau nếu cần.
- **Trạng thái máy chủ node:** `~/.openclaw/node.json` (id node + token ghép nối).
- **Máy chủ thực thi macOS:** chạy `system.run` bên trong ứng dụng macOS; dịch vụ máy chủ node chuyển tiếp yêu cầu qua IPC cục bộ.
- **Không có trợ giúp XPC:** tuân thủ Unix socket + token + kiểm tra ngang hàng.
## Các khái niệm chính

### Host

- `sandbox`: Docker exec (hành vi hiện tại).
- `gateway`: exec trên host Gateway.
- `node`: exec trên node runner qua Bridge (`system.run`).

### Security mode

- `deny`: luôn chặn.
- `allowlist`: chỉ cho phép các kết quả khớp.
- `full`: cho phép mọi thứ (tương đương với elevated).

### Ask mode

- `off`: không bao giờ hỏi.
- `on-miss`: chỉ hỏi khi allowlist không khớp.
- `always`: hỏi mỗi lần.

Ask là **độc lập** với allowlist; allowlist có thể được sử dụng với `always` hoặc `on-miss`.

### Policy resolution (mỗi exec)

1. Resolve `exec.host` (tool param → agent override → global default).
2. Resolve `exec.security` và `exec.ask` (cùng mức ưu tiên).
3. Nếu host là `sandbox`, tiến hành với local sandbox exec.
4. Nếu host là `gateway` hoặc `node`, áp dụng security + ask policy trên host đó.
## Bảo mật mặc định

- Mặc định `exec.host = sandbox`.
- Mặc định `exec.security = deny` cho `gateway` và `node`.
- Mặc định `exec.ask = on-miss` (chỉ liên quan nếu bảo mật cho phép).
- Nếu không đặt node binding, **agent có thể nhắm mục tiêu bất kỳ node nào**, nhưng chỉ nếu chính sách cho phép.
## Bề mặt cấu hình

### Tham số công cụ

- `exec.host` (tùy chọn): `sandbox | gateway | node`.
- `exec.security` (tùy chọn): `deny | allowlist | full`.
- `exec.ask` (tùy chọn): `off | on-miss | always`.
- `exec.node` (tùy chọn): id/tên node để sử dụng khi `host=node`.

### Các khóa cấu hình (toàn cục)

- `tools.exec.host`
- `tools.exec.security`
- `tools.exec.ask`
- `tools.exec.node` (liên kết node mặc định)

### Các khóa cấu hình (cho mỗi agent)

- `agents.list[].tools.exec.host`
- `agents.list[].tools.exec.security`
- `agents.list[].tools.exec.ask`
- `agents.list[].tools.exec.node`

### Bí danh

- `/elevated on` = đặt `tools.exec.host=gateway`, `tools.exec.security=full` cho phiên agent.
- `/elevated off` = khôi phục cài đặt exec trước đó cho phiên agent.
## Kho lưu trữ phê duyệt (JSON)

Đường dẫn: `~/.openclaw/exec-approvals.json`

Mục đích:

- Chính sách cục bộ + danh sách cho phép cho **máy chủ thực thi** (gateway hoặc node runner).
- Yêu cầu dự phòng khi không có UI khả dụng.
- Thông tin xác thực IPC cho các client UI.

Lược đồ được đề xuất (v1):

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Notes:

- No legacy allowlist formats.
- `askFallback` applies only when `ask` is required and no UI is reachable.
- File permissions: `0600`.
## Dịch vụ Runner (headless)

### Vai trò

- Thực thi `exec.security` + `exec.ask` cục bộ.
- Thực thi các lệnh hệ thống và trả về kết quả.
- Phát các sự kiện Bridge cho vòng đời exec (tùy chọn nhưng được khuyến nghị).

### Vòng đời dịch vụ

- Launchd/daemon trên macOS; dịch vụ hệ thống trên Linux/Windows.
- JSON phê duyệt là cục bộ trên máy chủ thực thi.
- UI lưu trữ một Unix socket cục bộ; các runner kết nối theo yêu cầu.
## Tích hợp giao diện (ứng dụng macOS)

### IPC

- Unix socket tại `~/.openclaw/exec-approvals.sock` (0600).
- Token được lưu trữ trong `exec-approvals.json` (0600).
- Kiểm tra peer: chỉ cùng UID.
- Challenge/response: nonce + HMAC(token, request-hash) để ngăn chặn replay.
- TTL ngắn (ví dụ: 10s) + payload tối đa + giới hạn tốc độ.

### Luồng Ask (máy chủ thực thi ứng dụng macOS)

1. Dịch vụ Node nhận `system.run` từ gateway.
2. Dịch vụ Node kết nối đến socket cục bộ và gửi yêu cầu prompt/exec.
3. Ứng dụng xác thực peer + token + HMAC + TTL, sau đó hiển thị hộp thoại nếu cần.
4. Ứng dụng thực thi lệnh trong ngữ cảnh giao diện và trả về kết quả.
5. Dịch vụ Node trả về kết quả cho gateway.

Nếu giao diện bị thiếu:

- Áp dụng `askFallback` (`deny|allowlist|full`).

### Sơ đồ (SCI)

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```
## Danh tính node + ràng buộc

- Sử dụng `nodeId` hiện có từ Bridge pairing.
- Mô hình ràng buộc:
  - `tools.exec.node` hạn chế agent đến một node cụ thể.
  - Nếu không được đặt, agent có thể chọn bất kỳ node nào (chính sách vẫn thực thi các giá trị mặc định).
- Phân giải lựa chọn node:
  - `nodeId` khớp chính xác
  - `displayName` (chuẩn hóa)
  - `remoteIp`
  - `nodeId` tiền tố (>= 6 ký tự)
## Sự kiện

### Ai nhìn thấy sự kiện

- Sự kiện hệ thống là **mỗi phiên** và được hiển thị cho agent ở lời nhắc tiếp theo.
- Được lưu trữ trong hàng đợi bộ nhớ của gateway (`enqueueSystemEvent`).

### Văn bản sự kiện

- `Exec started (node=<id>, id=<runId>)`
- `Exec finished (node=<id>, id=<runId>, code=<code>)` + đầu ra tùy chọn
- `Exec denied (node=<id>, id=<runId>, <reason>)`

### Giao thức truyền tải

Tùy chọn A (được khuyến nghị):

- Runner gửi các khung Bridge `event` `exec.started` / `exec.finished`.
- Gateway `handleBridgeEvent` ánh xạ những khung này thành `enqueueSystemEvent`.

Tùy chọn B:

- Gateway `exec` công cụ xử lý vòng đời trực tiếp (chỉ đồng bộ).
## Luồng thực thi

### Sandbox host

- Hành vi `exec` hiện có (Docker hoặc host khi không sandboxed).
- PTY được hỗ trợ chỉ ở chế độ non-sandbox.

### Gateway host

- Quá trình Gateway thực thi trên máy của nó.
- Thực thi `exec-approvals.json` cục bộ (bảo mật/yêu cầu/danh sách cho phép).

### Node host

- Gateway gọi `node.invoke` với `system.run`.
- Runner thực thi phê duyệt cục bộ.
- Runner trả về stdout/stderr tổng hợp.
- Các sự kiện Bridge tùy chọn cho bắt đầu/kết thúc/từ chối.
## Giới hạn đầu ra

- Giới hạn stdout+stderr kết hợp ở **200k**; giữ lại **20k cuối cùng** cho các sự kiện.
- Cắt ngắn với một hậu tố rõ ràng (ví dụ: `"… (truncated)"`).
## Lệnh gạch chéo

- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- Ghi đè theo agent, theo phiên; không lưu trữ lâu dài trừ khi được lưu qua cấu hình.
- `/elevated on|off|ask|full` vẫn là phím tắt cho `host=gateway security=full` (với `full` bỏ qua phê duyệt).
## Câu chuyện đa nền tảng

- Dịch vụ runner là mục tiêu thực thi có thể di động.
- UI là tùy chọn; nếu không có, `askFallback` áp dụng.
- Windows/Linux hỗ trợ cùng một giao thức JSON phê duyệt + socket.
## Các giai đoạn triển khai

### Giai đoạn 1: định tuyến cấu hình + thực thi

- Thêm schema cấu hình cho `exec.host`, `exec.security`, `exec.ask`, `exec.node`.
- Cập nhật tool plumbing để tôn trọng `exec.host`.
- Thêm lệnh slash `/exec` và giữ lại bí danh `/elevated`.

### Giai đoạn 2: kho phê duyệt + thực thi Gateway

- Triển khai reader/writer `exec-approvals.json`.
- Thực thi các chế độ allowlist + ask cho host `gateway`.
- Thêm output caps.

### Giai đoạn 3: thực thi node runner

- Cập nhật node runner để thực thi allowlist + ask.
- Thêm Unix socket prompt bridge tới UI ứng dụng macOS.
- Kết nối `askFallback`.

### Giai đoạn 4: sự kiện

- Thêm sự kiện Bridge từ node → gateway cho vòng đời thực thi.
- Ánh xạ tới `enqueueSystemEvent` cho các lời nhắc agent.

### Giai đoạn 5: hoàn thiện UI

- Ứng dụng Mac: trình chỉnh sửa allowlist, bộ chuyển đổi theo agent, UI chính sách ask.
- Điều khiển node binding (tùy chọn).
## Kế hoạch kiểm thử

- Unit tests: khớp danh sách cho phép (glob + không phân biệt chữ hoa/thường).
- Unit tests: ưu tiên phân giải chính sách (tham số công cụ → ghi đè agent → toàn cục).
- Integration tests: luồng từ chối/cho phép/hỏi của node runner.
- Bridge event tests: định tuyến sự kiện node → sự kiện hệ thống.
## Những rủi ro mở

- UI không khả dụng: đảm bảo `askFallback` được tuân thủ.
- Lệnh chạy lâu: dựa vào timeout + giới hạn đầu ra.
- Tính mơ hồ đa node: lỗi trừ khi có ràng buộc node hoặc tham số node rõ ràng.
## Tài liệu liên quan

- [Công cụ Exec](/tools/exec)
- [Phê duyệt Exec](/tools/exec-approvals)
- [Nodes](/nodes)
- [Chế độ nâng cao](/tools/elevated)