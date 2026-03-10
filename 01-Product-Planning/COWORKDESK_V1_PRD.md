# AutoClaw v1 Product Requirements Document (PRD)

> **版本**: v1.0.4
> **发布日期**: 2026-02-24
> **状态**: 已发布

---

## 1. 产品概述

### 1.1 产品定位

AutoClaw 是一个面向 OpenClaw 重度用户的 macOS 原生桌面 AI 协作平台，通过 WebSocket 连接 OpenClaw Gateway，提供：

- 智能对话（多模型切换、流式响应、tool calling）
- 多会话管理（创建、搜索、归档、删除）
- 会话级 token 统计与成本追踪
- Skills / MCP 插件生态管理
- IM 频道集成（飞书、Telegram、Discord、Slack 等）
- 会话导出与导入
- 多主题皮肤系统（Raycast、Dracula、Neon Noir、Nord、Monokai）

### 1.2 目标用户

- **主要用户**: OpenClaw 重度用户、开发者、技术团队
- **使用场景**: 日常编程辅助、文档分析、任务自动化、跨平台消息路由
- **技能水平**: 中高级用户，熟悉命令行和 AI 工具

### 1.3 核心价值主张

1. **原生体验**: macOS 原生 UI，性能优于 Web 版本
2. **会话持久化**: 本地会话管理 + Gateway 同步，永不丢失对话历史
3. **多模型无缝切换**: 顶部快捷切换，支持智谱、MiniMax、Claude、GPT 等
4. **可视化透明度**: Tool calls 内联展示、实时 Activity timeline、Artifacts 追踪
5. **跨频道整合**: 一个界面管理多个 IM 频道，统一 AI 助手体验

---

## 2. 功能规格

### 2.1 Chat 对话模块

#### 2.1.1 核心对话功能

**功能描述**:
- 支持 Markdown 渲染的流式响应
- Tool calls 内联展示为可折叠卡片
- 实时 streaming cursor 指示 AI 工作状态
- 消息气泡显示模型标签（切换后自动更新）
- 代码块语法高亮 + 一键复制 + 20+ 语言专属颜色标签
- 数学公式渲染（KaTeX 支持）

**技术实现**:
- `ChatPanel/index.tsx` - 主聊天面板
- `MessageItem.tsx` - 消息组件，支持流式更新
- `ChatInput.tsx` - 输入框，支持多行、自动扩展（168px 上限）
- `StreamingCursor.tsx` - 动态光标组件
- `CodeBlock.tsx` - 代码块组件，集成 rehype-highlight + KaTeX
- `ToolCallBlock.tsx` - Tool call 卡片，20+ 语义化图标映射

**数据流**:
```
用户输入 → ChatInput.onSend()
  → chatStore.sendMessage()
  → window.electronAPI.agent.send({ sessionKey, message })
  → [Main] ipc.ts handler
  → [Main] gatewayClient.sendAgentMessage()
  → [Gateway] chat.send RPC
  → [Gateway] chat/agent events (streaming)
  → [Main] client.on('chat', event)
  → [Main] ipcMain.emit('agent:stream', event)
  → [Renderer] chatStore.handleAgentStream(event)
  → MessageItem re-render (流式更新)
```

**验收标准**:
- [ ] 消息发送后在 100ms 内开始流式输出
- [ ] Tool calls 展示包含图标、工具名称、耗时、状态
- [ ] 代码块支持 20+ 语言语法高亮
- [ ] 数学公式正确渲染（行内 + 块级）

#### 2.1.2 Stop 按钮与请求取消

**功能描述**:
- 流式输出时，输入框按钮自动变为 Stop
- 点击 Stop 立即中断当前请求
- 500ms 内反馈"已停止"状态

**技术实现**:
- `chatStore.ts` - `stopGeneration()` 方法
- WebSocket `chat.stop` RPC 调用

**验收标准**:
- [ ] Stop 响应时间 < 500ms
- [ ] 中断后不再接收新的流式数据
- [ ] 按钮状态实时更新

