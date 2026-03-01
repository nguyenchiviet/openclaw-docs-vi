---
summary: >-
  Kiểm tra những gì có thể chi tiêu tiền, những khóa nào được sử dụng, và cách
  xem mức sử dụng
read_when:
  - Bạn muốn hiểu những tính năng nào có thể gọi các API trả phí
  - 'Bạn cần kiểm toán khóa, chi phí và khả năng hiển thị sử dụng'
  - Bạn đang giải thích báo cáo chi phí /status hoặc /usage
title: Sử dụng API và Chi phí
x-i18n:
  source_path: reference\api-usage-costs.md
  source_hash: 3bcefd373326fb77128c162981a04d51b8a4ca85e84833c929ca06e2d6a8f1c5
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:19:38.880Z'
---

# Sử dụng API & chi phí

Tài liệu này liệt kê **các tính năng có thể gọi các khóa API** và nơi chi phí của chúng xuất hiện. Nó tập trung vào
các tính năng OpenClaw có thể tạo ra sử dụng nhà cung cấp hoặc các lệnh gọi API trả phí.
## Nơi chi phí xuất hiện (chat + CLI)

**Ảnh chụp chi phí mỗi phiên**

- `/status` hiển thị mô hình phiên hiện tại, mức sử dụng ngữ cảnh và token phản hồi cuối cùng.
- Nếu mô hình sử dụng **xác thực API-key**, `/status` cũng hiển thị **chi phí ước tính** cho phản hồi cuối cùng.

**Chân trang chi phí mỗi tin nhắn**

- `/usage full` thêm chân trang sử dụng vào mỗi phản hồi, bao gồm **chi phí ước tính** (chỉ API-key).
- `/usage tokens` hiển thị token chỉ; các luồng OAuth ẩn chi phí theo đô la.

**Cửa sổ sử dụng CLI (hạn ngạch nhà cung cấp)**

- `openclaw status --usage` và `openclaw channels list` hiển thị **cửa sổ sử dụng** nhà cung cấp
  (ảnh chụp hạn ngạch, không phải chi phí mỗi tin nhắn).

Xem [Sử dụng token & chi phí](/reference/token-use) để biết chi tiết và ví dụ.
## Cách khóa được khám phá

OpenClaw có thể nhận thông tin xác thực từ:

- **Auth profiles** (theo agent, được lưu trữ trong `auth-profiles.json`).
- **Biến môi trường** (ví dụ: `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
- **Config** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
- **Skills** (`skills.entries.<name>.apiKey`) có thể xuất khóa sang biến môi trường của quá trình skill.
## Các tính năng có thể sử dụng khóa

### 1) Phản hồi mô hình cốt lõi (chat + công cụ)

Mỗi phản hồi hoặc lệnh gọi công cụ sử dụng **nhà cung cấp mô hình hiện tại** (OpenAI, Anthropic, v.v.). Đây là nguồn chính của mức sử dụng và chi phí.

Xem [Models](/providers/models) để cấu hình giá và [Token use & costs](/reference/token-use) để hiển thị.

### 2) Hiểu biết về phương tiện (âm thanh/hình ảnh/video)

Phương tiện đến có thể được tóm tắt/chuyển đổi thành văn bản trước khi phản hồi chạy. Điều này sử dụng các API của mô hình/nhà cung cấp.

- Âm thanh: OpenAI / Groq / Deepgram (hiện **được bật tự động** khi khóa tồn tại).
- Hình ảnh: OpenAI / Anthropic / Google.
- Video: Google.

Xem [Media understanding](/nodes/media-understanding).

### 3) Nhúng bộ nhớ + tìm kiếm ngữ nghĩa

Tìm kiếm bộ nhớ ngữ nghĩa sử dụng **API nhúng** khi được cấu hình cho các nhà cung cấp từ xa:

- `memorySearch.provider = "openai"` → Nhúng OpenAI
- `memorySearch.provider = "gemini"` → Nhúng Gemini
- `memorySearch.provider = "voyage"` → Nhúng Voyage
- `memorySearch.provider = "mistral"` → Nhúng Mistral
- Tùy chọn quay lại nhà cung cấp từ xa nếu nhúng cục bộ không thành công

Bạn có thể giữ nó cục bộ với `memorySearch.provider = "local"` (không sử dụng API).

Xem [Memory](/concepts/memory).

### 4) Công cụ tìm kiếm web (Brave / Perplexity qua OpenRouter)

`web_search` sử dụng khóa API và có thể phát sinh chi phí sử dụng:

- **Brave Search API**: `BRAVE_API_KEY` hoặc `tools.web.search.apiKey`
- **Perplexity** (qua OpenRouter): `PERPLEXITY_API_KEY` hoặc `OPENROUTER_API_KEY`

**Brave miễn phí (rất hào phóng):**

- **2.000 yêu cầu/tháng**
- **1 yêu cầu/giây**
- **Thẻ tín dụng bắt buộc** để xác minh (không tính phí trừ khi bạn nâng cấp)

Xem [Web tools](/tools/web).

### 5) Công cụ tìm nạp web (Firecrawl)

`web_fetch` có thể gọi **Firecrawl** khi khóa API có mặt:

- `FIRECRAWL_API_KEY` hoặc `tools.web.fetch.firecrawl.apiKey`

Nếu Firecrawl không được cấu hình, công cụ sẽ quay lại tìm nạp trực tiếp + khả năng đọc (không có API trả phí).

Xem [Web tools](/tools/web).

### 6) Ảnh chụp nhanh mức sử dụng nhà cung cấp (trạng thái/sức khỏe)
Một số lệnh trạng thái gọi **các endpoint sử dụng nhà cung cấp** để hiển thị cửa sổ hạn ngạch hoặc tình trạng xác thực.
Đây thường là các lệnh gọi có khối lượng thấp nhưng vẫn truy cập các API của nhà cung cấp:

- `openclaw status --usage`
- `openclaw models status --json`

Xem [Models CLI](/cli/models).

### 7) Bảo vệ nén - tóm tắt

Bảo vệ nén có thể tóm tắt lịch sử phiên bằng **mô hình hiện tại**, điều này
gọi các API của nhà cung cấp khi nó chạy.

Xem [Session management + compaction](/reference/session-management-compaction).

### 8) Quét / kiểm tra mô hình

`openclaw models scan` có thể kiểm tra các mô hình OpenRouter và sử dụng `OPENROUTER_API_KEY` khi
bật tính năng kiểm tra.

Xem [Models CLI](/cli/models).

### 9) Talk (speech)

Chế độ Talk có thể gọi **ElevenLabs** khi được cấu hình:

- `ELEVENLABS_API_KEY` hoặc `talk.apiKey`

Xem [Talk mode](/nodes/talk).

### 10) Skills (API của bên thứ ba)

Skills có thể lưu trữ `apiKey` trong `skills.entries.<name>.apiKey`. Nếu một skill sử dụng khóa đó cho các
API bên ngoài, nó có thể phát sinh chi phí theo nhà cung cấp của skill.

Xem [Skills](/tools/skills).