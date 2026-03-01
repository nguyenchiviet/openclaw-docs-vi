---
x-i18n:
  source_path: design\kilo-gateway-integration.md
  source_hash: 6fc39ea9bc3c63e7912ec56b1b17ddfc1d1193a467abde9171b50d605bae3a53
  provider: anthropic
  model: claude-sonnet-4-20250514
  generated_at: '2026-02-27T19:52:44.667Z'
---

# Thiết kế Tích hợp Nhà cung cấp Kilo Gateway

## Tổng quan

Tài liệu này trình bày thiết kế để tích hợp "Kilo Gateway" như một nhà cung cấp hạng nhất trong OpenClaw, được mô hình hóa theo cách triển khai OpenRouter hiện có. Kilo Gateway sử dụng API hoàn thành tương thích với OpenAI với một URL cơ sở khác.
## Quyết định Thiết kế

### 1. Đặt tên Nhà cung cấp

**Khuyến nghị: `kilocode`**

Lý do:

- Phù hợp với ví dụ cấu hình người dùng được cung cấp (khóa nhà cung cấp `kilocode`)
- Nhất quán với các mẫu đặt tên nhà cung cấp hiện có (ví dụ: `openrouter`, `opencode`, `moonshot`)
- Ngắn gọn và dễ nhớ
- Tránh nhầm lẫn với các thuật ngữ chung như "kilo" hoặc "gateway"

Phương án thay thế được xem xét: `kilo-gateway` - bị từ chối vì tên có dấu gạch ngang ít phổ biến hơn trong codebase và `kilocode` ngắn gọn hơn.

### 2. Tham chiếu Mô hình Mặc định

**Khuyến nghị: `kilocode/anthropic/claude-opus-4.6`**

Lý do:

- Dựa trên ví dụ cấu hình người dùng
- Claude Opus 4.5 là một mô hình mặc định có khả năng
- Lựa chọn mô hình rõ ràng tránh phụ thuộc vào định tuyến tự động

### 3. Cấu hình URL Cơ sở

**Khuyến nghị: Mặc định được mã hóa cứng với ghi đè cấu hình**

- **URL Cơ sở Mặc định:** `https://api.kilo.ai/api/gateway/`
- **Có thể cấu hình:** Có, thông qua `models.providers.kilocode.baseUrl`

Điều này phù hợp với mẫu được sử dụng bởi các nhà cung cấp khác như Moonshot, Venice và Synthetic.

### 4. Quét Mô hình

**Khuyến nghị: Không có điểm cuối quét mô hình chuyên dụng ban đầu**

Lý do:

- Kilo Gateway proxy tới OpenRouter, vì vậy các mô hình là động
- Người dùng có thể cấu hình thủ công các mô hình trong cấu hình của họ
- Nếu Kilo Gateway hiển thị điểm cuối `/models` trong tương lai, có thể thêm quét

### 5. Xử lý Đặc biệt

**Khuyến nghị: Kế thừa hành vi OpenRouter cho các mô hình Anthropic**

Vì Kilo Gateway proxy tới OpenRouter, nên cùng một xử lý đặc biệt sẽ áp dụng:

- Tính đủ điều kiện TTL bộ nhớ đệm cho các mô hình `anthropic/*`
- Các tham số bổ sung (cacheControlTtl) cho các mô hình `anthropic/*`
- Chính sách transcript tuân theo các mẫu OpenRouter
## Các Tệp Cần Sửa Đổi

### Quản Lý Thông Tin Xác Thực Cốt Lõi

#### 1. `src/commands/onboard-auth.credentials.ts`

Thêm:

```typescript
export const KILOCODE_DEFAULT_MODEL_REF = "kilocode/anthropic/claude-opus-4.6";

export async function setKilocodeApiKey(key: string, agentDir?: string) {
  upsertAuthProfile({
    profileId: "kilocode:default",
    credential: {
      type: "api_key",
      provider: "kilocode",
      key,
    },
    agentDir: resolveAuthAgentDir(agentDir),
  });
}
```

#### 2. `src/agents/model-auth.ts`

Add to `envMap` in `resolveEnvApiKey()`:

```typescript
const envMap: Record<string, string> = {
  // ... existing entries
  kilocode: "KILOCODE_API_KEY",
};
```

#### 3. `src/config/io.ts`

Add to `SHELL_ENV_EXPECTED_KEYS`:

```typescript
const SHELL_ENV_EXPECTED_KEYS = [
  // ... existing entries
  "KILOCODE_API_KEY",
];
```

### Config Application

#### 4. `src/commands/onboard-auth.config-core.ts`

Add new functions:

