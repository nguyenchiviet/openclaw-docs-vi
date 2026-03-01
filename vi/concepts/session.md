---
summary: 'Quy tắc quản lý phiên, khóa và tính bền vững cho trò chuyện'
read_when:
  - Sửa đổi xử lý phiên hoặc lưu trữ
title: Quản lý phiên
x-i18n:
  source_path: concepts\session.md
  source_hash: ad92958d5cf6d8bd33f364b6a8738a0bf892926099d461bce7b4ed41d394738e
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-03-01T05:09:25.800Z'
---

# Quản lý phiên

OpenClaw coi **một phiên trò chuyện trực tiếp trên mỗi agent** là chính. Các cuộc trò chuyện trực tiếp được gộp vào `agent:<agentId>:<mainKey>` (mặc định `main`), trong khi các cuộc trò chuyện nhóm/kênh có khóa riêng. `session.mainKey` được tuân thủ.

Sử dụng `session.dmScope` để kiểm soát cách **tin nhắn trực tiếp** được nhóm:

- `main` (mặc định): tất cả các tin nhắn riêng chia sẻ phiên chính để duy trì tính liên tục.
- `per-peer`: cách ly theo ID người gửi trên các kênh.
- `per-channel-peer`: cách ly theo kênh + người gửi (được khuyến nghị cho hộp thư đến nhiều người dùng).
- `per-account-channel-peer`: cách ly theo tài khoản + kênh + người gửi (được khuyến nghị cho hộp thư đến nhiều tài khoản).
  Sử dụng `session.identityLinks` để ánh xạ các ID ngang hàng có tiền tố nhà cung cấp tới một danh tính chuẩn để cùng một người chia sẻ phiên tin nhắn riêng trên các kênh khi sử dụng `per-peer`, `per-channel-peer`, hoặc `per-account-channel-peer`.
## Chế độ Tin nhắn riêng an toàn (khuyên dùng cho thiết lập đa người dùng)

> **Cảnh báo bảo mật:** Nếu agent của bạn có thể nhận Tin nhắn riêng từ **nhiều người**, bạn nên cân nhắc kỹ việc bật chế độ Tin nhắn riêng an toàn. Nếu không có chế độ này, tất cả người dùng sẽ chia sẻ cùng một ngữ cảnh hội thoại, điều này có thể làm rò rỉ thông tin riêng tư giữa các người dùng.

**Ví dụ về vấn đề với cài đặt mặc định:**

- Alice (`<SENDER_A>`) nhắn tin cho agent của bạn về một chủ đề riêng tư (ví dụ: một cuộc hẹn y tế)
- Bob (__OC_I19N_0001__) nhắn tin cho agent của bạn hỏi "Chúng ta đang nói về chuyện gì vậy?"
- Vì cả hai Tin nhắn riêng đều chia sẻ cùng một phiên, mô hình có thể trả lời Bob bằng ngữ cảnh trước đó của Alice.

**Cách khắc phục:** Đặt __OC_I19N_0002__ để cách ly các phiên theo từng người dùng:

``__OC_I19N_0003__`__OC_I19N_0004__dmPolicy: "open"`
- Nhiều số điện thoại hoặc tài khoản có thể nhắn tin cho agent của bạn

Ghi chú:
- Mặc định là `dmScope: "main"` để duy trì tính liên tục (tất cả các tin nhắn riêng đều chia sẻ phiên chính). Điều này phù hợp cho các thiết lập một người dùng.
- Quá trình thiết lập ban đầu CLI cục bộ ghi `session.dmScope: "per-channel-peer"` theo mặc định khi chưa được đặt (các giá trị rõ ràng hiện có được giữ nguyên).
- Đối với hộp thư đến đa tài khoản trên cùng một kênh, ưu tiên `per-account-channel-peer`.
- Nếu cùng một người liên hệ với bạn trên nhiều kênh, hãy sử dụng `session.identityLinks` để gộp các phiên tin nhắn riêng của họ thành một danh tính chuẩn.
- Bạn có thể xác minh cài đặt tin nhắn riêng của mình bằng `openclaw security audit` (xem [bảo mật](/cli/security)).
## Gateway là nguồn thông tin đáng tin cậy

Tất cả trạng thái phiên đều **thuộc sở hữu của gateway** (OpenClaw "chủ"). Các ứng dụng giao diện người dùng (ứng dụng macOS, WebChat, v.v.) phải truy vấn gateway để lấy danh sách phiên và số lượng token thay vì đọc các tệp cục bộ.

- Ở **chế độ từ xa**, kho lưu trữ phiên mà bạn quan tâm nằm trên máy chủ gateway từ xa, không phải trên máy Mac của bạn.
- Số lượng token hiển thị trong giao diện người dùng đến từ các trường lưu trữ của gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Các ứng dụng khách không phân tích cú pháp bản ghi JSONL để "sửa" tổng số.