---

### 2.2 会话管理模块

#### 2.2.1 多会话系统

**功能描述**:
- 左侧 SessionSidebar 显示所有会话列表
- 支持新建、重命名、删除会话
- 会话搜索（按标题、最近内容）
- 会话归档与置顶（Pin）
- 会话导出（Markdown / JSON 格式）
- 批量操作（删除多个会话）

**技术实现**:
- `SessionSidebar/index.tsx` - 会话列表组件
- `sessionStore.ts` - Zustand store，管理 `sessions`、`localSessions`、`pinnedSessions`
- `ipc.ts` - `sessions.list`、`sessions.create`、`sessions.rename`、`sessions.delete`、`sessions.archive`、`sessions.export` handlers

**数据源**:
- Gateway sessions: `sessions.list` RPC → 远程会话列表
- Local sessions: 本地 `~/.openclaw-autoclaw/sessions.json` → 用户创建的会话
- IM sessions: 自动创建的 IM 频道会话（如 `agent:main:feishu:direct:ou_xxx`）

**验收标准**:
- [ ] 20+ 会话下首屏检索 < 200ms
- [ ] 搜索支持中文、英文、混合查询
- [ ] 删除会话后不再出现在列表
- [ ] 导出 Markdown 格式可完整保留对话历史
- [ ] Pin 的会话固定在列表顶部

#### 2.2.2 会话删除与恢复

**功能描述**:
- 删除会话后，新消息可以自动恢复该会话
- 删除后 1 秒内恢复（grace period）
- 恢复后保留历史消息

**技术实现**:
- `sessionStore.ts` - `deleteSession()` 记录 `deletedAt` 时间戳
- `chatStore.ts` - `sendMessage()` 检测到已删除会话 → 自动恢复

**验收标准**:
- [ ] 删除后 1 秒内新消息触发恢复
- [ ] 恢复后保留原有消息历史
- [ ] Grace period 过期后删除操作生效

---

### 2.3 Gateway 连接模块

#### 2.3.1 WebSocket 连接

**功能描述**:
- 自动连接到 `ws://127.0.0.1:18789`（默认）
- 支持 challenge-response 握手（ED25519 签名）
- 指数退避重连（1.5x factor，最大 30s，最多 20 次）
- 连接状态可视化（绿色脉冲 = connected，黄色 = connecting，红色 = error）
- 实时 ping 延迟显示

**技术实现**:
- `gateway/client.ts` - WebSocket 客户端
- `gateway/protocol.ts` - 协议定义与序列化
- `gatewayStore.ts` - 连接状态管理

**验收标准**:
- [ ] 首次连接时间 < 2s（gateway 运行中）
- [ ] 断连后自动重连，最多 20 次尝试
- [ ] 重连成功后自动恢复订阅
- [ ] Ping 延迟显示精确到 ms

#### 2.3.2 Gateway Manager

**功能描述**:
- 内置 openclaw gateway 子进程管理
- 自动启停 gateway
- Gateway 崩溃时自动重启
- 导入 `~/.openclaw` 配置

**技术实现**:
- `gateway/manager.ts` - 进程管理器
- `store/settings.ts` - Gateway 配置持久化
- ChildProcess API + 事件监听

**验收标准**:
- [ ] 点击"启动 Gateway"后 3s 内显示 connected 状态
- [ ] Gateway 崩溃后 5s 内自动重启
- [ ] 导入配置成功后自动重连

---

### 2.4 模型与 Provider 管理

#### 2.4.1 顶部模型选择器

**功能描述**:
- 顶部导航栏中央显示当前模型
- 点击展开模型下拉菜单
- 实时切换模型，无需重启应用
- 显示模型价格（$ input / output per 1M tokens）
- Rate limit 状态显示（Limited Xs 芯片）

**技术实现**:
- `MainLayout.tsx` - 模型选择器 UI + Popover 菜单
- `settingsStore.ts` - `setPrimaryModel()` 方法
- Gateway `config.patch` RPC 推送模型配置

