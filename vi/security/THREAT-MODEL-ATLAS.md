---
x-i18n:
  source_path: security\THREAT-MODEL-ATLAS.md
  source_hash: 97dce2d990a412691b52f0e640cf1bc5bcf577c9bf20bbf732bf9c70394ef102
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:24:04.570Z'
---

# Mô hình Mối đe dọa OpenClaw v1.0

## Khung MITRE ATLAS

**Phiên bản:** 1.0-draft
**Cập nhật lần cuối:** 2026-02-04
**Phương pháp:** MITRE ATLAS + Sơ đồ Luồng Dữ liệu
**Khung:** [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems)

### Quy định về Khung

Mô hình mối đe dọa này được xây dựng dựa trên [MITRE ATLAS](https://atlas.mitre.org/), khung tiêu chuẩn công nghiệp để ghi chép các mối đe dọa đối kháng đối với các hệ thống AI/ML. ATLAS được duy trì bởi [MITRE](https://www.mitre.org/) phối hợp với cộng đồng bảo mật AI.

**Tài nguyên ATLAS chính:**

- [Kỹ thuật ATLAS](https://atlas.mitre.org/techniques/)
- [Chiến thuật ATLAS](https://atlas.mitre.org/tactics/)
- [Các trường hợp nghiên cứu ATLAS](https://atlas.mitre.org/studies/)
- [GitHub ATLAS](https://github.com/mitre-atlas/atlas-data)
- [Đóng góp cho ATLAS](https://atlas.mitre.org/resources/contribute)

### Đóng góp cho Mô hình Mối đe dọa này

Đây là một tài liệu sống được duy trì bởi cộng đồng OpenClaw. Xem [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) để biết hướng dẫn về cách đóng góp:

- Báo cáo các mối đe dọa mới
- Cập nhật các mối đe dọa hiện có
- Đề xuất chuỗi tấn công
- Gợi ý các biện pháp giảm thiểu

---
## 1. Giới thiệu

### 1.1 Mục đích

Mô hình đe dọa này ghi lại các mối đe dọa đối kháng đối với nền tảng agent AI OpenClaw và thị trường kỹ năng ClawHub, sử dụng khung MITRE ATLAS được thiết kế đặc biệt cho các hệ thống AI/ML.

### 1.2 Phạm vi

| Thành phần              | Bao gồm | Ghi chú                                            |
| ---------------------- | -------- | ------------------------------------------------ |
| OpenClaw Agent Runtime | Có      | Thực thi agent cốt lõi, lệnh gọi công cụ, phiên       |
| Gateway                | Có      | Xác thực, định tuyến, tích hợp kênh     |
| Tích hợp Kênh   | Có      | WhatsApp, Telegram, Discord, Signal, Slack, v.v. |
| Thị trường ClawHub    | Có      | Xuất bản kỹ năng, kiểm duyệt, phân phối       |
| MCP Servers            | Có      | Nhà cung cấp công cụ bên ngoài                          |
| Thiết bị Người dùng           | Một phần  | Ứng dụng di động, ứng dụng máy tính để bàn                     |

### 1.3 Ngoài phạm vi

Không có gì được loại trừ rõ ràng khỏi mô hình đe dọa này.

---
## 2. Kiến trúc hệ thống

### 2.1 Ranh giới tin cậy

```
┌─────────────────────────────────────────────────────────────────┐
│                    UNTRUSTED ZONE                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 1: Channel Access                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Device Pairing (30s grace period)                      │   │
│  │  • AllowFrom / AllowList validation                       │   │
│  │  • Token/Password/Tailscale auth                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 2: Session Isolation              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT SESSIONS                          │   │
│  │  • Session key = agent:channel:peer                       │   │
│  │  • Tool policies per agent                                │   │
│  │  • Transcript logging                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 3: Tool Execution                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  EXECUTION SANDBOX                        │   │
│  │  • Docker sandbox OR Host (exec-approvals)                │   │
│  │  • Node remote execution                                  │   │
│  │  • SSRF protection (DNS pinning + IP blocking)            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 4: External Content               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              FETCHED URLs / EMAILS / WEBHOOKS             │   │
│  │  • External content wrapping (XML tags)                   │   │
│  │  • Security notice injection                              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 TRUST BOUNDARY 5: Supply Chain                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Skill publishing (semver, SKILL.md required)           │   │
│  │  • Pattern-based moderation flags                         │   │
│  │  • VirusTotal scanning (coming soon)                      │   │
│  │  • GitHub account age verification                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```
### 2.2 Luồng dữ liệu

| Luồng | Nguồn   | Đích        | Dữ liệu            | Bảo vệ               |
| ----- | ------- | ----------- | ------------------ | -------------------- |
| F1    | Kênh    | Gateway     | Tin nhắn người dùng | TLS, AllowFrom       |
| F2    | Gateway | Agent       | Tin nhắn được định tuyến | Cách ly phiên    |
| F3    | Agent   | Công cụ     | Gọi công cụ        | Thực thi chính sách  |
| F4    | Agent   | Bên ngoài   | Yêu cầu web_fetch  | Chặn SSRF           |
| F5    | ClawHub | Agent       | Mã Skill           | Kiểm duyệt, quét     |
| F6    | Agent   | Kênh        | Phản hồi            | Lọc đầu ra          |

---
## 3. Phân tích Mối đe dọa theo Chiến thuật ATLAS

### 3.1 Khám phá thiết bị (AML.TA0002)

#### T-RECON-001: Khám phá Điểm cuối Agent

| Attribute               | Value                                                                |
| ----------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0006 - Quét chủ động                                            |
| **Description**         | Kẻ tấn công quét các điểm cuối Gateway OpenClaw bị lộ               |
| **Attack Vector**       | Quét mạng, truy vấn shodan, liệt kê DNS                              |
| **Affected Components** | Gateway, các điểm cuối API bị lộ                                     |
| **Current Mitigations** | Tùy chọn xác thực Tailscale, liên kết với local loopback theo mặc định |
| **Residual Risk**       | Trung bình - Gateway công khai có thể khám phá được                  |
| **Recommendations**     | Ghi chép triển khai an toàn, thêm giới hạn tốc độ trên các điểm cuối khám phá |

#### T-RECON-002: Thăm dò Tích hợp Kênh

| Attribute               | Value                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0006 - Quét chủ động                                          |
| **Description**         | Kẻ tấn công thăm dò các kênh nhắn tin để xác định tài khoản do AI quản lý |
| **Attack Vector**       | Gửi tin nhắn kiểm tra, quan sát các mẫu phản hồi                   |
| **Affected Components** | Tất cả các tích hợp kênh                                            |
| **Current Mitigations** | Không có biện pháp cụ thể                                           |
| **Residual Risk**       | Thấp - Giá trị hạn chế từ khám phá một mình                         |
| **Recommendations**     | Xem xét ngẫu nhiên hóa thời gian phản hồi                           |

---

### 3.2 Truy cập Ban đầu (AML.TA0004)

#### T-ACCESS-001: Chặn Mã Ghép nối

| Attribute               | Value                                                    |
| ----------------------- | -------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - Truy cập API Suy luận Mô hình AI             |
| **Description**         | Kẻ tấn công chặn mã ghép nối trong thời gian ân hạn 30 giây |
| **Attack Vector**       | Nhìn qua vai, nghe lén mạng, kỹ thuật xã hội             |
| **Affected Components** | Hệ thống ghép nối thiết bị                               |
| **Current Mitigations** | Hết hạn 30 giây, mã được gửi qua kênh hiện có            |
| **Residual Risk**       | Trung bình - Thời gian ân hạn có thể bị khai thác        |
| **Recommendations**     | Giảm thời gian ân hạn, thêm bước xác nhận                |

#### T-ACCESS-002: Giả mạo AllowFrom

| Attribute               | Value                                                                          |
| ----------------------- | ------------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0040 - Truy cập API Suy luận Mô hình AI                                   |
| **Description**         | Kẻ tấn công giả mạo danh tính người gửi được phép trong kênh                   |
| **Attack Vector**       | Tùy thuộc vào kênh - giả mạo số điện thoại, mạo danh tên người dùng            |
| **Affected Components** | Xác thực AllowFrom cho mỗi kênh                                                |
| **Current Mitigations** | Xác minh danh tính cụ thể cho từng kênh                                        |
| **Residual Risk**       | Trung bình - Một số kênh dễ bị giả mạo                                         |
| **Recommendations**     | Ghi chép các rủi ro cụ thể cho từng kênh, thêm xác minh mật mã nơi có thể      |

#### T-ACCESS-003: Đánh cắp Token

| Attribute               | Value                                                       |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - Truy cập API Suy luận Mô hình AI                 |
| **Description**         | Kẻ tấn công đánh cắp mã thông báo xác thực từ tệp cấu hình   |
| **Attack Vector**       | Phần mềm độc hại, truy cập thiết bị trái phép, tiếp xúc sao lưu cấu hình |
| **Affected Components** | ~/.openclaw/credentials/, lưu trữ cấu hình                   |
| **Current Mitigations** | Quyền tệp                                                    |
| **Residual Risk**       | Cao - Mã thông báo được lưu trữ dưới dạng văn bản thuần      |
| **Recommendations**     | Triển khai mã hóa mã thông báo khi lưu trữ, thêm xoay vòng mã thông báo |

---

### 3.3 Thực thi (AML.TA0005)

#### T-EXEC-001: Tiêm Prompt Trực tiếp

| Attribute               | Value                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.000 - Tiêm Prompt LLM: Trực tiếp                                                |
| **Description**         | Kẻ tấn công gửi các prompt được tạo để thao túng hành vi agent                            |
| **Attack Vector**       | Tin nhắn kênh chứa các hướng dẫn đối kháng                                                |
| **Affected Components** | Agent LLM, tất cả các bề mặt đầu vào                                                      |
| **Current Mitigations** | Phát hiện mẫu, bao bọc nội dung bên ngoài                                                 |
| **Residual Risk**       | Nghiêm trọng - Chỉ phát hiện, không chặn; các cuộc tấn công tinh vi vượt qua             |
| **Recommendations**     | Triển khai phòng thủ đa lớp, xác thực đầu ra, xác nhận người dùng cho các hành động nhạy cảm |

#### T-EXEC-002: Tiêm Prompt Gián tiếp

| Attribute               | Value                                                       |
| ----------------------- | ----------------------------------------------------------- |
| **ATLAS ID**            | AML.T0051.001 - Tiêm Prompt LLM: Gián tiếp                  |
| **Description**         | Kẻ tấn công nhúng các hướng dẫn độc hại trong nội dung được tìm nạp |
| **Attack Vector**       | URL độc hại, email bị nhiễm độc, webhook bị xâm phạm         |
| **Affected Components** | web_fetch, nhập email, nguồn dữ liệu bên ngoài              |
| **Current Mitigations** | Bao bọc nội dung bằng thẻ XML và thông báo bảo mật           |
| **Residual Risk**       | Cao - LLM có thể bỏ qua hướng dẫn bao bọc                    |
| **Recommendations**     | Triển khai vệ sinh nội dung, ngữ cảnh thực thi riêng biệt    |

#### T-EXEC-003: Tiêm Đối số Công cụ

| Attribute               | Value                                                        |
| ----------------------- | ------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0051.000 - Tiêm Prompt LLM: Trực tiếp                   |
| **Description**         | Kẻ tấn công thao túng các đối số công cụ thông qua tiêm prompt |
| **Attack Vector**       | Các prompt được tạo ảnh hưởng đến giá trị tham số công cụ    |
| **Affected Components** | Tất cả các lệnh gọi công cụ                                  |
| **Current Mitigations** | Phê duyệt exec cho các lệnh nguy hiểm                        |
| **Residual Risk**       | Cao - Dựa vào phán đoán của người dùng                       |
| **Recommendations**     | Triển khai xác thực đối số, lệnh gọi công cụ được tham số hóa |

#### T-EXEC-004: Vượt qua Phê duyệt Exec

| Attribute               | Value                                                      |
| ----------------------- | ---------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Tạo Dữ liệu Đối kháng                          |
| **Description**         | Kẻ tấn công tạo các lệnh vượt qua danh sách cho phép phê duyệt |
| **Attack Vector**       | Che khuất lệnh, khai thác bí danh, thao túng đường dẫn      |
| **Affected Components** | exec-approvals.ts, danh sách cho phép lệnh                 |
| **Current Mitigations** | Danh sách cho phép + chế độ hỏi                             |
| **Residual Risk**       | Cao - Không vệ sinh lệnh                                    |
| **Recommendations**     | Triển khai chuẩn hóa lệnh, mở rộng danh sách chặn          |
### 3.4 Tính bền vững (AML.TA0006)

#### T-PERSIST-001: Cài đặt Skill độc hại

| Thuộc tính              | Giá trị                                                                  |
| ----------------------- | ------------------------------------------------------------------------ |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software                     |
| **Mô tả**               | Kẻ tấn công xuất bản skill độc hại lên ClawHub                           |
| **Vectơ tấn công**      | Tạo tài khoản, xuất bản skill với mã độc hại ẩn                          |
| **Thành phần bị ảnh hưởng** | ClawHub, tải skill, thực thi agent                                   |
| **Biện pháp giảm thiểu hiện tại** | Xác minh tuổi tài khoản GitHub, cờ điều độc lập dựa trên mẫu |
| **Rủi ro còn lại**      | Nghiêm trọng - Không có sandboxing, xem xét hạn chế                     |
| **Khuyến nghị**         | Tích hợp VirusTotal (đang tiến hành), sandboxing skill, xem xét cộng đồng |

#### T-PERSIST-002: Skill Update Poisoning

| Thuộc tính              | Giá trị                                                    |
| ----------------------- | -------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.001 - Supply Chain Compromise: AI Software           |
| **Mô tả**               | Kẻ tấn công xâm phạm skill phổ biến và đẩy bản cập nhật độc hại |
| **Vectơ tấn công**      | Xâm phạm tài khoản, kỹ thuật xã hội của chủ sở hữu skill       |
| **Thành phần bị ảnh hưởng** | Phiên bản ClawHub, luồng tự động cập nhật                  |
| **Biện pháp giảm thiểu hiện tại** | Dấu vân tay phiên bản                                 |
| **Rủi ro còn lại**      | Cao - Tự động cập nhật có thể kéo các phiên bản độc hại        |
| **Khuyến nghị**         | Triển khai ký cập nhật, khả năng quay lại, ghim phiên bản      |

#### T-PERSIST-003: Agent Configuration Tampering

| Thuộc tính              | Giá trị                                                           |
| ----------------------- | --------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0010.002 - Supply Chain Compromise: Data                   |
| **Mô tả**               | Kẻ tấn công sửa đổi cấu hình agent để duy trì quyền truy cập    |
| **Vectơ tấn công**      | Sửa đổi tệp cấu hình, tiêm cài đặt                              |
| **Thành phần bị ảnh hưởng** | Cấu hình agent, chính sách công cụ                          |
| **Biện pháp giảm thiểu hiện tại** | Quyền tệp                                            |
| **Rủi ro còn lại**      | Trung bình - Yêu cầu quyền truy cập cục bộ                      |
| **Khuyến nghị**         | Xác minh tính toàn vẹn cấu hình, ghi nhật ký kiểm toán cho các thay đổi cấu hình |

---

### 3.5 Tránh phòng thủ (AML.TA0007)

#### T-EVADE-001: Moderation Pattern Bypass

| Thuộc tính              | Giá trị                                                                  |
| ----------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Craft Adversarial Data                                     |
| **Mô tả**               | Kẻ tấn công tạo nội dung skill để tránh các mẫu điều độc                |
| **Vectơ tấn công**      | Unicode homoglyphs, mẹo mã hóa, tải động                               |
| **Thành phần bị ảnh hưởng** | ClawHub moderation.ts                                              |
| **Biện pháp giảm thiểu hiện tại** | FLAG_RULES dựa trên mẫu                                    |
| **Rủi ro còn lại**      | Cao - Regex đơn giản dễ bị vượt qua                                     |
| **Khuyến nghị**         | Thêm phân tích hành vi (VirusTotal Code Insight), phát hiện dựa trên AST |

#### T-EVADE-002: Content Wrapper Escape

| Thuộc tính              | Giá trị                                                     |
| ----------------------- | --------------------------------------------------------- |
| **ATLAS ID**            | AML.T0043 - Tạo dữ liệu đối kháng                      |
| **Description**         | Kẻ tấn công tạo nội dung thoát khỏi ngữ cảnh bao XML  |
| **Attack Vector**       | Thao tác thẻ, nhầm lẫn ngữ cảnh, ghi đè hướng dẫn     |
| **Affected Components** | Bao bọc nội dung bên ngoài                             |
| **Current Mitigations** | Thẻ XML + thông báo bảo mật                            |
| **Residual Risk**       | Trung bình - Các cách thoát mới được phát hiện thường xuyên |
| **Recommendations**     | Nhiều lớp bao bọc, xác thực phía đầu ra               |

---

### 3.6 Khám phá thiết bị (AML.TA0008)

#### T-DISC-001: Liệt kê công cụ

| Attribute               | Value                                                 |
| ----------------------- | ----------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - Truy cập API suy luận mô hình AI          |
| **Description**         | Kẻ tấn công liệt kê các công cụ có sẵn thông qua nhắc nhở |
| **Attack Vector**       | Các truy vấn kiểu "Bạn có những công cụ nào?"        |
| **Affected Components** | Sổ đăng ký công cụ agent                              |
| **Current Mitigations** | Không có cụ thể                                       |
| **Residual Risk**       | Thấp - Công cụ thường được ghi chép                   |
| **Recommendations**     | Xem xét các điều khiển khả năng hiển thị công cụ      |

#### T-DISC-002: Trích xuất dữ liệu phiên

| Attribute               | Value                                                 |
| ----------------------- | ----------------------------------------------------- |
| **ATLAS ID**            | AML.T0040 - Truy cập API suy luận mô hình AI          |
| **Description**         | Kẻ tấn công trích xuất dữ liệu nhạy cảm từ ngữ cảnh phiên |
| **Attack Vector**       | Các truy vấn "Chúng ta đã thảo luận gì?", thăm dò ngữ cảnh |
| **Affected Components** | Bảng ghi chép phiên, cửa sổ ngữ cảnh                  |
| **Current Mitigations** | Cách ly phiên theo người gửi                          |
| **Residual Risk**       | Trung bình - Dữ liệu trong phiên có thể truy cập      |
| **Recommendations**     | Triển khai biên tập dữ liệu nhạy cảm trong ngữ cảnh   |

---

### 3.7 Thu thập & Rò rỉ dữ liệu (AML.TA0009, AML.TA0010)

#### T-EXFIL-001: Đánh cắp dữ liệu qua web_fetch

| Attribute               | Value                                                                  |
| ----------------------- | ---------------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Thu thập                                                   |
| **Description**         | Kẻ tấn công rò rỉ dữ liệu bằng cách hướng dẫn agent gửi đến URL bên ngoài |
| **Attack Vector**       | Tiêm nhắc nhở khiến agent POST dữ liệu đến máy chủ kẻ tấn công        |
| **Affected Components** | Công cụ web_fetch                                                      |
| **Current Mitigations** | Chặn SSRF cho mạng nội bộ                                              |
| **Residual Risk**       | Cao - URL bên ngoài được phép                                          |
| **Recommendations**     | Triển khai danh sách cho phép URL, nhận thức phân loại dữ liệu         |

#### T-EXFIL-002: Gửi tin nhắn trái phép

| Attribute               | Value                                                            |
| ----------------------- | ---------------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Thu thập                                             |
| **Description**         | Kẻ tấn công khiến agent gửi tin nhắn chứa dữ liệu nhạy cảm      |
| **Attack Vector**       | Tiêm nhắc nhở khiến agent nhắn tin cho kẻ tấn công              |
| **Affected Components** | Công cụ tin nhắn, tích hợp kênh                                  |
| **Biện pháp giảm thiểu hiện tại** | Gating tin nhắn gửi đi                                    |
| **Rủi ro còn lại**       | Trung bình - Gating có thể bị vượt qua                    |
| **Khuyến nghị**     | Yêu cầu xác nhận rõ ràng cho những người nhận mới         |

#### T-EXFIL-003: Thu thập thông tin xác thực

| Thuộc tính               | Giá trị                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0009 - Collection                                  |
| **Mô tả**         | Skill độc hại thu thập thông tin xác thực từ ngữ cảnh agent |
| **Vectơ tấn công**       | Mã Skill đọc biến môi trường, tệp cấu hình    |
| **Thành phần bị ảnh hưởng** | Môi trường thực thi Skill                             |
| **Biện pháp giảm thiểu hiện tại** | Không có biện pháp cụ thể cho Skills                                 |
| **Rủi ro còn lại**       | Nghiêm trọng - Skills chạy với quyền agent             |
| **Khuyến nghị**     | Sandboxing Skill, cách ly thông tin xác thực                  |

---

### 3.8 Tác động (AML.TA0011)

#### T-IMPACT-001: Thực thi lệnh trái phép

| Thuộc tính               | Giá trị                                               |
| ----------------------- | --------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity                |
| **Mô tả**         | Kẻ tấn công thực thi các lệnh tùy ý trên hệ thống người dùng |
| **Vectơ tấn công**       | Prompt injection kết hợp với vượt qua phê duyệt exec |
| **Thành phần bị ảnh hưởng** | Công cụ Bash, thực thi lệnh                        |
| **Biện pháp giảm thiểu hiện tại** | Phê duyệt exec, tùy chọn sandbox Docker               |
| **Rủi ro còn lại**       | Nghiêm trọng - Thực thi trên host mà không sandbox           |
| **Khuyến nghị**     | Mặc định sandbox, cải thiện UX phê duyệt             |

#### T-IMPACT-002: Cạn kiệt tài nguyên (DoS)

| Thuộc tính               | Giá trị                                              |
| ----------------------- | -------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity               |
| **Mô tả**         | Kẻ tấn công cạn kiệt tín dụng API hoặc tài nguyên tính toán |
| **Vectơ tấn công**       | Tấn công lũ tin nhắn tự động, gọi công cụ tốn kém   |
| **Thành phần bị ảnh hưởng** | Gateway, phiên agent, nhà cung cấp API              |
| **Biện pháp giảm thiểu hiện tại** | Không có                                               |
| **Rủi ro còn lại**       | Cao - Không có giới hạn tốc độ                            |
| **Khuyến nghị**     | Triển khai giới hạn tốc độ theo người gửi, ngân sách chi phí     |

#### T-IMPACT-003: Thiệt hại danh tiếng

| Thuộc tính               | Giá trị                                                   |
| ----------------------- | ------------------------------------------------------- |
| **ATLAS ID**            | AML.T0031 - Erode AI Model Integrity                    |
| **Mô tả**         | Kẻ tấn công khiến agent gửi nội dung có hại/x冒phạm |
| **Vectơ tấn công**       | Prompt injection gây ra các phản hồi không phù hợp        |
| **Thành phần bị ảnh hưởng** | Tạo đầu ra, nhắn tin kênh                    |
| **Biện pháp giảm thiểu hiện tại** | Chính sách nội dung của nhà cung cấp LLM                           |
| **Rủi ro còn lại**       | Trung bình - Bộ lọc nhà cung cấp không hoàn hảo                     |
| **Khuyến nghị**     | Lớp lọc đầu ra, kiểm soát người dùng                   |

---
## 4. Phân tích Chuỗi Cung cấp ClawHub

### 4.1 Các Biện pháp Kiểm soát Bảo mật Hiện tại

| Biện pháp            | Triển khai                  | Hiệu quả                                             |
| -------------------- | --------------------------- | ---------------------------------------------------- |
| Tuổi Tài khoản GitHub | `requireGitHubAccountAge()` | Trung bình - Nâng cao rào cản cho những kẻ tấn công mới |
| Vệ sinh Đường dẫn    | `sanitizePath()`            | Cao - Ngăn chặn path traversal                       |
| Xác thực Loại Tệp    | `isTextFile()`              | Trung bình - Chỉ tệp văn bản, nhưng vẫn có thể độc hại |
| Giới hạn Kích thước  | 50MB tổng bundle           | Cao - Ngăn chặn cạn kiệt tài nguyên                  |
| SKILL.md Bắt buộc    | Readme bắt buộc             | Giá trị bảo mật thấp - Chỉ mang tính thông tin       |
| Kiểm duyệt Mẫu       | FLAG_RULES trong moderation.ts | Thấp - Dễ dàng bỏ qua                                |
| Trạng thái Kiểm duyệt | `moderationStatus` trường    | Trung bình - Có thể xem xét thủ công                 |

### 4.2 Mẫu Cờ Kiểm duyệt

Các mẫu hiện tại trong `moderation.ts`:

```javascript
// Known-bad identifiers
/(keepcold131\/ClawdAuthenticatorTool|ClawdAuthenticatorTool)/i

// Suspicious keywords
/(malware|stealer|phish|phishing|keylogger)/i
/(api[-_ ]?key|token|password|private key|secret)/i
/(wallet|seed phrase|mnemonic|crypto)/i
/(discord\.gg|webhook|hooks\.slack)/i
/(curl[^\n]+\|\s*(sh|bash))/i
/(bit\.ly|tinyurl\.com|t\.co|goo\.gl|is\.gd)/i
```

**Limitations:**

- Only checks slug, displayName, summary, frontmatter, metadata, file paths
- Does not analyze actual skill code content
- Simple regex easily bypassed with obfuscation
- No behavioral analysis

### 4.3 Planned Improvements

| Improvement            | Status                                | Impact                                                                |
| ---------------------- | ------------------------------------- | --------------------------------------------------------------------- |
| VirusTotal Integration | In Progress                           | High - Code Insight behavioral analysis                               |
| Community Reporting    | Partial (`skillReports` table exists) | Medium                                                                |
| Audit Logging          | Partial (`auditLogs` table exists)    | Medium                                                                |
| Badge System           | Implemented                           | Medium - `highlighted`, `official`, `deprecated`, `redactionApproved` |

---
## 5. Ma trận Rủi ro

### 5.1 Khả năng xảy ra so với Tác động

| ID Mối đe dọa | Khả năng xảy ra | Tác động  | Mức độ Rủi ro | Ưu tiên  |
| ------------- | --------------- | --------- | ------------ | -------- |
| T-EXEC-001    | Cao             | Nghiêm trọng | **Nghiêm trọng** | P0       |
| T-PERSIST-001 | Cao             | Nghiêm trọng | **Nghiêm trọng** | P0       |
| T-EXFIL-003   | Trung bình       | Nghiêm trọng | **Nghiêm trọng** | P0       |
| T-IMPACT-001  | Trung bình       | Nghiêm trọng | **Cao**      | P1       |
| T-EXEC-002    | Cao             | Cao       | **Cao**      | P1       |
| T-EXEC-004    | Trung bình       | Cao       | **Cao**      | P1       |
| T-ACCESS-003  | Trung bình       | Cao       | **Cao**      | P1       |
| T-EXFIL-001   | Trung bình       | Cao       | **Cao**      | P1       |
| T-IMPACT-002  | Cao             | Trung bình | **Cao**      | P1       |
| T-EVADE-001   | Cao             | Trung bình | **Trung bình** | P2       |
| T-ACCESS-001  | Thấp            | Cao       | **Trung bình** | P2       |
| T-ACCESS-002  | Thấp            | Cao       | **Trung bình** | P2       |
| T-PERSIST-002 | Thấp            | Cao       | **Trung bình** | P2       |

### 5.2 Chuỗi Tấn công Đường dẫn Quan trọng

**Chuỗi Tấn công 1: Đánh cắp Dữ liệu dựa trên Skill**

```
T-PERSIST-001 → T-EVADE-001 → T-EXFIL-003
(Publish malicious skill) → (Evade moderation) → (Harvest credentials)
```

**Attack Chain 2: Prompt Injection to RCE**

```
T-EXEC-001 → T-EXEC-004 → T-IMPACT-001
(Inject prompt) → (Bypass exec approval) → (Execute commands)
```

**Attack Chain 3: Indirect Injection via Fetched Content**

```
T-EXEC-002 → T-EXFIL-001 → External exfiltration
(Poison URL content) → (Agent fetches & follows instructions) → (Data sent to attacker)
```

---
## 6. Tóm tắt Khuyến nghị

### 6.1 Ngay lập tức (P0)

| ID    | Khuyến nghị                                    | Giải quyết                 |
| ----- | ---------------------------------------------- | -------------------------- |
| R-001 | Hoàn thành tích hợp VirusTotal                 | T-PERSIST-001, T-EVADE-001 |
| R-002 | Triển khai sandboxing cho Skills               | T-PERSIST-001, T-EXFIL-003 |
| R-003 | Thêm xác thực đầu ra cho các hành động nhạy cảm | T-EXEC-001, T-EXEC-002     |

### 6.2 Ngắn hạn (P1)

| ID    | Khuyến nghị                                  | Giải quyết   |
| ----- | -------------------------------------------- | ------------ |
| R-004 | Triển khai giới hạn tốc độ                   | T-IMPACT-002 |
| R-005 | Thêm mã hóa token khi lưu trữ                | T-ACCESS-003 |
| R-006 | Cải thiện UX và xác thực phê duyệt exec      | T-EXEC-004   |
| R-007 | Triển khai danh sách cho phép URL cho web_fetch | T-EXFIL-001  |

### 6.3 Trung hạn (P2)

| ID    | Khuyến nghị                                              | Giải quyết    |
| ----- | -------------------------------------------------------- | ------------- |
| R-008 | Thêm xác minh kênh mã hóa nơi có thể                    | T-ACCESS-002  |
| R-009 | Triển khai xác minh tính toàn vẹn cấu hình              | T-PERSIST-003 |
| R-010 | Thêm ký tên cập nhật và ghim phiên bản                  | T-PERSIST-002 |

---
## 7. Phụ lục

### 7.1 Ánh xạ Kỹ thuật ATLAS

| ATLAS ID      | Tên Kỹ thuật                   | Mối đe dọa OpenClaw                                              |
| ------------- | ------------------------------ | ---------------------------------------------------------------- |
| AML.T0006     | Quét tích cực                  | T-RECON-001, T-RECON-002                                         |
| AML.T0009     | Thu thập dữ liệu               | T-EXFIL-001, T-EXFIL-002, T-EXFIL-003                            |
| AML.T0010.001 | Chuỗi cung ứng: Phần mềm AI    | T-PERSIST-001, T-PERSIST-002                                     |
| AML.T0010.002 | Chuỗi cung ứng: Dữ liệu        | T-PERSIST-003                                                    |
| AML.T0031     | Xói mòn tính toàn vẹn mô hình AI | T-IMPACT-001, T-IMPACT-002, T-IMPACT-003                         |
| AML.T0040     | Truy cập API suy luận mô hình AI | T-ACCESS-001, T-ACCESS-002, T-ACCESS-003, T-DISC-001, T-DISC-002 |
| AML.T0043     | Tạo dữ liệu đối kháng           | T-EXEC-004, T-EVADE-001, T-EVADE-002                             |
| AML.T0051.000 | Tiêm nhắc LLM: Trực tiếp       | T-EXEC-001, T-EXEC-003                                           |
| AML.T0051.001 | Tiêm nhắc LLM: Gián tiếp       | T-EXEC-002                                                       |

### 7.2 Các tệp bảo mật chính

| Đường dẫn                           | Mục đích                        | Mức độ rủi ro |
| ----------------------------------- | --------------------------- | ------------ |
| `src/infra/exec-approvals.ts`       | Logic phê duyệt lệnh        | **Quan trọng** |
| `src/gateway/auth.ts`               | Xác thực Gateway            | **Quan trọng** |
| `src/web/inbound/access-control.ts` | Kiểm soát truy cập kênh     | **Quan trọng** |
| `src/infra/net/ssrf.ts`             | Bảo vệ SSRF                 | **Quan trọng** |
| `src/security/external-content.ts`  | Giảm thiểu tiêm nhắc        | **Quan trọng** |
| `src/agents/sandbox/tool-policy.ts` | Thực thi chính sách công cụ | **Quan trọng** |
| `convex/lib/moderation.ts`          | Kiểm duyệt ClawHub          | **Cao**      |
| `convex/lib/skillPublish.ts`        | Quy trình xuất bản Skill    | **Cao**      |
| `src/routing/resolve-route.ts`      | Cách ly phiên               | **Trung bình** |

### 7.3 Thuật ngữ

| Thuật ngữ            | Định nghĩa                                                |
| -------------------- | --------------------------------------------------------- |
| **ATLAS**            | Cảnh quan mối đe dọa đối kháng của MITRE cho các hệ thống AI |
| **ClawHub**          | Thị trường Skill của OpenClaw                             |
| **Gateway**          | Lớp định tuyến tin nhắn và xác thực của OpenClaw          |
| **MCP**              | Model Context Protocol - giao diện nhà cung cấp công cụ  |
| **Prompt Injection** | Cuộc tấn công nhúng các hướng dẫn độc hại vào đầu vào    |
| **Skill**            | Tiện ích mở rộng có thể tải xuống cho agent OpenClaw     |
| **SSRF**             | Server-Side Request Forgery                               |

---

_Mô hình mối đe dọa này là một tài liệu sống. Báo cáo các vấn đề bảo mật cho security@openclaw.ai_