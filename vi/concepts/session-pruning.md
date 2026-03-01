---
title: Cắt tỉa Phiên làm việc
summary: >-
  Cắt tỉa phiên làm việc: cắt ngắn kết quả công cụ để giảm tình trạng quá tải
  ngữ cảnh
read_when:
  - Bạn muốn giảm sự tăng trưởng ngữ cảnh LLM từ các đầu ra của công cụ
  - Bạn đang điều chỉnh agents.defaults.contextPruning
x-i18n:
  source_path: concepts\session-pruning.md
  source_hash: aedb9f60dddc17a92834e74b264ccba63e1561cf41bb5d56f4e55d83cd544b83
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:41:48.651Z'
---

# Cắt tỉa phiên

Cắt tỉa phiên loại bỏ **kết quả công cụ cũ** khỏi ngữ cảnh trong bộ nhớ ngay trước mỗi lệnh gọi LLM. Nó **không** viết lại lịch sử phiên trên đĩa (`*.jsonl`).
## Khi nó chạy

- Khi `mode: "cache-ttl"` được bật và lệnh gọi Anthropic cuối cùng cho phiên này cũ hơn `ttl`.
- Chỉ ảnh hưởng đến các tin nhắn được gửi đến mô hình cho yêu cầu đó.
- Chỉ hoạt động cho các lệnh gọi API Anthropic (và các mô hình Anthropic của OpenRouter).
- Để có kết quả tốt nhất, khớp `ttl` với chính sách `cacheRetention` của mô hình của bạn (`short` = 5m, `long` = 1h).
- Sau khi cắt tỉa, cửa sổ TTL được đặt lại để các yêu cầu tiếp theo giữ bộ nhớ cache cho đến khi `ttl` hết hạn.
## Giá trị mặc định thông minh (Anthropic)

- **OAuth hoặc setup-token** profiles: bật `cache-ttl` pruning và đặt heartbeat thành `1h`.
- **API key** profiles: bật `cache-ttl` pruning, đặt heartbeat thành `30m`, và mặc định `cacheRetention: "short"` trên các mô hình Anthropic.
- Nếu bạn đặt bất kỳ giá trị nào trong số này một cách rõ ràng, OpenClaw sẽ **không** ghi đè chúng.
## Những gì điều này cải thiện (chi phí + hành vi bộ nhớ đệm)

- **Tại sao cắt tỉa:** Prompt caching của Anthropic chỉ áp dụng trong TTL. Nếu một phiên không hoạt động quá TTL, yêu cầu tiếp theo sẽ tái lưu toàn bộ prompt trừ khi bạn cắt tỉa nó trước.
- **Những gì trở nên rẻ hơn:** cắt tỉa giảm kích thước **cacheWrite** cho yêu cầu đầu tiên sau khi TTL hết hạn.
- **Tại sao việc đặt lại TTL lại quan trọng:** sau khi cắt tỉa chạy, cửa sổ bộ nhớ đệm được đặt lại, vì vậy các yêu cầu tiếp theo có thể tái sử dụng prompt được lưu vào bộ nhớ đệm mới thay vì tái lưu toàn bộ lịch sử một lần nữa.
- **Những gì nó không làm:** cắt tỉa không thêm token hoặc "nhân đôi" chi phí; nó chỉ thay đổi những gì được lưu vào bộ nhớ đệm trên yêu cầu đầu tiên sau TTL.
## Những gì có thể được cắt tỏa

- Chỉ các tin nhắn `toolResult`.
- Tin nhắn của người dùng + trợ lý **không bao giờ** được sửa đổi.
- `keepLastAssistants` tin nhắn trợ lý cuối cùng được bảo vệ; kết quả công cụ sau điểm cắt đó không được cắt tỏa.
- Nếu không có đủ tin nhắn trợ lý để thiết lập điểm cắt, cắt tỏa sẽ bị bỏ qua.
- Kết quả công cụ chứa **khối hình ảnh** sẽ bị bỏ qua (không bao giờ được cắt ngắn/xóa).
## Ước tính cửa sổ ngữ cảnh

Pruning sử dụng một cửa sổ ngữ cảnh ước tính (ký tự ≈ token × 4). Cửa sổ cơ sở được giải quyết theo thứ tự này:

1. `models.providers.*.models[].contextWindow` ghi đè.
2. Định nghĩa mô hình `contextWindow` (từ sổ đăng ký mô hình).
3. `200000` token mặc định.

Nếu `agents.defaults.contextTokens` được đặt, nó được coi là một giới hạn (tối thiểu) trên cửa sổ được giải quyết.
## Chế độ

### cache-ttl

- Pruning chỉ chạy nếu lệnh gọi Anthropic cuối cùng cũ hơn `ttl` (mặc định `5m`).
- Khi nó chạy: hành vi soft-trim + hard-clear giống như trước.
## Cắt tỉa mềm so với cắt tỉa cứng

- **Cắt tỉa mềm**: chỉ dành cho kết quả công cụ quá lớn.
  - Giữ phần đầu + phần cuối, chèn `...`, và thêm ghi chú với kích thước ban đầu.
  - Bỏ qua kết quả có khối hình ảnh.
- **Xóa hoàn toàn**: thay thế toàn bộ kết quả công cụ bằng `hardClear.placeholder`.
## Lựa chọn công cụ

- `tools.allow` / `tools.deny` hỗ trợ `*` wildcards.
- Deny có ưu tiên.
- Khớp không phân biệt chữ hoa/thường.
- Danh sách cho phép trống => tất cả công cụ được phép.
## Tương tác với các giới hạn khác

- Các công cụ tích hợp sẵn đã cắt ngắn đầu ra của chính chúng; cắt tỉa phiên là một lớp bổ sung ngăn chặn các cuộc trò chuyện chạy lâu dài tích lũy quá nhiều đầu ra công cụ trong ngữ cảnh mô hình.
- Nén là riêng biệt: nén tóm tắt và lưu trữ, cắt tỉa là tạm thời cho mỗi yêu cầu. Xem [/concepts/compaction](/concepts/compaction).
## Mặc định (khi được bật)

- `ttl`: `"5m"`
- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3`
- `hardClearRatio`: `0.5`
- `minPrunableToolChars`: `50000`
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`
## Ví dụ

Mặc định (tắt):

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

Enable TTL-aware pruning:

```json5
{
  agents: { defaults: { contextPruning: { mode: "cache-ttl", ttl: "5m" } } },
}
```

Restrict pruning to specific tools:

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl",
        tools: { allow: ["exec", "read"], deny: ["*image*"] },
      },
    },
  },
}
```

Xem tham chiếu cấu hình: [Cấu hình Gateway](/gateway/configuration)