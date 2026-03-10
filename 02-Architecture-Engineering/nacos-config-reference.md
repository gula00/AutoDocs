# AutoClaw Nacos 配置参考

> 每个 DataId 对应一个配置端点，客户端通过 `https://{host}/autoclaw-proxy/proxy/{DataId}` 获取。
> 格式统一为 JSON，所有 key 为 String 类型。
> 后端返回 `{}` 时客户端会使用内置回退值，所以可以按需只配部分端点。

---

## 1. `autoclaw-model-config`

控制客户端可用的 ZhipuAI 模型列表。

```json
{
  "models": [
    {
      "id": "glm-4.7",
      "name": "GLM-4.7",
      "reasoning": true,
      "input": ["text"],
      "contextWindow": 204800,
      "maxTokens": 131072
    },
    {
      "id": "glm-4.7-flashx",
      "name": "GLM-4.7-FlashX",
      "reasoning": true,
      "input": ["text"],
      "contextWindow": 204800,
      "maxTokens": 131072
    },
    {
      "id": "glm-4.7-flash",
      "name": "GLM-4.7-Flash",
      "reasoning": true,
      "input": ["text"],
      "contextWindow": 204800,
      "maxTokens": 131072
    },
    {
      "id": "glm-5",
      "name": "GLM-5",
      "reasoning": true,
      "input": ["text"],
      "contextWindow": 204800,
      "maxTokens": 131072
    }
  ]
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 模型 ID，用于 API 调用 |
| `name` | string | 否 | 显示名称，默认用 id |
| `reasoning` | boolean | 否 | 是否为推理模型 |
| `input` | string[] | 否 | 支持的输入类型，如 `["text"]`、`["text", "image"]` |
| `contextWindow` | number | 否 | 上下文窗口大小（tokens） |
| `maxTokens` | number | 否 | 最大输出 tokens |

---

## 2. `autoclaw-provider-config`

控制第三方 LLM 提供商的 baseUrl、API 协议和环境变量映射。

```json
{
  "providers": {
    "zhipu": {
      "baseUrl": "https://open.bigmodel.cn/api/paas/v4",
      "api": "openai-completions",
      "envVar": "ZAI_API_KEY"
    },
    "openai": {
      "baseUrl": "https://api.openai.com/v1",
      "api": "openai-completions",
      "envVar": "OPENAI_API_KEY"
    },
    "anthropic": {
      "baseUrl": "https://api.anthropic.com",
      "api": "anthropic",
      "envVar": "ANTHROPIC_API_KEY"
    },
    "google": {
      "baseUrl": "https://generativelanguage.googleapis.com/v1beta",
      "api": "google",
      "envVar": "GEMINI_API_KEY"
    },
    "gemini": {
      "baseUrl": "https://generativelanguage.googleapis.com/v1beta",
      "api": "google",
      "envVar": "GEMINI_API_KEY"
    },
    "deepseek": {
      "baseUrl": "https://api.deepseek.com",
      "api": "openai-completions",
      "envVar": "DEEPSEEK_API_KEY"
    },
    "groq": {
      "baseUrl": "https://api.groq.com/openai/v1",
      "api": "openai-completions",
      "envVar": "GROQ_API_KEY"
    },
    "mistral": {
      "baseUrl": "https://api.mistral.ai/v1",
      "api": "openai-completions",
      "envVar": "MISTRAL_API_KEY"
    },
    "xai": {
      "baseUrl": "https://api.x.ai/v1",
      "api": "openai-completions",
      "envVar": "XAI_API_KEY"
    },
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "envVar": "OLLAMA_API_KEY"
    },
    "openrouter": {
      "baseUrl": "https://openrouter.ai/api/v1",
      "api": "openai-completions",
      "envVar": "OPENROUTER_API_KEY"
    },
    "together": {
      "baseUrl": "https://api.together.xyz/v1",
      "api": "openai-completions",
      "envVar": "TOGETHER_API_KEY"
    },
    "minimax": {
      "baseUrl": "https://api.minimax.chat/v1/text/chatcompletion_v2",
      "api": "openai-completions",
      "envVar": "MINIMAX_API_KEY"
    }
  }
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `baseUrl` | string | 是 | 提供商 API 基础 URL |
| `api` | string | 是 | API 协议类型：`"openai-completions"` / `"anthropic"` / `"google"` |
| `envVar` | string | 否 | 对应的环境变量名（用于 API Key 传递） |

