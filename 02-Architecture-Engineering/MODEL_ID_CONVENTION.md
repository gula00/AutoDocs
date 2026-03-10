# 后端模型 ID 配置规范

本文档定义后端（Nacos / 模型代理）配置模型时 `id` 字段的命名规则。
客户端网关在启动时会对模型 ID 做清洗（见 `GATEWAY_PATCHES.md` #5），将内部 ID 转换为面向用户的展示名。配置时必须遵循以下规则，否则展示名会出错。

---

## 清洗规则

客户端对模型 ID 执行两步裁切：

```
原始 ID           ──step 1──>    去掉 provider    ──step 2──>    去掉 routing 前缀
zai/huawei_glm-5  ──────────>    huawei_glm-5     ──────────>    glm-5
```

1. **去 provider 前缀**：以第一个 `/` 为分隔，取后半段
2. **去 routing 前缀**：以第一个 `_` 为分隔，取后半段

> 两步都是"找第一个分隔符"，不是最后一个。

---

## 模型 ID 格式

```
{provider}/{routingPrefix}_{displayName}
```

| 段 | 说明 | 示例 |
|---|---|---|
| `provider` | 模型提供方标识，客户端用于路由，不展示给用户 | `zai` |
| `routingPrefix` | 内部路由标识（机房/网关/渠道），不展示给用户 | `huawei`、`apigateway` |
| `displayName` | 用户最终看到的模型名称 | `glm-5`、`pony-alpha-2` |

### 完整示例

| 后端配置的 ID | 用户看到的名称 |
|---|---|
| `zai/huawei_glm-5` | `glm-5` |
| `zai/huawei_glm-4-plus` | `glm-4-plus` |
| `zai/apigateway_pony-alpha-2` | `pony-alpha-2` |
| `zai/huawei_glm-4.7-flash` | `glm-4.7-flash` |
| `zai/openai_gpt-4o` | `gpt-4o` |

---

## 命名约束

### 1. `displayName` 中禁止使用下划线

清洗用第一个 `_` 做分割。如果 `displayName` 也含 `_`，不会被误裁（只裁第一个），**但前提是有 `routingPrefix` 存在**。

| 场景 | ID | 展示名 | 是否正确 |
|---|---|---|---|
| 有 routing 前缀 | `zai/huawei_glm_4_plus` | `glm_4_plus` | 正确，但不推荐 |
| **无 routing 前缀** | `zai/gpt_4o` | `4o` | **错误！被误裁** |

**规则：展示名一律用 `-`（连字符）连接，不用 `_`（下划线）。**

```
glm-4-plus     ✅
glm_4_plus     ❌ 有风险
```

### 2. `routingPrefix` 中禁止使用下划线

routing 前缀本身不能含 `_`，否则裁切位置会错：

| ID | 预期展示名 | 实际展示名 |
|---|---|---|
| `zai/api_gateway_glm-5` | `glm-5` | `gateway_glm-5` ❌ |

**规则：routing 前缀用纯字母或字母+数字，不含 `_`。**

```
huawei       ✅
apigateway   ✅
api_gateway  ❌
```

### 3. `provider` 中禁止使用 `/`

provider 前缀不能含 `/`，否则裁切位置会错。

### 4. 无 routing 前缀时可以省略

如果不需要内部路由标识，直接写 `{provider}/{displayName}`：

| ID | 展示名 |
|---|---|
| `zai/glm-5` | `glm-5` |
| `zai/deepseek-r1` | `deepseek-r1` |

此时 `displayName` **绝对不能包含 `_`**。

---

## 第三方模型（用户自配）

用户通过设置页面添加的第三方模型（如 `anthropic/claude-3.5-sonnet`、`openai/gpt-4o`）也会经过同样的清洗，但因为这些模型名称天然不含 `_` routing 前缀，所以不受影响：

| ID | 清洗后 |
|---|---|
| `anthropic/claude-3.5-sonnet` | `claude-3.5-sonnet` |
| `openai/gpt-4o` | `gpt-4o` |
| `google/gemini-2.0-flash` | `gemini-2.0-flash` |
| `deepseek/deepseek-r1` | `deepseek-r1` |

---

## 清洗生效的位置

清洗只作用于两处面向 LLM 的输出，不影响 API 调用和路由：

| 位置 | 原始值 | 清洗后 |
|---|---|---|
| Agent system prompt `## Runtime` 段的 `model=` | `model=zai/huawei_glm-5` | `model=glm-5` |
| `session_status` 工具结果的 `🧠 Model:` 行 | `🧠 Model: zai/huawei_glm-5` | `🧠 Model: glm-5` |

其他地方（API 请求 body、header、日志等）仍使用完整的内部 ID，不受影响。

---

## 检查清单

配置新模型时，按以下步骤验证：

1. **手动模拟裁切**：对你的 ID 执行两步规则，确认展示名正确
2. **启动客户端**：`npm run dev`，查看日志确认 `Patched N dist file(s)` 出现
3. **问模型**：在对话中问"你是什么模型？"，确认回答中不含内部前缀
4. **检查 session_status**：确认 `🧠 Model:` 行显示的是干净的展示名

---

## 相关文档

- `docs/GATEWAY_PATCHES.md` — 网关运行时补丁详细说明
- `docs/nacos-config-reference.md` — Nacos 配置端点参考（`autoclaw-model-config` 等）
- `src/main/gateway/manager.ts` — `patchGatewayDistModelIdentity()` 实现
