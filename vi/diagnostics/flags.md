---
summary: Cờ chẩn đoán để ghi nhật ký gỡ lỗi có mục tiêu
read_when:
  - >-
    Bạn cần các log gỡ lỗi được nhắm mục tiêu mà không cần nâng mức ghi nhật ký
    toàn cục
  - Bạn cần thu thập nhật ký dành riêng cho từng hệ thống con để hỗ trợ
title: Cờ Chẩn Đoán
x-i18n:
  source_path: diagnostics\flags.md
  source_hash: daf0eca0e6bd1cbc2c400b2e94e1698709a96b9cdba1a8cf00bd580a61829124
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:32.544Z'
---

# Cờ Chẩn đoán

Cờ chẩn đoán cho phép bạn bật nhật ký gỡ lỗi được nhắm mục tiêu mà không cần bật ghi nhật ký chi tiết ở mọi nơi. Các cờ là tùy chọn và không có tác dụng trừ khi một hệ thống con kiểm tra chúng.
## Cách hoạt động

- Cờ là các chuỗi (không phân biệt chữ hoa/thường).
- Bạn có thể bật cờ trong cấu hình hoặc thông qua ghi đè biến môi trường.
- Ký tự đại diện được hỗ trợ:
  - `telegram.*` khớp với `telegram.http`
  - `*` bật tất cả các cờ
## Bật qua cấu hình

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Multiple flags:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Khởi động lại Gateway sau khi thay đổi các cờ.
## Ghi đè biến môi trường (một lần)

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Disable all flags:

```bash
OPENCLAW_DIAGNOSTICS=0
```
## Nơi lưu trữ nhật ký

Các cờ phát ra nhật ký vào tệp nhật ký chẩn đoán tiêu chuẩn. Theo mặc định:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

If you set `logging.file`, use that path instead. Logs are JSONL (one JSON object per line). Redaction still applies based on `logging.redactSensitive`.
## Trích xuất nhật ký

Chọn tệp nhật ký mới nhất:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filter for Telegram HTTP diagnostics:

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

Or tail while reproducing:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

For remote gateways, you can also use `openclaw logs --follow` (xem [/cli/logs](/cli/logs)).
## Ghi chú

- Nếu `logging.level` được đặt cao hơn `warn`, các nhật ký này có thể bị loại bỏ. `info` mặc định là được.
- Các cờ an toàn để bật; chúng chỉ ảnh hưởng đến khối lượng nhật ký cho hệ thống con cụ thể.
- Sử dụng [/logging](/logging) để thay đổi đích đến nhật ký, mức độ và biên tập.