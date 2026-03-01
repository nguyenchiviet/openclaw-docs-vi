---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình ứng dụng Google Chat'
read_when:
  - Đang làm việc trên các tính năng kênh Google Chat
title: Google Chat
x-i18n:
  source_path: channels\googlechat.md
  source_hash: 4d00a1c42a412c3b61cd4f7b1947743c545a8f308100cfe64fc5f0192ccfbf44
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:18:17.044Z'
---

# Google Chat (Chat API)

Trạng thái: sẵn sàng cho tin nhắn riêng + không gian thông qua webhook Google Chat API (chỉ HTTP).
## Thiết lập nhanh (người mới bắt đầu)

1. Tạo một dự án Google Cloud và bật **Google Chat API**.
   - Truy cập: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - Bật API nếu chưa được bật.
2. Tạo một **Service Account**:
   - Nhấn **Create Credentials** > **Service Account**.
   - Đặt tên tùy ý (ví dụ: `openclaw-chat`).
   - Để trống quyền hạn (nhấn **Continue**).
   - Để trống principals có quyền truy cập (nhấn **Done**).
3. Tạo và tải xuống **JSON Key**:
   - Trong danh sách service accounts, nhấp vào tài khoản vừa tạo.
   - Chuyển đến tab **Keys**.
   - Nhấp **Add Key** > **Create new key**.
   - Chọn **JSON** và nhấn **Create**.
