---
title: Bộ nhớ
summary: Cách hoạt động của bộ nhớ OpenClaw (tệp workspace + tự động xóa bộ nhớ)
read_when:
  - Bạn muốn bố cục tệp bộ nhớ và quy trình làm việc
  - Bạn muốn điều chỉnh việc xả bộ nhớ tự động trước khi nén dữ liệu
x-i18n:
  source_path: concepts\memory.md
  source_hash: b31753d59496ceec64e4fd3554fff5ad3c698915caf15729f278399385c40f4d
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:40:35.011Z'
---

# Bộ nhớ

Bộ nhớ OpenClaw là **Markdown thuần túy trong không gian làm việc của agent**. Các tệp là nguồn sự thật; mô hình chỉ "nhớ" những gì được ghi vào đĩa.

Các công cụ tìm kiếm bộ nhớ được cung cấp bởi plugin bộ nhớ hoạt động (mặc định:
`memory-core`). Vô hiệu hóa các plugin bộ nhớ bằng `plugins.slots.memory = "none"`.
## Tệp bộ nhớ (Markdown)

Bố cục không gian làm việc mặc định sử dụng hai lớp bộ nhớ:

- `memory/YYYY-MM-DD.md`
  - Nhật ký hàng ngày (chỉ nối thêm).
  - Đọc hôm nay + hôm qua khi bắt đầu phiên.
- `MEMORY.md` (tùy chọn)
  - Bộ nhớ dài hạn được sắp xếp.
  - **Chỉ tải trong phiên chính, riêng tư** (không bao giờ trong bối cảnh nhóm).

Các tệp này nằm dưới không gian làm việc (`agents.defaults.workspace`, mặc định
`~/.openclaw/workspace`). Xem [Agent workspace](/concepts/agent-workspace) để biết bố cục đầy đủ.
## Công cụ bộ nhớ

OpenClaw cung cấp hai công cụ hướng tới agent cho các tệp Markdown này:

- `memory_search` — gọi lại ngữ nghĩa trên các đoạn được lập chỉ mục.
- `memory_get` — đọc được nhắm mục tiêu của một tệp Markdown/phạm vi dòng cụ thể.

`memory_get` hiện **giảm mức độ một cách duyên dáng khi tệp không tồn tại** (ví dụ,
nhật ký hàng ngày hôm nay trước lần ghi đầu tiên). Cả trình quản lý tích hợp và backend QMD
đều trả về `{ text: "", path }` thay vì ném `ENOENT`, vì vậy các agent có thể
xử lý "chưa có ghi chép gì" và tiếp tục quy trình làm việc của họ mà không cần bao bọc lệnh gọi công cụ trong logic try/catch.
## Khi nào viết bộ nhớ

- Các quyết định, sở thích và sự kiện bền vững đi vào `MEMORY.md`.
- Ghi chú hàng ngày và ngữ cảnh đang chạy đi vào `memory/YYYY-MM-DD.md`.
- Nếu ai đó nói "hãy nhớ điều này," hãy ghi lại (không giữ nó trong RAM).
- Khu vực này vẫn đang phát triển. Sẽ hữu ích nếu nhắc nhở mô hình lưu trữ bộ nhớ; nó sẽ biết phải làm gì.
- Nếu bạn muốn điều gì đó tồn tại, **yêu cầu bot viết nó** vào bộ nhớ.
## Tự động xóa bộ nhớ (ping trước nén)

Khi một phiên **sắp tới auto-compaction**, OpenClaw kích hoạt một **lượt agentic im lặng** nhắc nhở mô hình ghi lại bộ nhớ bền vững **trước khi** ngữ cảnh được nén. Các prompt mặc định rõ ràng nói rằng mô hình _có thể trả lời_, nhưng thường `NO_REPLY` là phản hồi chính xác để người dùng không bao giờ thấy lượt này.

Điều này được kiểm soát bởi `agents.defaults.compaction.memoryFlush`:

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Details:

- **Soft threshold**: flush triggers when the session token estimate crosses
  `contextWindow - reserveTokensFloor - softThresholdTokens`.