```typescript
export const KILOCODE_BASE_URL = "https://api.kilo.ai/api/gateway/";

export function applyKilocodeProviderConfig(cfg: OpenClawConfig): OpenClawConfig {
  const models = { ...cfg.agents?.defaults?.models };
  models[KILOCODE_DEFAULT_MODEL_REF] = {
    ...models[KILOCODE_DEFAULT_MODEL_REF],
    alias: models[KILOCODE_DEFAULT_MODEL_REF]?.alias ?? "Kilo Gateway",
  };

  const providers = { ...cfg.models?.providers };
  const existingProvider = providers.kilocode;
  const { apiKey: existingApiKey, ...existingProviderRest } = (existingProvider ?? {}) as Record<
    string,
    unknown
  > as { apiKey?: string };
  const resolvedApiKey = typeof existingApiKey === "string" ? existingApiKey : undefined;
  const normalizedApiKey = resolvedApiKey?.trim();

  providers.kilocode = {
    ...existingProviderRest,
    baseUrl: KILOCODE_BASE_URL,
    api: "openai-completions",
    ...(normalizedApiKey ? { apiKey: normalizedApiKey } : {}),
  };

  return {
    ...cfg,
    agents: {
      ...cfg.agents,
      defaults: {
        ...cfg.agents?.defaults,
        models,
      },
    },
    models: {
      mode: cfg.models?.mode ?? "merge",
      providers,
    },
  };
}

export function applyKilocodeConfig(cfg: OpenClawConfig): OpenClawConfig {
  const next = applyKilocodeProviderConfig(cfg);
  const existingModel = next.agents?.defaults?.model;
  return {
    ...next,
    agents: {
      ...next.agents,
      defaults: {
        ...next.agents?.defaults,
        model: {
          ...(existingModel && "fallbacks" in (existingModel as Record<string, unknown>)
            ? {
                fallbacks: (existingModel as { fallbacks?: string[] }).fallbacks,
              }
            : undefined),
          primary: KILOCODE_DEFAULT_MODEL_REF,
        },
      },
    },
  };
}
```
### Hệ thống Lựa chọn Xác thực

#### 5. `src/commands/onboard-types.ts`

Thêm vào `AuthChoice` type:

```typescript
export type AuthChoice =
  // ... existing choices
  "kilocode-api-key";
// ...
```

Add to `OnboardOptions`:

```typescript
export type OnboardOptions = {
  // ... existing options
  kilocodeApiKey?: string;
  // ...
};
```

#### 6. `src/commands/auth-choice-options.ts`

Add to `AuthChoiceGroupId`:

```typescript
export type AuthChoiceGroupId =
  // ... existing groups
  "kilocode";
// ...
```

Add to `AUTH_CHOICE_GROUP_DEFS`:

```typescript
{
  value: "kilocode",
  label: "Kilo Gateway",
  hint: "API key (OpenRouter-compatible)",
  choices: ["kilocode-api-key"],
},
```

Add to `buildAuthChoiceOptions()`:

```typescript
options.push({
  value: "kilocode-api-key",
  label: "Kilo Gateway API key",
  hint: "OpenRouter-compatible gateway",
});
```

#### 7. `src/commands/auth-choice.preferred-provider.ts`

Thêm ánh xạ:
```typescript
const PREFERRED_PROVIDER_BY_AUTH_CHOICE: Partial<Record<AuthChoice, string>> = {
  // ... existing mappings
  "kilocode-api-key": "kilocode",
};
```

### Auth Choice Application

#### 8. `src/commands/auth-choice.apply.api-providers.ts`

Add import:

```typescript
import {
  // ... existing imports
  applyKilocodeConfig,
  applyKilocodeProviderConfig,
  KILOCODE_DEFAULT_MODEL_REF,
  setKilocodeApiKey,
} from "./onboard-auth.js";
```

Add handling for `kilocode-api-key`:

