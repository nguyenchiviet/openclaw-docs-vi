---
summary: Node + tsx "__name is not a function" lỗi crash và các cách khắc phục
read_when:
  - Gỡ lỗi các script dev chỉ dành cho Node hoặc lỗi chế độ watch
  - Điều tra các sự cố crash của tsx/esbuild loader trong OpenClaw
title: Node + tsx Crash
x-i18n:
  source_path: debug\node-issue.md
  source_hash: f5beab7cdfe7679680f65176234a617293ce495886cfffb151518adfa61dc8dc
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:36.636Z'
---

# Node + tsx "__name is not a function" crash

## Tóm tắt

Chạy OpenClaw qua Node với `tsx` gặp lỗi khi khởi động:

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

This began after switching dev scripts from Bun to `tsx` (commit `2871657e`, 2026-01-06). Cùng một đường dẫn runtime này đã hoạt động với Bun.
## Môi trường

- Node: v25.x (quan sát trên v25.3.0)
- tsx: 4.21.0
- OS: macOS (có khả năng xảy ra trên các nền tảng khác chạy Node 25)
## Tái tạo (Chỉ Node)

```bash
# in repo root
node --version
pnpm install
node --import tsx src/entry.ts status
```
## Tái tạo tối thiểu trong kho lưu trữ

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```
## Kiểm tra phiên bản Node

- Node 25.3.0: không thành công
- Node 22.22.0 (Homebrew `node@22`): không thành công
- Node 24: chưa được cài đặt ở đây; cần xác minh
## Ghi chú / giả thuyết

- `tsx` sử dụng esbuild để chuyển đổi TS/ESM. `keepNames` của esbuild phát ra một helper `__name` và bao bọc các định nghĩa hàm bằng `__name(...)`.
- Sự cố chỉ ra rằng `__name` tồn tại nhưng không phải là một hàm tại thời điểm chạy, điều này ngụ ý rằng helper bị thiếu hoặc bị ghi đè cho module này trong đường dẫn loader Node 25.
- Các vấn đề helper `__name` tương tự đã được báo cáo trong các consumer esbuild khác khi helper bị thiếu hoặc được viết lại.
## Lịch sử hồi quy

- `2871657e` (2026-01-06): các script được thay đổi từ Bun sang tsx để làm cho Bun trở thành tùy chọn.
- Trước đó (đường dẫn Bun), `openclaw status` và `gateway:watch` đã hoạt động.
## Giải pháp thay thế

- Sử dụng Bun cho các tập lệnh dev (hoàn nguyên tạm thời hiện tại).
- Sử dụng Node + tsc watch, sau đó chạy đầu ra được biên dịch:

  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```

- Confirmed locally: `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` works on Node 25.
- Disable esbuild keepNames in the TS loader if possible (prevents `__name` helper insertion); tsx does not currently expose this.
- Test Node LTS (22/24) with `tsx` để xem liệu vấn đề có phải là dành riêng cho Node 25 hay không.
## Tài liệu tham khảo

- [https://opennext.js.org/cloudflare/howtos/keep_names](https://opennext.js.org/cloudflare/howtos/keep_names)
- [https://esbuild.github.io/api/#keep-names](https://esbuild.github.io/api/#keep-names)
- [https://github.com/evanw/esbuild/issues/1031](https://github.com/evanw/esbuild/issues/1031)
## Bước tiếp theo

- Tái hiện trên Node 22/24 để xác nhận hồi quy Node 25.
- Kiểm tra `tsx` hàng đêm hoặc ghim vào phiên bản trước nếu tồn tại hồi quy đã biết.
- Nếu tái hiện trên Node LTS, hãy gửi một bản tái hiện tối thiểu upstream với dấu vết ngăn xếp `__name`.