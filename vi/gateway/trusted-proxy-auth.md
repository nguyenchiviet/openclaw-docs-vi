---
summary: >-
  Ủy quyền xác thực gateway cho một reverse proxy đáng tin cậy (Pomerium, Caddy,
  nginx + OAuth)
read_when:
  - Chạy OpenClaw phía sau một proxy nhận dạng danh tính
  - 'Thiết lập Pomerium, Caddy, hoặc nginx với OAuth phía trước OpenClaw'
  - Sửa lỗi WebSocket 1008 unauthorized với cấu hình reverse proxy
  - Quyết định nơi đặt HSTS và các header HTTP hardening khác
x-i18n:
  source_path: gateway\trusted-proxy-auth.md
  source_hash: 99cd59e634d2b7b5fe8244192e082101e2bf460678367da735eea13632c9d922
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:00:38.802Z'
---

# Xác thực Proxy Đáng tin cậy

> ⚠️ **Tính năng nhạy cảm về bảo mật.** Chế độ này ủy quyền xác thực hoàn toàn cho reverse proxy của bạn. Cấu hình sai có thể làm lộ Gateway của bạn cho truy cập trái phép. Hãy đọc kỹ trang này trước khi bật.
## Khi nào sử dụng

Sử dụng chế độ xác thực `trusted-proxy` khi:

- Bạn chạy OpenClaw phía sau một **proxy nhận dạng danh tính** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Proxy của bạn xử lý tất cả xác thực và chuyển danh tính người dùng qua các tiêu đề
- Bạn đang ở trong môi trường Kubernetes hoặc container nơi proxy là đường dẫn duy nhất đến Gateway
- Bạn gặp lỗi WebSocket `1008 unauthorized` vì trình duyệt không thể chuyển token trong tải trọng WS
## Khi KHÔNG Sử Dụng

- Nếu proxy của bạn không xác thực người dùng (chỉ là TLS terminator hoặc load balancer)
- Nếu có bất kỳ đường dẫn nào đến Gateway mà bỏ qua proxy (lỗ tường lửa, truy cập mạng nội bộ)
- Nếu bạn không chắc liệu proxy của bạn có đúng cách xóa/ghi đè các tiêu đề được chuyển tiếp hay không
- Nếu bạn chỉ cần truy cập cá nhân cho một người dùng (hãy cân nhắc sử dụng Tailscale Serve + local loopback để thiết lập đơn giản hơn)
## Cách Hoạt Động

1. Reverse proxy của bạn xác thực người dùng (OAuth, OIDC, SAML, v.v.)
2. Proxy thêm một header với danh tính người dùng đã xác thực (ví dụ: `x-forwarded-user: nick@example.com`)
3. OpenClaw kiểm tra rằng yêu cầu đến từ một **IP proxy đáng tin cậy** (được cấu hình trong `gateway.trustedProxies`)
4. OpenClaw trích xuất danh tính người dùng từ header được cấu hình
5. Nếu mọi thứ đều ổn, yêu cầu được phép
## Kiểm soát Hành vi Ghép nối UI

Khi `gateway.auth.mode = "trusted-proxy"` hoạt động và yêu cầu vượt qua
kiểm tra trusted-proxy, các phiên Control UI WebSocket có thể kết nối mà không cần danh tính ghép nối thiết bị.

Hàm ý:

- Ghép nối không còn là cổng chính để truy cập Control UI ở chế độ này.
- Chính sách xác thực reverse proxy của bạn và `allowUsers` trở thành kiểm soát truy cập hiệu quả.
- Giữ ingress gateway bị khóa chỉ cho các IP trusted proxy (`gateway.trustedProxies` + tường lửa).
## Cấu hình

```json5
{
  gateway: {
    // Use loopback for same-host proxy setups; use lan/custom for remote proxy hosts
    bind: "loopback",

    // CRITICAL: Only add your proxy's IP(s) here
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: "x-forwarded-user",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

If `gateway.bind` is `loopback`, include a loopback proxy address in
`gateway.trustedProxies` (`127.0.0.1`, `::1`, or an equivalent loopback CIDR).

### Configuration Reference

| Field                                       | Required | Description                                                                 |
| ------------------------------------------- | -------- | --------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Yes      | Array of proxy IP addresses to trust. Requests from other IPs are rejected. |
| `gateway.auth.mode`                         | Yes      | Must be `"trusted-proxy"`                                                   |
| `gateway.auth.trustedProxy.userHeader`      | Yes      | Header name containing the authenticated user identity                      |
| `gateway.auth.trustedProxy.requiredHeaders` | No       | Additional headers that must be present for the request to be trusted       |
| `gateway.auth.trustedProxy.allowUsers`      | Không       | Danh sách cho phép các danh tính người dùng. Trống có nghĩa là cho phép tất cả người dùng được xác thực.    |
## Chấm dứt TLS và HSTS

Sử dụng một điểm chấm dứt TLS và áp dụng HSTS ở đó.

### Mẫu được khuyến nghị: chấm dứt TLS tại proxy

Khi reverse proxy của bạn xử lý HTTPS cho `https://control.example.com`, đặt
`Strict-Transport-Security` tại proxy cho miền đó.

- Phù hợp tốt cho các triển khai hướng tới internet.
- Giữ chứng chỉ + chính sách cứng hóa HTTP ở một nơi.
- OpenClaw có thể ở trên local loopback HTTP phía sau proxy.

