---
summary: Sử dụng Claude Max/Pro subscription làm điểm cuối API tương thích với OpenAI
read_when:
  - Bạn muốn sử dụng gói đăng ký Claude Max với các công cụ tương thích OpenAI
  - Bạn muốn một máy chủ API cục bộ bao bọc Claude Code CLI
  - Bạn muốn tiết kiệm tiền bằng cách sử dụng subscription thay vì API keys
title: Claude Max API Proxy
x-i18n:
  source_path: providers\claude-max-api-proxy.md
  source_hash: 43d0ab1461dd6f1da7974b54bd9c8fe033ad3abbad892953baad4a93c8b16b5b
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:15:25.054Z'
---

# Claude Max API Proxy

**claude-max-api-proxy** là một công cụ cộng đồng cho phép bạn công khai đăng ký Claude Max/Pro của mình dưới dạng điểm cuối API tương thích với OpenAI. Điều này cho phép bạn sử dụng đăng ký của mình với bất kỳ công cụ nào hỗ trợ định dạng API OpenAI.
## Tại Sao Sử Dụng Điều Này?

| Phương pháp             | Chi phí                                                | Phù hợp nhất với                           |
| ----------------------- | --------------------------------------------------- | ------------------------------------------ |
| Anthropic API           | Trả theo token (~$15/M đầu vào, $75/M đầu ra cho Opus) | Ứng dụng sản xuất, khối lượng cao               |
| Claude Max subscription | $200/tháng cố định                                     | Sử dụng cá nhân, phát triển, sử dụng không giới hạn |

Nếu bạn có đăng ký Claude Max và muốn sử dụng nó với các công cụ tương thích OpenAI, proxy này có thể giúp bạn tiết kiệm chi phí đáng kể.
## Cách hoạt động

```
Your App → claude-max-api-proxy → Claude Code CLI → Anthropic (via subscription)
     (OpenAI format)              (converts format)      (uses your login)
```

The proxy:

1. Accepts OpenAI-format requests at `http://localhost:3456/v1/chat/completions`
2. Chuyển đổi chúng thành các lệnh Claude Code CLI
3. Trả về các phản hồi ở định dạng OpenAI (hỗ trợ truyền phát)
## Cài đặt

```bash
# Requires Node.js 20+ and Claude Code CLI
npm install -g claude-max-api-proxy

# Verify Claude CLI is authenticated
claude --version
```
## Cách sử dụng

### Khởi động máy chủ

```bash
claude-max-api
# Server runs at http://localhost:3456
```

### Test it

```bash
# Health check
curl http://localhost:3456/health

# List models
curl http://localhost:3456/v1/models

# Chat completion
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### With OpenClaw

You can point OpenClaw at the proxy as a custom OpenAI-compatible endpoint:

```json5
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```
## Các Mô Hình Có Sẵn

| ID Mô Hình        | Ánh xạ tới      |
| ----------------- | --------------- |
| `claude-opus-4`   | Claude Opus 4   |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4`  | Claude Haiku 4  |
## Tự động khởi động trên macOS

Tạo một LaunchAgent để chạy proxy tự động:

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```
## Liên kết

- **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
- **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
- **Issues:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)
## Ghi chú

- Đây là một **công cụ cộng đồng**, không được hỗ trợ chính thức bởi Anthropic hoặc OpenClaw
- Yêu cầu đăng ký Claude Max/Pro hoạt động với Claude Code CLI được xác thực
- Proxy chạy cục bộ và không gửi dữ liệu đến bất kỳ máy chủ bên thứ ba nào
- Các phản hồi truyền phát được hỗ trợ đầy đủ
## Xem thêm

- [Nhà cung cấp Anthropic](/providers/anthropic) - Tích hợp OpenClaw gốc với Claude setup-token hoặc khóa API
- [Nhà cung cấp OpenAI](/providers/openai) - Dành cho các gói đăng ký OpenAI/Codex