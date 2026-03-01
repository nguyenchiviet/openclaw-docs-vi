---
summary: Phát sóng tin nhắn WhatsApp tới nhiều agent
read_when:
  - Cấu hình nhóm phát sóng
  - Gỡ lỗi phản hồi đa tác nhân trong WhatsApp
status: experimental
title: Nhóm Phát Sóng
x-i18n:
  source_path: channels\broadcast-groups.md
  source_hash: 25866bc0d519552d601ae27e5c3c97816f220f05de210e609f8b9f3712fababf
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:13:48.479Z'
---

# Nhóm Phát Sóng

**Trạng thái:** Thử nghiệm  
**Phiên bản:** Được thêm vào 2026.1.9
## Tổng quan

Broadcast Groups cho phép nhiều agent xử lý và phản hồi cùng một tin nhắn đồng thời. Điều này cho phép bạn tạo ra các nhóm agent chuyên biệt làm việc cùng nhau trong một nhóm WhatsApp hoặc tin nhắn riêng — tất cả đều sử dụng một số điện thoại.

Phạm vi hiện tại: **Chỉ WhatsApp** (kênh web).

Broadcast groups được đánh giá sau danh sách cho phép kênh và quy tắc kích hoạt nhóm. Trong các nhóm WhatsApp, điều này có nghĩa là broadcast xảy ra khi OpenClaw thường phản hồi (ví dụ: khi được nhắc đến, tùy thuộc vào cài đặt nhóm của bạn).
## Trường hợp sử dụng

### 1. Nhóm Agent chuyên biệt

Triển khai nhiều agent với trách nhiệm nguyên tử và tập trung:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Each agent processes the same message and provides its specialized perspective.

### 2. Multi-Language Support

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Quality Assurance Workflows

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Task Automation

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```
## Cấu hình

### Thiết lập cơ bản

Thêm một phần `broadcast` ở cấp cao nhất (bên cạnh `bindings`). Các khóa là id đồng đẳng WhatsApp:

- nhóm chat: group JID (ví dụ `120363403215116621@g.us`)
- Tin nhắn riêng: số điện thoại E.164 (ví dụ `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Result:** When OpenClaw would reply in this chat, it will run all three agents.

### Processing Strategy

Control how agents process messages:

#### Parallel (Default)

All agents process simultaneously:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Sequential

Agents process in order (one waits for previous to finish):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Complete Example

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```
## Cách Hoạt Động

### Luồng Tin Nhắn

1. **Tin nhắn đến** xuất hiện trong nhóm WhatsApp
2. **Kiểm tra broadcast**: Hệ thống kiểm tra xem peer ID có trong `broadcast` không
3. **Nếu có trong danh sách broadcast**:
   - Tất cả các agent được liệt kê sẽ xử lý tin nhắn
   - Mỗi agent có khóa phiên riêng và ngữ cảnh tách biệt
   - Các agent xử lý song song (mặc định) hoặc tuần tự
4. **Nếu không có trong danh sách broadcast**:
   - Áp dụng định tuyến bình thường (ràng buộc khớp đầu tiên)

Lưu ý: các nhóm broadcast không bỏ qua danh sách cho phép kênh hoặc quy tắc kích hoạt nhóm (mentions/lệnh/v.v.). Chúng chỉ thay đổi _agent nào chạy_ khi tin nhắn đủ điều kiện để xử lý.

### Cách Ly Phiên

Mỗi agent trong nhóm broadcast duy trì hoàn toàn tách biệt:

- **Khóa phiên** (`agent:alfred:whatsapp:group:120363...` so với `agent:baerbel:whatsapp:group:120363...`)
- **Lịch sử hội thoại** (agent không thấy tin nhắn của các agent khác)
- **Không gian làm việc** (sandbox tách biệt nếu được cấu hình)
- **Quyền truy cập công cụ** (danh sách cho phép/từ chối khác nhau)
- **Bộ nhớ/ngữ cảnh** (IDENTITY.md, SOUL.md riêng biệt, v.v.)
- **Bộ đệm ngữ cảnh nhóm** (tin nhắn nhóm gần đây được sử dụng làm ngữ cảnh) được chia sẻ theo peer, vì vậy tất cả các agent broadcast đều thấy cùng ngữ cảnh khi được kích hoạt

Điều này cho phép mỗi agent có:

- Tính cách khác nhau
- Quyền truy cập công cụ khác nhau (ví dụ: chỉ đọc so với đọc-ghi)
- Mô hình khác nhau (ví dụ: opus so với sonnet)
- Skills được cài đặt khác nhau

### Ví dụ: Phiên Tách Biệt

Trong nhóm `120363403215116621@g.us` với các agent `["alfred", "baerbel"]`:

**Ngữ cảnh của Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel's context:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```
## Thực hành tốt nhất

