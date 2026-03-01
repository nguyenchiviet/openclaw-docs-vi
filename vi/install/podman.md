---
summary: Chạy OpenClaw trong một container Podman không có quyền root
read_when:
  - Bạn muốn một gateway được đóng gói trong container với Podman thay vì Docker
title: Podman
x-i18n:
  source_path: install\podman.md
  source_hash: 3e55f060943038e6824286f2e0c27e564d05605a8848973f9d5ab8f7bbdd3f38
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:09:46.508Z'
---

# Podman

Chạy Gateway OpenClaw trong một container Podman **không có quyền root**. Sử dụng cùng một image như Docker (xây dựng từ [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) của repo).
## Yêu cầu

- Podman (rootless)
- Sudo cho thiết lập một lần (tạo người dùng, xây dựng hình ảnh)
## Bắt đầu nhanh

**1. Thiết lập một lần** (từ thư mục gốc repo; tạo người dùng, xây dựng image, cài đặt script khởi chạy):

```bash
./setup-podman.sh
```

This also creates a minimal `~openclaw/.openclaw/openclaw.json` (sets `gateway.mode="local"`) so the gateway can start without running the wizard.

By default the container is **not** installed as a systemd service, you start it manually (see below). For a production-style setup with auto-start and restarts, install it as a systemd Quadlet user service instead:

```bash
./setup-podman.sh --quadlet
```

(Or set `OPENCLAW_PODMAN_QUADLET=1`; use `--container` to install only the container and launch script.)

**2. Start gateway** (manual, for quick smoke testing):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding wizard** (e.g. to add channels or providers):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Then open `http://127.0.0.1:18789/` and use the token from `~openclaw/.openclaw/.env` (hoặc giá trị được in ra bởi setup).
## Systemd (Quadlet, tùy chọn)

Nếu bạn chạy `./setup-podman.sh --quadlet` (hoặc `OPENCLAW_PODMAN_QUADLET=1`), một đơn vị [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) được cài đặt để gateway chạy như một dịch vụ người dùng systemd cho người dùng openclaw. Dịch vụ được bật và khởi động ở cuối quá trình thiết lập.

- **Khởi động:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Dừng:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Trạng thái:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Nhật ký:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Tệp quadlet nằm tại `~openclaw/.config/containers/systemd/openclaw.container`. Để thay đổi cổng hoặc biến môi trường, chỉnh sửa tệp đó (hoặc `.env` mà nó tham chiếu), sau đó `sudo systemctl --machine openclaw@ --user daemon-reload` và khởi động lại dịch vụ. Khi khởi động, dịch vụ sẽ tự động khởi động nếu lingering được bật cho openclaw (thiết lập sẽ làm điều này khi loginctl khả dụng).

Để thêm quadlet **sau** một thiết lập ban đầu không sử dụng nó, chạy lại: `./setup-podman.sh --quadlet`.
## Người dùng openclaw (không đăng nhập)

`setup-podman.sh` tạo một người dùng hệ thống chuyên dụng `openclaw`:

- **Shell:** `nologin` — không có đăng nhập tương tác; giảm bề mặt tấn công.
- **Home:** ví dụ `/home/openclaw` — chứa `~/.openclaw` (cấu hình, không gian làm việc) và tập lệnh khởi chạy `run-openclaw-podman.sh`.
- **Rootless Podman:** Người dùng phải có phạm vi **subuid** và **subgid**. Nhiều bản phân phối gán các phạm vi này tự động khi người dùng được tạo. Nếu thiết lập in cảnh báo, thêm các dòng vào `/etc/subuid` và `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Then start the gateway as that user (e.g. from cron or systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** Only `openclaw` and root can access `/home/openclaw/.openclaw`. To edit config: use the Control UI once the gateway is running, or `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.
## Môi trường và cấu hình

