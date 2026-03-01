---
summary: 'Sổ tay vận hành cho dịch vụ Gateway, vòng đời và hoạt động'
read_when:
  - Chạy hoặc gỡ lỗi quy trình Gateway
title: Sổ tay hướng dẫn Gateway
x-i18n:
  source_path: gateway\index.md
  source_hash: 38e67f594affc017c4f67811b048ba71434744b611fdb336b504bfc90fcd4a75
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:57:43.309Z'
---

# Gateway runbook

Sử dụng trang này để khởi động ngày thứ nhất và vận hành ngày thứ hai của dịch vụ Gateway.

<CardGroup cols={2}>
  <Card title="Khắc phục sự cố nâu cao" icon="siren" href="/gateway/troubleshooting">
    Chẩn đoán theo triệu chứng với các bước lệnh chính xác và chữ ký nhật ký.
  </Card>
  <Card title="Cấu hình" icon="sliders" href="/gateway/configuration">
    Hướng dẫn thiết lập theo nhiệm vụ + tham chiếu cấu hình đầy đủ.
  </Card>
  <Card title="Quản lý bí mật" icon="key-round" href="/gateway/secrets">
    Hợp đồng SecretRef, hành vi chụp nhanh thời gian chạy và các hoạt động di chuyển/tải lại.
  </Card>
  <Card title="Hợp đồng kế hoạch bí mật" icon="shield-check" href="/gateway/secrets-plan-contract">
    Các quy tắc `secrets apply` target/path chính xác và hành vi xác thực auth-profile chỉ ref.
  </Card>
</CardGroup>
## Khởi động cục bộ 5 phút

<Steps>
  <Step title="Khởi động Gateway">

```bash
openclaw gateway --port 18789
# debug/trace mirrored to stdio
openclaw gateway --port 18789 --verbose
# force-kill listener on selected port, then start
openclaw gateway --force
```

  </Step>

  <Step title="Verify service health">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Healthy baseline: `Runtime: running` and `RPC probe: ok`.

  </Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  </Step>
</Steps>

<Note>
Gateway config reload watches the active config file path (resolved from profile/state defaults, or `OPENCLAW_CONFIG_PATH` when set).
Default mode is `gateway.reload.mode="hybrid"`.
</Note>
## Mô hình Runtime

- Một quy trình luôn hoạt động cho định tuyến, mặt phẳng điều khiển và kết nối kênh.
- Cổng được ghép kênh duy nhất cho:
  - Điều khiển WebSocket/RPC
  - API HTTP (tương thích OpenAI, Phản hồi, gọi công cụ)
  - UI điều khiển và hooks
- Chế độ liên kết mặc định: `loopback`.
- Xác thực được yêu cầu theo mặc định (`gateway.auth.token` / `gateway.auth.password`, hoặc `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Thứ tự ưu tiên cổng và liên kết

| Cài đặt      | Thứ tự phân giải                                              |
| ------------ | ------------------------------------------------------------- |
| Cổng Gateway | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Chế độ liên kết    | CLI/ghi đè → `gateway.bind` → `loopback`                    |

### Chế độ tải lại nóng

| `gateway.reload.mode` | Hành vi                                   |
| --------------------- | ------------------------------------------ |
| `off`                 | Không tải lại cấu hình                           |
| `hot`                 | Chỉ áp dụng các thay đổi an toàn nóng                |
| `restart`             | Khởi động lại khi có thay đổi yêu cầu tải lại         |
| `hybrid` (mặc định)    | Áp dụng nóng khi an toàn, khởi động lại khi cần thiết |
## Tập lệnh Operator

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```
## Truy cập từ xa

Ưu tiên: Tailscale/VPN.
Dự phòng: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Then connect clients to `ws://127.0.0.1:18789` locally.

<Warning>
If gateway auth is configured, clients still must send auth (`token`/`password`) ngay cả qua SSH tunnels.
</Warning>

Xem: [Remote Gateway](/gateway/remote), [Xác thực](/gateway/authentication), [Tailscale](/gateway/tailscale).
## Giám sát và vòng đời dịch vụ

Sử dụng các lần chạy được giám sát để có độ tin cậy giống như môi trường sản xuất.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent labels are `ai.openclaw.gateway` (default) or `ai.openclaw.<profile>` (named profile). `openclaw doctor` audits and repairs service config drift.

  </Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

For persistence after logout, enable lingering:

```bash
sudo loginctl enable-linger <user>
```

  </Tab>

  <Tab title="Linux (system service)">

Use a system unit for multi-user/always-on hosts.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  </Tab>
</Tabs>
## Nhiều Gateway trên một máy chủ

Hầu hết các thiết lập nên chạy **một** Gateway.
Chỉ sử dụng nhiều Gateway cho mục đích cách ly/dự phòng nghiêm ngặt (ví dụ như một hồ sơ cứu hộ).

Danh sách kiểm tra cho mỗi instance:

- `gateway.port` duy nhất
- `OPENCLAW_CONFIG_PATH` duy nhất
- `OPENCLAW_STATE_DIR` duy nhất
- `agents.defaults.workspace` duy nhất

Ví dụ:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

See: [Multiple gateways](/gateway/multiple-gateways).

### Dev profile quick path

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Defaults include isolated state/config and base gateway port `19001`.
## Tham chiếu giao thức nhanh (chế độ xem của nhà điều hành)

- Khung client đầu tiên phải là `connect`.
- Gateway trả về ảnh chụp `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, giới hạn/chính sách).
- Yêu cầu: `req(method, params)` → `res(ok/payload|error)`.
- Sự kiện phổ biến: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Các lần chạy agent có hai giai đoạn:

1. Xác nhận chấp nhận ngay lập tức (`status:"accepted"`)
2. Phản hồi hoàn thành cuối cùng (`status:"ok"|"error"`), với các sự kiện `agent` được truyền phát giữa các giai đoạn.

Xem tài liệu giao thức đầy đủ: [Gateway Protocol](/gateway/protocol).
## Kiểm tra hoạt động

### Liveness

- Mở WS và gửi `connect`.
- Mong đợi `hello-ok` phản hồi với snapshot.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Gap recovery

Events are not replayed. On sequence gaps, refresh state (`health`, `system-presence`) trước khi tiếp tục.
## Các dấu hiệu lỗi phổ biến

| Dấu hiệu                                                      | Vấn đề có khả năng xảy ra                             |
| -------------------------------------------------------------- | ---------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Liên kết không phải local loopback mà không có token/mật khẩu |
| `another gateway instance is already listening` / `EADDRINUSE` | Xung đột cổng                            |
| `Gateway start blocked: set gateway.mode=local`                | Cấu hình được đặt thành chế độ từ xa                |
| `unauthorized` trong quá trình kết nối                                  | Xác thực không khớp giữa client và Gateway |

Để có các bước chẩn đoán đầy đủ, hãy sử dụng [Khắc phục sự cố Gateway](/gateway/troubleshooting).
## Đảm bảo an toàn

- Các client của Gateway protocol sẽ thất bại nhanh chóng khi Gateway không khả dụng (không có fallback kênh trực tiếp ngầm định).
- Các frame không hợp lệ/không kết nối đầu tiên bị từ chối và đóng.
- Tắt graceful phát ra sự kiện `shutdown` trước khi đóng socket.

---

Liên quan:

- [Khắc phục sự cố](/gateway/troubleshooting)
- [Quy trình nền](/gateway/background-process)
- [Cấu hình](/gateway/configuration)
- [Sức khỏe](/gateway/health)
- [Doctor](/gateway/doctor)
- [Xác thực](/gateway/authentication)