**预置模型**:
- 智谱: GLM-4.7、GLM-4.7-Flash、GLM-5
- MiniMax: M2.5
- Anthropic: Claude 3.5 Sonnet、Claude Opus 4.6
- OpenAI: GPT-4o
- 自定义: 支持用户添加任意 provider

**验收标准**:
- [ ] 模型切换时间 < 3s（包含 gateway 重连）
- [ ] 切换后自动保持当前对话上下文
- [ ] Rate limit 时显示剩余时间（秒）
- [ ] 价格信息准确对应模型

#### 2.4.2 多 Provider 支持

**功能描述**:
- 支持 6+ provider：智谱、MiniMax、Anthropic、OpenAI、Ollama、自定义
- 每个独立配置 API Key、Base URL
- 自动路由到正确的 provider

**技术实现**:
- `SettingsView.tsx` - Models & API tab
- `shared/types.ts` - `ModelConfig`、`ProviderConfig` 类型
- Gateway 模型配置推送

**验收标准**:
- [ ] 可以添加多个同 provider 的模型
- [ ] API Key 错误时显示清晰错误提示
- [ ] 切换 provider 后立即生效

---

### 2.5 Token 统计与成本追踪

#### 2.5.1 会话级 Token 统计

**功能描述**:
- 左侧 SessionSidebar 显示每个会话的：
  - Total tokens
  - Estimated cost（USD）
- Token 数据从 Gateway usage events 采集
- 成本计算基于 `MODEL_PRICING` 常量表

**技术实现**:
- `sessionStore.ts` - `updateSessionMetrics()` 方法
- `chatStore.ts` - usage 事件监听
- `shared/constants.ts` - `MODEL_PRICING` 价格表

**验收标准**:
- [ ] Token 统计误差 < 2%
- [ ] 成本计算与 Gateway 一致
- [ ] 数据实时更新（每次对话后）

---

### 2.6 Skills 管理

#### 2.6.1 Skills Library

**功能描述**:
- 右侧 ContextSidebar 显示已安装的 skills
- 支持 skill 启用/禁用（toggle switch）
- 显示 skill 描述和图标
- 点击 skill 显示详情

**技术实现**:
- `ContextSidebar/index.tsx` - Skills 列表
- `gateway client` - `skills.list` RPC
- 本地配置 `.opencode/config.json` 读写

**验收标准**:
- [ ] Toggle switch 状态实时同步到 Gateway
- [ ] 禁用的 skill 不被调用
- [ ] Skill 列表按字母排序

---

### 2.7 MCP 服务器管理

#### 2.7.1 MCP Servers 配置

**功能描述**:
- Settings > MCP tab 显示已配置的 MCP servers
- 支持添加、删除、配置 MCP servers
- 显示 server 状态（Connected / Disconnected / Needs Auth）
- 支持 stdio 和 HTTP 两种连接方式

**技术实现**:
- `SettingsView.tsx` - MCP tab
- `gateway client` - `mcp.list`、`mcp.add`、`mcp.remove` RPCs
- 本地配置 `.opencode/config.json` 读写

**验收标准**:
- [ ] 可以添加 HTTP 和 stdio MCP servers
- [ ] Server 状态实时更新
- [ ] 删除 server 后不再出现在列表

---

### 2.8 IM 频道集成

#### 2.8.1 多频道支持

**功能描述**:
- 支持飞书、Telegram、Discord、Slack、Signal、Google Chat、IRC 等频道
- 每个频道自动创建独立 session
- 顶部导航栏显示配置的频道数量
- 点击频道图标跳转到 Settings > Channels

**技术实现**:
- `MainLayout.tsx` - IM Channel 健康指示器
- `ipc.ts` - `imChannels.list` handler
- Gateway IM 插件系统（bundle plugin extensions）

**频道分类**:
- **DM (Direct Message)**: 一对一对话（如飞书私聊）
- **Group**: 群聊
- **Channel**: 频道