> **增删提供商：** 直接增减 providers 对象的 key 即可。key 是提供商标识符（小写），值是配置对象。

---

## 3. `autoclaw-model-pricing`

按模型 ID 配置定价（单位：美元 / 百万 tokens）。用于客户端显示费用估算。

```json
{
  "pricing": {
    "claude-opus-4": { "input": 15, "output": 75 },
    "claude-sonnet-4": { "input": 3, "output": 15 },
    "claude-3-5-sonnet": { "input": 3, "output": 15 },
    "claude-3-5-haiku": { "input": 0.8, "output": 4 },
    "claude-3-haiku": { "input": 0.25, "output": 1.25 },
    "claude-3-opus": { "input": 15, "output": 75 },
    "gpt-4o": { "input": 2.5, "output": 10 },
    "gpt-4o-mini": { "input": 0.15, "output": 0.6 },
    "gpt-4-turbo": { "input": 10, "output": 30 },
    "gpt-4": { "input": 30, "output": 60 },
    "o1-pro": { "input": 150, "output": 600 },
    "o1": { "input": 15, "output": 60 },
    "o3": { "input": 10, "output": 40 },
    "o3-mini": { "input": 1.1, "output": 4.4 },
    "o4-mini": { "input": 1.1, "output": 4.4 },
    "deepseek-r1": { "input": 0.55, "output": 2.19 },
    "deepseek-chat": { "input": 0.27, "output": 1.1 },
    "deepseek-v3": { "input": 0.27, "output": 1.1 },
    "gemini-2.5-pro": { "input": 1.25, "output": 10 },
    "gemini-2.5-flash": { "input": 0.15, "output": 0.6 },
    "gemini-2.0-flash": { "input": 0.1, "output": 0.4 },
    "gemini-1.5-pro": { "input": 1.25, "output": 5 },
    "glm-5": { "input": 0.28, "output": 0.28 },
    "glm-4.7-flash": { "input": 0, "output": 0 },
    "glm-4.7": { "input": 0.07, "output": 0.07 },
    "glm-4-plus": { "input": 0.7, "output": 0.7 },
    "glm-4": { "input": 0.14, "output": 0.14 },
    "glm-4-flash": { "input": 0, "output": 0 },
    "glm-z1-plus": { "input": 0.7, "output": 0.7 },
    "glm-z1-flash": { "input": 0, "output": 0 },
    "MiniMax-M2.5": { "input": 0.29, "output": 1.16 },
    "MiniMax-M2": { "input": 0.29, "output": 1.16 },
    "MiniMax-M1": { "input": 0.29, "output": 1.16 },
    "qwen-max": { "input": 1.6, "output": 6.4 },
    "qwen-plus": { "input": 0.3, "output": 0.6 },
    "qwen-turbo": { "input": 0.04, "output": 0.12 }
  }
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `input` | number | 是 | 输入价格（USD / 1M tokens） |
| `output` | number | 是 | 输出价格（USD / 1M tokens） |
| `cacheRead` | number | 否 | 缓存读取价格 |
| `cacheWrite` | number | 否 | 缓存写入价格 |

> **key 是模型 ID**，需要与用户实际使用的模型 ID 匹配（支持前缀匹配）。

---

## 4. `autoclaw-subscription`

控制订阅方案（决定 ZhipuAI 模型的访问方式和 baseUrl）。

```json
{
  "plans": {
    "default": {
      "name": "默认方案",
      "description": "通过内部代理访问 ZhipuAI 模型，无需 API Key"
    },
    "zhipu-coding": {
      "name": "智谱 Coding 方案",
      "baseUrl": "https://open.bigmodel.cn/api/coding/paas/v4",
      "apiKeyRequired": true,
      "description": "使用个人 ZhipuAI Coding API Key 访问"
    }
  }
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 方案显示名称 |
| `baseUrl` | string | 否 | 自定义 API baseUrl（不填则走内部代理） |
| `apiKeyRequired` | boolean | 否 | 是否需要用户提供 API Key |
| `description` | string | 否 | 方案描述 |

> **key 是方案 ID**。`"default"` 是默认方案，必须存在。

---

## 5. `autoclaw-builtin-skills`

控制客户端启动时部署哪些内置技能到工作区。

```json
{
  "skills": [
    {
      "name": "zhipu-web-search",
      "description": "ZhipuAI Web Search",
      "bundled": true
    },
    {
      "name": "find-skills",
      "description": "Skills discovery helper",
      "bundled": true
    },
    {
      "name": "feishu-screenshot",
      "description": "Feishu screenshot",
      "bundled": true
    },
    {
      "name": "feishu-chat-history",
      "description": "Feishu chat history",
      "bundled": true
    },
    {
      "name": "feishu-send-file",
      "description": "Feishu file sending",
      "bundled": true
    }
  ]
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 技能目录名，需与 `resources/skills/{name}/` 一致 |
| `description` | string | 是 | 技能描述 |
| `bundled` | boolean | 否 | 是否为应用内置技能（默认 true） |
| `source` | string | 否 | 外部安装来源（预留，如 ClawHub URL） |

> **控制部署范围：** 从列表中移除一个技能 = 该技能不再自动部署。设置 `"bundled": false` 也会跳过部署。

---

## 6. `autoclaw-error-mapping`

配置错误信息映射规则，将原始错误码/文本转换为用户友好的提示。

```json
{
  "mappings": [
    {
      "pattern": "99991672",
      "message": "飞书 Bot 未启用或配置错误",
      "isRegex": false
    },
    {
      "pattern": "99991663",
      "message": "飞书接口调用频率超限",
      "isRegex": false
    },
    {
      "pattern": "99991664",
      "message": "飞书接口调用频率超限",
      "isRegex": false
    },
    {
      "pattern": "99991668",
      "message": "飞书消息发送过于频繁",
      "isRegex": false
    },
    {
      "pattern": "99991401",
      "message": "飞书 Token 过期，请重新配置",
      "isRegex": false
    },
    {
      "pattern": "ECONNREFUSED|ENOTFOUND|EHOSTUNREACH|EAI_AGAIN",
      "message": "网络连接失败，请检查网络设置",
      "isRegex": true
    },
    {
      "pattern": "HTTP 401|Unauthorized|Invalid token|authentication",
      "message": "API 密钥无效或已过期，请在设置中更新",
      "isRegex": true
    },
    {
      "pattern": "HTTP 429|rate limit|too many requests",
      "message": "请求频率过高，请稍后再试",
      "isRegex": true
    },
    {
      "pattern": "HTTP 402|quota|insufficient_balance|billing",
      "message": "账户额度不足，请充值后重试",
      "isRegex": true
    },
    {
      "pattern": "timed? ?out|ETIMEDOUT|ESOCKETTIMEDOUT|deadline exceeded",
      "message": "请求超时，请检查网络后重试",
      "isRegex": true
    },
    {
      "pattern": "HTTP 5\\d{2}|Internal Server Error|Bad Gateway|Service Unavailable",
      "message": "服务暂时不可用，请稍后重试",
      "isRegex": true
    },
    {
      "pattern": "context[_ ]?length|too long|max.*token",
      "message": "输入内容过长，请缩短后重试",
      "isRegex": true
    }
  ]
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `pattern` | string | 是 | 匹配模式（字符串或正则） |
| `message` | string | 是 | 用户友好的中文提示 |
| `message_en` | string | 否 | 英文提示（可选） |
| `isRegex` | boolean | 否 | 是否为正则表达式（默认 true） |

> 匹配顺序为数组顺序，第一个命中的规则生效。

---

## 7. `autoclaw-mcp-service-templates`

控制设置页面 MCP 快速添加模板（standard = 标准模板区，internet-access = Agent Reach 区）。

```json
{
  "templates": [
    {
      "name": "filesystem",
      "description": "Read/write local files",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "~/Desktop"],
      "category": "standard"
    },
    {
      "name": "brave-search",
      "description": "Web search via Brave",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "category": "standard"
    },
    {
      "name": "sqlite",
      "description": "Query SQLite databases",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite", "~/test.db"],
      "category": "standard"
    },
    {
      "name": "fetch",
      "description": "Fetch web content",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"],
      "category": "standard"
    },
    {
      "name": "xiaohongshu",
      "description": "XiaoHongShu (小红书)",
      "transport": "http",
      "url": "http://localhost:3001/mcp",
      "category": "internet-access"
    },
    {
      "name": "exa-search",
      "description": "Exa AI Search",
      "transport": "stdio",
      "command": "npx",
      "args": ["-y", "exa-mcp-server"],
      "category": "internet-access"
    },
    {
      "name": "linkedin",
      "description": "LinkedIn",
      "transport": "http",
      "url": "http://localhost:3002/mcp",
      "category": "internet-access"
    },
    {
      "name": "boss-zhipin",
      "description": "Boss直聘",
      "transport": "http",
      "url": "http://localhost:3003/mcp",
      "category": "internet-access"
    }
  ]
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | MCP 服务名称 |
| `description` | string | 是 | 按钮显示文本 |
| `transport` | string | 是 | `"stdio"` 或 `"http"` |
| `command` | string | 条件 | stdio 类型必填，启动命令 |
| `args` | string[] | 否 | stdio 类型的命令参数 |
| `url` | string | 条件 | http 类型必填，服务 URL |
| `category` | string | 否 | `"standard"` = 标准模板区，`"internet-access"` = Agent Reach 区 |

> **增删模板：** 直接增减数组元素。新增的模板会立即出现在对应区域。

---

## 8. `autoclaw-sensitive-data-rules`

配置敏感数据检测和脱敏规则，用于聊天消息和工具调用结果的自动脱敏。

```json
{
  "rules": [
    {
      "name": "api-key-prefixes",
      "pattern": "(sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36,}|gho_[a-zA-Z0-9]{36,}|xox[bpras]-[a-zA-Z0-9-]{10,}|glpat-[a-zA-Z0-9-]{20,}|AIza[a-zA-Z0-9_-]{35})",
      "replacement": "***REDACTED_KEY***",
      "flags": "g"
    },
    {
      "name": "bearer-token",
      "pattern": "(Bearer\\s+)[a-zA-Z0-9._\\-/+]{20,}",
      "replacement": "$1***REDACTED_TOKEN***",
      "flags": "gi"
    },
    {
      "name": "json-secrets",
      "pattern": "(\"(?:api[_-]?key|secret[_-]?key|access[_-]?token|password|credential|private[_-]?key|app[_-]?secret|client[_-]?secret|signing[_-]?key)\"\\s*:\\s*\")[^\"]{8,}(\")",
      "replacement": "$1***REDACTED***$2",
      "flags": "gi"
    },
    {
      "name": "aws-access-key",
      "pattern": "AKIA[0-9A-Z]{16}",
      "replacement": "***REDACTED_AWS_KEY***",
      "flags": "g"
    }
  ]
}
```

**字段说明：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 规则名称（用于日志） |
| `pattern` | string | 是 | JavaScript 正则表达式 |
| `replacement` | string | 是 | 替换文本（支持 `$1` 等捕获组引用） |
| `flags` | string | 否 | 正则 flags，默认 `"gi"` |

> **注意：** `pattern` 中的反斜杠需要 JSON 转义，即 `\s` 写成 `\\s`，`\d` 写成 `\\d`。

---

## 快速操作指南

### 不需要配的端点

如果某个端点暂时不需要自定义，保持 `{}` 即可，客户端会使用内置回退值。

### 只配一个端点

只需要在对应的 Nacos DataId 里粘贴对应的 JSON 即可，其他端点不受影响。

### 热更新

客户端会在启动时拉取一次。用户也可以在设置页手动刷新。Nacos 修改后不需要重启客户端，用户刷新即可生效。

### DataId 与端点对照

| Nacos DataId | 用途 |
|---|---|
| `autoclaw-model-config` | ZhipuAI 模型列表 |
| `autoclaw-provider-config` | 第三方提供商配置 |
| `autoclaw-model-pricing` | 模型定价 |
| `autoclaw-subscription` | 订阅方案 |
| `autoclaw-builtin-skills` | 内置技能部署 |
| `autoclaw-error-mapping` | 错误提示映射 |
| `autoclaw-mcp-service-templates` | MCP 快速添加模板 |
| `autoclaw-sensitive-data-rules` | 敏感数据脱敏规则 |