- **Silent** by default: prompts include `NO_REPLY` so nothing is delivered.
- **Two prompts**: a user prompt plus a system prompt append the reminder.
- **One flush per compaction cycle** (tracked in `sessions.json`).
- **Workspace must be writable**: if the session runs sandboxed with
  `workspaceAccess: "ro"` or `"none"`, việc xóa sẽ bị bỏ qua.

Để xem toàn bộ vòng đời nén, hãy xem
[Quản lý phiên + nén](/reference/session-management-compaction).
## Tìm kiếm bộ nhớ vector

OpenClaw có thể xây dựng một chỉ mục vector nhỏ trên `MEMORY.md` và `memory/*.md` để
các truy vấn ngữ nghĩa có thể tìm thấy các ghi chú liên quan ngay cả khi cách diễn đạt khác nhau.

Mặc định:

- Được bật theo mặc định.
- Theo dõi các tệp bộ nhớ để phát hiện thay đổi (có debounce).
- Cấu hình tìm kiếm bộ nhớ dưới `agents.defaults.memorySearch` (không phải `memorySearch`
  cấp cao nhất).
- Sử dụng embeddings từ xa theo mặc định. Nếu `memorySearch.provider` không được đặt, OpenClaw tự động chọn:
  1. `local` nếu `memorySearch.local.modelPath` được cấu hình và tệp tồn tại.
  2. `openai` nếu có thể giải quyết khóa OpenAI.
  3. `gemini` nếu có thể giải quyết khóa Gemini.
  4. `voyage` nếu có thể giải quyết khóa Voyage.
  5. `mistral` nếu có thể giải quyết khóa Mistral.
  6. Nếu không, tìm kiếm bộ nhớ vẫn bị vô hiệu hóa cho đến khi được cấu hình.
- Chế độ cục bộ sử dụng node-llama-cpp và có thể yêu cầu `pnpm approve-builds`.
- Sử dụng sqlite-vec (khi có sẵn) để tăng tốc độ tìm kiếm vector bên trong SQLite.

Embeddings từ xa **yêu cầu** khóa API cho nhà cung cấp embedding. OpenClaw
giải quyết khóa từ các hồ sơ xác thực, `models.providers.*.apiKey`, hoặc các
biến môi trường. Codex OAuth chỉ bao gồm chat/completions và **không** thỏa mãn
embeddings cho tìm kiếm bộ nhớ. Đối với Gemini, sử dụng `GEMINI_API_KEY` hoặc
`models.providers.google.apiKey`. Đối với Voyage, sử dụng `VOYAGE_API_KEY` hoặc
`models.providers.voyage.apiKey`. Đối với Mistral, sử dụng `MISTRAL_API_KEY` hoặc
`models.providers.mistral.apiKey`.
Khi sử dụng điểm cuối tương thích OpenAI tùy chỉnh,
đặt `memorySearch.remote.apiKey` (và `memorySearch.remote.headers` tùy chọn).

### QMD backend (thử nghiệm)

