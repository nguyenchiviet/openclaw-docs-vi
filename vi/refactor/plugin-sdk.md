---
summary: >-
  Kế hoạch: một SDK plugin sạch + runtime duy nhất cho tất cả các bộ kết nối
  nhắn tin
read_when:
  - Định nghĩa hoặc tái cấu trúc kiến trúc plugin
  - Chuyển đổi các trình kết nối kênh sang Plugin SDK/runtime
title: Refactor Plugin SDK
x-i18n:
  source_path: refactor\plugin-sdk.md
  source_hash: b9247cf0c555170862b2ed97ffba0f787fe15d176893bfe8f4bd38f4e94a87c9
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T20:18:08.990Z'
---

# Kế hoạch Tái cấu trúc Plugin SDK + Runtime

Mục tiêu: mọi trình kết nối nhắn tin là một plugin (được đóng gói hoặc bên ngoài) sử dụng một API ổn định.
Không có plugin nào nhập từ `src/**` trực tiếp. Tất cả các phụ thuộc đều đi qua SDK hoặc runtime.
## Tại sao bây giờ

- Các connector hiện tại trộn lẫn các mẫu: nhập core trực tiếp, cầu nối chỉ dist, và trợ giúp tùy chỉnh.
- Điều này làm cho nâng cấp dễ vỡ và chặn một bề mặt plugin bên ngoài sạch sẽ.
## Kiến trúc mục tiêu (hai lớp)

### 1) Plugin SDK (thời gian biên dịch, ổn định, có thể xuất bản)

Phạm vi: các kiểu, trợ giúp và tiện ích cấu hình. Không có trạng thái runtime, không có tác dụng phụ.

Nội dung (ví dụ):

- Các kiểu: `ChannelPlugin`, adapters, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
- Trợ giúp cấu hình: `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
- Trợ giúp ghép nối: `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
- Trợ giúp thiết lập ban đầu: `promptChannelAccessConfig`, `addWildcardAllowFrom`, các kiểu thiết lập ban đầu.
- Trợ giúp tham số công cụ: `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
- Trợ giúp liên kết tài liệu: `formatDocsLink`.

Phân phối:

- Xuất bản dưới dạng `openclaw/plugin-sdk` (hoặc xuất từ core dưới `openclaw/plugin-sdk`).
- Semver với các bảo đảm ổn định rõ ràng.

### 2) Plugin Runtime (bề mặt thực thi, được tiêm)

Phạm vi: mọi thứ liên quan đến hành vi runtime cốt lõi.
Được truy cập thông qua `OpenClawPluginApi.runtime` để các plugin không bao giờ nhập `src/**`.

Bề mặt được đề xuất (tối thiểu nhưng hoàn chỉnh):

```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: OpenClawConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: OpenClawConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: {
            text?: string;
            mediaUrls?: string[];
            mediaUrl?: string;
          }) => void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // adapter for Teams-style flows
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: RoutePeerKind; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: OpenClawConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
      ): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: OpenClawConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: OpenClawConfig): string;
  };
};
```
Ghi chú:

- Runtime là cách duy nhất để truy cập hành vi cốt lõi.
- SDK được thiết kế cố ý để nhỏ gọn và ổn định.
- Mỗi phương thức runtime ánh xạ tới một triển khai cốt lõi hiện có (không trùng lặp).
## Kế hoạch di chuyển (từng giai đoạn, an toàn)

### Giai đoạn 0: xây dựng cơ sở

- Giới thiệu `openclaw/plugin-sdk`.
- Thêm `api.runtime` vào `OpenClawPluginApi` với bề mặt ở trên.
- Duy trì các import hiện có trong cửa sổ chuyển đổi (cảnh báo không dùng nữa).

### Giai đoạn 1: dọn dẹp bridge (rủi ro thấp)

- Thay thế `core-bridge.ts` cho mỗi extension bằng `api.runtime`.
- Di chuyển BlueBubbles, Zalo, Zalo Personal trước (đã gần hoàn thành).
- Xóa mã bridge trùng lặp.

### Giai đoạn 2: plugin nhập trực tiếp nhẹ

- Di chuyển Matrix sang SDK + runtime.
- Xác thực logic thiết lập ban đầu, thư mục, đề cập nhóm.

### Giai đoạn 3: plugin nhập trực tiếp nặng

- Di chuyển MS Teams (bộ trợ giúp runtime lớn nhất).
- Đảm bảo ngữ nghĩa trả lời/gõ phù hợp với hành vi hiện tại.

### Giai đoạn 4: pluginization iMessage

- Di chuyển iMessage vào `extensions/imessage`.
- Thay thế các lệnh gọi core trực tiếp bằng `api.runtime`.
- Giữ nguyên các khóa cấu hình, hành vi CLI và tài liệu.

### Giai đoạn 5: thực thi

- Thêm quy tắc lint / kiểm tra CI: không có import `extensions/**` từ `src/**`.
- Thêm kiểm tra tương thích plugin SDK/phiên bản (runtime + SDK semver).
## Tương thích và phiên bản

- SDK: semver, được công bố, các thay đổi được ghi chép.
- Runtime: được phiên bản theo bản phát hành cốt lõi. Thêm `api.runtime.version`.
- Các plugin khai báo một phạm vi runtime bắt buộc (ví dụ: `openclawRuntime: ">=2026.2.0"`).
## Chiến lược kiểm thử

- Kiểm thử đơn vị cấp adapter (các hàm runtime được thực thi với triển khai core thực tế).
- Kiểm thử golden cho mỗi plugin: đảm bảo không có sự thay đổi hành vi (định tuyến, ghép nối, danh sách cho phép, gating đề cập).
- Một mẫu plugin end-to-end duy nhất được sử dụng trong CI (cài đặt + chạy + kiểm tra cơ bản).
## Các câu hỏi mở

- Nơi lưu trữ các loại SDK: gói riêng biệt hay xuất từ core?
- Phân phối loại runtime: trong SDK (chỉ các loại) hay trong core?
- Cách để hiển thị các liên kết tài liệu cho các plugin được đóng gói so với các plugin bên ngoài?
- Chúng ta có cho phép nhập core trực tiếp hạn chế cho các plugin trong kho lưu trữ trong quá trình chuyển đổi không?
## Tiêu chí thành công

- Tất cả các trình kết nối kênh là plugin sử dụng SDK + runtime.
- Không có `extensions/**` imports từ `src/**`.
- Các mẫu trình kết nối mới chỉ phụ thuộc vào SDK + runtime.
- Các plugin bên ngoài có thể được phát triển và cập nhật mà không cần truy cập vào mã nguồn cốt lõi.

Tài liệu liên quan: [Plugins](/tools/plugin), [Channels](/channels/index), [Configuration](/gateway/configuration).