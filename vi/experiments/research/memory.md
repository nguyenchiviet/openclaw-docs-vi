---
summary: >-
  Ghi chú nghiên cứu: hệ thống bộ nhớ ngoại tuyến cho các không gian làm việc
  Clawd (Markdown nguồn-sự-thật + chỉ mục dẫn xuất)
read_when:
  - >-
    Thiết kế bộ nhớ không gian làm việc (~/.openclaw/workspace) vượt ra ngoài
    các tệp Markdown hàng ngày
  - Deciding: standalone CLI vs deep OpenClaw integration
  - Thêm khả năng ghi nhớ ngoại tuyến + suy ngẫm (retain/recall/reflect)
title: Nghiên cứu Bộ nhớ Không gian làm việc
x-i18n:
  source_path: experiments\research\memory.md
  source_hash: 1753c8ee6284999fab4a94ff5fae7421c85233699c9d3088453d0c2133ac0feb
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:55:44.946Z'
---

# Workspace Memory v2 (offline): ghi chú nghiên cứu

Target: Không gian làm việc kiểu Clawd (`agents.defaults.workspace`, mặc định `~/.openclaw/workspace`) nơi "bộ nhớ" được lưu trữ dưới dạng một tệp Markdown mỗi ngày (`memory/YYYY-MM-DD.md`) cộng với một tập hợp nhỏ các tệp ổn định (ví dụ: `memory.md`, `SOUL.md`).

Tài liệu này đề xuất một kiến trúc bộ nhớ **offline-first** giữ Markdown làm nguồn sự thật chính tắc, có thể xem xét, nhưng thêm **ghi nhớ có cấu trúc** (tìm kiếm, tóm tắt thực thể, cập nhật độ tin cậy) thông qua một chỉ mục dẫn xuất.
## Tại sao thay đổi?

Thiết lập hiện tại (một tệp mỗi ngày) rất tốt cho:

- ghi nhật ký "chỉ nối thêm"
- chỉnh sửa thủ công
- độ bền + khả năng kiểm toán được hỗ trợ bởi git
- thu thập thông tin không ma sát ("chỉ cần viết nó xuống")

Nó yếu cho:

- truy xuất có độ nhớ cao ("chúng ta đã quyết định gì về X?", "lần cuối cùng chúng ta thử Y?")
- câu trả lời tập trung vào thực thể ("cho tôi biết về Alice / The Castle / warelay") mà không cần đọc lại nhiều tệp
- ổn định ý kiến/sở thích (và bằng chứng khi nó thay đổi)
- ràng buộc thời gian ("điều gì là đúng trong tháng 11 năm 2025?") và giải quyết xung đột
## Mục tiêu thiết kế

- **Offline**: hoạt động mà không cần mạng; có thể chạy trên laptop/Castle; không phụ thuộc vào cloud.
- **Có thể giải thích**: các mục được truy xuất phải có thể quy kết (tệp + vị trí) và tách biệt khỏi suy luận.
- **Ít nghi thức**: ghi nhật ký hàng ngày vẫn là Markdown, không cần công việc schema nặng nề.
- **Tăng dần**: v1 hữu ích với FTS duy nhất; semantic/vector và graphs là các nâng cấp tùy chọn.
- **Thân thiện với agent**: giúp "gọi lại trong ngân sách token" dễ dàng (trả về các gói sự kiện nhỏ).
## Mô hình hướng dẫn (Hindsight × Letta)

Hai phần cần kết hợp:

1. **Vòng lặp điều khiển kiểu Letta/MemGPT**

- giữ một "lõi" nhỏ luôn trong ngữ cảnh (nhân cách + các sự kiện chính của người dùng)
- mọi thứ khác nằm ngoài ngữ cảnh và được truy xuất thông qua các công cụ
- các ghi nhớ được thực hiện thông qua các lệnh gọi công cụ rõ ràng (append/replace/insert), được lưu trữ, sau đó được tiêm lại vào lượt tiếp theo

2. **Cơ chế lưu trữ bộ nhớ kiểu Hindsight**

- phân biệt giữa những gì được quan sát so với những gì được tin tưởng so với những gì được tóm tắt
- hỗ trợ retain/recall/reflect
- các ý kiến có mức độ tin cậy có thể phát triển theo bằng chứng
- truy xuất nhận thức về thực thể + truy vấn theo thời gian (ngay cả khi không có đầy đủ đồ thị kiến thức)
## Kiến trúc được đề xuất (Markdown là nguồn sự thật + chỉ mục dẫn xuất)