**验收标准**:
- [ ] 新消息正确路由到对应的 IM session
- [ ] 频道数量每 30 秒刷新
- [ ] 未配置频道时显示"Add IM"提示按钮

#### 2.8.2 会话路由与 dmScope

**功能描述**:
- IM 频道会话自动分配独立的 session key（如 `agent:main:feishu:direct:ou_xxx`）
- 支持从会话中提取 dmScope（per-channel-peer）
- 删除后新消息自动恢复

**技术实现**:
- `sessionStore.ts` - `sanitizeOpenclawConfig()` 修复 dmScope
- Gateway session data enrichment

**验收标准**:
- [ ] 飞书 DM 不再合并到 `agent:main:main`
- [ ] 每个 IM 频道有独立的对话历史
- [ ] 删除后恢复的会话保留 dmScope

---

### 2.9 Artifacts 追踪

#### 2.9.1 文件写入追踪

**功能描述**:
- 右侧 ContextSidebar 的 Artifacts tab 显示所有 Write/Edit 工具产生的文件
- 点击文件用系统默认程序打开
- 宽松匹配识别（不依赖 tool name 白名单）

**技术实现**:
- `ContextSidebar/index.tsx` - Artifacts tab
- `chatStore.ts` - `extractArtifactContent()` 内容提取
- `extractArtifactContent` 支持多字段内容提取（content、patch、result）

**验收标准**:
- [ ] Write tool 产生的文件出现在 Artifacts 列表
- [ ] 文件名和路径显示正确
- [ ] 点击文件可打开系统预览

---

### 2.10 主题系统

#### 2.10.1 多主题支持

**功能描述**:
- 5 套完整主题：Raycast、Dracula、Neon Noir、Nord、Monokai
- 每套主题包含：
  - 完整的 CSS 变量映射（`--bg`、`--text-pri`、`--accent1` 等）
  - 专属光晕效果（ambient lighting）
  - 阴影变量（`--shadow-sm`、`--shadow-float`）
- 实时切换主题

**技术实现**:
- `global.css` - 主题 CSS 变量
- `themes.css` - 各主题样式覆盖
- `SkinSwitcher.tsx` - 主题切换组件
- `settingsStore.ts` - 主题持久化

**验收标准**:
- [ ] 切换主题后所有组件立即更新
- [ ] 所有 5 套主题视觉一致
- [ ] OLED 纯黑主题支持（Raycast、Neon Noir）

---

### 2.11 导出与导入

#### 2.11.1 会话导出

**功能描述**:
- 支持 Markdown 和 JSON 两种格式导出
- 批量导出多个会话
- 导出文件包含完整对话历史、tool calls、元数据

**技术实现**:
- `sessionStore.ts` - `exportSessions()` 方法
- Electron `dialog.showSaveDialog()` 文件选择
- 文件系统操作

**验收标准**:
- [ ] 导出的 Markdown 格式可读性强
- [ ] JSON 格式包含所有字段
- [ ] 批量导出支持多选

---

## 3. 技术架构

### 3.1 进程架构