- **Token:** Được lưu trữ trong `~openclaw/.openclaw/.env` dưới dạng `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` và `run-openclaw-podman.sh` tạo nó nếu bị thiếu (sử dụng `openssl`, `python3`, hoặc `od`).
- **Tùy chọn:** Trong `.env` đó, bạn có thể đặt các khóa nhà cung cấp (ví dụ: `GROQ_API_KEY`, `OLLAMA_API_KEY`) và các biến môi trường OpenClaw khác.
- **Cổng máy chủ:** Theo mặc định, tập lệnh ánh xạ `18789` (gateway) và `18790` (bridge). Ghi đè ánh xạ cổng **máy chủ** với `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` và `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` khi khởi chạy.
- **Liên kết Gateway:** Theo mặc định, `run-openclaw-podman.sh` khởi động gateway với `--bind loopback` để truy cập cục bộ an toàn. Để hiển thị trên LAN, hãy đặt `OPENCLAW_GATEWAY_BIND=lan` và cấu hình `gateway.controlUi.allowedOrigins` (hoặc bật rõ ràng fallback host-header) trong `openclaw.json`.
- **Đường dẫn:** Cấu hình máy chủ và không gian làm việc mặc định là `~openclaw/.openclaw` và `~openclaw/.openclaw/workspace`. Ghi đè các đường dẫn máy chủ được sử dụng bởi tập lệnh khởi chạy với `OPENCLAW_CONFIG_DIR` và `OPENCLAW_WORKSPACE_DIR`.
## Các lệnh hữu ích

- **Logs:** Với quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Với script: `sudo -u openclaw podman logs -f openclaw`
- **Dừng:** Với quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Với script: `sudo -u openclaw podman stop openclaw`
- **Khởi động lại:** Với quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Với script: chạy lại script khởi chạy hoặc `podman start openclaw`
- **Xóa container:** `sudo -u openclaw podman rm -f openclaw` — cấu hình và workspace trên máy chủ được giữ lại
## Khắc phục sự cố

- **Permission denied (EACCES) trên config hoặc auth-profiles:** Container mặc định là `--userns=keep-id` và chạy với cùng uid/gid của người dùng host chạy script. Đảm bảo `OPENCLAW_CONFIG_DIR` và `OPENCLAW_WORKSPACE_DIR` của bạn được sở hữu bởi người dùng đó.
- **Gateway start bị chặn (thiếu `gateway.mode=local`):** Đảm bảo `~openclaw/.openclaw/openclaw.json` tồn tại và đặt `gateway.mode="local"`. `setup-podman.sh` tạo file này nếu thiếu.
- **Rootless Podman thất bại cho người dùng openclaw:** Kiểm tra `/etc/subuid` và `/etc/subgid` chứa một dòng cho `openclaw` (ví dụ `openclaw:100000:65536`). Thêm nó nếu thiếu và khởi động lại.
- **Tên container đang được sử dụng:** Script khởi chạy sử dụng `podman run --replace`, vì vậy container hiện có sẽ được thay thế khi bạn bắt đầu lại. Để dọn dẹp thủ công: `podman rm -f openclaw`.
- **Script không tìm thấy khi chạy dưới dạng openclaw:** Đảm bảo `setup-podman.sh` đã được chạy để `run-openclaw-podman.sh` được sao chép vào home của openclaw (ví dụ `/home/openclaw/run-openclaw-podman.sh`).
- **Dịch vụ Quadlet không tìm thấy hoặc không khởi động:** Chạy `sudo systemctl --machine openclaw@ --user daemon-reload` sau khi chỉnh sửa file `.container`. Quadlet yêu cầu cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` phải hiển thị `2`.
## Tùy chọn: chạy với người dùng của bạn

Để chạy gateway với người dùng bình thường của bạn (không có người dùng openclaw chuyên dụng): xây dựng image, tạo `~/.openclaw/.env` với `OPENCLAW_GATEWAY_TOKEN`, và chạy container với `--userns=keep-id` và mount đến `~/.openclaw` của bạn. Script khởi chạy được thiết kế cho quy trình openclaw-user; đối với thiết lập một người dùng, bạn có thể chạy lệnh `podman run` từ script theo cách thủ công, trỏ config và workspace đến home của bạn. Được khuyến nghị cho hầu hết người dùng: sử dụng `setup-podman.sh` và chạy với người dùng openclaw để config và process được cách ly.