## Nơi trạng thái được lưu trữ

- Trên **máy chủ gateway**:
  - Tệp lưu trữ: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (cho mỗi agent).
- Bản ghi: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (các phiên chủ đề Telegram sử dụng `.../<SessionId>-topic-<threadId>.jsonl`).
- Kho lưu trữ là một bản đồ `sessionKey -> { sessionId, updatedAt, ... }`. Việc xóa các mục là an toàn; chúng được tạo lại theo yêu cầu.
- Các mục nhóm có thể bao gồm `displayName`, `channel`, `subject`, `room`, và `space` để gắn nhãn các phiên trong giao diện người dùng.
- Các mục phiên bao gồm siêu dữ liệu `origin` (nhãn + gợi ý định tuyến) để giao diện người dùng có thể giải thích nguồn gốc của một phiên.
- OpenClaw **không** đọc các thư mục phiên Pi/Tau cũ.

## Bảo trì

OpenClaw áp dụng bảo trì kho lưu trữ phiên để giữ cho `sessions.json` và các tạo phẩm bản ghi được giới hạn theo thời gian.

### Mặc định

- `session.maintenance.mode`: `warn`
- `session.maintenance.pruneAfter`: `30d`
- `session.maintenance.maxEntries`: `500`
- `session.maintenance.rotateBytes`: `10mb`
- `session.maintenance.resetArchiveRetention`: mặc định là `pruneAfter` (`30d`)
- `session.maintenance.maxDiskBytes`: chưa đặt (đã tắt)
- `session.maintenance.highWaterBytes`: mặc định là `80%` của `maxDiskBytes` khi bật ngân sách

### Cách hoạt động

Bảo trì chạy trong quá trình ghi vào kho lưu trữ phiên và bạn có thể kích hoạt nó theo yêu cầu bằng `openclaw sessions cleanup`.

- `mode: "warn"`: báo cáo những gì sẽ bị trục xuất nhưng không làm thay đổi các mục nhập/bản ghi.
- `mode: "enforce"`: áp dụng dọn dẹp theo thứ tự này:
  1. cắt bỏ các mục nhập cũ hơn `pruneAfter`
  2. giới hạn số lượng mục nhập ở `maxEntries` (cũ nhất trước)
  3. lưu trữ các tệp bản ghi cho các mục nhập đã xóa không còn được tham chiếu
  4. xóa các kho lưu trữ `*.deleted.<timestamp>` và `*.reset.<timestamp>` cũ theo chính sách lưu giữ
  5. xoay vòng `sessions.json` khi nó vượt quá `rotateBytes`
  6. nếu `maxDiskBytes` được đặt, thực thi ngân sách đĩa hướng tới `highWaterBytes` (các tạo phẩm cũ nhất trước, sau đó là các phiên cũ nhất)

### Lưu ý về hiệu suất đối với các kho lưu trữ lớn

Các kho lưu trữ phiên lớn thường thấy trong các thiết lập có khối lượng lớn. Công việc bảo trì là công việc trên đường ghi, vì vậy các kho lưu trữ rất lớn có thể làm tăng độ trễ ghi.
Điều gì làm tăng chi phí nhiều nhất:

- giá trị `session.maintenance.maxEntries` rất cao
- cửa sổ `pruneAfter` dài giữ các mục cũ
- nhiều bản ghi/tệp lưu trữ trong `~/.openclaw/agents/<agentId>/sessions/`
- bật ngân sách đĩa (`maxDiskBytes`) mà không có giới hạn cắt tỉa/giới hạn hợp lý

Những việc cần làm:

- sử dụng __OC_I19N_0004__ trong môi trường sản xuất để tăng trưởng được giới hạn tự động
- đặt cả giới hạn thời gian và số lượng (__OC_I19N_0005__ + __OC_I19N_0006__), không chỉ một
- đặt __OC_I19N_0007__ + __OC_I19N_0008__ cho các giới hạn trên cứng trong các triển khai lớn
- giữ __OC_I19N_0009__ thấp hơn đáng kể so với __OC_I19N_0010__ (mặc định là 80%)
- chạy __OC_I19N_0011__ sau khi thay đổi cấu hình để xác minh tác động dự kiến trước khi thực thi
- đối với các phiên hoạt động thường xuyên, truyền __OC_I19N_0012__ khi chạy dọn dẹp thủ công

### Tùy chỉnh ví dụ

Sử dụng chính sách thực thi thận trọng:

``__OC_I19N_0013__``
Bật giới hạn dung lượng ổ cứng cho thư mục phiên:

