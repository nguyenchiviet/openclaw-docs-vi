---
summary: >-
  Kế hoạch sản xuất cho giám sát quy trình tương tác đáng tin cậy (PTY +
  non-PTY) với quyền sở hữu rõ ràng, vòng đời thống nhất và dọn dẹp xác định
read_when:
  - Đang làm việc trên quyền sở hữu và dọn dẹp vòng đời exec/process
  - Gỡ lỗi hành vi giám sát PTY và non-PTY
owner: openclaw
status: in-progress
last_updated: '2026-02-15'
title: Kế hoạch Giám sát PTY và Quy trình
x-i18n:
  source_path: experiments\plans\pty-process-supervision.md
  source_hash: cc45c8a9862d59261f3f3ef57a6a332f3b7691613bf338e1088c924c43dc103f
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:55:42.570Z'
---

# PTY và Kế hoạch Giám sát Quy trình

## 1. Vấn đề và mục tiêu

Chúng tôi cần một vòng đời đáng tin cậy cho việc thực thi lệnh chạy lâu dài trên:

- `exec` chạy nền trước
- `exec` chạy nền
- `process` các hành động tiếp theo (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
- Các quy trình con của CLI agent runner

Mục tiêu không chỉ là hỗ trợ PTY. Mục tiêu là quyền sở hữu có thể dự đoán, hủy, timeout và dọn dẹp mà không có bất kỳ heuristic khớp quy trình không an toàn nào.
## 2. Phạm vi và ranh giới

- Giữ triển khai nội bộ trong `src/process/supervisor`.
- Không tạo gói mới cho điều này.
- Giữ tương thích hành vi hiện tại khi thực tế.
- Không mở rộng phạm vi sang phát lại terminal hoặc tính bền vững phiên kiểu tmux.
## 3. Được triển khai trong nhánh này

### Đường cơ sở Supervisor đã có sẵn

- Mô-đun Supervisor nằm trong `src/process/supervisor/*`.
- Exec runtime và CLI runner đã được định tuyến qua supervisor spawn và wait.
- Registry finalization là idempotent.

### Lần này hoàn thành

1. Hợp đồng lệnh PTY rõ ràng

- `SpawnInput` hiện là một discriminated union trong `src/process/supervisor/types.ts`.
- PTY runs yêu cầu `ptyCommand` thay vì tái sử dụng generic `argv`.
- Supervisor không còn xây dựng lại chuỗi lệnh PTY từ argv joins trong `src/process/supervisor/supervisor.ts`.
- Exec runtime hiện chuyển `ptyCommand` trực tiếp trong `src/agents/bash-tools.exec-runtime.ts`.

2. Tách rời kiểu lớp xử lý

- Các kiểu Supervisor không còn import `SessionStdin` từ agents.
- Hợp đồng stdin cục bộ xử lý nằm trong `src/process/supervisor/types.ts` (`ManagedRunStdin`).
- Các adapter hiện chỉ phụ thuộc vào các kiểu cấp xử lý:
  - `src/process/supervisor/adapters/child.ts`
  - `src/process/supervisor/adapters/pty.ts`

3. Cải thiện quyền sở hữu vòng đời công cụ xử lý

- `src/agents/bash-tools.process.ts` hiện yêu cầu hủy thông qua supervisor trước tiên.
- `process kill/remove` hiện sử dụng fallback termination của process-tree khi supervisor lookup bị bỏ lỡ.
- `remove` giữ hành vi remove xác định bằng cách loại bỏ các mục phiên đang chạy ngay sau khi yêu cầu termination.

4. Mặc định watchdog nguồn duy nhất

- Đã thêm các mặc định được chia sẻ trong `src/agents/cli-watchdog-defaults.ts`.
- `src/agents/cli-backends.ts` sử dụng các mặc định được chia sẻ.
- `src/agents/cli-runner/reliability.ts` sử dụng các mặc định được chia sẻ tương tự.

5. Dọn dẹp helper chết

- Đã xóa đường dẫn helper `killSession` không được sử dụng từ `src/agents/bash-tools.shared.ts`.

6. Các bài kiểm tra đường dẫn supervisor trực tiếp được thêm

- Đã thêm `src/agents/bash-tools.process.supervisor.test.ts` để bao gồm kill và remove routing thông qua supervisor cancellation.

7. Các bản sửa lỗi khoảng cách độ tin cậy hoàn thành

- `src/agents/bash-tools.process.ts` hiện fallback đến termination xử lý OS-level thực khi supervisor lookup bị bỏ lỡ.
- `src/process/supervisor/adapters/child.ts` hiện sử dụng ngữ nghĩa termination của process-tree cho các đường dẫn kill/timeout cancel mặc định.
- Đã thêm tiện ích process-tree được chia sẻ trong `src/process/kill-tree.ts`.

8. Bảo phủ trường hợp biên hợp đồng PTY được thêm

- Đã thêm `src/process/supervisor/supervisor.pty-command.test.ts` cho chuyển tiếp lệnh PTY nguyên văn và từ chối lệnh trống.
- Đã thêm `src/process/supervisor/adapters/child.test.ts` cho hành vi kill của process-tree trong child adapter cancellation.
## 4. Các khoảng trống và quyết định còn lại

### Trạng thái độ tin cậy

Hai khoảng trống độ tin cậy bắt buộc cho lần này hiện đã được đóng lại:

- `process kill/remove` hiện có fallback kết thúc OS thực khi supervisor lookup bị bỏ lỡ.
- child cancel/timeout hiện sử dụng ngữ nghĩa kill process-tree cho đường dẫn kill mặc định.
- Các bài kiểm tra hồi quy đã được thêm cho cả hai hành vi.

### Độ bền và đối sánh khởi động lại

Hành vi khởi động lại hiện được định nghĩa rõ ràng chỉ là vòng đời trong bộ nhớ.

- `reconcileOrphans()` vẫn là no-op trong `src/process/supervisor/supervisor.ts` theo thiết kế.
- Các lần chạy hoạt động không được khôi phục sau khi khởi động lại quy trình.
- Ranh giới này là cố ý cho lần thực hiện này để tránh rủi ro lưu trữ một phần.

### Các công việc tiếp theo về khả năng bảo trì

1. `runExecProcess` trong `src/agents/bash-tools.exec-runtime.ts` vẫn xử lý nhiều trách nhiệm và có thể được chia thành các trợ giúp tập trung trong một lần tiếp theo.
## 5. Kế hoạch triển khai

Lần triển khai cho các mục yêu cầu về độ tin cậy và hợp đồng đã hoàn thành.

Đã hoàn thành:

- `process kill/remove` fallback real termination
- process-tree cancellation for child adapter default kill path
- regression tests for fallback kill and child adapter kill path
- PTY command edge-case tests under explicit `ptyCommand`
- explicit in-memory restart boundary with `reconcileOrphans()` no-op by design

Theo dõi tùy chọn:

- split `runExecProcess` into focused helpers with no behavior drift
## 6. Bản đồ tệp

### Giám sát quy trình

- `src/process/supervisor/types.ts` được cập nhật với đầu vào spawn phân biệt và hợp đồng stdin cục bộ quy trình.
- `src/process/supervisor/supervisor.ts` được cập nhật để sử dụng `ptyCommand` rõ ràng.
- `src/process/supervisor/adapters/child.ts` và `src/process/supervisor/adapters/pty.ts` được tách rời khỏi các loại agent.
- `src/process/supervisor/registry.ts` hoàn tất idempotent không thay đổi và được giữ lại.

### Tích hợp Exec và quy trình

- `src/agents/bash-tools.exec-runtime.ts` được cập nhật để chuyển lệnh PTY rõ ràng và giữ đường dẫn dự phòng.
- `src/agents/bash-tools.process.ts` được cập nhật để hủy thông qua giám sát với kết thúc cây quy trình thực tế dự phòng.
- `src/agents/bash-tools.shared.ts` đã xóa đường dẫn trợ giúp kill trực tiếp.

### Độ tin cậy CLI

- `src/agents/cli-watchdog-defaults.ts` được thêm vào làm đường cơ sở được chia sẻ.
- `src/agents/cli-backends.ts` và `src/agents/cli-runner/reliability.ts` hiện tiêu thụ các giá trị mặc định giống nhau.
## 7. Xác thực chạy trong lần này

Bài kiểm tra đơn vị:

- `pnpm vitest src/process/supervisor/registry.test.ts`
- `pnpm vitest src/process/supervisor/supervisor.test.ts`
- `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
- `pnpm vitest src/process/supervisor/adapters/child.test.ts`
- `pnpm vitest src/agents/cli-backends.test.ts`
- `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
- `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
- `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
- `pnpm vitest src/process/exec.test.ts`

Mục tiêu E2E:

- `pnpm vitest src/agents/cli-runner.test.ts`
- `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

Ghi chú kiểm tra kiểu:

- Sử dụng `pnpm build` (và `pnpm check` cho cổng lint/docs đầy đủ) trong kho này. Các ghi chú cũ hơn đề cập đến `pnpm tsgo` đã lỗi thời.
## 8. Các bảo đảm hoạt động được bảo tồn

- Hành vi cứng hóa môi trường thực thi không thay đổi.
- Luồng phê duyệt và danh sách cho phép không thay đổi.
- Vệ sinh đầu ra và giới hạn đầu ra không thay đổi.
- Bộ điều hợp PTY vẫn đảm bảo giải quyết chờ đợi khi buộc tắt và xử lý loại bỏ trình nghe.
## 9. Định nghĩa hoàn thành

1. Supervisor là chủ sở hữu vòng đời cho các lần chạy được quản lý.
2. PTY spawn sử dụng hợp đồng lệnh rõ ràng mà không cần xây dựng lại argv.
3. Lớp quy trình không có phụ thuộc kiểu vào lớp agent cho các hợp đồng stdin của supervisor.
4. Các giá trị mặc định của Watchdog là nguồn duy nhất.
5. Các bài kiểm tra đơn vị và e2e được nhắm mục tiêu vẫn xanh.
6. Ranh giới độ bền khởi động lại được ghi chép rõ ràng hoặc được triển khai đầy đủ.
## 10. Tóm tắt

Nhánh hiện có một cấu trúc giám sát nhất quán và an toàn hơn:

- hợp đồng PTY rõ ràng
- phân lớp quy trình sạch hơn
- đường dẫn hủy do supervisor điều khiển cho các hoạt động quy trình
- chấm dứt dự phòng thực sự khi tra cứu supervisor bị bỏ lỡ
- hủy cây quy trình cho các đường dẫn kill mặc định chạy con
- các giá trị mặc định watchdog thống nhất
- ranh giới khởi động lại trong bộ nhớ rõ ràng (không có đối sánh yêu tinh trên khởi động lại trong lần này)