Giá trị tiêu đề ví dụ:

```text
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Gateway TLS termination

If OpenClaw itself serves HTTPS directly (no TLS-terminating proxy), set:

```json5
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: "max-age=31536000; includeSubDomains",
      },
    },
  },
}
```

`strictTransportSecurity` accepts a string header value, or `false` to disable explicitly.

### Rollout guidance

- Start with a short max age first (for example `max-age=300`) while validating traffic.
- Increase to long-lived values (for example `max-age=31536000`) only after confidence is high.
- Add `includeSubDomains` chỉ khi mọi tên miền phụ đều sẵn sàng cho HTTPS.
- Chỉ sử dụng preload nếu bạn có ý định đáp ứng các yêu cầu preload cho bộ miền đầy đủ của mình.
- Phát triển cục bộ chỉ dành cho loopback không được hưởng lợi từ HSTS.
## Ví dụ Cấu hình Proxy

### Pomerium

Pomerium truyền danh tính trong `x-pomerium-claim-email` (hoặc các tiêu đề yêu cầu khác) và JWT trong `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium's IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Pomerium config snippet:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy with OAuth

Caddy with the `caddy-security` plugin can authenticate users and pass identity headers.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (if on same host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile snippet:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```
### nginx + oauth2-proxy

oauth2-proxy xác thực người dùng và chuyển danh tính trong `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx config snippet:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik with Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik container IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```
## Danh sách kiểm tra bảo mật

Trước khi bật xác thực trusted-proxy, hãy xác minh:

- [ ] **Proxy là đường dẫn duy nhất**: Cổng Gateway được tường lửa khỏi mọi thứ ngoại trừ proxy của bạn
- [ ] **trustedProxies tối thiểu**: Chỉ các IP proxy thực tế của bạn, không phải toàn bộ subnet
- [ ] **Proxy loại bỏ tiêu đề**: Proxy của bạn ghi đè (không nối thêm) các tiêu đề `x-forwarded-*` từ máy khách
- [ ] **Kết thúc TLS**: Proxy của bạn xử lý TLS; người dùng kết nối qua HTTPS
- [ ] **allowUsers được đặt** (được khuyến nghị): Hạn chế đối với những người dùng đã biết thay vì cho phép bất kỳ ai được xác thực
## Kiểm tra Bảo mật

`openclaw security audit` sẽ đánh dấu xác thực trusted-proxy với mức độ nghiêm trọng **critical**. Điều này là cố ý — đó là một lời nhắc nhở rằng bạn đang ủy thác bảo mật cho thiết lập proxy của mình.

Kiểm tra bảo mật kiểm tra:

- Cấu hình `trustedProxies` bị thiếu
- Cấu hình `userHeader` bị thiếu
- `allowUsers` trống (cho phép bất kỳ người dùng nào được xác thực)
## Khắc phục sự cố

### "trusted_proxy_untrusted_source"

Yêu cầu không đến từ IP trong `gateway.trustedProxies`. Kiểm tra:

- IP proxy có chính xác không? (IP container Docker có thể thay đổi)
- Có load balancer nào ở phía trước proxy của bạn không?
- Sử dụng `docker inspect` hoặc `kubectl get pods -o wide` để tìm IP thực tế

### "trusted_proxy_user_missing"

Header người dùng trống hoặc bị thiếu. Kiểm tra:

- Proxy của bạn có được cấu hình để chuyển tiếp header nhận dạng không?
- Tên header có chính xác không? (không phân biệt chữ hoa/thường, nhưng chính tả quan trọng)
- Người dùng có thực sự được xác thực tại proxy không?

### "trusted*proxy_missing_header*\*"

Header bắt buộc không có. Kiểm tra:

- Cấu hình proxy của bạn cho những header cụ thể đó
- Liệu header có bị loại bỏ ở đâu đó trong chuỗi không

### "trusted_proxy_user_not_allowed"

Người dùng được xác thực nhưng không có trong `allowUsers`. Thêm họ hoặc xóa danh sách cho phép.

### WebSocket Vẫn Gặp Lỗi

Đảm bảo proxy của bạn:

- Hỗ trợ nâng cấp WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Chuyển tiếp header nhận dạng trên các yêu cầu nâng cấp WebSocket (không chỉ HTTP)
- Không có đường dẫn xác thực riêng cho kết nối WebSocket
## Di chuyển từ Token Auth

Nếu bạn đang chuyển từ token auth sang trusted-proxy:

1. Cấu hình proxy của bạn để xác thực người dùng và chuyển tiếp các header
2. Kiểm tra thiết lập proxy độc lập (curl với các header)
3. Cập nhật cấu hình OpenClaw với xác thực trusted-proxy
4. Khởi động lại Gateway
5. Kiểm tra kết nối WebSocket từ Control UI
6. Chạy `openclaw security audit` và xem xét các phát hiện
## Liên quan

- [Bảo mật](/gateway/security) — hướng dẫn bảo mật đầy đủ
- [Cấu hình](/gateway/configuration) — tham chiếu cấu hình
- [Truy cập từ xa](/gateway/remote) — các mẫu truy cập từ xa khác
- [Tailscale](/gateway/tailscale) — giải pháp thay thế đơn giản hơn cho truy cập chỉ tailnet