```json5
{
  session: {
    maintenance: {
      mode: "enforce",
      maxDiskBytes: "1gb",
      highWaterBytes: "800mb",
    },
  },
}
```

Tune for larger installs (example):

```json5
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "14d",
      maxEntries: 2000,
      rotateBytes: "25mb",
      maxDiskBytes: "2gb",
      highWaterBytes: "1.6gb",
    },
  },
}
```
Xem trước hoặc buộc bảo trì từ CLI:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```
## Cắt tỉa phiên

OpenClaw mặc định sẽ cắt tỉa **kết quả công cụ cũ** khỏi ngữ cảnh trong bộ nhớ ngay trước các lệnh gọi LLM.
Điều này **không** ghi lại lịch sử JSONL. Xem [/concepts/session-pruning](/concepts/session-pruning).
## Xả bộ nhớ trước khi nén

Khi một phiên gần đến giai đoạn tự động nén, OpenClaw có thể chạy một lượt **xả bộ nhớ im lặng**
nhắc nhở mô hình ghi các ghi chú bền vững vào đĩa. Điều này chỉ chạy khi
không gian làm việc có thể ghi được. Xem [Bộ nhớ](/concepts/memory) và
[Nén](/concepts/compaction).

## Ánh xạ giao thức truyền tải → khóa phiên

- Tin nhắn riêng tuân theo `session.dmScope` (mặc định `main`).
  - `main`: `agent:<agentId>:<mainKey>` (tính liên tục trên các thiết bị/kênh).
    - Nhiều số điện thoại và kênh có thể ánh xạ tới cùng một khóa chính của agent; chúng hoạt động như các giao thức truyền tải vào một cuộc hội thoại.
  - `per-peer`: `agent:<agentId>:dm:<peerId>`.
  - `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  - `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (accountId mặc định là `default`).
  - Nếu `session.identityLinks` khớp với một ID ngang hàng có tiền tố nhà cung cấp (ví dụ `telegram:123`), khóa chính tắc sẽ thay thế `<peerId>` để cùng một người chia sẻ một phiên trên các kênh.
- Trò chuyện nhóm cách ly trạng thái: `agent:<agentId>:<channel>:group:<id>` (phòng/kênh sử dụng `agent:<agentId>:<channel>:channel:<id>`).
  - Chủ đề diễn đàn Telegram thêm `:topic:<threadId>` vào ID nhóm để cách ly.
  - Các khóa `group:<id>` cũ vẫn được nhận diện để di chuyển.
- Các ngữ cảnh đến vẫn có thể sử dụng `group:<id>`; kênh được suy ra từ `Provider` và được chuẩn hóa thành dạng chính tắc `agent:<agentId>:<channel>:group:<id>`.
- Các nguồn khác:
  - Cron jobs: `cron:<job.id>`
  - Webhooks: `hook:<uuid>` (trừ khi được hook đặt rõ ràng)
  - Chạy node: `node-<nodeId>`
## Vòng đời

- Chính sách đặt lại: các phiên được tái sử dụng cho đến khi chúng hết hạn, và thời điểm hết hạn được đánh giá dựa trên tin nhắn đến tiếp theo.
- Đặt lại hàng ngày: mặc định là **4:00 sáng giờ địa phương trên máy chủ Gateway**. Một phiên được coi là cũ khi lần cập nhật cuối cùng của nó sớm hơn thời gian đặt lại hàng ngày gần nhất.
- Đặt lại khi không hoạt động (tùy chọn): `idleMinutes` thêm một cửa sổ không hoạt động trượt. Khi cả đặt lại hàng ngày và đặt lại khi không hoạt động được cấu hình, **cái nào hết hạn trước** sẽ buộc một phiên mới.
- Chế độ chỉ không hoạt động cũ: nếu bạn đặt `session.idleMinutes` mà không có bất kỳ cấu hình `session.reset`/`resetByType` nào, OpenClaw vẫn ở chế độ chỉ không hoạt động để tương thích ngược.
- Ghi đè theo loại (tùy chọn): `resetByType` cho phép bạn ghi đè chính sách cho các phiên `direct`, `group` và `thread` (luồng = luồng Slack/Discord, chủ đề Telegram, luồng Matrix khi được cung cấp bởi trình kết nối).
- Ghi đè theo kênh (tùy chọn): `resetByChannel` ghi đè chính sách đặt lại cho một kênh (áp dụng cho tất cả các loại phiên cho kênh đó và ưu tiên hơn `reset`/`resetByType`).
- Kích hoạt đặt lại: chính xác `/new` hoặc `/reset` (cộng với bất kỳ phần bổ sung nào trong `resetTriggers`) bắt đầu một ID phiên mới và chuyển phần còn lại của tin nhắn đi. `/new <model>` chấp nhận một bí danh mô hình, `provider/model`, hoặc tên nhà cung cấp (khớp mờ) để đặt mô hình phiên mới. Nếu `/new` hoặc `/reset` được gửi riêng, OpenClaw sẽ chạy một lượt chào "hello" ngắn để xác nhận việc đặt lại.
- Đặt lại thủ công: xóa các khóa cụ thể khỏi kho lưu trữ hoặc xóa bản ghi JSONL; tin nhắn tiếp theo sẽ tạo lại chúng.
- Các cron job bị cô lập luôn tạo một `sessionId` mới cho mỗi lần chạy (không tái sử dụng khi không hoạt động).
## Chính sách gửi (tùy chọn)

Chặn gửi cho các loại phiên cụ thể mà không cần liệt kê từng ID riêng lẻ.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // Match the raw session key (including the `agent:<id>:` prefix).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

Runtime override (owner only):

- `/send on` → allow for this session
- `/send off` → deny for this session
- `/send inherit` → xóa ghi đè và sử dụng các quy tắc cấu hình
  Gửi các tin nhắn này dưới dạng tin nhắn độc lập để chúng được ghi nhận.
## Cấu hình (ví dụ đổi tên tùy chọn)

``__OC_I19N_0000__``
## Kiểm tra

- `openclaw status` — hiển thị đường dẫn lưu trữ và các phiên gần đây.
- `openclaw sessions --json` — xuất mọi mục nhập (lọc bằng `--active <minutes>`).
- `openclaw gateway call sessions.list --params '{}'` — lấy các phiên từ Gateway đang chạy (sử dụng `--url`/`--token` để truy cập Gateway từ xa).
- Gửi `/status` dưới dạng tin nhắn độc lập trong cuộc trò chuyện để xem liệu agent có thể truy cập được không, bao nhiêu ngữ cảnh phiên được sử dụng, các chế độ bật/tắt suy nghĩ/chi tiết hiện tại và lần cuối cùng thông tin đăng nhập WhatsApp web của bạn được làm mới là khi nào (giúp phát hiện nhu cầu liên kết lại).
- Gửi `/context list` hoặc `/context detail` để xem nội dung trong lời nhắc hệ thống và các tệp không gian làm việc được chèn (và những yếu tố đóng góp ngữ cảnh lớn nhất).
- Gửi `/stop` (hoặc các cụm từ hủy bỏ độc lập như `stop`, `stop action`, `stop run`, `stop openclaw`) để hủy bỏ lần chạy hiện tại, xóa các phản hồi tiếp theo đã xếp hàng cho phiên đó và dừng mọi lần chạy sub-agent được tạo ra từ đó (phản hồi bao gồm số lượng đã dừng).
- Gửi `/compact` (hướng dẫn tùy chọn) dưới dạng tin nhắn độc lập để tóm tắt ngữ cảnh cũ hơn và giải phóng không gian cửa sổ. Xem [/concepts/compaction](/concepts/compaction).
- Bản ghi JSONL có thể được mở trực tiếp để xem lại toàn bộ lượt.

## Mẹo

- Giữ khóa chính dành riêng cho lưu lượng 1:1; để các nhóm giữ khóa riêng của họ.
- Khi tự động hóa việc dọn dẹp, hãy xóa từng khóa riêng lẻ thay vì toàn bộ kho lưu trữ để bảo toàn ngữ cảnh ở những nơi khác.
## Siêu dữ liệu nguồn gốc phiên

Mỗi mục phiên ghi lại nguồn gốc của nó (nỗ lực tốt nhất) trong `origin`:

- `label`: nhãn người dùng (được giải quyết từ nhãn cuộc trò chuyện + chủ đề/kênh nhóm)
- `provider`: ID kênh đã chuẩn hóa (bao gồm các tiện ích mở rộng)
- `from`/`to`: ID định tuyến thô từ phong bì đến
- `accountId`: ID tài khoản nhà cung cấp (khi có nhiều tài khoản)
- `threadId`: ID luồng/chủ đề khi kênh hỗ trợ
  Các trường nguồn gốc được điền cho tin nhắn trực tiếp, kênh và nhóm. Nếu một
  trình kết nối chỉ cập nhật định tuyến phân phối (ví dụ: để giữ phiên chính DM
  luôn mới), nó vẫn nên cung cấp ngữ cảnh đến để phiên giữ
  siêu dữ liệu giải thích của nó. Các tiện ích mở rộng có thể làm điều này bằng cách gửi `ConversationLabel`,
  `GroupSubject`, `GroupChannel`, `GroupSpace`, và `SenderName` trong ngữ cảnh đến
  và gọi `recordSessionMetaFromInbound` (hoặc truyền cùng ngữ cảnh
  đến `updateLastRoute`).