```typescript
if (authChoice === "kilocode-api-key") {
  const store = ensureAuthProfileStore(params.agentDir, {
    allowKeychainPrompt: false,
  });
  const profileOrder = resolveAuthProfileOrder({
    cfg: nextConfig,
    store,
    provider: "kilocode",
  });
  const existingProfileId = profileOrder.find((profileId) => Boolean(store.profiles[profileId]));
  const existingCred = existingProfileId ? store.profiles[existingProfileId] : undefined;
  let profileId = "kilocode:default";
  let mode: "api_key" | "oauth" | "token" = "api_key";
  let hasCredential = false;

  if (existingProfileId && existingCred?.type) {
    profileId = existingProfileId;
    mode =
      existingCred.type === "oauth" ? "oauth" : existingCred.type === "token" ? "token" : "api_key";
    hasCredential = true;
  }

  if (!hasCredential && params.opts?.token && params.opts?.tokenProvider === "kilocode") {
    await setKilocodeApiKey(normalizeApiKeyInput(params.opts.token), params.agentDir);
    hasCredential = true;
  }

  if (!hasCredential) {
    const envKey = resolveEnvApiKey("kilocode");
    if (envKey) {
      const useExisting = await params.prompter.confirm({
        message: `Sử dụng KILOCODE_API_KEY hiện có (${envKey.source}, ${formatApiKeyPreview(envKey.apiKey)})?`,
        initialValue: true,
      });
      if (useExisting) {
        await setKilocodeApiKey(envKey.apiKey, params.agentDir);
        hasCredential = true;
      }
    }
  }

  if (!hasCredential) {
    const key = await params.prompter.text({
      message: "Enter Kilo Gateway API key",
      validate: validateApiKeyInput,
    });
    await setKilocodeApiKey(normalizeApiKeyInput(String(key)), params.agentDir);
    hasCredential = true;
  }

  if (hasCredential) {
    nextConfig = applyAuthProfileConfig(nextConfig, {
      profileId,
      provider: "kilocode",
      mode,
    });
  }
  {
    const applied = await applyDefaultModelChoice({
      config: nextConfig,
      setDefaultModel: params.setDefaultModel,
      defaultModel: KILOCODE_DEFAULT_MODEL_REF,
      applyDefaultConfig: applyKilocodeConfig,
      applyProviderConfig: applyKilocodeProviderConfig,
      noteDefault: KILOCODE_DEFAULT_MODEL_REF,
      noteAgentModel,
      prompter: params.prompter,
    });
    nextConfig = applied.config;
    agentModelOverride = applied.agentModelOverride ?? agentModelOverride;
  }
  return { config: nextConfig, agentModelOverride };
}
```
Cũng thêm ánh xạ tokenProvider ở đầu hàm:

```typescript
if (params.opts.tokenProvider === "kilocode") {
  authChoice = "kilocode-api-key";
}
```

### CLI Registration

#### 9. `src/cli/program/register.onboard.ts`

Add CLI option:

```typescript
.option("--kilocode-api-key <key>", "Kilo Gateway API key")
```

Add to action handler:

```typescript
kilocodeApiKey: opts.kilocodeApiKey as string | undefined,
```

Update auth-choice help text:

```typescript
.option(
  "--auth-choice <choice>",
  "Auth: setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|kilocode-api-key|ai-gateway-api-key|...",
)
```

### Non-Interactive Onboarding

#### 10. `src/commands/onboard-non-interactive/local/auth-choice.ts`

Add handling for `kilocode-api-key`:

```typescript
if (authChoice === "kilocode-api-key") {
  const resolved = await resolveNonInteractiveApiKey({
    provider: "kilocode",
    cfg: baseConfig,
    flagValue: opts.kilocodeApiKey,
    flagName: "--kilocode-api-key",
    envVar: "KILOCODE_API_KEY",
  });
  await setKilocodeApiKey(resolved.apiKey, agentDir);
  nextConfig = applyAuthProfileConfig(nextConfig, {
    profileId: "kilocode:default",
    provider: "kilocode",
    mode: "api_key",
  });
  // ... apply default model
}
```

### Cập nhật Xuất
#### 11. `src/commands/onboard-auth.ts`

Thêm các export:

```typescript
export {
  // ... existing exports
  applyKilocodeConfig,
  applyKilocodeProviderConfig,
  KILOCODE_BASE_URL,
} from "./onboard-auth.config-core.js";

export {
  // ... existing exports
  KILOCODE_DEFAULT_MODEL_REF,
  setKilocodeApiKey,
} from "./onboard-auth.credentials.js";
```

### Special Handling (Optional)

#### 12. `src/agents/pi-embedded-runner/cache-ttl.ts`

Add Kilo Gateway support for Anthropic models:

```typescript
export function isCacheTtlEligibleProvider(provider: string, modelId: string): boolean {
  const normalizedProvider = provider.toLowerCase();
  const normalizedModelId = modelId.toLowerCase();
  if (normalizedProvider === "anthropic") return true;
  if (normalizedProvider === "openrouter" && normalizedModelId.startsWith("anthropic/"))
    return true;
  if (normalizedProvider === "kilocode" && normalizedModelId.startsWith("anthropic/")) return true;
  return false;
}
```

#### 13. `src/agents/transcript-policy.ts`

Add Kilo Gateway handling (similar to OpenRouter):

```typescript
const isKilocodeGemini = provider === "kilocode" && modelId.toLowerCase().includes("gemini");

// Include in needsNonImageSanitize check
const needsNonImageSanitize =
  isGoogle || isAnthropic || isMistral || isOpenRouterGemini || isKilocodeGemini;
```
## Cấu trúc Cấu hình