### Kho lưu trữ chính tắc (thân thiện với git)

Giữ `~/.openclaw/workspace` làm bộ nhớ chính tắc dễ đọc.

Bố cục không gian làm việc được đề xuất:

```
~/.openclaw/workspace/
  memory.md                    # small: durable facts + preferences (core-ish)
  memory/
    YYYY-MM-DD.md              # daily log (append; narrative)
  bank/                        # “typed” memory pages (stable, reviewable)
    world.md                   # objective facts about the world
    experience.md              # what the agent did (first-person)
    opinions.md                # subjective prefs/judgments + confidence + evidence pointers
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notes:

- **Daily log stays daily log**. No need to turn it into JSON.
- The `bank/` files are **curated**, produced by reflection jobs, and can still be edited by hand.
- `memory.md` remains “small + core-ish”: the things you want Clawd to see every session.

### Derived store (machine recall)

Add a derived index under the workspace (not necessarily git tracked):

```
~/.openclaw/workspace/.memory/index.sqlite
```

Hỗ trợ nó bằng:

- Lược đồ SQLite cho các sự kiện + liên kết thực thể + siêu dữ liệu ý kiến
- SQLite **FTS5** để truy xuất từ vựng (nhanh, nhỏ gọn, ngoại tuyến)
- bảng nhúng tùy chọn để truy xuất ngữ nghĩa (vẫn ngoại tuyến)

Chỉ mục luôn **có thể xây dựng lại từ Markdown**.
## Giữ lại / Nhớ lại / Phản ánh (vòng lặp hoạt động)

### Giữ lại: chuẩn hóa nhật ký hàng ngày thành "sự kiện"

Hiểu biết chính của Hindsight có ý nghĩa ở đây: lưu trữ **sự kiện tự chứa đựng, có tính chất tường thuật**, không phải những đoạn nhỏ.

Quy tắc thực tế cho `memory/YYYY-MM-DD.md`:

- vào cuối ngày (hoặc trong suốt ngày), thêm một phần `## Retain` với 2–5 dấu đầu dòng là:
  - tường thuật (bối cảnh xuyên suốt các lượt được bảo toàn)
  - tự chứa đựng (độc lập có ý nghĩa sau này)
  - được gắn thẻ với loại + đề cập thực thể

Ví dụ:

```
## Retain
- W @Peter: Currently in Marrakech (Nov 27–Dec 1, 2025) for Andy’s birthday.
- B @warelay: I fixed the Baileys WS crash by wrapping connection.update handlers in try/catch (see memory/2025-11-27.md).
- O(c=0.95) @Peter: Prefers concise replies (&lt;1500 chars) on WhatsApp; long content goes into files.
```

Minimal parsing:

- Type prefix: `W` (world), `B` (experience/biographical), `O` (opinion), `S` (observation/summary; usually generated)
- Entities: `@Peter`, `@warelay`, etc (slugs map to `bank/entities/*.md`)
- Opinion confidence: `O(c=0.0..1.0)` optional

If you don’t want authors to think about it: the reflect job can infer these bullets from the rest of the log, but having an explicit `## Giữ lại` section is the easiest “quality lever”.

### Recall: queries over the derived index

Recall should support:

- **lexical**: “find exact terms / names / commands” (FTS5)
- **entity**: “tell me about X” (entity pages + entity-linked facts)
- **temporal**: “what happened around Nov 27” / “since last week”
- **opinion**: “what does Peter prefer?” (with confidence + evidence)

Return format should be agent-friendly and cite sources:

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (source day, or extracted time range if present)
- `entities` (`["Peter","warelay"]`)
- `content` (the narrative fact)
- `source` (`memory/2025-11-27.md#L12` etc)

### Reflect: produce stable pages + update beliefs

Reflection is a scheduled job (daily or heartbeat `ultrathink`) that:

- updates `bank/entities/*.md` from recent facts (entity summaries)
- updates `bank/opinions.md` confidence based on reinforcement/contradiction
- optionally proposes edits to `memory.md` (“core-ish” durable facts)

Opinion evolution (simple, explainable):

- each opinion has:
  - statement
  - confidence `c ∈ [0,1]`
