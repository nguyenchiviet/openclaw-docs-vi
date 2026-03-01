---
x-i18n:
  source_path: security\CONTRIBUTING-THREAT-MODEL.md
  source_hash: fd7c528984d1ca5a6ece83d683210d775eca9cd3cc1dc7eadba5516b4adfa854
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:23:01.997Z'
---

# Đóng góp vào Mô hình Đe dọa OpenClaw

Cảm ơn bạn đã giúp làm cho OpenClaw an toàn hơn. Mô hình đe dọa này là một tài liệu sống động và chúng tôi hoan nghênh những đóng góp từ bất kỳ ai - bạn không cần phải là một chuyên gia bảo mật.
## Cách Đóng Góp

### Thêm Một Mối Đe Dọa

Phát hiện một vector tấn công hoặc rủi ro mà chúng tôi chưa đề cập? Mở một issue trên [openclaw/trust](https://github.com/openclaw/trust/issues) và mô tả nó bằng lời của bạn. Bạn không cần phải biết bất kỳ framework nào hoặc điền vào mọi trường - chỉ cần mô tả tình huống.

**Hữu ích để bao gồm (nhưng không bắt buộc):**

- Kịch bản tấn công và cách nó có thể bị khai thác
- Những phần nào của OpenClaw bị ảnh hưởng (CLI, gateway, channels, ClawHub, MCP servers, v.v.)
- Mức độ nghiêm trọng mà bạn nghĩ (thấp / trung bình / cao / nghiêm trọng)
- Bất kỳ liên kết nào đến nghiên cứu liên quan, CVE hoặc ví dụ thực tế

Chúng tôi sẽ xử lý ánh xạ ATLAS, ID mối đe dọa và đánh giá rủi ro trong quá trình xem xét. Nếu bạn muốn bao gồm những chi tiết đó, tuyệt vời - nhưng không bắt buộc.

> **Đây là để thêm vào mô hình mối đe dọa, không phải để báo cáo các lỗ hổng đang hoạt động.** Nếu bạn đã tìm thấy một lỗ hổng có thể khai thác, hãy xem [trang Trust](https://trust.openclaw.ai) của chúng tôi để biết hướng dẫn công khai trách nhiệm.

### Đề Xuất Một Biện Pháp Giảm Thiểu

Có ý tưởng về cách giải quyết một mối đe dọa hiện có? Mở một issue hoặc PR tham chiếu đến mối đe dọa. Các biện pháp giảm thiểu hữu ích là cụ thể và có thể thực hiện được - ví dụ, "giới hạn tốc độ trên mỗi người gửi là 10 tin nhắn/phút tại gateway" tốt hơn "triển khai giới hạn tốc độ."

### Đề Xuất Một Chuỗi Tấn Công

Chuỗi tấn công cho thấy cách nhiều mối đe dọa kết hợp thành một kịch bản tấn công thực tế. Nếu bạn thấy một sự kết hợp nguy hiểm, hãy mô tả các bước và cách kẻ tấn công sẽ kết nối chúng lại với nhau. Một câu chuyện ngắn về cách cuộc tấn công diễn ra trong thực tế có giá trị hơn một mẫu chính thức.

### Sửa Chữa hoặc Cải Thiện Nội Dung Hiện Có

Lỗi chính tả, làm rõ, thông tin lỗi thời, ví dụ tốt hơn - PR được chào đón, không cần issue.
## Những Gì Chúng Tôi Sử Dụng

### MITRE ATLAS

Mô hình mối đe dọa này được xây dựng dựa trên [MITRE ATLAS](https://atlas.mitre.org/) (Adversarial Threat Landscape for AI Systems), một framework được thiết kế đặc biệt cho các mối đe dọa AI/ML như prompt injection, tool misuse và agent exploitation. Bạn không cần phải biết về ATLAS để đóng góp - chúng tôi sẽ ánh xạ các bài nộp vào framework trong quá trình xem xét.

### ID Mối Đe Dọa

Mỗi mối đe dọa nhận một ID như `T-EXEC-003`. Các danh mục là:

| Mã      | Danh Mục                                   |
| ------- | ------------------------------------------ |
| RECON   | Reconnaissance - thu thập thông tin        |
| ACCESS  | Initial access - đạt được quyền truy cập  |
| EXEC    | Execution - thực thi các hành động độc hại |
| PERSIST | Persistence - duy trì quyền truy cập      |
| EVADE   | Defense evasion - tránh phát hiện          |
| DISC    | Discovery - tìm hiểu về môi trường         |
| EXFIL   | Exfiltration - đánh cắp dữ liệu            |
| IMPACT  | Impact - gây thiệt hại hoặc gián đoạn     |

ID được gán bởi những người bảo trì trong quá trình xem xét. Bạn không cần phải chọn một cái.

### Mức Độ Rủi Ro

| Mức Độ       | Ý Nghĩa                                                           |
| ------------ | ----------------------------------------------------------------- |
| **Critical** | Thỏa hiệp toàn bộ hệ thống, hoặc khả năng cao + tác động nghiêm trọng |
| **High**     | Thiệt hại đáng kể có khả năng xảy ra, hoặc khả năng trung bình + tác động nghiêm trọng |
| **Medium**   | Rủi ro vừa phải, hoặc khả năng thấp + tác động cao               |
| **Low**      | Không có khả năng xảy ra và tác động hạn chế                     |

Nếu bạn không chắc chắn về mức độ rủi ro, chỉ cần mô tả tác động và chúng tôi sẽ đánh giá nó.
## Quy trình Xem xét

1. **Phân loại** - Chúng tôi xem xét các bài nộp mới trong vòng 48 giờ
2. **Đánh giá** - Chúng tôi xác minh khả năng thực hiện, gán ánh xạ ATLAS và ID mối đe dọa, xác thực mức độ rủi ro
3. **Tài liệu** - Chúng tôi đảm bảo mọi thứ được định dạng và hoàn chỉnh
4. **Hợp nhất** - Được thêm vào mô hình mối đe dọa và trực quan hóa
## Tài nguyên

- [Trang web ATLAS](https://atlas.mitre.org/)
- [Kỹ thuật ATLAS](https://atlas.mitre.org/techniques/)
- [Các nghiên cứu trường hợp ATLAS](https://atlas.mitre.org/studies/)
- [Mô hình Mối đe dọa OpenClaw](./THREAT-MODEL-ATLAS.md)
## Liên hệ

- **Lỗ hổng bảo mật:** Xem [trang Trust](https://trust.openclaw.ai) của chúng tôi để biết hướng dẫn báo cáo
- **Câu hỏi về mô hình mối đe dọa:** Mở một issue trên [openclaw/trust](https://github.com/openclaw/trust/issues)
- **Trò chuyện chung:** Kênh #security trên Discord
## Ghi nhận

Những người đóng góp cho mô hình đe dọa được ghi nhận trong phần ghi nhận mô hình đe dọa, ghi chú phát hành và bảng vinh danh bảo mật OpenClaw vì những đóng góp đáng kể.