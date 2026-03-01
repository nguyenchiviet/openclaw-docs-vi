---
summary: >-
  Bộ kit kiểm thử: các bộ unit/e2e/live, Docker runners, và những gì mỗi bài
  kiểm thử bao gồm
read_when:
  - Chạy các bài kiểm tra cục bộ hoặc trong CI
  - Thêm các bài kiểm tra hồi quy cho các lỗi của mô hình/nhà cung cấp
  - Gỡ lỗi hành vi gateway + agent
title: Kiểm tra
x-i18n:
  source_path: help\testing.md
  source_hash: 5f737303e044e201c78d065268859c6afdcc99afd12dd6de7ff70c749ecbdff4
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:07:06.870Z'
---

# Kiểm thử

OpenClaw có ba bộ Vitest (unit/integration, e2e, live) và một tập hợp nhỏ các runner Docker.

Tài liệu này là hướng dẫn "cách chúng tôi kiểm thử":

- Mỗi bộ bao gồm những gì (và những gì nó cố ý _không_ bao gồm)
- Lệnh nào cần chạy cho các quy trình công việc phổ biến (local, pre-push, debugging)
- Cách live tests khám phá thông tin xác thực và chọn mô hình/nhà cung cấp
- Cách thêm regressions cho các vấn đề mô hình/nhà cung cấp trong thế giới thực
## Bắt đầu nhanh

Hầu hết các ngày:

- Full gate (dự kiến trước khi push): `pnpm build && pnpm check && pnpm test`

Khi bạn chạm vào các bài kiểm tra hoặc muốn có thêm sự tự tin:

- Coverage gate: `pnpm test:coverage`
- E2E suite: `pnpm test:e2e`

Khi gỡ lỗi các nhà cung cấp/mô hình thực tế (yêu cầu thông tin xác thực thực tế):

- Live suite (mô hình + gateway tool/image probes): `pnpm test:live`

Mẹo: khi bạn chỉ cần một trường hợp thất bại, hãy ưu tiên thu hẹp các bài kiểm tra live thông qua các biến môi trường allowlist được mô tả dưới đây.
## Bộ kiểm tra (chạy ở đâu)

Hãy coi các bộ kiểm tra là "tăng tính thực tế" (và tăng tính không ổn định/chi phí):

### Unit / integration (mặc định)

- Lệnh: `pnpm test`
- Cấu hình: `scripts/test-parallel.mjs` (chạy `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`)
- Tệp: `src/**/*.test.ts`, `extensions/**/*.test.ts`
- Phạm vi:
  - Kiểm tra unit thuần túy
  - Kiểm tra tích hợp trong quy trình (xác thực Gateway, định tuyến, công cụ, phân tích cú pháp, cấu hình)
  - Kiểm tra hồi quy xác định cho các lỗi đã biết
- Kỳ vọng:
  - Chạy trong CI
  - Không cần khóa thực
  - Nên nhanh và ổn định