```
┌─────────────────────────────────────────────────────┐
│ Renderer Process (React)                            │
│  ├── stores/ (Zustand)                              │
│  │   ├── settingsStore.ts                           │
│  │   ├── sessionStore.ts                            │
│  │   ├── gatewayStore.ts                            │
│  │   └── chatStore.ts                              │
│  ├── panels/  (布局面板)                             │
│  │   ├── ChatPanel/index.tsx                        │
│  │   ├── SessionSidebar/index.tsx                   │
│  │   └── ContextSidebar/index.tsx                    │
│  ├── components/ (可复用组件)                        │
│  │   ├── MessageItem.tsx                            │
│  │   ├── ChatInput.tsx                              │
│  │   ├── ToolCallBlock.tsx                          │
│  │   └── CodeBlock.tsx                              │
│  └── views/  (视图)                                  │
│      └── SettingsView.tsx                            │
├─────────────────────────────────────────────────────┤
│ Preload (contextBridge)                             │
│  ├── 暴露最小化 API                                  │
│  └── 无 raw Node 泄露                               │
├─────────────────────────────────────────────────────┤
│ Main Process (Node.js)                              │
│  ├── gateway/client.ts  (WebSocket → OpenClaw)      │
│  ├── gateway/manager.ts (子进程管理)                  │
│  ├── ipc.ts             (IPC handler 注册)          │
│  └── store/settings.ts  (持久化设置)                 │
├─────────────────────────────────────────────────────┤
│ OpenClaw Gateway (ws://127.0.0.1:18789)             │
│  ├── chat.send / chat.stop RPCs                       │
│  ├── skills.list / skills.toggle RPCs                 │
│  ├── mcp.list / mcp.add RPCs                         │
│  └── IM channel plugin system                        │
└─────────────────────────────────────────────────────┘
```

### 3.2 技术栈

| 类别 | 技术 | 版本 |
|------|------|------|
| Runtime | Electron | 33.4.0 |
| Node | Node.js | 20.x |
| Frontend | React | 19.0.0 |
| UI Library | Ant Design | 5.24.1 |
| State Management | Zustand | 5.0.3 |
| Build Tool | electron-vite | 5.0.0 |
| WebSocket | ws | 8.18.0 |
| Markdown | react-markdown | 9.0.3 |
| Code Highlight | rehype-highlight | 7.0.2 |
| Math Rendering | KaTeX | 0.16.33 |
| Icons | lucide-react | 0.575.0 |
| Database | better-sqlite3 | 11.8.0 |
| File Watcher | chokidar | 4.0.3 |
| Terminal | xterm | 5.3.0 |
| Type System | TypeScript | 5.7.3 |

### 3.3 数据持久化

| 数据 | 存储位置 | 用途 |
|------|----------|------|
| 用户设置 | `~/.openclaw-autoclaw/settings.json` | 主题、模型配置、快捷键 |
| 本地会话 | `~/.openclaw-autoclaw/sessions.json` | 会话元数据、重命名、归档 |
| Gateway 配置 | `~/.openclaw/config.json` | Provider keys、模型 catalog |
| Skills 配置 | `.opencode/config.json` | 已安装 skills 列表 |
| MCP 配置 | `.opencode/config.json` | MCP servers 配置 |

---

## 4. 用户流程

### 4.1 首次启动流程

```
1. 打开 AutoClaw
   ↓
2. 检测 Gateway 连接状态
   ├─ Gateway 未运行 → 显示错误提示 + "启动 Gateway" 按钮
   └─ Gateway 已运行 → 自动连接（ws://127.0.0.1:18789）
   ↓
3. 连接成功
   ├─ 显示欢迎界面（Main Thread 空状态）
   ├─ 自动加载会话列表
   ├─ 加载 Skills 列表
   └─ 加载 MCP servers 列表
   ↓
4. 用户开始对话
```

### 4.2 切换模型流程

```
1. 点击顶部模型选择器
   ↓
2. 展开模型下拉菜单
   ├─ 显示所有配置的模型
   ├─ 显示价格信息
   └─ 显示当前选中状态
   ↓
3. 点击目标模型
   ↓
4. 显示 loading 状态 "Switching..."
   ↓
5. 推送新模型配置到 Gateway
   ├─ Gateway 重启
   └─ 等待重连完成
   ↓
6. 显示成功 toast "Model switched to XXX"
   ↓
7. 继续当前对话（使用新模型）
```

### 4.3 创建新会话流程

```
1. 点击左侧 "+ New Chat" 按钮（或 Cmd+N）
   ↓
2. 创建新的 session key（随机生成）
   ↓
3. 切换到新会话（activeSessionKey 更新）
   ↓
4. 显示空状态 "Main Thread" + "Describe your objective..."
   ↓
5. 用户开始输入
```

### 4.4 删除会话流程

