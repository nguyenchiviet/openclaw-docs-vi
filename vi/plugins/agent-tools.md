---
summary: 'Viết các công cụ agent trong một plugin (schemas, optional tools, allowlists)'
read_when:
  - Bạn muốn thêm một công cụ agent mới trong một plugin
  - Bạn cần làm cho một công cụ được chọn tham gia thông qua danh sách cho phép.
title: Công cụ Agent Plugin
x-i18n:
  source_path: plugins\agent-tools.md
  source_hash: 4479462e9d8b17b664bf6b5f424f2efc8e7bedeaabfdb6a93126e051e635c659
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:14:46.934Z'
---

# Công cụ agent plugin

Các plugin OpenClaw có thể đăng ký **công cụ agent** (các hàm JSON‑schema) được hiển thị
cho LLM trong quá trình chạy agent. Công cụ có thể **bắt buộc** (luôn khả dụng) hoặc
**tùy chọn** (tham gia).

Công cụ agent được cấu hình dưới `tools` trong cấu hình chính, hoặc cho từng agent dưới
`agents.list[].tools`. Chính sách danh sách cho phép/từ chối kiểm soát những công cụ nào mà agent
có thể gọi.
## Công cụ cơ bản

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```
## Công cụ tùy chọn (tham gia)

Các công cụ tùy chọn **không bao giờ** được tự động bật. Người dùng phải thêm chúng vào danh sách cho phép của agent.

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Enable optional tools in `agents.list[].tools.allow` (or global `tools.allow`):

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // specific tool name
            "workflow", // plugin id (enables all tools from that plugin)
            "group:plugins", // all plugin tools
          ],
        },
      },
    ],
  },
}
```

Other config knobs that affect tool availability:

- Allowlists that only name plugin tools are treated as plugin opt-ins; core tools remain
  enabled unless you also include core tools or groups in the allowlist.
- `tools.profile` / `agents.list[].tools.profile` (base allowlist)
- `tools.byProvider` / `agents.list[].tools.byProvider` (provider‑specific allow/deny)
- `tools.sandbox.tools.*` (chính sách công cụ sandbox khi được sandboxed)
## Quy tắc + mẹo

- Tên công cụ **không được** xung đột với tên công cụ cốt lõi; các công cụ xung đột sẽ bị bỏ qua.
- ID plugin được sử dụng trong danh sách cho phép không được xung đột với tên công cụ cốt lõi.
- Ưu tiên `optional: true` cho các công cụ kích hoạt tác dụng phụ hoặc yêu cầu các tệp nhị phân/thông tin xác thực bổ sung.