- Ghi chú về nhóm:
  - OpenClaw sử dụng Vitest `vmForks` trên Node 22/23 để tăng tốc độ các phân đoạn unit.
  - Trên Node 24+, OpenClaw tự động quay lại `forks` thông thường để tránh lỗi liên kết Node VM (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`).
  - Ghi đè thủ công bằng `OPENCLAW_TEST_VM_FORKS=0` (buộc `forks`) hoặc `OPENCLAW_TEST_VM_FORKS=1` (buộc `vmForks`).

### E2E (Gateway smoke)

- Lệnh: `pnpm test:e2e`
- Cấu hình: `vitest.e2e.config.ts`
- Tệp: `src/**/*.e2e.test.ts`
- Mặc định thời gian chạy:
  - Sử dụng Vitest `vmForks` để khởi động tệp nhanh hơn.
  - Sử dụng worker thích ứng (CI: 2-4, cục bộ: 4-8).
  - Chạy ở chế độ im lặng theo mặc định để giảm chi phí I/O bảng điều khiển.
- Ghi đè hữu ích:
  - `OPENCLAW_E2E_WORKERS=<n>` để buộc số lượng worker (giới hạn ở 16).
  - `OPENCLAW_E2E_VERBOSE=1` để bật lại đầu ra bảng điều khiển chi tiết.
- Phạm vi:
  - Hành vi end-to-end Gateway đa phiên bản
  - Bề mặt WebSocket/HTTP, ghép nối node và mạng nặng hơn
- Kỳ vọng:
  - Chạy trong CI (khi được bật trong pipeline)
  - Không cần khóa thực
  - Nhiều bộ phận di chuyển hơn các kiểm tra unit (có thể chậm hơn)

### Live (nhà cung cấp thực + mô hình thực)

- Lệnh: `pnpm test:live`
- Cấu hình: `vitest.live.config.ts`
- Tệp: `src/**/*.live.test.ts`
- Mặc định: **được bật** bởi `pnpm test:live` (đặt `OPENCLAW_LIVE_TEST=1`)
- Phạm vi:
  - "Nhà cung cấp/mô hình này có thực sự hoạt động _hôm nay_ với thông tin xác thực thực không?"
  - Bắt các thay đổi định dạng nhà cung cấp, các điểm kỳ lạ gọi công cụ, vấn đề xác thực và hành vi giới hạn tốc độ
- Kỳ vọng:
  - Không ổn định CI theo thiết kế (mạng thực, chính sách nhà cung cấp thực, hạn ngạch, sự cố)
  - Chi phí tiền / sử dụng giới hạn tốc độ
  - Ưu tiên chạy các tập hợp con hẹp thay vì "tất cả"
  - Các lần chạy live sẽ lấy `~/.profile` để nhận các khóa API bị thiếu
- Xoay khóa API (dành riêng cho nhà cung cấp): đặt `*_API_KEYS` với định dạng dấu phẩy/dấu chấm phẩy hoặc `*_API_KEY_1`, `*_API_KEY_2` (ví dụ `OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`) hoặc ghi đè live cho mỗi thông qua `OPENCLAW_LIVE_*_KEY`; các kiểm tra sẽ thử lại khi nhận được phản hồi giới hạn tốc độ.
## Tôi nên chạy bộ kiểm tra nào?

Sử dụng bảng quyết định này:

- Chỉnh sửa logic/kiểm tra: chạy `pnpm test` (và `pnpm test:coverage` nếu bạn thay đổi rất nhiều)
- Chạm vào mạng Gateway / giao thức WS / ghép nối: thêm `pnpm test:e2e`
- Gỡ lỗi "bot của tôi bị down" / lỗi cụ thể của nhà cung cấp / gọi công cụ: chạy `pnpm test:live` hẹp hơn
## Live: Quét khả năng node Android

- Test: `src/gateway/android-node.capabilities.live.test.ts`
- Script: `pnpm android:test:integration`
- Goal: gọi **mọi lệnh hiện được quảng cáo** bởi một node Android được kết nối và xác nhận hành vi hợp đồng lệnh.
- Scope:
  - Thiết lập được điều kiện trước/thủ công (bộ test không cài đặt/chạy/ghép nối ứng dụng).
  - Xác thực gateway `node.invoke` từng lệnh cho node Android được chọn.
- Yêu cầu thiết lập trước:
  - Ứng dụng Android đã kết nối + ghép nối với gateway.
  - Ứng dụng được giữ ở tiền cảnh.
  - Quyền/sự đồng ý ghi hình được cấp cho các khả năng bạn dự kiến sẽ vượt qua.
- Ghi đè mục tiêu tùy chọn:
  - `OPENCLAW_ANDROID_NODE_ID` hoặc `OPENCLAW_ANDROID_NODE_NAME`.
  - `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`.
- Chi tiết thiết lập Android đầy đủ: [Android App](/platforms/android)
## Live: model smoke (profile keys)

Các bài kiểm tra trực tiếp được chia thành hai lớp để chúng tôi có thể cô lập các lỗi:

- "Direct model" cho chúng tôi biết nhà cung cấp/mô hình có thể trả lời được với khóa đã cho hay không.
- "Gateway smoke" cho chúng tôi biết toàn bộ đường ống gateway+agent hoạt động cho mô hình đó (phiên, lịch sử, công cụ, chính sách sandbox, v.v.).

### Layer 1: Direct model completion (no gateway)

- Test: `src/agents/models.profiles.live.test.ts`
- Mục tiêu:
  - Liệt kê các mô hình được khám phá
  - Sử dụng `getApiKeyForModel` để chọn các mô hình bạn có thông tin xác thực
  - Chạy một hoàn thành nhỏ cho mỗi mô hình (và các hồi quy được nhắm mục tiêu nếu cần)
- Cách bật:
  - `pnpm test:live` (hoặc `OPENCLAW_LIVE_TEST=1` nếu gọi Vitest trực tiếp)
- Đặt `OPENCLAW_LIVE_MODELS=modern` (hoặc `all`, bí danh cho hiện đại) để thực sự chạy bộ này; nếu không, nó sẽ bỏ qua để giữ `pnpm test:live` tập trung vào gateway smoke
- Cách chọn mô hình:
  - `OPENCLAW_LIVE_MODELS=modern` để chạy danh sách cho phép hiện đại (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  - `OPENCLAW_LIVE_MODELS=all` là bí danh cho danh sách cho phép hiện đại
  - hoặc `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (danh sách cho phép được phân tách bằng dấu phẩy)
- Cách chọn nhà cung cấp:
  - `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (danh sách cho phép được phân tách bằng dấu phẩy)
- Khóa đến từ đâu:
  - Theo mặc định: kho lưu trữ hồ sơ và dự phòng env
  - Đặt `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` để thực thi **kho lưu trữ hồ sơ** chỉ
- Tại sao điều này tồn tại:
  - Tách "API nhà cung cấp bị hỏng / khóa không hợp lệ" khỏi "đường ống agent gateway bị hỏng"
  - Chứa các hồi quy nhỏ, cô lập (ví dụ: OpenAI Responses/Codex Responses reasoning replay + tool-call flows)

### Layer 2: Gateway + dev agent smoke (what "@openclaw" actually does)

- Test: `src/gateway/gateway-models.profiles.live.test.ts`
- Mục tiêu:
  - Khởi động một gateway trong quy trình
  - Tạo/vá một phiên `agent:dev:*` (ghi đè mô hình cho mỗi lần chạy)
  - Lặp lại các mô hình có khóa và khẳng định:
    - phản hồi "có ý nghĩa" (không có công cụ)
    - một lệnh gọi công cụ thực sự hoạt động (đọc probe)
    - các probe công cụ bổ sung tùy chọn (exec+read probe)
    - các đường dẫn hồi quy OpenAI (tool-call-only → follow-up) tiếp tục hoạt động
- Chi tiết Probe (để bạn có thể giải thích các lỗi một cách nhanh chóng):
  - `read` probe: bài kiểm tra ghi một tệp nonce trong không gian làm việc và yêu cầu agent `read` nó và lặp lại nonce.
  - `exec+read` probe: bài kiểm tra yêu cầu agent `exec`-ghi một nonce vào tệp tạm thời, sau đó `read` nó trở lại.
  - image probe: bài kiểm tra đính kèm một PNG được tạo (mèo + mã ngẫu nhiên) và mong đợi mô hình trả về `cat <CODE>`.
  - Tham chiếu triển khai: `src/gateway/gateway-models.profiles.live.test.ts` và `src/gateway/live-image-probe.ts`.
- Cách bật:
  - `pnpm test:live` (hoặc `OPENCLAW_LIVE_TEST=1` nếu gọi Vitest trực tiếp)
- Cách chọn mô hình:
  - Mặc định: danh sách cho phép hiện đại (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  - `OPENCLAW_LIVE_GATEWAY_MODELS=all` là bí danh cho danh sách cho phép hiện đại
  - Hoặc đặt `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (hoặc danh sách được phân tách bằng dấu phẩy) để thu hẹp
- Cách chọn nhà cung cấp (tránh "OpenRouter mọi thứ"):
  - `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (danh sách cho phép được phân tách bằng dấu phẩy)
- Các probe công cụ + hình ảnh luôn bật trong bài kiểm tra trực tiếp này:
  - `read` probe + `exec+read` probe (tool stress)
  - image probe chạy khi mô hình quảng cáo hỗ trợ đầu vào hình ảnh
  - Luồng (cấp cao):
    - Bài kiểm tra tạo một PNG nhỏ với "CAT" + mã ngẫu nhiên (`src/gateway/live-image-probe.ts`)
    - Gửi nó qua `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
- Gateway phân tích các tệp đính kèm thành `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
- Agent nhúng chuyển tiếp một tin nhắn người dùng đa phương tiện đến mô hình
- Khẳng định: phản hồi chứa `cat` + mã (OCR tolerance: cho phép lỗi nhỏ)

Mẹo: để xem những gì bạn có thể kiểm tra trên máy của mình (và các `provider/model` ids chính xác), hãy chạy:

```bash
openclaw models list
openclaw models list --json
```
## Live: Thiết lập token Anthropic smoke

- Test: `src/agents/anthropic.setup-token.live.test.ts`
- Goal: xác minh Claude Code CLI setup-token (hoặc một hồ sơ setup-token được dán) có thể hoàn thành một lời nhắc Anthropic.
- Enable:
  - `pnpm test:live` (hoặc `OPENCLAW_LIVE_TEST=1` nếu gọi Vitest trực tiếp)
  - `OPENCLAW_LIVE_SETUP_TOKEN=1`
- Token sources (chọn một):
  - Profile: `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  - Raw token: `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
- Model override (tùy chọn):
  - `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

Setup example:

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```
## Live: CLI backend smoke (Claude Code CLI or other local CLIs)

- Test: `src/gateway/gateway-cli-backend.live.test.ts`
- Goal: xác thực pipeline Gateway + agent bằng cách sử dụng local CLI backend, mà không cần chạm vào cấu hình mặc định của bạn.
- Enable:
  - `pnpm test:live` (or `OPENCLAW_LIVE_TEST=1` if invoking Vitest directly)
  - `OPENCLAW_LIVE_CLI_BACKEND=1`
- Defaults:
  - Model: `claude-cli/claude-sonnet-4-6`
  - Command: `claude`
  - Args: `["-p","--output-format","json","--dangerously-skip-permissions"]`
- Overrides (optional):
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.3-codex"`
  - `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  - `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  - `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` để gửi một tệp đính kèm hình ảnh thực (các đường dẫn được chèn vào prompt).
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` để chuyển các đường dẫn tệp hình ảnh dưới dạng CLI args thay vì prompt injection.
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (or `"list"`) để kiểm soát cách các image args được truyền khi `IMAGE_ARG` được đặt.
  - `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` để gửi một lượt thứ hai và xác thực resume flow.
- `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` để giữ Claude Code CLI MCP config được bật (mặc định vô hiệu hóa MCP config bằng một tệp trống tạm thời).

Example:

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### Recommended live recipes

Narrow, explicit allowlists are fastest and least flaky:

- Single model, direct (no gateway):
  - `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

- Single model, gateway smoke:
  - `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Tool calling across several providers:
  - `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Google focus (Gemini API key + Antigravity):
  - Gemini (API key): `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  - Antigravity (OAuth): `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notes:

- `google/...` uses the Gemini API (API key).
- `google-antigravity/...` uses the Antigravity OAuth bridge (Cloud Code Assist-style agent endpoint).
- `google-gemini-cli/...` uses the local Gemini CLI on your machine (separate auth + tooling quirks).
- Gemini API vs Gemini CLI:
  - API: OpenClaw calls Google’s hosted Gemini API over HTTP (API key / profile auth); this is what most users mean by “Gemini”.
  - CLI: OpenClaw shells out to a local `gemini` binary; nó có xác thực riêng của nó và có thể hoạt động khác nhau (streaming/tool support/version skew).
## Live: ma trận mô hình (những gì chúng tôi bao gồm)

Không có danh sách "mô hình CI cố định" (live là tùy chọn), nhưng đây là những **mô hình được khuyến nghị** để bao phủ thường xuyên trên máy dev với các khóa.

### Bộ smoke hiện đại (gọi công cụ + hình ảnh)

Đây là lần chạy "mô hình phổ biến" mà chúng tôi mong đợi sẽ tiếp tục hoạt động:

- OpenAI (non-Codex): `openai/gpt-5.2` (tùy chọn: `openai/gpt-5.1`)
- OpenAI Codex: `openai-codex/gpt-5.3-codex` (tùy chọn: `openai-codex/gpt-5.3-codex-codex`)
- Anthropic: `anthropic/claude-opus-4-6` (hoặc `anthropic/claude-sonnet-4-5`)
- Google (Gemini API): `google/gemini-3-pro-preview` và `google/gemini-3-flash-preview` (tránh các mô hình Gemini 2.x cũ hơn)
- Google (Antigravity): `google-antigravity/claude-opus-4-6-thinking` và `google-antigravity/gemini-3-flash`
- Z.AI (GLM): `zai/glm-4.7`
- MiniMax: `minimax/minimax-m2.1`

Chạy gateway smoke với công cụ + hình ảnh:
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.3-codex,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### Cơ sở: gọi công cụ (Đọc + Thực thi tùy chọn)

Chọn ít nhất một cho mỗi họ nhà cung cấp:

- OpenAI: `openai/gpt-5.2` (hoặc `openai/gpt-5-mini`)
- Anthropic: `anthropic/claude-opus-4-6` (hoặc `anthropic/claude-sonnet-4-5`)
- Google: `google/gemini-3-flash-preview` (hoặc `google/gemini-3-pro-preview`)
- Z.AI (GLM): `zai/glm-4.7`
- MiniMax: `minimax/minimax-m2.1`

Bao phủ bổ sung tùy chọn (tốt để có):

- xAI: `xai/grok-4` (hoặc phiên bản mới nhất có sẵn)
- Mistral: `mistral/`… (chọn một mô hình "công cụ" có khả năng mà bạn đã bật)
- Cerebras: `cerebras/`… (nếu bạn có quyền truy cập)
- LM Studio: `lmstudio/`… (cục bộ; gọi công cụ phụ thuộc vào chế độ API)

### Tầm nhìn: gửi hình ảnh (tệp đính kèm → tin nhắn đa phương tiện)

Bao gồm ít nhất một mô hình có khả năng hình ảnh trong `OPENCLAW_LIVE_GATEWAY_MODELS` (các biến thể hỗ trợ tầm nhìn Claude/Gemini/OpenAI, v.v.) để thực hiện kiểm tra hình ảnh.

### Bộ tập hợp / gateway thay thế

Nếu bạn có các khóa được bật, chúng tôi cũng hỗ trợ kiểm tra qua:

- OpenRouter: `openrouter/...` (hàng trăm mô hình; sử dụng `openclaw models scan` để tìm các ứng cử viên có khả năng công cụ + hình ảnh)
- OpenCode Zen: `opencode/...` (xác thực qua `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Nhiều nhà cung cấp khác mà bạn có thể đưa vào ma trận live (nếu bạn có thông tin xác thực/cấu hình):

- Tích hợp sẵn: `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
- Qua `models.providers` (điểm cuối tùy chỉnh): `minimax` (đám mây/API), cộng với bất kỳ proxy tương thích OpenAI/Anthropic nào (LM Studio, vLLM, LiteLLM, v.v.)

Mẹo: đừng cố gắng mã hóa cứng "tất cả mô hình" trong tài liệu. Danh sách có thẩm quyền là bất kỳ `discoverModels(...)` nào trả về trên máy của bạn + bất kỳ khóa nào có sẵn.
## Thông tin xác thực (không bao giờ commit)

Các bài kiểm tra trực tiếp khám phá thông tin xác thực theo cách CLI thực hiện. Những hàm ý thực tế:

- Nếu CLI hoạt động, các bài kiểm tra trực tiếp sẽ tìm thấy các khóa tương tự.
- Nếu một bài kiểm tra trực tiếp nói "không có thông tin xác thực", hãy gỡ lỗi theo cách bạn sẽ gỡ lỗi `openclaw models list` / lựa chọn mô hình.

- Kho lưu trữ hồ sơ: `~/.openclaw/credentials/` (ưu tiên; "khóa hồ sơ" có nghĩa là gì trong các bài kiểm tra)
- Cấu hình: `~/.openclaw/openclaw.json` (hoặc `OPENCLAW_CONFIG_PATH`)

Nếu bạn muốn dựa vào các khóa env (ví dụ: được xuất trong `~/.profile`), hãy chạy các bài kiểm tra cục bộ sau `source ~/.profile`, hoặc sử dụng các trình chạy Docker bên dưới (chúng có thể gắn `~/.profile` vào container).
## Deepgram live (phiên âm âm thanh)

- Test: `src/media-understanding/providers/deepgram/audio.live.test.ts`
- Enable: `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`
## Kế hoạch mã hóa BytePlus trực tiếp

- Test: `src/agents/byteplus.live.test.ts`
- Enable: `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
- Optional model override: `BYTEPLUS_CODING_MODEL=ark-code-latest`
## Docker runners (kiểm tra "hoạt động trên Linux" tùy chọn)

Những cái này chạy `pnpm test:live` bên trong image Docker của repo, gắn thư mục cấu hình cục bộ và workspace của bạn (và sourcing `~/.profile` nếu được gắn):

- Direct models: `pnpm test:docker:live-models` (script: `scripts/test-live-models-docker.sh`)
- Gateway + dev agent: `pnpm test:docker:live-gateway` (script: `scripts/test-live-gateway-models-docker.sh`)
- Onboarding wizard (TTY, full scaffolding): `pnpm test:docker:onboard` (script: `scripts/e2e/onboard-docker.sh`)
- Gateway networking (hai containers, WS auth + health): `pnpm test:docker:gateway-network` (script: `scripts/e2e/gateway-network-docker.sh`)
- Plugins (custom extension load + registry smoke): `pnpm test:docker:plugins` (script: `scripts/e2e/plugins-docker.sh`)

Kiểm tra smoke thread plain-language ACP thủ công (không phải CI):

- `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
- Giữ script này cho các quy trình regression/debug. Nó có thể cần thiết lại cho xác thực định tuyến thread ACP, vì vậy đừng xóa nó.

Biến môi trường hữu ích:

- `OPENCLAW_CONFIG_DIR=...` (mặc định: `~/.openclaw`) được gắn vào `/home/node/.openclaw`
- `OPENCLAW_WORKSPACE_DIR=...` (mặc định: `~/.openclaw/workspace`) được gắn vào `/home/node/.openclaw/workspace`
- `OPENCLAW_PROFILE_FILE=...` (mặc định: `~/.profile`) được gắn vào `/home/node/.profile` và được sourcing trước khi chạy các bài kiểm tra
- `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` để thu hẹp lần chạy
- `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` để đảm bảo thông tin xác thực đến từ kho lưu trữ hồ sơ (không phải env)
## Kiểm tra tài liệu

Chạy kiểm tra tài liệu sau khi chỉnh sửa tài liệu: `pnpm docs:list`.
## Hồi quy ngoại tuyến (CI-safe)

Đây là những hồi quy "pipeline thực" mà không có nhà cung cấp thực:

- Gọi công cụ Gateway (mock OpenAI, gateway thực + vòng lặp agent): `src/gateway/gateway.test.ts` (case: "runs a mock OpenAI tool call end-to-end via gateway agent loop")
- Trình hướng dẫn Gateway (WS `wizard.start`/`wizard.next`, ghi cấu hình + xác thực được thực thi): `src/gateway/gateway.test.ts` (case: "runs wizard over ws and writes auth token config")
## Đánh giá độ tin cậy của Agent (Skills)

Chúng tôi đã có một vài bài kiểm tra an toàn cho CI hoạt động như "đánh giá độ tin cậy của agent":

- Gọi công cụ giả thông qua gateway thực + vòng lặp agent (`src/gateway/gateway.test.ts`).
- Luồng trình hướng dẫn end-to-end xác thực wiring phiên và hiệu ứng cấu hình (`src/gateway/gateway.test.ts`).

Những gì vẫn còn thiếu cho Skills (xem [Skills](/tools/skills)):

- **Decisioning:** khi các Skills được liệt kê trong prompt, agent có chọn đúng skill (hoặc tránh những skill không liên quan) không?
- **Compliance:** agent có đọc `SKILL.md` trước khi sử dụng và tuân theo các bước/đối số bắt buộc không?
- **Workflow contracts:** các kịch bản nhiều lượt khẳng định thứ tự công cụ, carryover lịch sử phiên và ranh giới sandbox.

Các đánh giá trong tương lai nên giữ tính xác định trước tiên:

- Một trình chạy kịch bản sử dụng các nhà cung cấp giả để khẳng định các lệnh gọi công cụ + thứ tự, đọc tệp skill và wiring phiên.
- Một bộ kịch bản nhỏ tập trung vào skill (sử dụng vs tránh, gating, prompt injection).
- Các đánh giá live tùy chọn (opt-in, env-gated) chỉ sau khi bộ suite an toàn cho CI được triển khai.
## Thêm các bài kiểm tra hồi quy (hướng dẫn)

Khi bạn sửa một vấn đề provider/model được phát hiện trong môi trường live:

- Thêm một bài kiểm tra hồi quy an toàn cho CI nếu có thể (mock/stub provider, hoặc ghi lại chính xác phép biến đổi hình dạng yêu cầu)
- Nếu nó vốn chỉ dành cho live (giới hạn tốc độ, chính sách xác thực), hãy giữ bài kiểm tra live hẹp và tùy chọn thông qua biến môi trường
- Ưu tiên nhắm mục tiêu vào lớp nhỏ nhất có thể bắt được lỗi:
  - lỗi chuyển đổi/phát lại yêu cầu provider → bài kiểm tra mô hình trực tiếp
  - lỗi pipeline phiên/lịch sử/công cụ gateway → bài kiểm tra smoke live gateway hoặc bài kiểm tra mock gateway an toàn cho CI