```
1. 右键点击会话 → "Delete"（或批量删除）
   ↓
2. 显示确认对话框 "Delete X session(s)?"
   ↓
3. 用户确认
   ↓
4. 标记会话为 deleted（deletedAt = Date.now()）
   ↓
5. 会话从列表消失
   ↓
6. Grace period (1秒)
   ├─ 无新消息 → 永久删除
   └─ 有新消息 → 自动恢复会话
```

---

## 5. 安全与隐私

### 5.1 数据安全

| 数据 | 存储方式 | 加密 |
|------|----------|------|
| API Keys | `settings.json` 明文 | ❌ 未加密 |
| 对话历史 | Gateway 远程存储 | ✅ 传输加密（TLS） |
| 本地会话元数据 | `sessions.json` 明文 | ❌ 未加密 |
| 用户设置 | `settings.json` 明文 | ❌ 未加密 |

**Note**: v1 版本未实现 Keychain 加密存储，文档中已明确标注为"本地文件存储"。

### 5.2 网络安全

- WebSocket 连接使用 `ws://`（本地）或 `wss://`（远程）
- 所有 Gateway RPC 调用通过 WebSocket 传输
- IM 频道连接使用各平台官方 SDK

### 5.3 权限管理

- macOS 屏幕录制权限（未实现，v2 预留）
- macOS 日历访问权限（未实现，v2 预留）
- 文件系统访问权限（仅读写用户指定路径）

---

## 6. 已知限制

### 6.1 功能限制

- ❌ 无本地数据库（会话数据依赖 Gateway）
- ❌ 无 proactive 主动智能
- ❌ 无 Tray / Pill 系统级交互
- ❌ 无日程规划与记忆系统
- ❌ 无自动模型路由
- ❌ Terminal 面板为空壳（未实现）
- ❌ Preview 面板为空壳（未实现）

### 6.2 平台限制

- ❌ 仅支持 macOS arm64
- ❌ 未测试 Windows / Linux

### 6.3 性能限制

- 大文件读取可能卡顿（>10MB）
- 20+ 会话时搜索性能下降（无索引优化）

---

## 7. 质量保证

### 7.1 测试覆盖

| 模块 | 单元测试 | 集成测试 | E2E 测试 |
|------|----------|----------|----------|
| Chat | ❌ | ❌ | ❌ |
| Session | ❌ | ❌ | ❌ |
| Gateway | ❌ | ✅（手动） | ❌ |
| Settings | ❌ | ✅（手动） | ❌ |

**Note**: v1 版本无自动化测试，依赖手动回归。

### 7.2 已修复 Bug

详见 git commit history（0d4c791 → v0.1.0）：

- ✅ IM session routing（dmScope）
- ✅ Delete/revive timing
- ✅ WebSocket reconnect stability
- ✅ 输入框自动扩展
- ✅ 外部链接覆盖主窗口问题
- ✅ Artifacts 文件识别
- ✅ 配置隔离
- ✅ MiniMax 支持
- ✅ 飞书 plugin loading

---

## 8. 发布历史

### v0.1.0 — 2026-02-20（首版发布）

**Milestone: AutoClaw v1 首版发布**

#### 已完成功能

| 模块 | 状态 | 说明 |
|------|------|------|
| Chat 对话 | ✅ Done | ChatPanel + MessageItem + 流式响应 + tool calling |
| Session 管理 | ✅ Done | SessionSidebar + sessionStore，多会话切换 |
| Gateway 连接 | ✅ Done | WebSocket 完整握手 + 自动重连 + 状态同步 |
| 模型选择 | ✅ Done | catalog + primary，支持运行时切换 |
| 多服务商支持 | ✅ Done | 智谱、MiniMax、Anthropic、OpenAI + 自定义 |
| 预置模型 | ✅ Done | GLM-4.7 / GLM-4.7-Flash / GLM-5 / MiniMax-M2.5 |
| Settings | ✅ Done | 7 tab 设置面板（General / Models & API / Integrations 等） |
| Skills 管理 | ✅ Done | ContextSidebar 内 skills 列表 + 启停 |
| MCP 管理 | ✅ Done | SettingsView 内 MCP server 配置 + 状态 |
| Gateway Manager | ✅ Done | 内置 openclaw gateway 子进程管理，自动启停 |
| 构建发布 | ✅ Done | electron-builder macOS arm64 DMG + ZIP，代码签名 |