4. Lưu trữ tệp JSON đã tải xuống trên máy chủ gateway của bạn (ví dụ: `~/.openclaw/googlechat-service-account.json`).
5. Tạo ứng dụng Google Chat trong [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
   - Điền vào **Application info**:
     - **App name**: (ví dụ: `OpenClaw`)
     - **Avatar URL**: (ví dụ: `https://openclaw.ai/logo.png`)
     - **Description**: (ví dụ: `Personal AI Assistant`)
   - Bật **Interactive features**.
   - Trong **Functionality**, chọn **Join spaces and group conversations**.
   - Trong **Connection settings**, chọn **HTTP endpoint URL**.
   - Trong **Triggers**, chọn **Use a common HTTP endpoint URL for all triggers** và đặt thành URL công khai của gateway của bạn theo sau bởi `/googlechat`.
     - _Mẹo: Chạy `openclaw status` để tìm URL công khai của gateway._
   - Trong **Visibility**, chọn **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**.
   - Nhập địa chỉ email của bạn (ví dụ: `user@example.com`) vào hộp văn bản.
   - Nhấp **Save** ở cuối trang.
6. **Bật trạng thái ứng dụng**:
   - Sau khi lưu, **làm mới trang**.
   - Tìm phần **App status** (thường ở gần đầu hoặc cuối trang sau khi lưu).
   - Thay đổi trạng thái thành **Live - available to users**.
   - Nhấp **Save** lần nữa.
7. Cấu hình OpenClaw với đường dẫn service account + webhook audience:
   - Env: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - Hoặc config: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Đặt loại + giá trị webhook audience (khớp với cấu hình ứng dụng Chat của bạn).
9. Khởi động gateway. Google Chat sẽ POST đến đường dẫn webhook của bạn.
## Thêm vào Google Chat

Khi Gateway đang chạy và email của bạn đã được thêm vào danh sách hiển thị:

1. Truy cập [Google Chat](https://chat.google.com/).
2. Nhấp vào biểu tượng **+** (dấu cộng) bên cạnh **Tin nhắn riêng**.
3. Trong thanh tìm kiếm (nơi bạn thường thêm người), nhập **Tên ứng dụng** mà bạn đã cấu hình trong Google Cloud Console.
   - **Lưu ý**: Bot sẽ _không_ xuất hiện trong danh sách duyệt "Marketplace" vì đây là ứng dụng riêng tư. Bạn phải tìm kiếm theo tên.
4. Chọn bot của bạn từ kết quả.
5. Nhấp **Thêm** hoặc **Chat** để bắt đầu cuộc trò chuyện 1:1.
6. Gửi "Hello" để kích hoạt trợ lý!
## URL Công khai (Chỉ Webhook)

Webhook Google Chat yêu cầu một endpoint HTTPS công khai. Vì lý do bảo mật, **chỉ expose đường dẫn `/googlechat`** ra internet. Giữ dashboard OpenClaw và các endpoint nhạy cảm khác trên mạng riêng của bạn.

### Tùy chọn A: Tailscale Funnel (Khuyến nghị)

Sử dụng Tailscale Serve cho dashboard riêng tư và Funnel cho đường dẫn webhook công khai. Điều này giữ `/` ở chế độ riêng tư trong khi chỉ expose `/googlechat`.

1. **Kiểm tra địa chỉ mà gateway của bạn đang bind:**

   ```bash
   ss -tlnp | grep 18789
   ```

   Note the IP address (e.g., `127.0.0.1`, `0.0.0.0`, or your Tailscale IP like `100.x.x.x`).

2. **Expose the dashboard to the tailnet only (port 8443):**

   ```bash
   # If bound to localhost (127.0.0.1 or 0.0.0.0):
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # If bound to Tailscale IP only (e.g., 100.106.161.80):
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Expose only the webhook path publicly:**

   ```bash
   # If bound to localhost (127.0.0.1 or 0.0.0.0):
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # If bound to Tailscale IP only (e.g., 100.106.161.80):
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Authorize the node for Funnel access:**
   If prompted, visit the authorization URL shown in the output to enable Funnel for this node in your tailnet policy.

5. **Verify the configuration:**

   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Your public webhook URL will be:
`https://<node-name>.<tailnet>.ts.net/googlechat`

Your private dashboard stays tailnet-only:
`https://<node-name>.<tailnet>.ts.net:8443/`

Use the public URL (without `:8443`) in the Google Chat app config.

> Note: This configuration persists across reboots. To remove it later, run `tailscale funnel reset` and `tailscale serve reset`.

### Tùy chọn B: Reverse Proxy (Caddy)

Nếu bạn sử dụng reverse proxy như Caddy, chỉ proxy đường dẫn cụ thể:
```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

With this config, any request to `your-domain.com/` will be ignored or returned as 404, while `your-domain.com/googlechat` is safely routed to OpenClaw.

### Option C: Cloudflare Tunnel

Configure your tunnel's ingress rules to only route the webhook path:

- **Path**: `/googlechat` -> `http://localhost:18789/googlechat`
- **Quy tắc mặc định**: HTTP 404 (Không tìm thấy)
## Cách thức hoạt động

1. Google Chat gửi webhook POST đến gateway. Mỗi yêu cầu bao gồm header `Authorization: Bearer <token>`.
2. OpenClaw xác minh token với `audienceType` + `audience` đã cấu hình:
   - `audienceType: "app-url"` → audience là URL webhook HTTPS của bạn.
   - `audienceType: "project-number"` → audience là số dự án Cloud.
3. Tin nhắn được định tuyến theo không gian:
   - Tin nhắn riêng sử dụng khóa phiên `agent:<agentId>:googlechat:dm:<spaceId>`.
   - Không gian sử dụng khóa phiên `agent:<agentId>:googlechat:group:<spaceId>`.
4. Truy cập tin nhắn riêng mặc định là ghép nối. Người gửi không xác định sẽ nhận mã ghép nối; phê duyệt bằng:
   - `openclaw pairing approve googlechat <code>`
5. Không gian nhóm mặc định yêu cầu @-mention. Sử dụng `botUser` nếu phát hiện mention cần tên người dùng của ứng dụng.
## Mục tiêu

Sử dụng các định danh này cho việc gửi và danh sách cho phép:

- Tin nhắn riêng: `users/<userId>` (khuyến nghị).
- Email thô `name@example.com` có thể thay đổi và chỉ được sử dụng để khớp danh sách cho phép trực tiếp khi `channels.googlechat.dangerouslyAllowNameMatching: true`.
- Không được khuyến khích: `users/<email>` được xử lý như một id người dùng, không phải danh sách cho phép email.
- Không gian: `spaces/<spaceId>`.
## Điểm nổi bật cấu hình

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // or serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; helps mention detection
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Notes:

- Service account credentials can also be passed inline with `serviceAccount` (JSON string).
- `serviceAccountRef` is also supported (env/file SecretRef), including per-account refs under `channels.googlechat.accounts.<id>.serviceAccountRef`.
- Default webhook path is `/googlechat` if `webhookPath` isn’t set.
- `dangerouslyAllowNameMatching` re-enables mutable email principal matching for allowlists (break-glass compatibility mode).
- Reactions are available via the `reactions` tool and `channels action` when `actions.reactions` is enabled.
- `typingIndicator` supports `none`, `message` (default), and `reaction` (reaction requires user OAuth).
- Attachments are downloaded through the Chat API and stored in the media pipeline (size capped by `mediaMaxMb`).

Chi tiết tham chiếu bí mật: [Quản lý bí mật](/gateway/secrets).
## Khắc phục sự cố

### 405 Method Not Allowed

Nếu Google Cloud Logs Explorer hiển thị lỗi như:

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

This means the webhook handler isn't registered. Common causes:

1. **Channel not configured**: The `channels.googlechat` section is missing from your config. Verify with:

   ```bash
   openclaw config get channels.googlechat
   ```

   If it returns "Config path not found", add the configuration (see [Config highlights](#config-highlights)).

2. **Plugin not enabled**: Check plugin status:

   ```bash
   openclaw plugins list | grep googlechat
   ```

   If it shows "disabled", add `plugins.entries.googlechat.enabled: true` to your config.

3. **Gateway not restarted**: After adding config, restart the gateway:

   ```bash
   openclaw gateway restart
   ```

Verify the channel is running:

```bash
openclaw channels status
# Should show: Google Chat default: enabled, configured, ...
```

### Other issues

- Check `openclaw channels status --probe` for auth errors or missing audience config.
- If no messages arrive, confirm the Chat app's webhook URL + event subscriptions.
- If mention gating blocks replies, set `botUser` to the app's user resource name and verify `requireMention`.
- Use `openclaw logs --follow` trong khi gửi tin nhắn thử nghiệm để xem liệu yêu cầu có đến được gateway hay không.

Tài liệu liên quan:

- [Cấu hình Gateway](/gateway/configuration)
- [Bảo mật](/gateway/security)
- [Phản ứng](/tools/reactions)