### Ví dụ Cấu hình Người dùng

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "kilocode": {
        "baseUrl": "https://api.kilo.ai/api/gateway/",
        "apiKey": "xxxxx",
        "api": "openai-completions",
        "models": [
          {
            "id": "anthropic/claude-opus-4.6",
            "name": "Anthropic: Claude Opus 4.6"
          },
          { "id": "minimax/minimax-m2.1:free", "name": "Minimax: Minimax M2.1" }
        ]
      }
    }
  }
}
```

### Auth Profile Structure

```json
{
  "profiles": {
    "kilocode:default": {
      "type": "api_key",
      "provider": "kilocode",
      "key": "xxxxx"
    }
  }
}
```
## Các Xem Xét Về Kiểm Thử

1. **Kiểm Thử Đơn Vị:**
   - Kiểm thử `setKilocodeApiKey()` ghi hồ sơ chính xác
   - Kiểm thử `applyKilocodeConfig()` đặt các giá trị mặc định chính xác
   - Kiểm thử `resolveEnvApiKey("kilocode")` trả về biến môi trường chính xác

2. **Kiểm Thử Tích Hợp:**
   - Kiểm thử luồng thiết lập ban đầu với `--auth-choice kilocode-api-key`
   - Kiểm thử thiết lập ban đầu không tương tác với `--kilocode-api-key`
   - Kiểm thử lựa chọn mô hình với tiền tố `kilocode/`

3. **Kiểm Thử E2E:**
   - Kiểm thử các lệnh gọi API thực tế thông qua Gateway Kilo (kiểm thử trực tiếp)
## Ghi chú về Quá trình Chuyển đổi

- Không cần quá trình chuyển đổi cho người dùng hiện tại
- Người dùng mới có thể sử dụng ngay lựa chọn xác thực `kilocode-api-key`
- Cấu hình thủ công hiện tại với nhà cung cấp `kilocode` sẽ tiếp tục hoạt động
## Những Cân Nhắc Trong Tương Lai

1. **Danh Mục Mô Hình:** Nếu Kilo Gateway công khai một điểm cuối `/models`, hãy thêm hỗ trợ quét tương tự như `scanOpenRouterModels()`

2. **Hỗ Trợ OAuth:** Nếu Kilo Gateway thêm OAuth, hãy mở rộng hệ thống xác thực tương ứng

3. **Giới Hạn Tốc Độ:** Cân nhắc thêm xử lý giới hạn tốc độ cụ thể cho Kilo Gateway nếu cần

4. **Tài Liệu:** Thêm tài liệu tại `docs/providers/kilocode.md` giải thích thiết lập và cách sử dụng
## Tóm tắt các thay đổi

| Tệp                                                        | Loại thay đổi | Mô tả                                                             |
| ----------------------------------------------------------- | ----------- | ----------------------------------------------------------------------- |
| `src/commands/onboard-auth.credentials.ts`                  | Thêm         | `KILOCODE_DEFAULT_MODEL_REF`, `setKilocodeApiKey()`                     |
| `src/agents/model-auth.ts`                                  | Sửa đổi      | Thêm `kilocode` vào `envMap`                                              |
| `src/config/io.ts`                                          | Sửa đổi      | Thêm `KILOCODE_API_KEY` vào các khóa shell env                                |
| `src/commands/onboard-auth.config-core.ts`                  | Thêm         | `applyKilocodeProviderConfig()`, `applyKilocodeConfig()`                |
| `src/commands/onboard-types.ts`                             | Sửa đổi      | Thêm `kilocode-api-key` vào `AuthChoice`, thêm `kilocodeApiKey` vào các tùy chọn |
| `src/commands/auth-choice-options.ts`                       | Sửa đổi      | Thêm nhóm `kilocode` và tùy chọn                                         |
| `src/commands/auth-choice.preferred-provider.ts`            | Sửa đổi      | Thêm ánh xạ `kilocode-api-key`                                          |
| `src/commands/auth-choice.apply.api-providers.ts`           | Sửa đổi      | Thêm xử lý `kilocode-api-key`                                         |
| `src/cli/program/register.onboard.ts`                       | Sửa đổi      | Thêm tùy chọn `--kilocode-api-key`                                         |
| `src/commands/onboard-non-interactive/local/auth-choice.ts` | Sửa đổi      | Thêm xử lý không tương tác                                            |
| `src/commands/onboard-auth.ts`                              | Sửa đổi      | Xuất các hàm mới                                                    |
| `src/agents/pi-embedded-runner/cache-ttl.ts`                | Sửa đổi      | Thêm hỗ trợ kilocode                                                    |
| `src/agents/transcript-policy.ts`                           | Sửa đổi      | Thêm xử lý kilocode Gemini                                            |