- last_updated
- evidence links (supporting + contradicting fact IDs)
- khi các sự kiện mới đến:
  - tìm các ý kiến ứng cử viên bằng cách trùng lặp thực thể + tương tự (FTS trước, embeddings sau)
  - cập nhật độ tin cậy bằng các delta nhỏ; những bước nhảy lớn yêu cầu mâu thuẫn mạnh + bằng chứng lặp lại
## Tích hợp CLI: độc lập vs tích hợp sâu

Khuyến nghị: **tích hợp sâu trong OpenClaw**, nhưng giữ một thư viện lõi có thể tách rời.

### Tại sao tích hợp vào OpenClaw?

- OpenClaw đã biết:
  - đường dẫn workspace (`agents.defaults.workspace`)
  - mô hình phiên + nhịp tim
  - các mẫu ghi nhật ký + khắc phục sự cố
- Bạn muốn agent tự gọi các công cụ:
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

### Tại sao vẫn tách một thư viện?

- giữ logic bộ nhớ có thể kiểm tra được mà không cần gateway/runtime
- tái sử dụng từ các ngữ cảnh khác (local scripts, ứng dụng desktop trong tương lai, v.v.)

Hình dạng:
Công cụ bộ nhớ được dự định là một lớp CLI + thư viện nhỏ, nhưng đây chỉ là khám phá.
## "S-Collide" / SuCo: khi nào sử dụng (nghiên cứu)

Nếu "S-Collide" đề cập đến **SuCo (Subspace Collision)**: đây là một phương pháp truy xuất ANN nhắm đến các tradeoff mạnh mẽ giữa recall/latency bằng cách sử dụng các va chạm được học/có cấu trúc trong các không gian con (bài báo: arXiv 2411.14754, 2024).

Cách tiếp cận thực tế cho `~/.openclaw/workspace`:

- **đừng bắt đầu** với SuCo.
- bắt đầu với SQLite FTS + (tùy chọn) embeddings đơn giản; bạn sẽ có được hầu hết những cải tiến UX ngay lập tức.
- chỉ xem xét các giải pháp SuCo/HNSW/ScaNN khi:
  - kho dữ liệu lớn (hàng chục/hàng trăm nghìn đoạn)
  - tìm kiếm embedding brute-force trở nên quá chậm
  - chất lượng recall bị ảnh hưởng đáng kể bởi tìm kiếm từ vựng

Các giải pháp thân thiện với chế độ ngoại tuyến (theo độ phức tạp tăng dần):

- SQLite FTS5 + bộ lọc metadata (không có ML)
- Embeddings + brute force (hoạt động tốt ngạc nhiên nếu số lượng đoạn thấp)
- Chỉ mục HNSW (phổ biến, mạnh mẽ; cần ràng buộc thư viện)
- SuCo (cấp độ nghiên cứu; hấp dẫn nếu có một triển khai vững chắc mà bạn có thể nhúng)

Câu hỏi mở:

- **mô hình embedding ngoại tuyến tốt nhất** cho "bộ nhớ trợ lý cá nhân" trên máy của bạn (laptop + desktop) là gì?
  - nếu bạn đã có Ollama: nhúng với một mô hình cục bộ; nếu không, hãy cung cấp một mô hình embedding nhỏ trong chuỗi công cụ.
## Pilot hữu ích nhất tối thiểu

Nếu bạn muốn một phiên bản tối thiểu nhưng vẫn hữu ích:

- Thêm các trang thực thể `bank/` và phần `## Retain` trong nhật ký hàng ngày.
- Sử dụng SQLite FTS để gọi lại với trích dẫn (đường dẫn + số dòng).
- Thêm embeddings chỉ khi chất lượng gọi lại hoặc quy mô yêu cầu.
## Tài liệu tham khảo

- Các khái niệm Letta / MemGPT: "khối bộ nhớ cốt lõi" + "bộ nhớ lưu trữ" + bộ nhớ tự chỉnh sửa được điều khiển bởi công cụ.
- Báo cáo Kỹ thuật Hindsight: "giữ lại / gọi lại / phản ánh", bộ nhớ bốn mạng, trích xuất sự kiện tường thuật, tiến hóa độ tin cậy ý kiến.
- SuCo: arXiv 2411.14754 (2024): "Subspace Collision" truy xuất láng giềng gần nhất xấp xỉ.