---
summary: 'Trạng thái hỗ trợ, khả năng và cấu hình bot Microsoft Teams'
read_when:
  - Đang làm việc trên các tính năng kênh MS Teams
title: Microsoft Teams
x-i18n:
  source_path: channels\msteams.md
  source_hash: 9559705b08291578f9d476ea4924d53aa77ae81d682a95d93e9b148430007e32
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:24:05.660Z'
---

# Microsoft Teams (plugin)

> "Hãy từ bỏ mọi hy vọng, những ai bước vào đây."

Cập nhật: 2026-01-21

Trạng thái: hỗ trợ văn bản + tệp đính kèm tin nhắn riêng; gửi tệp trong kênh/nhóm yêu cầu `sharePointSiteId` + quyền Graph (xem [Gửi tệp trong cuộc trò chuyện nhóm](#sending-files-in-group-chats)). Bình chọn được gửi qua Adaptive Cards.
## Plugin bắt buộc

Microsoft Teams được cung cấp dưới dạng plugin và không được tích hợp sẵn trong bản cài đặt cốt lõi.

**Thay đổi đột phá (2026.1.15):** MS Teams đã được tách ra khỏi cốt lõi. Nếu bạn sử dụng nó, bạn phải cài đặt plugin.

Giải thích: giúp các bản cài đặt cốt lõi nhẹ hơn và cho phép các phụ thuộc của MS Teams cập nhật độc lập.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/msteams
```

Local checkout (when running from a git repo):

```bash
openclaw plugins install ./extensions/msteams
```

Nếu bạn chọn Teams trong quá trình cấu hình/thiết lập ban đầu và phát hiện git checkout,
OpenClaw sẽ tự động đề xuất đường dẫn cài đặt cục bộ.

Chi tiết: [Plugins](/tools/plugin)
## Thiết lập nhanh (người mới bắt đầu)

1. Cài đặt plugin Microsoft Teams.
2. Tạo một **Azure Bot** (App ID + client secret + tenant ID).
3. Cấu hình OpenClaw với các thông tin xác thực đó.
4. Expose `/api/messages` (cổng 3978 theo mặc định) thông qua URL công khai hoặc tunnel.
5. Cài đặt gói ứng dụng Teams và khởi động gateway.

Cấu hình tối thiểu:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Note: group chats are blocked by default (`channels.msteams.groupPolicy: "allowlist"`). To allow group replies, set `channels.msteams.groupAllowFrom` (or use `groupPolicy: "open"` để cho phép bất kỳ thành viên nào, được kiểm soát bằng mention).
## Mục tiêu

- Trò chuyện với OpenClaw qua tin nhắn riêng Teams, trò chuyện nhóm, hoặc kênh.
- Giữ định tuyến xác định: phản hồi luôn quay lại kênh mà chúng đến.
- Mặc định hành vi kênh an toàn (yêu cầu đề cập trừ khi được cấu hình khác).
## Ghi cấu hình

Theo mặc định, Microsoft Teams được phép ghi các cập nhật cấu hình được kích hoạt bởi `/config set|unset` (yêu cầu `commands.config: true`).

Vô hiệu hóa bằng:

```json5
{
  channels: { msteams: { configWrites: false } },
}
```
## Kiểm soát truy cập (Tin nhắn riêng + nhóm)

**Truy cập tin nhắn riêng**

- Mặc định: `channels.msteams.dmPolicy = "pairing"`. Người gửi không xác định sẽ bị bỏ qua cho đến khi được phê duyệt.
- `channels.msteams.allowFrom` nên sử dụng ID đối tượng AAD ổn định.
- UPN/tên hiển thị có thể thay đổi; khớp trực tiếp bị vô hiệu hóa theo mặc định và chỉ được bật với `channels.msteams.dangerouslyAllowNameMatching: true`.
- Trình hướng dẫn có thể phân giải tên thành ID thông qua Microsoft Graph khi thông tin xác thực cho phép.

**Truy cập nhóm**

- Mặc định: `channels.msteams.groupPolicy = "allowlist"` (bị chặn trừ khi bạn thêm `groupAllowFrom`). Sử dụng `channels.defaults.groupPolicy` để ghi đè mặc định khi chưa được đặt.
- `channels.msteams.groupAllowFrom` kiểm soát người gửi nào có thể kích hoạt trong cuộc trò chuyện nhóm/kênh (quay lại `channels.msteams.allowFrom`).
- Đặt `groupPolicy: "open"` để cho phép bất kỳ thành viên nào (vẫn bị giới hạn bởi mention theo mặc định).
- Để không cho phép **kênh nào**, đặt `channels.msteams.groupPolicy: "disabled"`.

Ví dụ:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams + channel allowlist**

- Scope group/channel replies by listing teams and channels under `channels.msteams.teams`.
- Keys can be team IDs or names; channel keys can be conversation IDs or names.
- When `groupPolicy="allowlist"` and a teams allowlist is present, only listed teams/channels are accepted (mention‑gated).
- The configure wizard accepts `Team/Channel` entries and stores them for you.
- On startup, OpenClaw resolves team/channel and user allowlist names to IDs (when Graph permissions allow)
  and logs the mapping; unresolved entries are kept as typed.

Example:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```
## Cách thức hoạt động

1. Cài đặt plugin Microsoft Teams.
2. Tạo một **Azure Bot** (App ID + secret + tenant ID).
3. Xây dựng **gói ứng dụng Teams** tham chiếu đến bot và bao gồm các quyền RSC bên dưới.
4. Tải lên/cài đặt ứng dụng Teams vào một nhóm (hoặc phạm vi cá nhân cho tin nhắn riêng).
5. Cấu hình `msteams` trong `~/.openclaw/openclaw.json` (hoặc biến môi trường) và khởi động gateway.
6. Gateway lắng nghe lưu lượng webhook Bot Framework trên `/api/messages` theo mặc định.
## Thiết lập Azure Bot (Điều kiện tiên quyết)

Trước khi cấu hình OpenClaw, bạn cần tạo tài nguyên Azure Bot.

### Bước 1: Tạo Azure Bot

1. Truy cập [Tạo Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Điền vào tab **Cơ bản**:

   | Trường             | Giá trị                                                  |
   | ------------------ | -------------------------------------------------------- |
   | **Tên bot**        | Tên bot của bạn, ví dụ: `openclaw-msteams` (phải duy nhất) |
   | **Gói đăng ký**    | Chọn gói đăng ký Azure của bạn                          |
   | **Nhóm tài nguyên** | Tạo mới hoặc sử dụng nhóm hiện có                        |
   | **Gói giá**        | **Miễn phí** cho phát triển/thử nghiệm                   |
   | **Loại ứng dụng**  | **Đơn thuê bao** (khuyến nghị - xem ghi chú bên dưới)    |
   | **Loại tạo**       | **Tạo Microsoft App ID mới**                             |

> **Thông báo ngừng hỗ trợ:** Việc tạo bot đa thuê bao mới đã bị ngừng sau ngày 31/07/2025. Sử dụng **Đơn thuê bao** cho các bot mới.

3. Nhấp **Xem xét + tạo** → **Tạo** (chờ khoảng 1-2 phút)

### Bước 2: Lấy thông tin xác thực

1. Truy cập tài nguyên Azure Bot của bạn → **Cấu hình**
2. Sao chép **Microsoft App ID** → đây là `appId` của bạn
3. Nhấp **Quản lý mật khẩu** → chuyển đến App Registration
4. Trong **Chứng chỉ & bí mật** → **Bí mật client mới** → sao chép **Giá trị** → đây là `appPassword` của bạn
5. Truy cập **Tổng quan** → sao chép **ID thư mục (tenant)** → đây là `tenantId` của bạn

### Bước 3: Cấu hình điểm cuối nhắn tin

1. Trong Azure Bot → **Cấu hình**
2. Đặt **Điểm cuối nhắn tin** thành URL webhook của bạn:
   - Sản xuất: `https://your-domain.com/api/messages`
   - Phát triển cục bộ: Sử dụng tunnel (xem [Phát triển cục bộ](#local-development-tunneling) bên dưới)

### Bước 4: Kích hoạt kênh Teams

1. Trong Azure Bot → **Kênh**
2. Nhấp **Microsoft Teams** → Cấu hình → Lưu
3. Chấp nhận Điều khoản dịch vụ
## Phát triển cục bộ (Tunneling)

Các nhóm không thể truy cập `localhost`. Sử dụng tunnel cho phát triển cục bộ:

**Tùy chọn A: ngrok**

```bash
ngrok http 3978
# Copy the https URL, e.g., https://abc123.ngrok.io
# Set messaging endpoint to: https://abc123.ngrok.io/api/messages
```

**Option B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Use your Tailscale funnel URL as the messaging endpoint
```
## Teams Developer Portal (Thay thế)

Thay vì tạo tệp ZIP manifest thủ công, bạn có thể sử dụng [Teams Developer Portal](https://dev.teams.microsoft.com/apps):

1. Nhấp **+ New app**
2. Điền thông tin cơ bản (tên, mô tả, thông tin nhà phát triển)
3. Đi tới **App features** → **Bot**
4. Chọn **Enter a bot ID manually** và dán Azure Bot App ID của bạn
5. Chọn các phạm vi: **Personal**, **Team**, **Group Chat**
6. Nhấp **Distribute** → **Download app package**
7. Trong Teams: **Apps** → **Manage your apps** → **Upload a custom app** → chọn tệp ZIP

Cách này thường dễ dàng hơn so với việc chỉnh sửa manifest JSON thủ công.
## Kiểm tra Bot

**Tùy chọn A: Azure Web Chat (xác minh webhook trước)**

1. Trong Azure Portal → tài nguyên Azure Bot của bạn → **Test in Web Chat**
2. Gửi tin nhắn - bạn sẽ thấy phản hồi
3. Điều này xác nhận endpoint webhook của bạn hoạt động trước khi thiết lập Teams

**Tùy chọn B: Teams (sau khi cài đặt ứng dụng)**

1. Cài đặt ứng dụng Teams (sideload hoặc catalog tổ chức)
2. Tìm bot trong Teams và gửi tin nhắn riêng
3. Kiểm tra logs gateway để xem hoạt động đến
## Thiết lập (văn bản tối thiểu)

1. **Cài đặt plugin Microsoft Teams**
   - Từ npm: `openclaw plugins install @openclaw/msteams`
   - Từ bản checkout cục bộ: `openclaw plugins install ./extensions/msteams`

2. **Đăng ký bot**
   - Tạo Azure Bot (xem phía trên) và ghi chú:
     - App ID
     - Client secret (Mật khẩu ứng dụng)
     - Tenant ID (đơn tenant)

3. **Manifest ứng dụng Teams**
   - Bao gồm mục `bot` với `botId = <App ID>`.
   - Phạm vi: `personal`, `team`, `groupChat`.
   - `supportsFiles: true` (bắt buộc để xử lý tệp trong phạm vi cá nhân).
   - Thêm quyền RSC (bên dưới).
   - Tạo biểu tượng: `outline.png` (32x32) và `color.png` (192x192).
   - Nén cả ba tệp lại với nhau: `manifest.json`, `outline.png`, `color.png`.

4. **Cấu hình OpenClaw**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   You can also use environment variables instead of config keys:
   - `MSTEAMS_APP_ID`
   - `MSTEAMS_APP_PASSWORD`
   - `MSTEAMS_TENANT_ID`

5. **Bot endpoint**
   - Set the Azure Bot Messaging Endpoint to:
     - `https://<host>:3978/api/messages` (or your chosen path/port).

6. **Run the gateway**
   - The Teams channel starts automatically when the plugin is installed and `msteams` tồn tại với thông tin xác thực.
## Ngữ cảnh lịch sử

- `channels.msteams.historyLimit` kiểm soát số lượng tin nhắn gần đây trong kênh/nhóm được đưa vào prompt.
- Sẽ quay về `messages.groupChat.historyLimit`. Đặt `0` để vô hiệu hóa (mặc định 50).
- Lịch sử tin nhắn riêng có thể được giới hạn bằng `channels.msteams.dmHistoryLimit` (lượt người dùng). Ghi đè theo từng người dùng: `channels.msteams.dms["<user_id>"].historyLimit`.
## Quyền RSC Teams Hiện tại (Manifest)

Đây là các **quyền resourceSpecific hiện có** trong manifest ứng dụng Teams của chúng tôi. Chúng chỉ áp dụng bên trong team/chat nơi ứng dụng được cài đặt.

**Đối với kênh (phạm vi team):**

- `ChannelMessage.Read.Group` (Application) - nhận tất cả tin nhắn kênh mà không cần @mention
- `ChannelMessage.Send.Group` (Application)
- `Member.Read.Group` (Application)
- `Owner.Read.Group` (Application)
- `ChannelSettings.Read.Group` (Application)
- `TeamMember.Read.Group` (Application)
- `TeamSettings.Read.Group` (Application)

**Đối với chat nhóm:**

- `ChatMessage.Read.Chat` (Application) - nhận tất cả tin nhắn chat nhóm mà không cần @mention
## Ví dụ Teams Manifest (đã ẩn thông tin)

Ví dụ tối thiểu, hợp lệ với các trường bắt buộc. Thay thế các ID và URL.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### Manifest caveats (must-have fields)

- `bots[].botId` **must** match the Azure Bot App ID.
- `webApplicationInfo.id` **must** match the Azure Bot App ID.
- `bots[].scopes` must include the surfaces you plan to use (`personal`, `team`, `groupChat`).
- `bots[].supportsFiles: true` is required for file handling in personal scope.
- `authorization.permissions.resourceSpecific` phải bao gồm quyền đọc/gửi kênh nếu bạn muốn nhận lưu lượng kênh.

### Cập nhật ứng dụng hiện có
Để cập nhật ứng dụng Teams đã cài đặt (ví dụ: để thêm quyền RSC):

1. Cập nhật `manifest.json` của bạn với các cài đặt mới
2. **Tăng trường `version`** (ví dụ: `1.0.0` → `1.1.0`)
3. **Nén lại** manifest với các biểu tượng (`manifest.json`, `outline.png`, `color.png`)
4. Tải lên tệp zip mới:
   - **Tùy chọn A (Teams Admin Center):** Teams Admin Center → Teams apps → Manage apps → tìm ứng dụng của bạn → Upload new version
   - **Tùy chọn B (Sideload):** Trong Teams → Apps → Manage your apps → Upload a custom app
5. **Đối với kênh nhóm:** Cài đặt lại ứng dụng trong mỗi nhóm để quyền mới có hiệu lực
6. **Thoát hoàn toàn và khởi động lại Teams** (không chỉ đóng cửa sổ) để xóa metadata ứng dụng đã lưu trong bộ nhớ đệm
## Khả năng: Chỉ RSC so với Graph

### Với **Teams chỉ RSC** (ứng dụng đã cài đặt, không có quyền Graph API)

Hoạt động:

- Đọc nội dung **văn bản** tin nhắn kênh.
- Gửi nội dung **văn bản** tin nhắn kênh.
- Nhận tệp đính kèm **cá nhân (tin nhắn riêng)**.

KHÔNG hoạt động:

- **Hình ảnh hoặc nội dung tệp** kênh/nhóm (payload chỉ bao gồm HTML stub).
- Tải xuống tệp đính kèm được lưu trữ trong SharePoint/OneDrive.
- Đọc lịch sử tin nhắn (ngoài sự kiện webhook trực tiếp).

### Với **Teams RSC + quyền Microsoft Graph Application**

Thêm:

- Tải xuống nội dung được lưu trữ (hình ảnh được dán vào tin nhắn).
- Tải xuống tệp đính kèm được lưu trữ trong SharePoint/OneDrive.
- Đọc lịch sử tin nhắn kênh/chat qua Graph.

### RSC so với Graph API

| Khả năng                | Quyền RSC            | Graph API                                   |
| ----------------------- | -------------------- | ------------------------------------------- |
| **Tin nhắn thời gian thực** | Có (qua webhook)     | Không (chỉ polling)                         |
| **Tin nhắn lịch sử**    | Không                | Có (có thể truy vấn lịch sử)                |
| **Độ phức tạp thiết lập** | Chỉ app manifest     | Yêu cầu đồng ý admin + luồng token         |
| **Hoạt động offline**   | Không (phải đang chạy) | Có (truy vấn bất cứ lúc nào)               |

**Kết luận:** RSC dành cho lắng nghe thời gian thực; Graph API dành cho truy cập lịch sử. Để bắt kịp các tin nhắn bị bỏ lỡ khi offline, bạn cần Graph API với `ChannelMessage.Read.All` (yêu cầu đồng ý admin).
## Graph-enabled media + history (bắt buộc cho các kênh)

Nếu bạn cần hình ảnh/tệp trong **các kênh** hoặc muốn lấy **lịch sử tin nhắn**, bạn phải bật quyền Microsoft Graph và cấp sự đồng ý của quản trị viên.

1. Trong Entra ID (Azure AD) **App Registration**, thêm quyền **Application permissions** của Microsoft Graph:
   - `ChannelMessage.Read.All` (tệp đính kèm kênh + lịch sử)
   - `Chat.Read.All` hoặc `ChatMessage.Read.All` (trò chuyện nhóm)
2. **Cấp sự đồng ý của quản trị viên** cho tenant.
3. Tăng **phiên bản manifest** của ứng dụng Teams, tải lên lại và **cài đặt lại ứng dụng trong Teams**.
4. **Thoát hoàn toàn và khởi động lại Teams** để xóa metadata ứng dụng đã lưu trong bộ nhớ đệm.

**Quyền bổ sung cho việc nhắc đến người dùng:** Việc @nhắc đến người dùng hoạt động ngay lập tức cho những người dùng trong cuộc trò chuyện. Tuy nhiên, nếu bạn muốn tìm kiếm động và nhắc đến những người dùng **không có trong cuộc trò chuyện hiện tại**, hãy thêm quyền `User.Read.All` (Application) và cấp sự đồng ý của quản trị viên.
## Hạn chế đã biết

### Timeout webhook

Teams gửi tin nhắn qua HTTP webhook. Nếu quá trình xử lý mất quá nhiều thời gian (ví dụ: phản hồi LLM chậm), bạn có thể gặp:

- Timeout Gateway
- Teams thử lại tin nhắn (gây ra tin nhắn trùng lặp)
- Phản hồi bị mất

OpenClaw xử lý điều này bằng cách trả về nhanh chóng và gửi phản hồi chủ động, nhưng các phản hồi rất chậm vẫn có thể gây ra vấn đề.

### Định dạng

Markdown của Teams bị hạn chế hơn so với Slack hoặc Discord:

- Định dạng cơ bản hoạt động: **đậm**, _nghiêng_, `code`, liên kết
- Markdown phức tạp (bảng, danh sách lồng nhau) có thể không hiển thị đúng
- Adaptive Cards được hỗ trợ cho bình chọn và gửi thẻ tùy ý (xem bên dưới)
## Cấu hình

Các cài đặt chính (xem `/gateway/configuration` để biết các mẫu kênh được chia sẻ):

- `channels.msteams.enabled`: bật/tắt kênh.
- `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: thông tin xác thực bot.
- `channels.msteams.webhook.port` (mặc định `3978`)
- `channels.msteams.webhook.path` (mặc định `/api/messages`)
- `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: pairing)
- `channels.msteams.allowFrom`: danh sách cho phép tin nhắn riêng (khuyến nghị sử dụng ID đối tượng AAD). Trình hướng dẫn sẽ phân giải tên thành ID trong quá trình thiết lập khi có quyền truy cập Graph.
- `channels.msteams.dangerouslyAllowNameMatching`: công tắc khẩn cấp để bật lại tính năng khớp UPN/tên hiển thị có thể thay đổi.
- `channels.msteams.textChunkLimit`: kích thước khối văn bản gửi đi.
- `channels.msteams.chunkMode`: `length` (mặc định) hoặc `newline` để chia theo dòng trống (ranh giới đoạn văn) trước khi chia theo độ dài.
- `channels.msteams.mediaAllowHosts`: danh sách cho phép các máy chủ tệp đính kèm đến (mặc định là các miền Microsoft/Teams).
- `channels.msteams.mediaAuthAllowHosts`: danh sách cho phép đính kèm tiêu đề Authorization khi thử lại phương tiện (mặc định là các máy chủ Graph + Bot Framework).
- `channels.msteams.requireMention`: yêu cầu @mention trong kênh/nhóm (mặc định true).
- `channels.msteams.replyStyle`: `thread | top-level` (xem [Kiểu trả lời](#reply-style-threads-vs-posts)).
- `channels.msteams.teams.<teamId>.replyStyle`: ghi đè theo nhóm.
- `channels.msteams.teams.<teamId>.requireMention`: ghi đè theo nhóm.
- `channels.msteams.teams.<teamId>.tools`: ghi đè chính sách công cụ mặc định theo nhóm (`allow`/`deny`/`alsoAllow`) được sử dụng khi thiếu ghi đè kênh.
- `channels.msteams.teams.<teamId>.toolsBySender`: ghi đè chính sách công cụ mặc định theo nhóm theo người gửi (`"*"` hỗ trợ ký tự đại diện).
- `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: ghi đè theo kênh.
- `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: ghi đè theo kênh.
- `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: ghi đè chính sách công cụ theo kênh (`allow`/`deny`/`alsoAllow`).
- `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: ghi đè chính sách công cụ theo kênh theo người gửi (`"*"` hỗ trợ ký tự đại diện).
- `toolsBySender` các khóa nên sử dụng tiền tố rõ ràng:
  `id:`, `e164:`, `username:`, `name:` (các khóa cũ không có tiền tố vẫn ánh xạ chỉ đến `id:`).
- `channels.msteams.sharePointSiteId`: ID trang SharePoint để tải lên tệp trong cuộc trò chuyện nhóm/kênh (xem [Gửi tệp trong cuộc trò chuyện nhóm](#sending-files-in-group-chats)).
## Định tuyến & Phiên

- Khóa phiên tuân theo định dạng agent tiêu chuẩn (xem [/concepts/session](/concepts/session)):
  - Tin nhắn riêng chia sẻ phiên chính (`agent:<agentId>:<mainKey>`).
  - Tin nhắn kênh/nhóm sử dụng id cuộc trò chuyện:
    - `agent:<agentId>:msteams:channel:<conversationId>`
    - `agent:<agentId>:msteams:group:<conversationId>`
## Kiểu trả lời: Threads vs Posts

Teams gần đây đã giới thiệu hai kiểu giao diện kênh trên cùng một mô hình dữ liệu cơ bản:

| Kiểu                     | Mô tả                                                     | `replyStyle` được khuyến nghị |
| ------------------------ | --------------------------------------------------------- | ------------------------ |
| **Posts** (cổ điển)      | Tin nhắn xuất hiện dưới dạng thẻ với các phản hồi theo luồng bên dưới | `thread` (mặc định)       |
| **Threads** (giống Slack) | Tin nhắn chảy tuyến tính, giống Slack hơn                   | `top-level`              |

**Vấn đề:** API Teams không hiển thị kiểu giao diện mà kênh sử dụng. Nếu bạn sử dụng sai `replyStyle`:

- `thread` trong kênh kiểu Threads → các phản hồi xuất hiện lồng nhau một cách khó xử
- `top-level` trong kênh kiểu Posts → các phản hồi xuất hiện như các bài đăng cấp cao nhất riêng biệt thay vì trong luồng

**Giải pháp:** Cấu hình `replyStyle` theo từng kênh dựa trên cách kênh được thiết lập:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```
## Tệp đính kèm & Hình ảnh

**Hạn chế hiện tại:**

- **Tin nhắn riêng:** Hình ảnh và tệp đính kèm hoạt động thông qua API tệp bot Teams.
- **Kênh/nhóm:** Tệp đính kèm được lưu trữ trong M365 (SharePoint/OneDrive). Payload webhook chỉ bao gồm HTML stub, không phải byte tệp thực tế. **Cần quyền Graph API** để tải xuống tệp đính kèm kênh.

Không có quyền Graph, tin nhắn kênh có hình ảnh sẽ được nhận dưới dạng chỉ văn bản (nội dung hình ảnh không thể truy cập được bởi bot).
Theo mặc định, OpenClaw chỉ tải xuống media từ hostname Microsoft/Teams. Ghi đè bằng `channels.msteams.mediaAllowHosts` (sử dụng `["*"]` để cho phép bất kỳ host nào).
Header ủy quyền chỉ được đính kèm cho các host trong `channels.msteams.mediaAuthAllowHosts` (mặc định là Graph + Bot Framework hosts). Giữ danh sách này nghiêm ngặt (tránh hậu tố multi-tenant).
## Gửi tệp trong cuộc trò chuyện nhóm

Bot có thể gửi tệp trong tin nhắn riêng bằng luồng FileConsentCard (tích hợp sẵn). Tuy nhiên, **gửi tệp trong cuộc trò chuyện nhóm/kênh** cần thiết lập bổ sung:

| Ngữ cảnh                 | Cách gửi tệp                                 | Thiết lập cần thiết                             |
| ------------------------ | -------------------------------------------- | ----------------------------------------------- |
| **Tin nhắn riêng**       | FileConsentCard → người dùng chấp nhận → bot tải lên | Hoạt động ngay lập tức                          |
| **Cuộc trò chuyện nhóm/kênh** | Tải lên SharePoint → chia sẻ liên kết        | Cần `sharePointSiteId` + quyền Graph             |
| **Hình ảnh (mọi ngữ cảnh)** | Mã hóa Base64 nội tuyến                      | Hoạt động ngay lập tức                          |

### Tại sao cuộc trò chuyện nhóm cần SharePoint

Bot không có ổ đĩa OneDrive cá nhân (điểm cuối Graph API `/me/drive` không hoạt động với danh tính ứng dụng). Để gửi tệp trong cuộc trò chuyện nhóm/kênh, bot tải lên **trang SharePoint** và tạo liên kết chia sẻ.

### Thiết lập

1. **Thêm quyền Graph API** trong Entra ID (Azure AD) → Đăng ký ứng dụng:
   - `Sites.ReadWrite.All` (Ứng dụng) - tải tệp lên SharePoint
   - `Chat.Read.All` (Ứng dụng) - tùy chọn, cho phép liên kết chia sẻ theo người dùng

2. **Cấp đồng ý quản trị** cho tenant.

3. **Lấy ID trang SharePoint của bạn:**

   ```bash
   # Via Graph Explorer or curl with a valid token:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Example: for a site at "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # Response includes: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **Configure OpenClaw:**

   ```json5
   {
     channels: {
       msteams: {
         // ... other config ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
       },
     },
   }
   ```

### Sharing behavior

| Permission                              | Sharing behavior                                          |
| --------------------------------------- | --------------------------------------------------------- |
| `Sites.ReadWrite.All` only              | Organization-wide sharing link (anyone in org can access) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Per-user sharing link (only chat members can access)      |

Per-user sharing is more secure as only the chat participants can access the file. If `Chat.Read.All` bị thiếu, bot sẽ quay về chia sẻ toàn tổ chức.

### Hành vi dự phòng
| Tình huống                                        | Kết quả                                            |
| ------------------------------------------------- | -------------------------------------------------- |
| Trò chuyện nhóm + tệp + `sharePointSiteId` đã cấu hình | Tải lên SharePoint, gửi liên kết chia sẻ            |
| Trò chuyện nhóm + tệp + không có `sharePointSiteId`         | Thử tải lên OneDrive (có thể thất bại), chỉ gửi văn bản |
| Trò chuyện cá nhân + tệp                              | Luồng FileConsentCard (hoạt động mà không cần SharePoint)    |
| Bất kỳ ngữ cảnh nào + hình ảnh                               | Mã hóa Base64 nội tuyến (hoạt động mà không cần SharePoint)   |

### Vị trí lưu trữ tệp

Các tệp được tải lên được lưu trữ trong thư mục `/OpenClawShared/` trong thư viện tài liệu mặc định của trang SharePoint đã cấu hình.
## Khảo sát (Adaptive Cards)

OpenClaw gửi khảo sát Teams dưới dạng Adaptive Cards (không có API khảo sát Teams gốc).

- CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
- Phiếu bầu được gateway ghi lại trong `~/.openclaw/msteams-polls.json`.
- Gateway phải duy trì trực tuyến để ghi lại phiếu bầu.
- Khảo sát chưa tự động đăng tóm tắt kết quả (kiểm tra tệp lưu trữ nếu cần).
## Adaptive Cards (tùy ý)

Gửi bất kỳ JSON Adaptive Card nào đến người dùng hoặc cuộc trò chuyện Teams bằng công cụ `message` hoặc CLI.

Tham số `card` chấp nhận một đối tượng JSON Adaptive Card. Khi `card` được cung cấp, văn bản tin nhắn là tùy chọn.

**Công cụ agent:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello!" }]
  }
}
```

**CLI:**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Xem [tài liệu Adaptive Cards](https://adaptivecards.io/) để biết lược đồ thẻ và ví dụ. Để biết chi tiết về định dạng đích, xem [Định dạng đích](#target-formats) bên dưới.
## Định dạng đích

Các đích MSTeams sử dụng tiền tố để phân biệt giữa người dùng và cuộc hội thoại:

| Loại đích           | Định dạng                        | Ví dụ                                               |
| ------------------- | -------------------------------- | --------------------------------------------------- |
| Người dùng (theo ID)        | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`         |
| Người dùng (theo tên)      | `user:<display-name>`            | `user:John Smith` (yêu cầu Graph API)              |
| Nhóm/kênh       | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`            |
| Nhóm/kênh (thô) | `<conversation-id>`              | `19:abc123...@thread.tacv2` (nếu chứa `@thread`) |

**Ví dụ CLI:**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Send to a user by display name (triggers Graph API lookup)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Agent tool examples:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello" }]
  }
}
```

Note: Without the `user:` prefix, names default to group/team resolution. Always use `user:` khi nhắm đến mọi người theo tên hiển thị.
## Tin nhắn chủ động

- Tin nhắn chủ động chỉ có thể thực hiện **sau khi** người dùng đã tương tác, vì chúng tôi lưu trữ tham chiếu cuộc trò chuyện tại thời điểm đó.
- Xem `/gateway/configuration` để biết `dmPolicy` và cổng kiểm soát danh sách cho phép.
## ID Nhóm và Kênh (Lỗi Thường Gặp)

Tham số truy vấn `groupId` trong URL Teams **KHÔNG PHẢI** là ID nhóm được sử dụng để cấu hình. Thay vào đó, hãy trích xuất ID từ đường dẫn URL:

**URL Nhóm:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (URL-decode this)
```

**Channel URL:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**For config:**

- Team ID = path segment after `/team/` (URL-decoded, e.g., `19:Bk4j...@thread.tacv2`)
- Channel ID = path segment after `/channel/` (URL-decoded)
- **Ignore** the `groupId` query parameter
## Kênh riêng tư

Bot có hỗ trợ hạn chế trong các kênh riêng tư:

| Tính năng                    | Kênh tiêu chuẩn   | Kênh riêng tư          |
| ---------------------------- | ----------------- | ---------------------- |
| Cài đặt bot                  | Có                | Hạn chế                |
| Tin nhắn thời gian thực (webhook) | Có          | Có thể không hoạt động |
| Quyền RSC                    | Có                | Có thể hoạt động khác  |
| @mentions                    | Có                | Nếu bot có thể truy cập |
| Lịch sử Graph API            | Có                | Có (với quyền)         |

**Giải pháp thay thế nếu kênh riêng tư không hoạt động:**

1. Sử dụng kênh tiêu chuẩn cho tương tác bot
2. Sử dụng tin nhắn riêng - người dùng luôn có thể nhắn tin trực tiếp với bot
3. Sử dụng Graph API để truy cập lịch sử (yêu cầu `ChannelMessage.Read.All`)
## Khắc phục sự cố

### Các vấn đề thường gặp

- **Hình ảnh không hiển thị trong kênh:** Thiếu quyền Graph hoặc chưa có sự đồng ý của quản trị viên. Cài đặt lại ứng dụng Teams và thoát/mở lại Teams hoàn toàn.
- **Không có phản hồi trong kênh:** theo mặc định cần phải mention; đặt `channels.msteams.requireMention=false` hoặc cấu hình theo từng team/kênh.
- **Không khớp phiên bản (Teams vẫn hiển thị manifest cũ):** xóa + thêm lại ứng dụng và thoát Teams hoàn toàn để làm mới.
- **401 Unauthorized từ webhook:** Điều này bình thường khi kiểm tra thủ công mà không có Azure JWT - có nghĩa là endpoint có thể truy cập được nhưng xác thực thất bại. Sử dụng Azure Web Chat để kiểm tra đúng cách.

### Lỗi tải lên manifest

- **"Icon file cannot be empty":** Manifest tham chiếu đến các tệp icon có kích thước 0 byte. Tạo các icon PNG hợp lệ (32x32 cho `outline.png`, 192x192 cho `color.png`).
- **"webApplicationInfo.Id already in use":** Ứng dụng vẫn được cài đặt trong team/chat khác. Tìm và gỡ cài đặt trước, hoặc đợi 5-10 phút để lan truyền.
- **"Something went wrong" khi tải lên:** Thay vào đó hãy tải lên qua [https://admin.teams.microsoft.com](https://admin.teams.microsoft.com), mở DevTools của trình duyệt (F12) → tab Network, và kiểm tra response body để xem lỗi thực tế.
- **Sideload thất bại:** Thử "Upload an app to your org's app catalog" thay vì "Upload a custom app" - điều này thường bỏ qua các hạn chế sideload.

### Quyền RSC không hoạt động

1. Xác minh `webApplicationInfo.id` khớp chính xác với App ID của bot
2. Tải lại ứng dụng và cài đặt lại trong team/chat
3. Kiểm tra xem quản trị viên tổ chức có chặn quyền RSC không
4. Xác nhận bạn đang sử dụng đúng scope: `ChannelMessage.Read.Group` cho teams, `ChatMessage.Read.Chat` cho group chats
## Tài liệu tham khảo

- [Tạo Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - Hướng dẫn thiết lập Azure Bot
- [Teams Developer Portal](https://dev.teams.microsoft.com/apps) - tạo/quản lý ứng dụng Teams
- [Lược đồ manifest ứng dụng Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
- [Nhận tin nhắn kênh với RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
- [Tài liệu tham khảo quyền RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
- [Xử lý tệp bot Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (kênh/nhóm yêu cầu Graph)
- [Tin nhắn chủ động](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)