---
title: Phát hiện vòng lặp công cụ
description: >-
  Configure optional guardrails for preventing repetitive or stalled tool-call
  loops
summary: >-
  Làm thế nào để bật và điều chỉnh các guardrail nhằm phát hiện các vòng lặp gọi
  công cụ lặp đi lặp lại
read_when:
  - Một người dùng báo cáo các agents bị kẹt khi lặp lại các lệnh gọi công cụ.
  - Cần tinh chỉnh tính năng bảo vệ cuộc gọi lặp lại.
  - Bạn đang chỉnh sửa các chính sách của công cụ tác nhân và thời gian chạy.
x-i18n:
  source_path: tools\loop-detection.md
  source_hash: 80a3d0b6ec0b34daa75604f28b2f917e5cc71f99285a2fc259fd68a8dbccb5f3
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:17:50.171Z'
---

# Phát hiện vòng lặp công cụ

OpenClaw có thể ngăn các agent bị kẹt trong các mẫu gọi công cụ lặp đi lặp lại.
Cơ chế bảo vệ này **bị tắt theo mặc định**.

Chỉ bật nó khi cần thiết, vì nó có thể chặn các cuộc gọi lặp lại hợp lệ với các cài đặt nghiêm ngặt.
## Lý do tồn tại

- Phát hiện các chuỗi lặp lại không tạo ra tiến triển.
- Phát hiện các vòng lặp không có kết quả tần số cao (cùng công cụ, cùng đầu vào, lỗi lặp lại).
- Phát hiện các mẫu gọi lặp lại cụ thể cho các công cụ thăm dò đã biết.
## Khối cấu hình

Mặc định toàn cục:

```json5
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 20,
      detectorCooldownMs: 12000,
      repeatThreshold: 3,
      criticalThreshold: 6,
      detectors: {
        repeatedFailure: true,
        knownPollLoop: true,
        repeatingNoProgress: true,
      },
    },
  },
}
```

Per-agent override (optional):

```json5
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            repeatThreshold: 2,
            criticalThreshold: 5,
          },
        },
      },
    ],
  },
}
```
### Hành vi trường

- __OC_I19N_0000__: Công tắc chính. __OC_I19N_0001__ có nghĩa là không thực hiện phát hiện vòng lặp.
- __OC_I19N_0002__: số lượng lệnh gọi công cụ gần đây được giữ lại để phân tích.
- __OC_I19N_0003__: khoảng thời gian được sử dụng bởi bộ phát hiện không tiến triển.
- __OC_I19N_0004__: số lần lặp lại tối thiểu trước khi cảnh báo/chặn bắt đầu.
- __OC_I19N_0005__: ngưỡng mạnh hơn có thể kích hoạt xử lý nghiêm ngặt hơn.
- __OC_I19N_0006__: phát hiện các lần thử thất bại lặp lại trên cùng một đường dẫn gọi.
- __OC_I19N_0007__: phát hiện các vòng lặp kiểu thăm dò đã biết.
- __OC_I19N_0008__: phát hiện các lệnh gọi lặp lại tần số cao mà không thay đổi trạng thái.
## Thiết lập đề xuất

- Bắt đầu với `enabled: true`, giữ nguyên các giá trị mặc định.
- Nếu xảy ra dương tính giả:
  - tăng `repeatThreshold` và/hoặc `criticalThreshold`
  - chỉ tắt bộ phát hiện gây ra sự cố
  - giảm `historySize` để có ngữ cảnh lịch sử ít nghiêm ngặt hơn
## Nhật ký và hành vi dự kiến

Khi một vòng lặp được phát hiện, OpenClaw báo cáo một sự kiện vòng lặp và chặn hoặc làm giảm chu kỳ công cụ tiếp theo tùy thuộc vào mức độ nghiêm trọng.
Điều này bảo vệ người dùng khỏi việc tiêu tốn token không kiểm soát và bị khóa, đồng thời vẫn duy trì quyền truy cập công cụ bình thường.

- Ưu tiên cảnh báo và tạm thời chặn trước.
- Chỉ leo thang khi có bằng chứng lặp lại tích lũy.
## Ghi chú

- `tools.loopDetection` được hợp nhất với các ghi đè cấp agent.
- Cấu hình từng agent ghi đè hoặc mở rộng hoàn toàn các giá trị toàn cục.
- Nếu không có cấu hình nào tồn tại, các biện pháp bảo vệ sẽ vẫn tắt.