### v0.1.1 — 2026-02-21

- ✅ 配置隔离（每个用户独立的 settings.json）
- ✅ MiniMax 支持

### v0.1.2 — 2026-02-24

- ✅ 修复点击外部链接覆盖主窗口问题
- ✅ 修复 Write tool 产生的文件不在 Artifacts 显示
- ✅ Artifacts 文件识别改为宽松匹配

### v0.1.4 — 2026-02-27

- ✅ IM 频道管理
- ✅ 跨频道消息
- ✅ Token 统计
- ✅ 会话 dmScope 修复
- ✅ WebSocket 重连稳定性提升

---

## 9. 设计决策记录 (ADR)

| 决策 | 选择 | 理由 |
|------|------|------|
| 状态管理 | Zustand | 轻量、无样板代码、DevTools 支持 |
| UI 框架 | Ant Design | 组件丰富、文档完善、中文友好 |
| 构建工具 | electron-vite | Vite 生态、HMR 支持好 |
| WebSocket | ws（npm） | 原生实现、性能最优 |
| 进程间通信 | Electron IPC + contextBridge | 安全、标准化 |
| 主题系统 | CSS 变量 | 运行时切换、性能高 |
| 会话存储 | 本地 JSON 文件 | 简单、可读、可手动编辑 |
| Gateway 连接 | WebSocket | 实时双向通信、支持流式 |
| 文件导出 | JSON + Markdown | JSON 机器可读，Markdown 人类可读 |
| 图标库 | lucide-react | Tree-shakable、设计一致 |

---

## 10. 后续规划 (v2)

详见 `docs/DEVELOPMENT_PLAN.md`：

- 📅 Daily Planner（日程规划）
- 🧠 MemOS（语义记忆）
- 💰 Usage Dashboard（用量统计面板）
- 💊 Pill 浮动通知系统
- 🎯 Proactive 主动智能
- 🔄 Auto-Router 智能模型路由
- 🖥️ Tray / Menubar 集成
- 🖼️ Screen Vision 屏幕分析
- 📁 Filesystem Watcher 文件监听

---

## 11. 附录

### 11.1 键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd+N` | 新建会话 |
| `Cmd+,` | 打开/关闭设置 |
| `Esc` | 关闭设置 / 停止流式输出 |
| `Enter` | 发送消息 |
| `Shift+Enter` | 换行 |
| `Cmd+K` | （预留）命令面板 |

### 11.2 常用 RPC 列表

| RPC | 说明 |
|-----|------|
| `chat.send` | 发送消息 |
| `chat.stop` | 停止生成 |
| `chat.history` | 获取历史消息 |
| `sessions.list` | 获取会话列表 |
| `sessions.create` | 创建新会话 |
| `sessions.delete` | 删除会话 |
| `skills.list` | 获取 skills 列表 |
| `skills.toggle` | 启用/禁用 skill |
| `mcp.list` | 获取 MCP servers 列表 |
| `mcp.add` | 添加 MCP server |
| `mcp.remove` | 删除 MCP server |
| `config.patch` | 更新配置 |
| `imChannels.list` | 获取 IM 频道列表 |

### 11.3 文件路径

| 文件 | 路径 |
|------|------|
| 应用数据 | `~/Library/Application Support/autoclaw/` |
| Gateway 配置 | `~/.openclaw/config.json` |
| 本地会话 | `~/.openclaw-autoclaw/sessions.json` |
| 用户设置 | `~/.openclaw-autoclaw/settings.json` |
| Skills 配置 | `.opencode/config.json` |

---

**文档结束**
