# Gateway Runtime Patches

本文档记录了 `src/main/gateway/manager.ts` 中对 bundled gateway 文件的所有运行时 patch。

这些 patch 在 gateway 进程启动前（`startGateway()` 中）执行，通过文本替换修改 `resources/gateway/openclaw/` 下的 JS 文件。**每次升级 gateway 版本后需要验证这些 patch 是否仍然生效。**

---

## 概览

| # | 函数名 | 目标文件 | 作用 | 幂等标记 |
|---|--------|----------|------|----------|
| 1 | `patchPiAiModelsBaseUrl` | `node_modules/@mariozechner/pi-ai/dist/models.generated.js` | 将 zai 模型的 baseUrl 指向内部代理或 coding plan 端点 | URL 比对 |
| 2 | `patchOpenAICompletionsRequestId` | `node_modules/@mariozechner/pi-ai/dist/providers/openai-completions.js` | 每个 API 请求注入动态 `X-Request-Id`（UUID）和 `X-Request-Model` header | `content.includes('X-Request-Id')` |
| 3 | `patchOpenAICompletionsModelPrefix` | `node_modules/@mariozechner/pi-ai/dist/providers/openai-completions.js` | request body 的 `model` 字段去掉路由前缀（如 `huawei_glm-5` → `glm-5`），header 中保留完整 ID | `content.includes('.replace(/^[a-z]+_/')` |
| 4 | `patchPiAiModelsAuthHeader` | `node_modules/@mariozechner/pi-ai/dist/models.generated.js` | 为 zai 模型注入 `X-Authorization` 和 `X-Request-Model` header | 每次根据 token 状态重写 |
| 5 | `patchGatewayDistModelIdentity` | `dist/*.js` + `dist/plugin-sdk/*.js` | 清洗两处模型身份泄露点：(A) system prompt 的 `buildRuntimeLine`、(B) `buildStatusMessage` 的 modelLine。去掉 `provider/` 前缀和 `xxx_` 路由前缀（如 `zai/huawei_glm-5` → `glm-5`） | `/* model-identity-sanitised */` |

> 以上路径均相对于 `resources/gateway/openclaw/`。

---

## 详细说明

### 1. `patchPiAiModelsBaseUrl`

**目的**：将 pi-ai 内置模型注册表中 zai 模型的 `baseUrl` 从官方端点重定向到内部代理。

**改动**：
- 将所有已知的官方 URL（`open.bigmodel.cn`、`api.z.ai` 等）统一替换为 `ZAI_PROXY_BASE_URL`
- 如果用户选择了 zhipu-coding plan，则替换为 `https://open.bigmodel.cn/api/coding/paas/v4`

**升级注意**：新版 gateway 如果新增了 zai 模型或更换了 URL 格式，需要更新 `URLS_TO_REPLACE` 列表。

---

### 2. `patchOpenAICompletionsRequestId`

**目的**：后端代理需要每个请求有唯一的 trace ID 和模型标识。

**改动**：
```
// 原始
const headers = { ...model.headers };

// patch 后
import { randomUUID } from 'node:crypto';
const headers = { ...model.headers, "X-Request-Id": randomUUID(), "X-Request-Model": model.id };
```

**升级注意**：如果 `openai-completions.js` 中 `createClient()` 的 headers 构造方式变化，需要调整匹配字符串。

---

### 3. `patchOpenAICompletionsModelPrefix`

**目的**：后端代理用 `X-Request-Model` header（保留完整 ID 如 `huawei_glm-5`）做路由，但上游模型 API 只接受基础名称（如 `glm-5`）。

**改动**：
```
// 原始（buildParams 中）
model: model.id,

// patch 后
model: model.id.replace(/^[a-z]+_/, ''),
```

**升级注意**：依赖 `buildParams()` 函数中 `model: model.id,\n    messages,\n    stream: true,` 的固定格式。如果函数结构变化需要调整正则。

---

### 4. `patchPiAiModelsAuthHeader`

**目的**：pi-ai 内置模型注册表（`models.generated.js`）中的 zai 模型需要携带认证 header，但这些模型不从 `openclaw.json` 加载，所以必须直接 patch 生成文件。

**改动**：
- 为每个 `provider: "zai"` 的模型块注入 `X-Authorization: Bearer <token>` 和 `X-Request-Model: <modelId>`
- token 变更时会重新执行（见 `patchPiAiModelsAuthHeader` 在 token refresh 时的调用）