### 1. Giữ Agent tập trung

Thiết kế mỗi agent với một trách nhiệm duy nhất và rõ ràng:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Good:** Each agent has one job  
❌ **Bad:** One generic "dev-helper" agent

### 2. Use Descriptive Names

Make it clear what each agent does:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Configure Different Tool Access

Give agents only the tools they need:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. Monitor Performance

With many agents, consider:

- Using `"strategy": "parallel"` (mặc định) để tăng tốc độ
- Giới hạn nhóm broadcast từ 5-10 agent
- Sử dụng các mô hình nhanh hơn cho các agent đơn giản

### 5. Xử lý lỗi một cách khéo léo

Các agent thất bại độc lập. Lỗi của một agent không chặn các agent khác:
```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```
## Tương thích

### Nhà cung cấp

Nhóm phát sóng hiện tại hoạt động với:

- ✅ WhatsApp (đã triển khai)
- 🚧 Telegram (đã lên kế hoạch)
- 🚧 Discord (đã lên kế hoạch)
- 🚧 Slack (đã lên kế hoạch)

### Định tuyến

Nhóm phát sóng hoạt động cùng với định tuyến hiện có:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Only alfred responds (normal routing)
- `GROUP_B`: agent1 AND agent2 respond (broadcast)

**Precedence:** `broadcast` takes priority over `bindings`.
## Khắc phục sự cố

### Agent không phản hồi

**Kiểm tra:**

1. ID Agent tồn tại trong `agents.list`
2. Định dạng Peer ID đúng (ví dụ: `120363403215116621@g.us`)
3. Agent không nằm trong danh sách từ chối

**Gỡ lỗi:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Only One Agent Responding

**Cause:** Peer ID might be in `bindings` but not `broadcast`.

**Khắc phục:** Thêm vào cấu hình broadcast hoặc xóa khỏi bindings.

### Vấn đề hiệu suất

**Nếu chậm với nhiều agent:**

- Giảm số lượng agent trên mỗi nhóm
- Sử dụng mô hình nhẹ hơn (sonnet thay vì opus)
- Kiểm tra thời gian khởi động sandbox
## Ví dụ

### Ví dụ 1: Nhóm Đánh giá Mã nguồn

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**User sends:** Code snippet  
**Responses:**

- code-formatter: "Fixed indentation and added type hints"
- security-scanner: "⚠️ SQL injection vulnerability in line 12"
- test-coverage: "Coverage is 45%, missing tests for error cases"
- docs-checker: "Missing docstring for function `process_data`"

### Example 2: Multi-Language Support

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```
## Tham khảo API

### Lược đồ cấu hình

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Fields

- `strategy` (optional): How to process agents
  - `"parallel"` (default): All agents process simultaneously
  - `"sequential"`: Agents process in array order
- `[peerId]`: WhatsApp group JID, số E.164, hoặc ID peer khác
  - Giá trị: Mảng các ID agent sẽ xử lý tin nhắn
## Hạn chế

1. **Số lượng agent tối đa:** Không có giới hạn cứng, nhưng 10+ agent có thể chậm
2. **Ngữ cảnh chia sẻ:** Các agent không thấy phản hồi của nhau (theo thiết kế)
3. **Thứ tự tin nhắn:** Các phản hồi song song có thể đến theo bất kỳ thứ tự nào
4. **Giới hạn tốc độ:** Tất cả agent đều tính vào giới hạn tốc độ của WhatsApp
## Cải tiến trong tương lai

Các tính năng được lên kế hoạch:

- [ ] Chế độ ngữ cảnh chia sẻ (các agent thấy phản hồi của nhau)
- [ ] Phối hợp agent (các agent có thể gửi tín hiệu cho nhau)
- [ ] Lựa chọn agent động (chọn agent dựa trên nội dung tin nhắn)
- [ ] Ưu tiên agent (một số agent phản hồi trước những agent khác)
## Xem thêm

- [Cấu hình đa Agent](/tools/multi-agent-sandbox-tools)
- [Cấu hình định tuyến](/channels/channel-routing)
- [Quản lý phiên](/concepts/sessions)