Đặt `memory.backend = "qmd"` để thay thế trình lập chỉ mục SQLite tích hợp bằng
[QMD](https://github.com/tobi/qmd): một sidecar tìm kiếm ưu tiên cục bộ kết hợp
BM25 + vectors + reranking. Markdown vẫn là nguồn sự thật; OpenClaw gọi
ra QMD để truy xuất. Các điểm chính:

**Điều kiện tiên quyết**

- Bị vô hiệu hóa theo mặc định. Chọn tham gia cho mỗi cấu hình (`memory.backend = "qmd"`).
- Cài đặt CLI QMD riêng biệt (`bun install -g https://github.com/tobi/qmd` hoặc tải xuống
  một bản phát hành) và đảm bảo tệp nhị phân `qmd` nằm trên `PATH` của gateway.
- QMD cần một bản dựng SQLite cho phép các tiện ích mở rộng (`brew install sqlite` trên
  macOS).
- QMD chạy hoàn toàn cục bộ thông qua Bun + `node-llama-cpp` và tự động tải xuống các mô hình GGUF
  từ HuggingFace khi sử dụng lần đầu (không cần daemon Ollama riêng biệt).
- Gateway chạy QMD trong một XDG home tự chứa dưới
  `~/.openclaw/agents/<agentId>/qmd/` bằng cách đặt `XDG_CONFIG_HOME` và
  `XDG_CACHE_HOME`.
- Hỗ trợ hệ điều hành: macOS và Linux hoạt động ngay lập tức sau khi Bun + SQLite được
  cài đặt. Windows được hỗ trợ tốt nhất thông qua WSL2.

**Cách sidecar chạy**

- Gateway ghi một XDG home tự chứa QMD dưới
  `~/.openclaw/agents/<agentId>/qmd/` (cấu hình + bộ nhớ cache + cơ sở dữ liệu sqlite).
- Các bộ sưu tập được tạo thông qua `qmd collection add` từ `memory.qmd.paths`
  (cộng với các tệp bộ nhớ không gian làm việc mặc định), sau đó `qmd update` + `qmd embed` chạy
  khi khởi động và theo một khoảng thời gian có thể cấu hình (`memory.qmd.update.interval`,
mặc định 5 m).
- Gateway hiện khởi tạo trình quản lý QMD khi khởi động, vì vậy các bộ hẹn giờ cập nhật định kỳ được kích hoạt ngay cả trước lệnh `memory_search` đầu tiên.
- Làm mới khởi động hiện chạy ở chế độ nền theo mặc định để khởi động trò chuyện không bị chặn; đặt `memory.qmd.update.waitForBootSync = true` để giữ hành vi chặn trước đó.
- Tìm kiếm chạy qua `memory.qmd.searchMode` (mặc định `qmd search --json`; cũng hỗ trợ `vsearch` và `query`). Nếu chế độ được chọn từ chối cờ trên bản dựng QMD của bạn, OpenClaw sẽ thử lại với `qmd query`. Nếu QMD không thành công hoặc tệp nhị phân bị thiếu, OpenClaw sẽ tự động quay lại trình quản lý SQLite tích hợp để các công cụ bộ nhớ tiếp tục hoạt động.
- OpenClaw hiện không cung cấp điều chỉnh kích thước lô nhúng QMD; hành vi lô được kiểm soát bởi chính QMD.
- **Tìm kiếm đầu tiên có thể chậm**: QMD có thể tải xuống các mô hình GGUF cục bộ (xếp hạng lại/mở rộng truy vấn) trên lần chạy `qmd query` đầu tiên.
  - OpenClaw đặt `XDG_CONFIG_HOME`/`XDG_CACHE_HOME` tự động khi chạy QMD.
  - Nếu bạn muốn tải xuống các mô hình theo cách thủ công trước (và làm ấm cùng một chỉ mục mà OpenClaw sử dụng), hãy chạy một truy vấn một lần với các thư mục XDG của agent.

    Trạng thái QMD của OpenClaw nằm dưới **thư mục trạng thái** của bạn (mặc định là `~/.openclaw`).
    Bạn có thể trỏ `qmd` đến chỉ mục chính xác bằng cách xuất các biến XDG giống như OpenClaw sử dụng:

    ```bash
    # Pick the same state dir OpenClaw uses
    STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"

    export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
    export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"

    # (Optional) force an index refresh + embeddings
    qmd update
    qmd embed

    # Warm up / trigger first-time model downloads
    qmd query "test" -c memory-root --json >/dev/null 2>&1
    ```

**Config surface (`memory.qmd.*`)**

- `command` (default `qmd`): override the executable path.
- `searchMode` (default `search`): pick which QMD command backs
  `memory_search` (`search`, `vsearch`, `query`).
- `includeDefaultMemory` (default `true`): auto-index `MEMORY.md` + `memory/**/*.md`.
- `paths[]`: add extra directories/files (`path`, optional `pattern`, optional
  stable `name`).
- `sessions`: opt into session JSONL indexing (`enabled`, `retentionDays`,
  `exportDir`).
- `update`: controls refresh cadence and maintenance execution:
  (`interval`, `debounceMs`, `onBoot`, `waitForBootSync`, `embedInterval`,
  `commandTimeoutMs`, `updateTimeoutMs`, `embedTimeoutMs`).
- `limits`: clamp recall payload (`maxResults`, `maxSnippetChars`,
  `maxInjectedChars`, `timeoutMs`).
- `scope`: same schema as [`session.sendPolicy`](/gateway/configuration#session).
  Default is DM-only (`deny` all, `allow` direct chats); loosen it to surface QMD
  hits in groups/channels.
  - `match.keyPrefix` matches the **normalized** session key (lowercased, with any
    leading `agent:<id>:` stripped). Example: `discord:channel:`.
  - `match.rawKeyPrefix` matches the **raw** session key (lowercased), including
    `agent:<id>:`. Example: `agent:main:discord:`.
- Legacy: `match.keyPrefix: "agent:..."` vẫn được coi là tiền tố raw-key,
    nhưng nên sử dụng `rawKeyPrefix` để rõ ràng hơn.
- Khi `scope` từ chối tìm kiếm, OpenClaw ghi lại cảnh báo với
  `channel`/`chatType` được tính toán để dễ dàng gỡ lỗi kết quả trống.
- Các đoạn mã được lấy từ bên ngoài không gian làm việc sẽ hiển thị dưới dạng
  `qmd/<collection>/<relative-path>` trong kết quả `memory_search`; `memory_get`
  hiểu tiền tố đó và đọc từ gốc bộ sưu tập QMD được cấu hình.
- Khi `memory.qmd.sessions.enabled = true`, OpenClaw xuất các bản ghi phiên đã được làm sạch
  (các lượt User/Assistant) vào bộ sưu tập QMD chuyên dụng dưới
  `~/.openclaw/agents/<id>/qmd/sessions/`, để `memory_search` có thể nhớ lại các cuộc trò chuyện gần đây
  mà không cần chạm vào chỉ mục SQLite tích hợp.
- Các đoạn mã `memory_search` hiện bao gồm chân trang `Source: <path#line>` khi
  `memory.citations` là `auto`/`on`; đặt `memory.citations = "off"` để giữ
  siêu dữ liệu đường dẫn ở bên trong (agent vẫn nhận được đường dẫn cho
  `memory_get`, nhưng văn bản đoạn mã bỏ qua chân trang và lời nhắc hệ thống
  cảnh báo agent không được trích dẫn nó).

**Ví dụ**

```json5
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // Normalized session-key prefix (strips `agent:<id>:`).
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // Raw session-key prefix (includes `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**Citations & fallback**

- `memory.citations` applies regardless of backend (`auto`/`on`/`off`).
- When `qmd` runs, we tag `status().backend = "qmd"` so diagnostics show which
  engine served the results. If the QMD subprocess exits or JSON output can’t be
  parsed, the search manager logs a warning and returns the builtin provider
  (existing Markdown embeddings) until QMD recovers.

### Additional memory paths

If you want to index Markdown files outside the default workspace layout, add
explicit paths:

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```
Ghi chú:

- Các đường dẫn có thể là tuyệt đối hoặc tương đối với workspace.
- Các thư mục được quét đệ quy để tìm các tệp `.md`.
- Chỉ các tệp Markdown được lập chỉ mục.
- Các liên kết tượng trưng bị bỏ qua (tệp hoặc thư mục).

### Gemini embeddings (native)

Đặt nhà cung cấp thành `gemini` để sử dụng trực tiếp API nhúng Gemini:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Notes:

- `remote.baseUrl` is optional (defaults to the Gemini API base URL).
- `remote.headers` lets you add extra headers if needed.
- Default model: `gemini-embedding-001`.

If you want to use a **custom OpenAI-compatible endpoint** (OpenRouter, vLLM, or a proxy),
you can use the `remote` configuration with the OpenAI provider:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

If you don't want to set an API key, use `memorySearch.provider = "local"` or set
`memorySearch.fallback = "none"`.

Fallbacks:

- `memorySearch.fallback` can be `openai`, `gemini`, `voyage`, `mistral`, `local`, or `none`.
- Nhà cung cấp dự phòng chỉ được sử dụng khi nhà cung cấp nhúng chính bị lỗi.

Lập chỉ mục theo lô (OpenAI + Gemini + Voyage):
- Tắt theo mặc định. Đặt `agents.defaults.memorySearch.remote.batch.enabled = true` để bật cho việc lập chỉ mục corpus lớn (OpenAI, Gemini, và Voyage).
- Hành vi mặc định chờ hoàn thành batch; điều chỉnh `remote.batch.wait`, `remote.batch.pollIntervalMs`, và `remote.batch.timeoutMinutes` nếu cần.
- Đặt `remote.batch.concurrency` để kiểm soát số lượng batch job chúng tôi gửi song song (mặc định: 2).
- Chế độ batch áp dụng khi `memorySearch.provider = "openai"` hoặc `"gemini"` và sử dụng khóa API tương ứng.
- Các batch job của Gemini sử dụng async embeddings batch endpoint và yêu cầu tính khả dụng của Gemini Batch API.

Tại sao OpenAI batch nhanh + rẻ:

- Đối với các backfill lớn, OpenAI thường là tùy chọn nhanh nhất mà chúng tôi hỗ trợ vì chúng tôi có thể gửi nhiều yêu cầu embedding trong một batch job duy nhất và để OpenAI xử lý chúng một cách không đồng bộ.
- OpenAI cung cấp giá chiết khấu cho các khối lượng công việc Batch API, vì vậy các lần chạy lập chỉ mục lớn thường rẻ hơn so với việc gửi các yêu cầu tương tự một cách đồng bộ.
- Xem tài liệu và giá cả Batch API của OpenAI để biết chi tiết:
  - [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
  - [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

Ví dụ cấu hình:

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Tools:

- `memory_search` — returns snippets with file + line ranges.
- `memory_get` — read memory file content by path.

Local mode:

- Set `agents.defaults.memorySearch.provider = "local"`.
- Provide `agents.defaults.memorySearch.local.modelPath` (GGUF or `hf:` URI).
- Optional: set `agents.defaults.memorySearch.fallback = "none"` to avoid remote fallback.

### How the memory tools work

- `memory_search` semantically searches Markdown chunks (~400 token target, 80-token overlap) from `MEMORY.md` + `memory/**/*.md`. It returns snippet text (capped ~700 chars), file path, line range, score, provider/model, and whether we fell back from local → remote embeddings. No full file payload is returned.
- `memory_get` reads a specific memory Markdown file (workspace-relative), optionally from a starting line and for N lines. Paths outside `MEMORY.md` / `memory/` are rejected.
- Both tools are enabled only when `memorySearch.enabled` resolves true for the agent.

### What gets indexed (and when)

- File type: Markdown only (`MEMORY.md`, `memory/**/*.md`).
- Index storage: per-agent SQLite at `~/.openclaw/memory/<agentId>.sqlite` (configurable via `agents.defaults.memorySearch.store.path`, supports `{agentId}` token).
- Freshness: watcher on `MEMORY.md` + `memory/` đánh dấu chỉ mục là bẩn (debounce 1.5s). Đồng bộ hóa được lên lịch khi bắt đầu phiên, khi tìm kiếm, hoặc theo khoảng thời gian và chạy không đồng bộ. Các bản ghi phiên sử dụng ngưỡng delta để kích hoạt đồng bộ hóa nền.
- Kích hoạt lập chỉ mục lại: chỉ mục lưu trữ **dấu vân tay nhà cung cấp/mô hình + endpoint + các tham số chunking**. Nếu bất kỳ thay đổi nào trong số đó, OpenClaw sẽ tự động đặt lại và lập chỉ mục lại toàn bộ kho lưu trữ.

### Tìm kiếm hybrid (BM25 + vector)

Khi được bật, OpenClaw kết hợp:
- **Tương đồng vector** (khớp ngữ nghĩa, cách diễn đạt có thể khác)
- **Mức độ liên quan từ khóa BM25** (các token chính xác như ID, biến môi trường, ký hiệu mã)

Nếu tìm kiếm toàn văn bản không khả dụng trên nền tảng của bạn, OpenClaw sẽ quay lại tìm kiếm chỉ vector.

#### Tại sao là hybrid?

Tìm kiếm vector rất tốt ở "điều này có nghĩa giống nhau":

- "Mac Studio gateway host" vs "máy chạy gateway"
- "debounce file updates" vs "tránh lập chỉ mục trên mỗi lần ghi"

Nhưng nó có thể yếu ở các token chính xác, tín hiệu cao:

- ID (`a828e60`, `b3b9895a…`)
- ký hiệu mã (`memorySearch.query.hybrid`)
- chuỗi lỗi ("sqlite-vec unavailable")

BM25 (toàn văn bản) là ngược lại: mạnh ở các token chính xác, yếu hơn ở các paraphrase.
Tìm kiếm hybrid là giải pháp thực tế ở giữa: **sử dụng cả hai tín hiệu truy xuất** để bạn nhận được
kết quả tốt cho cả truy vấn "ngôn ngữ tự nhiên" và truy vấn "tìm kim trong đống cỏ".

#### Cách chúng tôi hợp nhất kết quả (thiết kế hiện tại)

Phác thảo triển khai:

1. Truy xuất một nhóm ứng cử viên từ cả hai phía:

- **Vector**: top `maxResults * candidateMultiplier` theo độ tương đồng cosine.
- **BM25**: top `maxResults * candidateMultiplier` theo xếp hạng BM25 FTS5 (thấp hơn là tốt hơn).

2. Chuyển đổi xếp hạng BM25 thành điểm 0..1-ish:

- `textScore = 1 / (1 + max(0, bm25Rank))`

3. Hợp nhất ứng cử viên theo chunk id và tính điểm có trọng số:

- `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Ghi chú:

- `vectorWeight` + `textWeight` được chuẩn hóa thành 1.0 trong phân giải cấu hình, vì vậy trọng số hoạt động như phần trăm.
- Nếu embedding không khả dụng (hoặc nhà cung cấp trả về zero-vector), chúng tôi vẫn chạy BM25 và trả về các khớp từ khóa.
- Nếu FTS5 không thể được tạo, chúng tôi giữ tìm kiếm chỉ vector (không có lỗi cứng).

Đây không phải là "hoàn hảo theo lý thuyết IR", nhưng nó đơn giản, nhanh chóng và có xu hướng cải thiện recall/precision trên các ghi chú thực tế.
Nếu chúng tôi muốn tinh vi hơn sau này, các bước tiếp theo phổ biến là Reciprocal Rank Fusion (RRF) hoặc chuẩn hóa điểm
(min/max hoặc z-score) trước khi trộn.

#### Đường ống hậu xử lý

Sau khi hợp nhất điểm vector và từ khóa, hai giai đoạn hậu xử lý tùy chọn
tinh chỉnh danh sách kết quả trước khi nó đến agent:

```
Vector + Keyword → Weighted Merge → Temporal Decay → Sort → MMR → Top-K Results
```

Cả hai giai đoạn đều **tắt theo mặc định** và có thể được bật độc lập.
#### Xếp hạng lại MMR (đa dạng)

Khi tìm kiếm hybrid trả về kết quả, nhiều đoạn có thể chứa nội dung tương tự hoặc trùng lặp.
Ví dụ, tìm kiếm "home network setup" có thể trả về năm đoạn gần như giống hệt nhau
từ các ghi chú hàng ngày khác nhau mà tất cả đều đề cập đến cùng một cấu hình router.

**MMR (Maximal Marginal Relevance)** xếp hạng lại kết quả để cân bằng giữa độ liên quan và đa dạng,
đảm bảo các kết quả hàng đầu bao gồm các khía cạnh khác nhau của truy vấn thay vì lặp lại cùng một thông tin.

Cách hoạt động:

1. Kết quả được chấm điểm theo độ liên quan ban đầu (điểm vector + BM25 có trọng số).
2. MMR lặp lại chọn kết quả tối đa hóa: `λ × relevance − (1−λ) × max_similarity_to_selected`.
3. Độ tương tự giữa các kết quả được đo bằng độ tương tự văn bản Jaccard trên nội dung được tokenize.

Tham số `lambda` kiểm soát sự cân bằng:

- `lambda = 1.0` → độ liên quan thuần túy (không có hình phạt đa dạng)
- `lambda = 0.0` → đa dạng tối đa (bỏ qua độ liên quan)
- Mặc định: `0.7` (cân bằng, thiên về độ liên quan nhẹ)

**Ví dụ — truy vấn: "home network setup"**

Cho các tệp bộ nhớ này:

```
memory/2026-02-10.md  → "Configured Omada router, set VLAN 10 for IoT devices"
memory/2026-02-08.md  → "Configured Omada router, moved IoT to VLAN 10"
memory/2026-02-05.md  → "Set up AdGuard DNS on 192.168.10.2"
memory/network.md     → "Router: Omada ER605, AdGuard: 192.168.10.2, VLAN 10: IoT"
```

Without MMR — top 3 results:

```
1. memory/2026-02-10.md  (score: 0.92)  ← router + VLAN
2. memory/2026-02-08.md  (score: 0.89)  ← router + VLAN (near-duplicate!)
3. memory/network.md     (score: 0.85)  ← reference doc
```

With MMR (λ=0.7) — top 3 results:

```
1. memory/2026-02-10.md  (score: 0.92)  ← router + VLAN
2. memory/network.md     (score: 0.85)  ← reference doc (diverse!)
3. memory/2026-02-05.md  (score: 0.78)  ← AdGuard DNS (diverse!)
```

The near-duplicate from Feb 8 drops out, and the agent gets three distinct pieces of information.

**When to enable:** If you notice `memory_search` trả về các đoạn dư thừa hoặc gần như trùng lặp,
đặc biệt là với các ghi chú hàng ngày thường lặp lại thông tin tương tự trong nhiều ngày.

#### Suy giảm theo thời gian (tăng cường tính gần đây)

Các agent có ghi chú hàng ngày tích lũy hàng trăm tệp có ngày tháng theo thời gian. Nếu không có suy giảm,
một ghi chú được viết tốt từ sáu tháng trước có thể xếp hạng cao hơn bản cập nhật của hôm qua về cùng một chủ đề.

**Suy giảm theo thời gian** áp dụng một bộ nhân hàm mũ cho điểm dựa trên tuổi của mỗi kết quả,
vì vậy những ký ức gần đây tự nhiên xếp hạng cao hơn trong khi những ký ức cũ mờ đi:

```
decayedScore = score × e^(-λ × ageInDays)
```

where `λ = ln(2) / halfLifeDays`.

With the default half-life of 30 days:

- Today's notes: **100%** of original score
- 7 days ago: **~84%**
- 30 days ago: **50%**
- 90 days ago: **12.5%**
- 180 days ago: **~1.6%**

**Evergreen files are never decayed:**

- `MEMORY.md` (root memory file)
- Non-dated files in `memory/` (e.g., `memory/projects.md`, `memory/network.md`)
- These contain durable reference information that should always rank normally.

**Dated daily files** (`memory/YYYY-MM-DD.md`) use the date extracted from the filename.
Other sources (e.g., session transcripts) fall back to file modification time (`mtime`).

**Example — query: "what's Rod's work schedule?"**

Given these memory files (today is Feb 10):

```
memory/2025-09-15.md  → "Rod works Mon-Fri, standup at 10am, pairing at 2pm"  (148 days old)
memory/2026-02-10.md  → "Rod has standup at 14:15, 1:1 with Zeb at 14:45"    (today)
memory/2026-02-03.md  → "Rod started new team, standup moved to 14:15"        (7 days old)
```

Without decay:

```
1. memory/2025-09-15.md  (score: 0.91)  ← best semantic match, but stale!
2. memory/2026-02-10.md  (score: 0.82)
3. memory/2026-02-03.md  (score: 0.80)
```

With decay (halfLife=30):

```
1. memory/2026-02-10.md  (score: 0.82 × 1.00 = 0.82)  ← today, no decay
2. memory/2026-02-03.md  (score: 0.80 × 0.85 = 0.68)  ← 7 days, mild decay
3. memory/2025-09-15.md  (score: 0.91 × 0.03 = 0.03)  ← 148 days, nearly gone
```

The stale September note drops to the bottom despite having the best raw semantic match.

**When to enable:** If your agent has months of daily notes and you find that old,
stale information outranks recent context. A half-life of 30 days works well for
daily-note-heavy workflows; increase it (e.g., 90 days) if you reference older notes frequently.

#### Configuration

Both features are configured under `memorySearch.query.hybrid`:
```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4,
          // Diversity: reduce redundant results
          mmr: {
            enabled: true,    // default: false
            lambda: 0.7       // 0 = max diversity, 1 = max relevance
          },
          // Recency: boost newer memories
          temporalDecay: {
            enabled: true,    // default: false
            halfLifeDays: 30  // score halves every 30 days
          }
        }
      }
    }
  }
}
```

You can enable either feature independently:

- **MMR only** — useful when you have many similar notes but age doesn't matter.
- **Temporal decay only** — useful when recency matters but your results are already diverse.
- **Both** — recommended for agents with large, long-running daily note histories.

### Embedding cache

OpenClaw can cache **chunk embeddings** in SQLite so reindexing and frequent updates (especially session transcripts) don't re-embed unchanged text.

Config:

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

### Session memory search (experimental)

You can optionally index **session transcripts** and surface them via `memory_search`.
This is gated behind an experimental flag.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```
Ghi chú:

- Lập chỉ mục phiên là **tùy chọn** (tắt theo mặc định).
- Cập nhật phiên được khử nhiễu và **lập chỉ mục không đồng bộ** sau khi vượt qua ngưỡng delta (nỗ lực tốt nhất).
- `memory_search` không bao giờ chặn lập chỉ mục; kết quả có thể hơi cũ cho đến khi đồng bộ hóa nền tảng hoàn tất.
- Kết quả vẫn chỉ bao gồm các đoạn trích; `memory_get` vẫn bị giới hạn ở các tệp bộ nhớ.
- Lập chỉ mục phiên được cách ly cho mỗi agent (chỉ nhật ký phiên của agent đó được lập chỉ mục).
- Nhật ký phiên nằm trên đĩa (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Bất kỳ quy trình/người dùng nào có quyền truy cập hệ thống tệp đều có thể đọc chúng, vì vậy hãy coi quyền truy cập đĩa là ranh giới tin cậy. Để cách ly nghiêm ngặt hơn, hãy chạy các agent dưới các người dùng hoặc máy chủ OS riêng biệt.

Ngưỡng Delta (giá trị mặc định được hiển thị):

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 KB
          deltaMessages: 50     // JSONL lines
        }
      }
    }
  }
}
```

### SQLite vector acceleration (sqlite-vec)

When the sqlite-vec extension is available, OpenClaw stores embeddings in a
SQLite virtual table (`vec0`) and performs vector distance queries in the
database. This keeps search fast without loading every embedding into JS.

Configuration (optional):

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Notes:

- `enabled` defaults to true; when disabled, search falls back to in-process
  cosine similarity over stored embeddings.
- If the sqlite-vec extension is missing or fails to load, OpenClaw logs the
  error and continues with the JS fallback (no vector table).
- `extensionPath` ghi đè đường dẫn sqlite-vec được đóng gói (hữu ích cho các bản dựng tùy chỉnh hoặc vị trí cài đặt không chuẩn).

### Tải xuống nhúng cục bộ tự động
- Mô hình nhúng cục bộ mặc định: `hf:ggml-org/embeddinggemma-300m-qat-q8_0-GGUF/embeddinggemma-300m-qat-Q8_0.gguf` (~0.6 GB).
- Khi `memorySearch.provider = "local"`, `node-llama-cpp` giải quyết `modelPath`; nếu GGUF bị thiếu, nó **tự động tải xuống** vào bộ nhớ cache (hoặc `local.modelCacheDir` nếu được đặt), sau đó tải nó. Các tải xuống tiếp tục khi thử lại.
- Yêu cầu xây dựng gốc: chạy `pnpm approve-builds`, chọn `node-llama-cpp`, sau đó `pnpm rebuild node-llama-cpp`.
- Dự phòng: nếu thiết lập cục bộ không thành công và `memorySearch.fallback = "openai"`, chúng tôi tự động chuyển sang nhúng từ xa (`openai/text-embedding-3-small` trừ khi bị ghi đè) và ghi lại lý do.

### Ví dụ về điểm cuối tương thích OpenAI tùy chỉnh

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Notes:

- `remote.*` takes precedence over `models.providers.openai.*`.
- `remote.headers` merge with OpenAI headers; remote wins on key conflicts. Omit `remote.headers` để sử dụng các giá trị mặc định của OpenAI.