**升级注意**：依赖 `models.generated.js` 中 zai 模型块的 `provider: "zai"` 和 `headers: { ... }` 结构。

---

### 5. `patchGatewayDistModelIdentity`

**目的**：gateway 有两个地方会把 `provider/model`（如 `zai/huawei_glm-5`）原样暴露给 LLM，导致模型知道并泄露自身真实 ID。此 patch 对这两处做清洗，只保留 `_` 后面的真实模型名。

**清洗逻辑**（通过注入 IIFE 在运行时执行）：
1. 去掉 `/` 前面的 provider 前缀（`zai/huawei_glm-5` → `huawei_glm-5`）
2. 去掉 `_` 前面的路由前缀（`huawei_glm-5` → `glm-5`）

**泄露点与改动**：

**(A) `buildRuntimeLine()` — system prompt 的 `## Runtime` 部分**
```js
// 原始
runtimeInfo?.model ? `model=${runtimeInfo.model}` : ""
runtimeInfo?.defaultModel ? `default_model=${runtimeInfo.defaultModel}` : ""

// patch 后 → model=glm-5（去掉 zai/ 和 huawei_）
runtimeInfo?.model ? `model=${sanitise(runtimeInfo.model)}` : ""
runtimeInfo?.defaultModel ? `default_model=${sanitise(runtimeInfo.defaultModel)}` : ""
```

**(B) `buildStatusMessage()` — `session_status` 工具返回的状态卡片**
```js
// 原始
const modelLine = `🧠 Model: ${model ? `${provider}/${model}` : "unknown"}...`

// patch 后 → 🧠 Model: glm-5（去掉 provider/ 和 xxx_ 前缀）
const modelLine = `🧠 Model: ${model ? sanitise(model) : "unknown"}...`
```

> `sanitise` 是一个 IIFE：`((v) => { strip "/" prefix, then strip "_" prefix })(value)`

**影响的文件**（截至当前版本）：
- `dist/pi-embedded-Cn8f5u97.js`
- `dist/pi-embedded-CHb5giY2.js`
- `dist/reply-B4B0jUCM.js`
- `dist/subagent-registry-DOZpiiys.js`
- `dist/plugin-sdk/reply-Bsg9j6AP.js`

> 文件名中的 hash 部分会随 gateway 版本变化。patch 通过扫描 `dist/` + `dist/plugin-sdk/` 中所有 `.js` 文件来自动适配，不依赖固定文件名。

**升级注意**：如果 `buildRuntimeLine` 或 `buildStatusMessage` 中的模板字符串格式变化，需要调整对应的正则。

---

## 升级 Gateway 的 Checklist

1. 替换 `resources/gateway/openclaw/` 下的文件为新版本
2. 启动 app（`npm run dev`），检查日志中是否有 patch 成功的 log：
   ```
   [GatewayManager] Patched pi-ai models.generated.js: zai baseUrl → ...
   [GatewayManager] Patched openai-completions.js with dynamic X-Request-Id and X-Request-Model
   [GatewayManager] Patched openai-completions.js: strip routing prefix from request body model field
   [GatewayManager] Patched headers into models.generated.js
   [GatewayManager] Patched N dist file(s): sanitised model identity in system prompt and session_status
   ```
3. 如果有 patch 失败的 log（`Failed to patch ...`），需要检查对应文件的结构是否变化并调整 patch 逻辑
4. 功能验证：
   - 模型 API 请求 header 中包含 `X-Request-Id`、`X-Request-Model`、`X-Authorization`
   - 请求 body 中 `model` 字段不含路由前缀
   - 问模型"你是什么模型"或触发 `session_status` 工具，回复中应显示清洗后的名称（如 `glm-5`），不应出现 provider 前缀或路由前缀

---

## 执行时序

在 `startGateway()` 中，所有 patch 在 gateway 进程 spawn 之前按以下顺序执行：

```
patchPiAiModelsBaseUrl()            // 1. baseUrl 重定向
patchOpenAICompletionsRequestId()   // 2. 注入 request header
patchOpenAICompletionsModelPrefix() // 3. model body 去前缀
patchGatewayDistModelIdentity()     // 4. 清洗 system prompt 和 session_status 中的模型 ID
patchPiAiModelsAuthHeader()         // 5. 注入认证 header
```

Patch #4 和 #5 之间没有顺序依赖，但保持当前顺序即可。

---

*最后更新：2026-03-09*
