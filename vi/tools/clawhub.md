---
summary: 'Hướng dẫn ClawHub: registry kỹ năng công khai + quy trình làm việc CLI'
read_when:
  - Giới thiệu ClawHub cho những người dùng mới
  - 'Cài đặt, tìm kiếm hoặc xuất bản Skills'
  - Giải thích các cờ CLI của ClawHub và hành vi đồng bộ
title: ClawHub
x-i18n:
  source_path: tools\clawhub.md
  source_hash: b572473a1124635744cfe537143249cd57659b511d060bb631ffddba7c8c8315
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:29:51.263Z'
---

# ClawHub

ClawHub là **sổ đăng ký Skills công khai cho OpenClaw**. Đây là một dịch vụ miễn phí: tất cả các Skills đều công khai, mở và hiển thị cho mọi người để chia sẻ và tái sử dụng. Một Skill chỉ là một thư mục có chứa tệp `SKILL.md` (cộng với các tệp văn bản hỗ trợ). Bạn có thể duyệt các Skills trong ứng dụng web hoặc sử dụng CLI để tìm kiếm, cài đặt, cập nhật và xuất bản các Skills.

Trang web: [clawhub.ai](https://clawhub.ai)
## ClawHub là gì

- Một sổ đăng ký công khai cho các Skills của OpenClaw.
- Một kho lưu trữ có phiên bản của các gói Skills và siêu dữ liệu.
- Một bề mặt khám phá để tìm kiếm, thẻ và tín hiệu sử dụng.
## Cách hoạt động

1. Người dùng xuất bản một gói skill (tệp + siêu dữ liệu).
2. ClawHub lưu trữ gói, phân tích siêu dữ liệu và gán phiên bản.
3. Sổ đăng ký lập chỉ mục skill để tìm kiếm và khám phá.
4. Người dùng duyệt, tải xuống và cài đặt skill trong OpenClaw.
## Những gì bạn có thể làm

- Xuất bản các Skills mới và các phiên bản mới của các Skills hiện có.
- Khám phá Skills theo tên, thẻ hoặc tìm kiếm.
- Tải xuống các gói Skills và kiểm tra các tệp của chúng.
- Báo cáo các Skills lạm dụng hoặc không an toàn.
- Nếu bạn là người điều hành, hãy ẩn, hiển thị lại, xóa hoặc cấm.
## Dành cho ai (thân thiện với người mới bắt đầu)

Nếu bạn muốn thêm các khả năng mới cho agent OpenClaw của mình, ClawHub là cách dễ nhất để tìm và cài đặt Skills. Bạn không cần biết backend hoạt động như thế nào. Bạn có thể:

- Tìm kiếm Skills bằng ngôn ngữ tự nhiên.
- Cài đặt một Skill vào không gian làm việc của bạn.
- Cập nhật Skills sau này bằng một lệnh.
- Sao lưu các Skills của riêng bạn bằng cách xuất bản chúng.
## Bắt đầu nhanh (không kỹ thuật)

1. Cài đặt CLI (xem phần tiếp theo).
2. Tìm kiếm thứ gì đó bạn cần:
   - `clawhub search "calendar"`
3. Cài đặt một skill:
   - `clawhub install <skill-slug>`
4. Bắt đầu một phiên OpenClaw mới để nó nhận diện skill mới.
## Cài đặt CLI

Chọn một:

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```
## Cách nó phù hợp với OpenClaw

Theo mặc định, CLI cài đặt Skills vào `./skills` trong thư mục làm việc hiện tại của bạn. Nếu một không gian làm việc OpenClaw được cấu hình, `clawhub` sẽ quay lại không gian làm việc đó trừ khi bạn ghi đè `--workdir` (hoặc `CLAWHUB_WORKDIR`). OpenClaw tải Skills không gian làm việc từ `<workspace>/skills` và sẽ chọn chúng trong phiên **tiếp theo**. Nếu bạn đã sử dụng `~/.openclaw/skills` hoặc Skills được đóng gói, Skills không gian làm việc sẽ được ưu tiên.

Để biết thêm chi tiết về cách Skills được tải, chia sẻ và kiểm soát, xem
[Skills](/tools/skills).
## Tổng quan về hệ thống Skills

Một skill là một gói tệp có phiên bản giúp OpenClaw học cách thực hiện một
nhiệm vụ cụ thể. Mỗi lần xuất bản tạo ra một phiên bản mới, và registry lưu giữ
lịch sử các phiên bản để người dùng có thể kiểm tra các thay đổi.

Một skill điển hình bao gồm:

- Tệp `SKILL.md` với mô tả chính và hướng dẫn sử dụng.
- Các cấu hình tùy chọn, tập lệnh hoặc tệp hỗ trợ được sử dụng bởi skill.
- Siêu dữ liệu như thẻ, tóm tắt và yêu cầu cài đặt.

ClawHub sử dụng siêu dữ liệu để cung cấp khám phá và an toàn tiếp xúc các khả năng của skill.
Registry cũng theo dõi các tín hiệu sử dụng (chẳng hạn như sao và lượt tải xuống) để cải thiện
xếp hạng và khả năng hiển thị.
## Những gì dịch vụ cung cấp (tính năng)

- **Duyệt công khai** các Skills và nội dung `SKILL.md` của chúng.
- **Tìm kiếm** được hỗ trợ bởi embeddings (tìm kiếm vector), không chỉ từ khóa.
- **Quản lý phiên bản** với semver, nhật ký thay đổi và thẻ (bao gồm `latest`).
- **Tải xuống** dưới dạng zip cho mỗi phiên bản.
- **Sao và bình luận** để nhận phản hồi từ cộng đồng.
- **Kiểm duyệt** hooks để phê duyệt và kiểm toán.
- **API thân thiện với CLI** để tự động hóa và viết kịch bản.
## Bảo mật và kiểm duyệt

ClawHub mở theo mặc định. Bất kỳ ai cũng có thể tải lên Skills, nhưng tài khoản GitHub phải có tuổi ít nhất một tuần để xuất bản. Điều này giúp làm chậm lạm dụng mà không chặn những người đóng góp hợp pháp.

Báo cáo và kiểm duyệt:

- Bất kỳ người dùng nào đã đăng nhập đều có thể báo cáo một Skill.
- Lý do báo cáo là bắt buộc và được ghi lại.
- Mỗi người dùng có thể có tối đa 20 báo cáo hoạt động cùng một lúc.
- Skills có hơn 3 báo cáo duy nhất sẽ bị ẩn tự động theo mặc định.
- Những người kiểm duyệt có thể xem Skills ẩn, bỏ ẩn chúng, xóa chúng hoặc cấm người dùng.
- Lạm dụng tính năng báo cáo có thể dẫn đến cấm tài khoản.

Quan tâm đến việc trở thành người kiểm duyệt? Hỏi trong Discord OpenClaw và liên hệ với một người kiểm duyệt hoặc người duy trì.
## Các lệnh CLI và tham số

Tùy chọn toàn cục (áp dụng cho tất cả các lệnh):

- `--workdir <dir>`: Thư mục làm việc (mặc định: thư mục hiện tại; quay lại không gian làm việc OpenClaw).
- `--dir <dir>`: Thư mục Skills, tương đối với workdir (mặc định: `skills`).
- `--site <url>`: URL cơ sở trang web (đăng nhập trình duyệt).
- `--registry <url>`: URL cơ sở API Registry.
- `--no-input`: Vô hiệu hóa lời nhắc (không tương tác).
- `-V, --cli-version`: In phiên bản CLI.

Xác thực:

- `clawhub login` (luồng trình duyệt) hoặc `clawhub login --token <token>`
- `clawhub logout`
- `clawhub whoami`

Tùy chọn:

- `--token <token>`: Dán mã thông báo API.
- `--label <label>`: Nhãn được lưu trữ cho mã thông báo đăng nhập trình duyệt (mặc định: `CLI token`).
- `--no-browser`: Không mở trình duyệt (yêu cầu `--token`).

Tìm kiếm:

- `clawhub search "query"`
- `--limit <n>`: Số kết quả tối đa.

Cài đặt:

- `clawhub install <slug>`
- `--version <version>`: Cài đặt một phiên bản cụ thể.
- `--force`: Ghi đè nếu thư mục đã tồn tại.

Cập nhật:

- `clawhub update <slug>`
- `clawhub update --all`
- `--version <version>`: Cập nhật lên một phiên bản cụ thể (chỉ slug duy nhất).
- `--force`: Ghi đè khi các tệp cục bộ không khớp với bất kỳ phiên bản được xuất bản nào.

Danh sách:

- `clawhub list` (đọc `.clawhub/lock.json`)

Xuất bản:

- `clawhub publish <path>`
- `--slug <slug>`: Slug Skill.
- `--name <name>`: Tên hiển thị.
- `--version <version>`: Phiên bản Semver.
- `--changelog <text>`: Văn bản nhật ký thay đổi (có thể trống).
- `--tags <tags>`: Các thẻ được phân tách bằng dấu phẩy (mặc định: `latest`).

Xóa/khôi phục (chỉ chủ sở hữu/quản trị viên):

- `clawhub delete <slug> --yes`
- `clawhub undelete <slug> --yes`

Đồng bộ (quét các Skills cục bộ + xuất bản các Skills mới/đã cập nhật):
- `clawhub sync`
- `--root <dir...>`: Các thư mục gốc quét bổ sung.
- `--all`: Tải lên mọi thứ mà không cần nhắc.
- `--dry-run`: Hiển thị những gì sẽ được tải lên.
- `--bump <type>`: `patch|minor|major` để cập nhật (mặc định: `patch`).
- `--changelog <text>`: Nhật ký thay đổi cho các bản cập nhật không tương tác.
- `--tags <tags>`: Các thẻ được phân tách bằng dấu phẩy (mặc định: `latest`).
- `--concurrency <n>`: Kiểm tra registry (mặc định: 4).
## Quy trình công việc phổ biến cho agent

### Tìm kiếm Skills

```bash
clawhub search "postgres backups"
```

### Download new skills

```bash
clawhub install my-skill-pack
```

### Update installed skills

```bash
clawhub update --all
```

### Back up your skills (publish or sync)

For a single skill folder:

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

To scan and back up many skills at once:

```bash
clawhub sync --all
```
## Chi tiết nâng cao (kỹ thuật)

### Phiên bản và thẻ

- Mỗi lần xuất bản tạo ra một **semver** `SkillVersion` mới.
- Thẻ (như `latest`) trỏ đến một phiên bản; di chuyển thẻ cho phép bạn quay lại.
- Nhật ký thay đổi được đính kèm theo từng phiên bản và có thể trống khi đồng bộ hóa hoặc xuất bản cập nhật.

### Thay đổi cục bộ so với phiên bản đăng ký

Cập nhật so sánh nội dung skill cục bộ với các phiên bản đăng ký bằng cách sử dụng hàm băm nội dung. Nếu các tệp cục bộ không khớp với bất kỳ phiên bản được xuất bản nào, CLI sẽ yêu cầu trước khi ghi đè (hoặc yêu cầu `--force` trong các lần chạy không tương tác).

### Quét đồng bộ hóa và thư mục gốc dự phòng

`clawhub sync` quét thư mục làm việc hiện tại của bạn trước tiên. Nếu không tìm thấy skill nào, nó sẽ quay lại các vị trí kế thừa đã biết (ví dụ `~/openclaw/skills` và `~/.openclaw/skills`). Điều này được thiết kế để tìm các cài đặt skill cũ hơn mà không cần các cờ bổ sung.

### Lưu trữ và tệp khóa

- Các skill được cài đặt được ghi lại trong `.clawhub/lock.json` dưới thư mục làm việc của bạn.
- Mã thông báo xác thực được lưu trữ trong tệp cấu hình CLI ClawHub (ghi đè qua `CLAWHUB_CONFIG_PATH`).

### Telemetry (số lượng cài đặt)

Khi bạn chạy `clawhub sync` trong khi đã đăng nhập, CLI sẽ gửi một ảnh chụp tối thiểu để tính toán số lượng cài đặt. Bạn có thể tắt hoàn toàn điều này:

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```
## Biến môi trường

- `CLAWHUB_SITE`: Ghi đè URL trang web.
- `CLAWHUB_REGISTRY`: Ghi đè URL API đăng ký.
- `CLAWHUB_CONFIG_PATH`: Ghi đè vị trí lưu trữ token/cấu hình của CLI.
- `CLAWHUB_WORKDIR`: Ghi đè thư mục làm việc mặc định.
- `CLAWHUB_DISABLE_TELEMETRY=1`: Vô hiệu hóa telemetry trên `sync`.