---
summary: Thanh menu trạng thái logic và những gì được hiển thị cho người dùng
read_when:
  - Điều chỉnh giao diện menu Mac hoặc logic trạng thái
title: Thanh Menu
x-i18n:
  source_path: platforms\mac\menu-bar.md
  source_hash: 8eb73c0e671a76aae4ebb653c65147610bf3e6d3c9c0943d150e292e7761d16d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:13:21.672Z'
---

# Lôgic Trạng thái Thanh Menu

## Những gì được hiển thị

- Chúng tôi hiển thị trạng thái công việc hiện tại của agent trong biểu tượng thanh menu và trong hàng trạng thái đầu tiên của menu.
- Trạng thái sức khỏe bị ẩn khi công việc đang hoạt động; nó sẽ quay lại khi tất cả các phiên ở trạng thái không hoạt động.
- Khối "Nodes" trong menu chỉ liệt kê **thiết bị** (các nút được ghép nối qua `node.list`), không phải các mục client/presence.
- Một phần "Usage" xuất hiện dưới Context khi các ảnh chụp nhanh sử dụng nhà cung cấp có sẵn.
## Mô hình trạng thái

- Phiên: các sự kiện đến với `runId` (mỗi lần chạy) cộng với `sessionKey` trong tải trọng. Phiên "chính" là khóa `main`; nếu không có, chúng tôi quay lại phiên được cập nhật gần đây nhất.
- Ưu tiên: phiên chính luôn thắng. Nếu phiên chính hoạt động, trạng thái của nó được hiển thị ngay lập tức. Nếu phiên chính không hoạt động, phiên không phải chính được kích hoạt gần đây nhất sẽ được hiển thị. Chúng tôi không chuyển đổi giữa hoạt động; chúng tôi chỉ chuyển đổi khi phiên hiện tại không hoạt động hoặc phiên chính trở nên hoạt động.
- Loại hoạt động:
  - `job`: thực thi lệnh cấp cao (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` với `toolName` và `meta/args`.
## IconState enum (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (ghi đè gỡ lỗi)

### ActivityKind → glyph

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- mặc định → 🛠️

### Ánh xạ hình ảnh

- `idle`: critter bình thường.
- `workingMain`: badge với glyph, tint đầy đủ, hoạt ảnh chân "đang hoạt động".
- `workingOther`: badge với glyph, tint muted, không scurry.
- `overridden`: sử dụng glyph/tint đã chọn bất kể hoạt động.
## Văn bản hàng trạng thái (menu)

- Khi công việc đang hoạt động: `<Session role> · <activity label>`
  - Ví dụ: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Khi không hoạt động: quay lại bản tóm tắt tình trạng.
## Nhập sự kiện

- Nguồn: sự kiện control‑channel `agent` (`ControlChannel.handleAgentEvent`).
- Các trường được phân tích:
  - `stream: "job"` với `data.state` để bắt đầu/dừng.
  - `stream: "tool"` với `data.phase`, `name`, `meta`/`args` tùy chọn.
- Nhãn:
  - `exec`: dòng đầu tiên của `args.command`.
  - `read`/`write`: đường dẫn rút gọn.
  - `edit`: đường dẫn cộng với loại thay đổi được suy ra từ `meta`/số lượng diff.
  - dự phòng: tên công cụ.
## Ghi đè Gỡ lỗi

- Cài đặt ▸ Gỡ lỗi ▸ Bộ chọn "Ghi đè biểu tượng":
  - `System (auto)` (mặc định)
  - `Working: main` (theo loại công cụ)
  - `Working: other` (theo loại công cụ)
  - `Idle`
- Được lưu trữ qua `@AppStorage("iconOverride")`; ánh xạ tới `IconState.overridden`.
## Danh sách kiểm tra thử nghiệm

- Kích hoạt công việc phiên chính: xác minh biểu tượng chuyển đổi ngay lập tức và hàng trạng thái hiển thị nhãn chính.
- Kích hoạt công việc phiên không phải chính trong khi chính ở trạng thái chờ: biểu tượng/trạng thái hiển thị không phải chính; giữ ổn định cho đến khi hoàn thành.
- Bắt đầu chính trong khi các phiên khác hoạt động: biểu tượng chuyển đổi sang chính ngay lập tức.
- Các đợt công cụ nhanh: đảm bảo huy hiệu không nhấp nháy (TTL grace trên kết quả công cụ).
- Hàng sức khỏe xuất hiện lại khi tất cả các phiên ở trạng thái chờ.