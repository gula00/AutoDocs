# AutoClaw Development Plan

---

## Release History

### v0.1.0 — 2025-02-24 Released

**Milestone: AutoClaw v1 首版发布**

本版本完成了 AutoClaw 桌面端的核心功能闭环，面向种子用户发布。

#### 已完成功能

| 模块 | 状态 | 说明 |
|------|------|------|
| Chat 对话 | Done | ChatPanel + MessageItem + 流式响应 + tool calling |
| Session 管理 | Done | SessionSidebar + sessionStore，多会话切换 |
| Gateway 连接 | Done | WebSocket 完整握手 + 自动重连 + 状态同步 |
| 模型选择 | Done | catalog + primary，支持运行时切换 |
| 多服务商支持 | Done | 智谱、MiniMax、Anthropic、OpenAI + 自定义 |
| 预置模型 | Done | GLM-4.7 / GLM-4.7-Flash / GLM-5 / MiniMax-M2.5 |
| Settings | Done | 7 tab 设置面板（General / Models & API / Integrations 等） |
| Skills 管理 | Done | ContextSidebar 内 skills 列表 + 启停 |
| MCP 管理 | Done | SettingsView 内 MCP server 配置 + 状态 |
| Gateway Manager | Done | 内置 openclaw gateway 子进程管理，自动启停 |
| 构建发布 | Done | electron-builder macOS arm64 DMG + ZIP，代码签名 |

#### 技术栈

- Electron 33 + Node 20
- React 19 + Ant Design 5 + Zustand 5
- electron-vite 5 + Vite 6
- WebSocket (ws) + OpenClaw JSON-RPC v3
- lucide-react

#### 已知限制

- 仅 macOS arm64 平台
- 无本地数据库（会话数据依赖 gateway）
- 无 usage 统计
- 无 proactive 主动智能
- 无 Tray / Pill 系统级交互
- 无日程规划与记忆系统

---

# CoworkDesk v2 Development Plan

> 基于 openclaw-ultimate-desktop-v13.html 原型，在 v0.1.0 基础上进行功能升级。
> UI 实现严格参照原型文件。

---

## 1. 项目概述

### 1.1 产品定位

CoworkDesk 是一个 macOS 桌面 AI 协作平台，通过 WebSocket 连接 OpenClaw Gateway，提供：

- 智能对话（多模型切换、流式响应、tool calling）
- 日程规划（本地 + 系统日历同步）
- 语义记忆（MemOS fact store）
- Skills / MCP 插件生态管理
- 用量追踪与成本分析
- 系统级浮动 Pill 通知（Tray 级交互）
- Proactive 主动智能（屏幕视觉、文件监听、GitHub 监控、后台进程监控、每日 Rollover）

### 1.2 当前状态 vs 目标状态

| 模块 | 现有 | 原型目标 | 工作量 |
|------|------|----------|--------|
| Chat 对话 | 已实现（ChatPanel + MessageItem + streaming） | UI 重构对齐原型样式 | 中 |
| Session 管理 | 已实现（SessionSidebar + sessionStore） | 改名 "Continuous Streams"，UI 调整 | 小 |
| Settings | 已实现（7 tab SettingsView） | 精简为 4 tab，对齐原型 | 小 |
| Gateway 连接 | 已实现（完整握手 + 重连） | 无变化 | 无 |
| 模型选择 | 已实现（catalog + primary） | UI 改为原型风格下拉 | 小 |
| Skills 管理 | 已实现（ContextSidebar 内） | 独立 Skills Library 视图 | 中 |
| MCP 管理 | 已实现（SettingsView 内） | 独立 MCP Servers 视图 | 中 |
| **Daily Planner** | **不存在** | **全新模块** | **大** |
| **MemOS / Memory** | **不存在** | **全新模块** | **大** |
| **Usage Dashboard** | **不存在** | **全新模块** | **中** |
| **Pill 浮动通知** | **不存在** | **全新模块** | **大** |
| **Tray / Menubar** | **不存在** | **全新模块** | **中** |
| **系统日历集成** | **不存在** | **全新能力** | **中** |
| **Proactive 主动智能** | **不存在** | **全新模块（5 种触发源）** | **大** |

### 1.3 技术栈（不变）

- **Runtime**: Electron 33 + Node 20
- **Frontend**: React 19 + Ant Design 5 + Zustand 5
- **Build**: electron-vite 5 + Vite 6
- **Local DB**: better-sqlite3（已在依赖中）
- **Gateway**: WebSocket (ws) + 自定义 JSON-RPC 协议 v3
- **Icons**: lucide-react（已在依赖中）

---

## 2. 架构设计

### 2.1 进程架构（不变）

```
┌─────────────────────────────────────────────────────┐
│ Renderer Process (React)                            │
│  ├── stores/ (Zustand)                              │
│  ├── views/  (全屏视图)                              │
│  ├── panels/ (布局面板)                              │
│  └── components/ (可复用组件)                         │
├─────────────────────────────────────────────────────┤
│ Preload (contextBridge)                             │
├─────────────────────────────────────────────────────┤
│ Main Process (Node.js)                              │
│  ├── gateway/client.ts  (WebSocket → OpenClaw)      │
│  ├── ipc.ts             (IPC handler 注册)          │
│  ├── store/settings.ts  (持久化设置)                 │
│  ├── calendar/           ← NEW: 系统日历桥接         │
│  ├── db/                 ← NEW: SQLite 数据层        │
│  ├── tray/               ← NEW: Tray/Menubar        │
│  └── proactive/          ← NEW: 主动智能触发引擎     │
├─────────────────────────────────────────────────────┤
│ OpenClaw Gateway (ws://127.0.0.1:18789)             │
└─────────────────────────────────────────────────────┘
```

### 2.2 新增 IPC Channels

在 `src/shared/ipc-channels.ts` 中追加：

```typescript
// Planner（日程规划）
PLANNER_LIST_EVENTS: 'planner:list-events',
PLANNER_CREATE_EVENT: 'planner:create-event',
PLANNER_UPDATE_EVENT: 'planner:update-event',
PLANNER_DELETE_EVENT: 'planner:delete-event',
PLANNER_SYNC_SYSTEM_CALENDAR: 'planner:sync-system-calendar',
PLANNER_WRITE_SYSTEM_CALENDAR: 'planner:write-system-calendar',

// MemOS（语义记忆）
MEMOS_LIST_FACTS: 'memos:list-facts',
MEMOS_ADD_FACT: 'memos:add-fact',
MEMOS_UPDATE_FACT: 'memos:update-fact',
MEMOS_DELETE_FACT: 'memos:delete-fact',
MEMOS_SEARCH_FACTS: 'memos:search-facts',
MEMOS_GET_DIRECTIVE: 'memos:get-directive',
MEMOS_SET_DIRECTIVE: 'memos:set-directive',

// Usage（用量统计）
USAGE_GET_SUMMARY: 'usage:get-summary',
USAGE_GET_TIMELINE: 'usage:get-timeline',
USAGE_RECORD: 'usage:record',

// Tray
TRAY_UPDATE_STATUS: 'tray:update-status',
TRAY_ACTIVITY: 'tray:activity',

// Proactive（主动智能）
PROACTIVE_GET_SETTINGS: 'proactive:get-settings',
PROACTIVE_UPDATE_SETTINGS: 'proactive:update-settings',
PROACTIVE_TRIGGER: 'proactive:trigger',           // 手动触发某个 watcher
PROACTIVE_SUGGESTION: 'proactive:suggestion',     // Main → Renderer 推送建议
PROACTIVE_DISMISS: 'proactive:dismiss',           // 用户忽略建议
PROACTIVE_APPROVE: 'proactive:approve',           // 用户批准建议

// Auto-Router（智能模型路由）
ROUTER_GET_SETTINGS: 'router:get-settings',
ROUTER_UPDATE_SETTINGS: 'router:update-settings',
ROUTER_CLASSIFY: 'router:classify',               // 手动触发分类（调试用）
```

### 2.3 新增 Shared Types

在 `src/shared/types.ts` 中追加：

```typescript
// ─── Planner ───

export interface PlannerEvent {
  id: string
  title: string
  description?: string
  startTime: number      // Unix timestamp ms
  endTime: number
  allDay: boolean
  source: 'local' | 'system-calendar' | 'mcp-feishu'
  sourceId?: string      // 系统日历 event ID，用于回写
  calendarName?: string
  color?: string
  completed?: boolean
  createdAt: number
  updatedAt: number
}

export interface PlannerTask {
  id: string
  title: string
  completed: boolean
  dueDate?: number
  source: 'user' | 'agent'   // 用户手动创建 or AI 提取
  sessionKey?: string         // 关联的对话 session
  createdAt: number
  updatedAt: number
}

// ─── MemOS ───

export type FactCategory =
  | 'USER_PREFERENCE'
  | 'PROJECT_CONTEXT'
  | 'CODING_STYLE'
  | 'WORKFLOW'
  | 'CUSTOM'

export interface MemOSFact {
  id: string
  category: FactCategory
  content: string
  source: 'user' | 'agent-extracted'
  sessionKey?: string     // 从哪次对话提取的
  createdAt: number
  updatedAt: number
}

export interface MemOSDirective {
  content: string
  updatedAt: number
}

// ─── Usage ───

export interface UsageRecord {
  id: string
  sessionKey: string
  model: string
  provider: string
  inputTokens: number
  outputTokens: number
  totalTokens: number
  costUsd: number
  timestamp: number
}

export interface UsageSummary {
  totalTokens: number
  totalCostUsd: number
  totalTasks: number
  autoRouterTasks: number
  byModel: Record<string, { tokens: number; cost: number; count: number }>
  periodStart: number
  periodEnd: number
}

export interface UsageTimelinePoint {
  date: string       // 'YYYY-MM-DD'
  tokens: number
  cost: number
  tasks: number
}

// ─── Proactive（主动智能）───

export type ProactiveSource =
  | 'screen-vision'       // 屏幕截图分析
  | 'github-watcher'      // GitHub 事件监控
  | 'fs-watcher'          // 文件系统变化
  | 'process-watcher'     // 后台进程/终端错误
  | 'daily-rollover'      // 每日任务检查

export interface ProactiveSettings {
  screenVision: {
    enabled: boolean
    intervalMs: number       // 截屏分析间隔，默认 30000
    visionModel?: string     // 指定 vision 模型（需支持图片输入）
  }
  github: {
    enabled: boolean
    intervalMs: number       // 轮询间隔，默认 60000
  }
  filesystem: {
    enabled: boolean
    watchPaths: string[]     // 监听路径列表，默认 ['~/Downloads']
    debounceMs: number       // 变化合并窗口，默认 5000
  }
  terminal: {
    enabled: boolean         // 默认 true
  }
  dailyRollover: {
    enabled: boolean         // 默认 true
  }
}

export interface ProactiveSuggestion {
  id: string
  source: ProactiveSource
  title: string
  description: string
  preview?: string           // 代码/内容预览（HTML 或 markdown）
  previewType?: 'code' | 'markdown' | 'system'
  actions: ProactiveAction[]
  createdAt: number
  dismissed: boolean
  executed: boolean
}

export interface ProactiveAction {
  label: string              // 按钮文字，如 "Fix it", "Merge CSVs"
  type: 'approve' | 'open' | 'custom'
  payload?: unknown          // 执行所需的参数
}

// ─── AppSettings 扩展 ───
// 在现有 AppSettings interface 中新增：
//   proactive: ProactiveSettings
// DEFAULT_SETTINGS 中的默认值：
//   proactive: {
//     screenVision: { enabled: false, intervalMs: 30000 },
//     github: { enabled: true, intervalMs: 60000 },
//     filesystem: { enabled: true, watchPaths: ['~/Downloads'], debounceMs: 5000 },
//     terminal: { enabled: true },
//     dailyRollover: { enabled: true }
//   }
//   router: RouterSettings

// ─── Auto-Router（智能模型路由）───

export type RouterTier = 'heavy' | 'medium' | 'light'

export interface RouterSettings {
  enabled: boolean                        // 默认 false，需用户主动开启
  llmClassifyEnabled: boolean             // 规则不确定时是否用 LLM 分类，默认 true
  classifyModel?: string                  // 分类用的模型，默认用 catalog 中最便宜的
  tierMap: Record<RouterTier, {
    modelRef: string                      // gateway model ref，如 'anthropic/claude-opus-4-6'
    thinkingLevel: string                 // 'off' | 'low' | 'medium' | 'high'
  }>
  costAwareFallback: boolean              // rate limit 时自动降级，默认 true
}

export interface RouterDecision {
  tier: RouterTier
  modelRef: string
  thinkingLevel: string
  reason: 'rule' | 'llm-classify' | 'rate-limit-fallback' | 'user-override'
}

// DEFAULT_SETTINGS 中的默认值：
//   router: {
//     enabled: false,
//     llmClassifyEnabled: true,
//     tierMap: {
//       heavy:  { modelRef: 'anthropic/claude-opus-4-6', thinkingLevel: 'high' },
//       medium: { modelRef: 'zai/glm-4.7', thinkingLevel: 'medium' },
//       light:  { modelRef: 'zai/glm-4.7-flash', thinkingLevel: 'off' }
//     },
//     costAwareFallback: true
//   }
```

### 2.4 本地数据库设计 (SQLite)

存储路径：`app.getPath('userData')/coworkdesk.db`

```sql
-- 日程事件
CREATE TABLE planner_events (
  id            TEXT PRIMARY KEY,
  title         TEXT NOT NULL,
  description   TEXT,
  start_time    INTEGER NOT NULL,
  end_time      INTEGER NOT NULL,
  all_day       INTEGER DEFAULT 0,
  source        TEXT DEFAULT 'local',
  source_id     TEXT,
  calendar_name TEXT,
  color         TEXT,
  completed     INTEGER DEFAULT 0,
  created_at    INTEGER NOT NULL,
  updated_at    INTEGER NOT NULL
);

-- 待办任务
CREATE TABLE planner_tasks (
  id            TEXT PRIMARY KEY,
  title         TEXT NOT NULL,
  completed     INTEGER DEFAULT 0,
  due_date      INTEGER,
  source        TEXT DEFAULT 'user',
  session_key   TEXT,
  created_at    INTEGER NOT NULL,
  updated_at    INTEGER NOT NULL
);

-- 语义记忆 facts
CREATE TABLE memos_facts (
  id            TEXT PRIMARY KEY,
  category      TEXT NOT NULL,
  content       TEXT NOT NULL,
  source        TEXT DEFAULT 'user',
  session_key   TEXT,
  created_at    INTEGER NOT NULL,
  updated_at    INTEGER NOT NULL
);

-- Core Directive（单行）
CREATE TABLE memos_directive (
  id            INTEGER PRIMARY KEY CHECK (id = 1),
  content       TEXT NOT NULL DEFAULT '',
  updated_at    INTEGER NOT NULL
);

-- 用量记录
CREATE TABLE usage_records (
  id            TEXT PRIMARY KEY,
  session_key   TEXT NOT NULL,
  model         TEXT NOT NULL,
  provider      TEXT NOT NULL,
  input_tokens  INTEGER DEFAULT 0,
  output_tokens INTEGER DEFAULT 0,
  total_tokens  INTEGER DEFAULT 0,
  cost_usd      REAL DEFAULT 0,
  timestamp     INTEGER NOT NULL
);

-- Proactive 建议日志（用于去重 + 历史回溯）
CREATE TABLE proactive_log (
  id            TEXT PRIMARY KEY,
  source        TEXT NOT NULL,
  title         TEXT NOT NULL,
  description   TEXT,
  dismissed     INTEGER DEFAULT 0,
  executed      INTEGER DEFAULT 0,
  timestamp     INTEGER NOT NULL
);

-- Auto-Router 路由日志（用于统计 + 回溯）
CREATE TABLE router_log (
  id            TEXT PRIMARY KEY,
  session_key   TEXT NOT NULL,
  tier          TEXT NOT NULL,
  model_ref     TEXT NOT NULL,
  reason        TEXT NOT NULL,
  message_preview TEXT,
  timestamp     INTEGER NOT NULL
);

-- 索引
CREATE INDEX idx_planner_events_time ON planner_events(start_time);
CREATE INDEX idx_planner_tasks_due ON planner_tasks(due_date);
CREATE INDEX idx_memos_facts_category ON memos_facts(category);
CREATE INDEX idx_usage_records_time ON usage_records(timestamp);
CREATE INDEX idx_usage_records_model ON usage_records(model);
CREATE INDEX idx_proactive_log_time ON proactive_log(timestamp);
CREATE INDEX idx_proactive_log_source ON proactive_log(source);
CREATE INDEX idx_router_log_time ON router_log(timestamp);
CREATE INDEX idx_router_log_tier ON router_log(tier);
```

---

## 3. 模块开发规格

### 3.1 视图导航系统（重构）

原型中左侧 sidebar 变为多视图导航，不再只是 session 列表：

**导航项：**
1. **Chat** (对话) — 默认视图
2. **Planner** (日程规划) — 新增
3. **Skills** (技能库) — 从 ContextSidebar 独立
4. **MCP** (服务连接) — 从 SettingsView 独立
5. **MemOS** (记忆系统) — 新增

**实现：**
- 扩展 `settingsStore.ts` 的 `currentView` 类型：
  ```typescript
  type AppView = 'chat' | 'planner' | 'skills' | 'mcp' | 'memos' | 'settings'
  ```
- `MainLayout.tsx` 根据 `currentView` 切换渲染不同视图
- 左侧栏上部为视图导航 icon 列，下部在 chat 视图时显示 session 列表

### 3.2 Chat 视图（重构 UI）

**现有文件改动：**
- `panels/ChatPanel/index.tsx` — 样式对齐原型
- `components/MessageItem.tsx` — 气泡样式调整
- `components/ChatInput.tsx` — 添加 Quick Actions 行

**原型新增元素：**
- 消息气泡上方显示模型 tag（如 "Claude 3.5 Sonnet"）
- Input 框上方 Quick Actions 按钮行（Add Task、Schedule Block、Draft Email）
- Quick Actions 触发对应功能：
  - `Add Task` → 调用 planner:create-event
  - `Schedule Block` → 在 input 预填模板
  - `Draft Email` → 在 input 预填模板

**Gateway 数据流（无变化）：**
```
用户输入 → agent:send IPC → chat.send RPC → chat events → agent:stream IPC → chatStore
```

### 3.3 Daily Planner 视图（全新）

**新增文件：**
```
src/renderer/views/PlannerView.tsx        # 主视图
src/renderer/stores/plannerStore.ts       # 状态管理
src/main/calendar/index.ts               # macOS 日历桥接
src/main/db/planner.ts                   # SQLite CRUD
```

**视图结构（对照原型）：**
```
PlannerView
├── Header: 日期标题 + Sync 按钮
├── Today's Agenda
│   ├── Timeline 时间轴（合并本地事件 + 系统日历）
│   └── 每个事件卡片：时间、标题、来源标签
├── Rollover Tasks
│   ├── 未完成待办列表（checkbox + 标题）
│   └── 已完成的划线显示
├── Quick Actions
│   ├── Add Task 按钮
│   ├── Schedule Block 按钮
│   └── Draft Email 按钮
└── Input Bar: "Tell me what you want to plan or schedule..."
```

**plannerStore 状态：**
```typescript
interface PlannerState {
  events: PlannerEvent[]
  tasks: PlannerTask[]
  selectedDate: string          // 'YYYY-MM-DD'
  syncing: boolean
  lastSyncAt: number | null

  // actions
  fetchEvents: (date: string) => Promise<void>
  fetchTasks: () => Promise<void>
  createEvent: (event: Omit<PlannerEvent, 'id' | 'createdAt' | 'updatedAt'>) => Promise<void>
  createTask: (title: string, dueDate?: number) => Promise<void>
  toggleTask: (id: string) => Promise<void>
  deleteTask: (id: string) => Promise<void>
  syncSystemCalendar: () => Promise<void>
  writeToSystemCalendar: (eventId: string) => Promise<void>
}
```

**macOS 系统日历集成 (`src/main/calendar/index.ts`)：**

```typescript
/**
 * 通过 osascript 读取 macOS Calendar 事件
 * 首次调用会触发系统权限弹窗
 */
export async function readSystemCalendarEvents(
  startDate: Date,
  endDate: Date
): Promise<SystemCalendarEvent[]>

/**
 * 通过 osascript 写入事件到 macOS Calendar
 * 写入后 iCloud 自动同步到所有 Apple 设备
 */
export async function writeSystemCalendarEvent(event: {
  title: string
  startDate: Date
  endDate: Date
  notes?: string
  calendar?: string
}): Promise<{ success: boolean; eventId?: string }>

/**
 * 检查日历访问权限状态
 */
export async function checkCalendarPermission(): Promise<boolean>
```

**Agent Tool 集成（通过 gateway 的 tool calling）：**

当用户在对话中提到日程相关意图时：
1. Agent 发出 `tool_use` 事件，tool name = `create_calendar_event`
2. Gateway 发 `exec.approval.requested` 事件
3. Pill 组件弹出审批卡片
4. 用户 Approve → `exec.approval.resolve` → Electron 执行 `writeSystemCalendarEvent()`
5. 写入系统日历 → iCloud 同步到所有设备
6. `tool_result` 返回 agent，agent 确认 "已创建，所有设备会收到提醒"

### 3.4 MemOS / Context & Memory 视图（全新）

**新增文件：**
```
src/renderer/views/MemosView.tsx          # 主视图
src/renderer/stores/memosStore.ts         # 状态管理
src/main/db/memos.ts                      # SQLite CRUD
```

**视图结构（对照原型）：**
```
MemosView
├── Header: "Context & MemOS" + Add Fact 按钮
├── 左栏: Core Directive
│   ├── Textarea（可编辑的全局系统指令）
│   ├── Save Directive 按钮
│   └── Active Contexts（tag chips: 项目/Repo 关联）
└── 右栏: Semantic Facts
    ├── 搜索栏
    └── Fact 卡片列表
        ├── 分类标签 (USER_PREFERENCE / PROJECT_CONTEXT / ...)
        ├── 内容文本
        ├── 时间戳
        └── Edit / Delete 操作
```

**memosStore 状态：**
```typescript
interface MemosState {
  facts: MemOSFact[]
  directive: MemOSDirective
  searchQuery: string
  loading: boolean

  // actions
  fetchFacts: () => Promise<void>
  addFact: (fact: Omit<MemOSFact, 'id' | 'createdAt' | 'updatedAt'>) => Promise<void>
  updateFact: (id: string, content: string) => Promise<void>
  deleteFact: (id: string) => Promise<void>
  searchFacts: (query: string) => Promise<void>
  loadDirective: () => Promise<void>
  saveDirective: (content: string) => Promise<void>
}
```

**与 Agent 的集成：**
- Core Directive 内容在每次 `chat.send` 前通过 `chat.inject` 注入为 system prompt
- Agent 对话中提取的 facts（如 "用户偏好 React FC"）可通过 agent tool 自动写入 MemOS
- Active Contexts 列表用于向 agent 提供当前工作上下文

### 3.5 Skills Library 视图（从 ContextSidebar 独立）

**新增文件：**
```
src/renderer/views/SkillsView.tsx         # 独立视图
```

**现有逻辑复用：**
- 数据源不变：`skills.status` gateway RPC → `skills:list` IPC
- Store 不变：可在 `settingsStore` 中管理，或新建 `skillsStore.ts`

**视图结构（对照原型）：**
```
SkillsView
├── Header: "Skills Library" + Create Skill 按钮
└── 2列网格卡片
    ├── Skill 图标 + 颜色
    ├── Skill 名称
    ├── 描述
    └── 启用/禁用 Toggle Switch
```

**操作：**
- Toggle Switch → `skills:toggle` IPC → 修改 `.opencode/config.json`
- Create Skill → 打开文件选择器或模板
- 卡片 hover 效果 + 点击进入详情

### 3.6 MCP Servers 视图（从 SettingsView 独立）

**新增文件：**
```
src/renderer/views/McpView.tsx            # 独立视图
```

**现有逻辑复用：**
- 数据源不变：`mcp:list` IPC → 读取 `.opencode/config.json`
- Store：可复用 `settingsStore` 或新建 `mcpStore.ts`

**视图结构（对照原型）：**
```
McpView
├── Header: "MCP Servers" + Add Server 按钮
└── 列表（每行一个 MCP server）
    ├── 图标 + Server 名称 + package tag
    ├── 描述（连接信息）
    ├── 状态 Badge (Connected / Needs Auth / Disconnected)
    └── Settings / Connect 按钮
```

**状态映射：**
- `connected` → 绿色 badge "Connected"
- `connecting` → 蓝色 badge "Connecting..."
- `disconnected` → 灰色 badge "Disconnected"
- `error` → 红色 badge "Error"
- `disabled` → 灰色 badge "Disabled"
- 未配置 auth → "Needs Auth" + Connect 按钮

### 3.7 Usage Dashboard（全新 Modal）

**新增文件：**
```
src/renderer/components/UsageDashboard.tsx   # Modal 组件
src/renderer/stores/usageStore.ts            # 状态管理
src/main/db/usage.ts                         # SQLite 聚合查询
```

**视图结构（对照原型）：**
```
UsageDashboard (Modal)
├── Header: "Usage Dashboard" + 当前账单周期
├── 3列统计卡片
│   ├── Tokens Used (累计 + 环比)
│   ├── Est. Cost (金额 + 主模型)
│   └── Tasks Executed (总数 + Auto-Router 数)
└── Activity Timeline 柱状图
    └── 最近 7 天每日 token 用量
```

**数据采集（自动，无需用户操作）：**
- 每次 `AgentStreamEvent.type === 'usage'` 时，自动写入 `usage_records` 表
- 位置：在 `chatStore.handleAgentStream()` 中添加 side effect
- 调用 `usage:record` IPC → `src/main/db/usage.ts` INSERT

**聚合查询：**
```typescript
// src/main/db/usage.ts
export function getUsageSummary(periodStart: number, periodEnd: number): UsageSummary
export function getUsageTimeline(days: number): UsageTimelinePoint[]
```

**入口：**
- TopNav 中的 dashboard 图标按钮
- 键盘快捷键（可选）

### 3.8 Pill 浮动通知系统（全新）

**新增文件：**
```
src/renderer/components/FloatingPill.tsx     # Pill UI 组件
src/renderer/stores/pillStore.ts             # 通知状态
```

**Pill 生命周期：**
```
隐藏 → 收起态（单行: 图标 + 文字 + Approve/Dismiss）
     → 展开态（标题 + 描述 + 预览 + 输入框 + 发送）
     → 执行中（Spinner + 进度文字）
     → 完成（Check 图标 + 结果文字 → 自动消失）
```

**触发来源：**
1. `exec.approval.requested` 事件 → Agent 请求执行审批（文件写入、系统日历、shell 命令等）
2. `agent` 事件中的 `tool_use` → 展示 tool 执行预览
3. 主动通知（如 MCP server 断连、skill 安装完成）

**pillStore 状态：**
```typescript
interface PillState {
  visible: boolean
  mode: 'compact' | 'expanded'
  scenario: PillScenario | null
  executing: boolean

  // actions
  showPill: (scenario: PillScenario) => void
  hidePill: () => void
  expand: () => void
  collapse: () => void
  approve: () => Promise<void>
  dismiss: () => void
}

interface PillScenario {
  type: 'approval' | 'tool-preview' | 'notification'
  icon: string           // lucide icon name
  iconColor: string
  title: string
  description: string
  preview?: string       // 代码/内容预览
  sourceTag: string      // e.g. "GitHub MCP", "Local FS"
  approvalId?: string    // exec.approval.requested 的 ID
}
```

**与 Gateway 的对接：**
- `exec.approval.requested` → 构造 PillScenario → `pillStore.showPill()`
- 用户 Approve → `exec.approval.resolve` RPC → 显示执行中 → 完成后自动收起

### 3.9 Tray / Menubar 集成（全新）

**新增文件：**
```
src/main/tray/index.ts                       # Tray 管理
src/main/tray/menubar.ts                     # Menubar 构建
```

**功能：**
- 系统托盘图标（macOS menu bar）
- 状态指示：
  - 绿色 = gateway connected + idle
  - 蓝色脉冲 = agent 执行中
  - 红色 = disconnected / error
- 右键菜单：
  - 显示/隐藏主窗口
  - 当前模型名称
  - Quick Actions
  - 最近活动
  - 设置 / 退出

**与 Renderer 的同步：**
- Gateway 状态变化 → `tray:update-status` IPC → 更新 tray 图标颜色
- Agent 活动 → `tray:activity` IPC → 更新 tray 动画

### 3.10 Settings Modal（精简重构）

**现有文件改动：**
- `views/SettingsView.tsx` — 从 7 tab 精简为 5 tab

**原型 Tab 结构：**
1. **General** — 主题、IDE 路径、开机启动
2. **Models & API** — 模型 catalog 管理（保留现有逻辑）
3. **Integrations** — Gateway URL/Token 配置 + Import from ~/.openclaw
4. **Proactive** — 主动智能各触发源的开关和参数配置
5. **Privacy & Data** — 数据导出、清除历史、隐私设置

### 3.11 Proactive 主动智能系统（全新）

这是产品的核心差异化特性。Agent 不再只是被动应答，而是主动感知环境变化并给出建议。

**核心架构：Electron 主进程作为触发器，Gateway Agent 作为大脑**

```
┌─────────────────────────────────────────────────────────────┐
│ ProactiveManager (src/main/proactive/manager.ts)            │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ScreenVision  │  │GitHubWatcher │  │  FsWatcher   │      │
│  │ desktopCapt. │  │ MCP 轮询/    │  │  chokidar    │      │
│  │ + Vision模型 │  │ webhook      │  │              │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │               │
│  ┌──────┴───────┐  ┌──────┴───────┐                        │
│  │ProcessWatcher│  │DailyRollover │                        │
│  │ exit code ≠0 │  │ app启动检查  │                        │
│  └──────┬───────┘  └──────┬───────┘                        │
│         │                 │                                 │
│         ▼                 ▼                                 │
│  ┌─────────────────────────────────┐                        │
│  │     Trigger Router              │                        │
│  │  构造上下文 → 发送到后台 session │                        │
│  └──────────────┬──────────────────┘                        │
│                 │ chat.send (后台 session)                   │
│                 ▼                                           │
│  ┌─────────────────────────────────┐                        │
│  │   Gateway Agent 分析            │                        │
│  │   返回 ProactiveSuggestion      │                        │
│  └──────────────┬──────────────────┘                        │
│                 │ proactive:suggestion IPC                   │
│                 ▼                                           │
│  ┌─────────────────────────────────┐                        │
│  │   Pill UI (Renderer)            │                        │
│  │   展示建议 → 用户 Approve/Dismiss│                       │
│  └─────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

**新增文件：**
```
src/main/proactive/
├── manager.ts                # ProactiveManager 统一调度器
├── screen-vision.ts          # 屏幕截图分析
├── github-watcher.ts         # GitHub 事件轮询
├── fs-watcher.ts             # 文件系统监听
├── process-watcher.ts        # 后台进程监控
├── daily-rollover.ts         # 每日任务 Rollover 检查
└── trigger-router.ts         # 触发事件 → Agent 后台 session 转发
```

#### 3.11.1 ProactiveManager — 统一调度器

```typescript
// src/main/proactive/manager.ts

export class ProactiveManager {
  private watchers: Map<ProactiveSource, BaseWatcher> = new Map()
  private backgroundSessions: Map<ProactiveSource, string> = new Map()

  constructor(
    private gatewayClient: GatewayClient,
    private db: Database,
    private onSuggestion: (suggestion: ProactiveSuggestion) => void
  ) {}

  /** 根据用户设置启动/停止各触发源 */
  applySettings(settings: ProactiveSettings): void {
    // 对每个 source：设置 enabled 则启动，否则停止
    this.toggleWatcher('screen-vision', settings.screenVision)
    this.toggleWatcher('github-watcher', settings.github)
    this.toggleWatcher('fs-watcher', settings.filesystem)
    this.toggleWatcher('process-watcher', settings.terminal)
    this.toggleWatcher('daily-rollover', settings.dailyRollover)
  }

  /** app 启动时调用 */
  async startup(): Promise<void> {
    // 立即执行 daily-rollover 检查
    await this.watchers.get('daily-rollover')?.trigger()
  }

  /** 全部停止（app 退出时） */
  shutdown(): void {
    for (const watcher of this.watchers.values()) watcher.stop()
  }
}
```

#### 3.11.2 Screen Vision — 看屏幕，主动发现问题

**触发方式：** Electron `desktopCapturer` API + 定时截屏

```typescript
// src/main/proactive/screen-vision.ts
import { desktopCapturer, systemPreferences } from 'electron'

export class ScreenVisionWatcher extends BaseWatcher {
  private intervalId: NodeJS.Timeout | null = null

  start(config: { intervalMs: number; visionModel?: string }): void {
    this.intervalId = setInterval(() => this.capture(), config.intervalMs)
  }

  stop(): void {
    if (this.intervalId) clearInterval(this.intervalId)
  }

  private async capture(): Promise<void> {
    // 1. 检查屏幕录制权限（macOS）
    const hasAccess = systemPreferences.getMediaAccessStatus('screen')
    if (hasAccess !== 'granted') return

    // 2. 截取当前屏幕
    const sources = await desktopCapturer.getSources({
      types: ['screen'],
      thumbnailSize: { width: 1920, height: 1080 }
    })
    const screenshot = sources[0]?.thumbnail.toDataURL()
    if (!screenshot) return

    // 3. 发送到后台 session，让 Vision 模型分析
    this.emit('trigger', {
      source: 'screen-vision',
      context: {
        type: 'screen_analysis',
        image: screenshot,
        instruction: '分析屏幕内容。如果发现代码错误、可优化项或用户可能需要帮助的地方，返回建议。如果没有发现问题，返回 {"noAction": true}。'
      }
    })
  }
}
```

**隐私控制：**
- **默认关闭**，需用户在 Settings > Proactive 中主动开启
- 首次启用时弹出 macOS 屏幕录制权限请求
- 截图仅发给用户配置的模型 API（本地 / 私有部署优先）
- 不持久化截图数据，分析完即丢弃
- 频率可调（默认 30 秒，可改为手动触发）

#### 3.11.3 GitHub Watcher — 监听仓库事件

**触发方式：** 通过已连接的 GitHub MCP server 定时轮询

```typescript
// src/main/proactive/github-watcher.ts

export class GitHubWatcher extends BaseWatcher {
  private intervalId: NodeJS.Timeout | null = null

  start(config: { intervalMs: number }): void {
    this.intervalId = setInterval(() => this.poll(), config.intervalMs)
  }

  private async poll(): Promise<void> {
    // 通过后台 session 让 agent 使用 GitHub MCP tool 查询
    this.emit('trigger', {
      source: 'github-watcher',
      context: {
        instruction: `检查我的 GitHub 仓库近期事件：
1. 是否有新的 PR review 请求分配给我
2. 是否有新的 issue 被分配给我
3. 是否有 CI/CD 构建失败
4. 是否有新的 PR comment 需要回复
只返回需要我立即关注的事件。如果没有，返回 {"noAction": true}。`
      }
    })
  }
}
```

**前置条件：**
- GitHub MCP server 必须已连接（状态为 `connected`）
- 如果 MCP 未连接，此 watcher 自动跳过，不报错

**进阶方案（可选，后续迭代）：**
- 在 Electron 中启动本地 HTTP server 接收 GitHub Webhook
- 通过 smee.io 代理实现实时推送，零延迟

#### 3.11.4 Filesystem Watcher — 监听文件变化

**触发方式：** `chokidar`（已在 package.json 依赖中）

```typescript
// src/main/proactive/fs-watcher.ts
import chokidar from 'chokidar'

export class FsWatcher extends BaseWatcher {
  private watcher: chokidar.FSWatcher | null = null
  private pendingChanges: FileChange[] = []
  private debounceTimer: NodeJS.Timeout | null = null

  start(config: { watchPaths: string[]; debounceMs: number }): void {
    this.watcher = chokidar.watch(config.watchPaths, {
      ignoreInitial: true,
      ignored: [
        '**/node_modules/**',
        '**/.git/**',
        '**/out/**',
        '**/.DS_Store'
      ]
    })

    this.watcher.on('all', (event, path) => {
      this.pendingChanges.push({ type: event, path, timestamp: Date.now() })

      // debounce：合并短时间内的批量变化
      if (this.debounceTimer) clearTimeout(this.debounceTimer)
      this.debounceTimer = setTimeout(() => {
        this.flushChanges()
      }, config.debounceMs)
    })
  }

  private flushChanges(): void {
    if (this.pendingChanges.length === 0) return
    const changes = [...this.pendingChanges]
    this.pendingChanges = []

    // 发给 agent 分析
    this.emit('trigger', {
      source: 'fs-watcher',
      context: {
        changes: changes.map(c => ({ event: c.type, path: c.path })),
        instruction: `用户的文件系统发生了以下变化：
${changes.map(c => `  ${c.type}: ${c.path}`).join('\n')}
分析这些变化，判断是否有可以帮助用户的操作：
- 多个同类文件下载 → 提议合并/整理
- 代码文件变化 → 检查是否引入错误
- 配置文件变化 → 提醒可能的副作用
如果没有值得提醒的，返回 {"noAction": true}。`
      }
    })
  }

  stop(): void {
    this.watcher?.close()
  }
}
```

**默认监听路径：**
- `~/Downloads` — 检测新下载的文件
- 当前 workspace 目录 — 检测代码 / 配置变化
- 用户可在 Settings 中自定义追加路径

#### 3.11.5 Process Watcher — 后台进程 / 终端错误监控

**触发方式：** 监听 agent tool calling 中 bash/exec 的 exit code

```typescript
// src/main/proactive/process-watcher.ts

export class ProcessWatcher extends BaseWatcher {
  /**
   * 当 gateway 返回 tool_result 且 isError === true 时调用
   * 由 GatewayClient 的事件处理器触发
   */
  onToolError(event: {
    toolCallId: string
    name: string        // e.g. 'bash', 'exec'
    output: string      // stderr 内容
    sessionKey: string
  }): void {
    this.emit('trigger', {
      source: 'process-watcher',
      context: {
        toolName: event.name,
        errorOutput: event.output,
        sessionKey: event.sessionKey,
        instruction: `后台执行的 ${event.name} 命令失败了：
${event.output}
分析错误原因，给出修复建议。如果是缺少依赖，建议安装命令。如果是代码错误，指出错误位置和修复方法。`
      }
    })
  }
}
```

**触发点（在现有代码中挂钩）：**
- `src/main/gateway/client.ts` 中处理 `agent` event 的 `tool_result` 时
- 如果 `isError === true`，调用 `processWatcher.onToolError()`

#### 3.11.6 Daily Rollover — 每日任务检查

**触发方式：** App 启动时自动执行

```typescript
// src/main/proactive/daily-rollover.ts

export class DailyRolloverWatcher extends BaseWatcher {
  constructor(private db: Database) { super() }

  async trigger(): Promise<void> {
    const yesterday = Date.now() - 86_400_000
    const incompleteTasks = this.db.prepare(`
      SELECT * FROM planner_tasks
      WHERE completed = 0 AND created_at < ?
      ORDER BY created_at DESC
    `).all(yesterday) as PlannerTask[]

    if (incompleteTasks.length === 0) return

    // 直接构造建议（不需要 agent 分析）
    this.emit('suggestion', {
      id: `rollover-${Date.now()}`,
      source: 'daily-rollover',
      title: `${incompleteTasks.length} 个未完成任务`,
      description: `昨天有 ${incompleteTasks.length} 个任务未完成，已自动滚入今天。`,
      preview: incompleteTasks.map(t => `- ${t.title}`).join('\n'),
      previewType: 'markdown',
      actions: [
        { label: 'View in Planner', type: 'open', payload: { view: 'planner' } },
        { label: 'Let AI prioritize', type: 'approve', payload: { tasks: incompleteTasks } }
      ],
      createdAt: Date.now(),
      dismissed: false,
      executed: false
    })

    // 如果用户选择 "Let AI prioritize"，则发送到 agent
    // agent 返回重新排序和建议的日程安排
  }
}
```

**额外触发（可选）：**
- 每天首次打开 app 时触发
- 或者设定固定时间（如每天 9:00）通过 `setTimeout` 触发

#### 3.11.7 Trigger Router — 统一转发到 Gateway Agent

```typescript
// src/main/proactive/trigger-router.ts

export class TriggerRouter {
  // 每个 source 使用独立的后台 session，避免互相干扰
  private sessionKeys: Record<ProactiveSource, string> = {
    'screen-vision':    '__proactive_screen__',
    'github-watcher':   '__proactive_github__',
    'fs-watcher':       '__proactive_fs__',
    'process-watcher':  '__proactive_terminal__',
    'daily-rollover':   '__proactive_planner__'
  }

  constructor(private gatewayClient: GatewayClient) {}

  async route(trigger: {
    source: ProactiveSource
    context: Record<string, unknown>
  }): Promise<ProactiveSuggestion | null> {
    const sessionKey = this.sessionKeys[trigger.source]

    // 发送到 gateway agent
    const result = await this.gatewayClient.sendAgentMessage({
      sessionKey,
      message: JSON.stringify(trigger.context)
    })

    // 解析 agent 返回
    // 如果 agent 返回 {"noAction": true}，返回 null（不弹 Pill）
    // 否则构造 ProactiveSuggestion
    return this.parseSuggestion(trigger.source, result)
  }
}
```

**关键设计：后台 session 不显示在 UI 的 session 列表中**
- session key 以 `__proactive_` 前缀标识
- `sessionStore` 过滤掉这些 session，不在 sidebar 中显示
- 后台 session 不计入 usage dashboard（可选，或单独分类）

#### 3.11.8 Pill 与 Proactive 的对接

Proactive 建议通过 Pill 展示，复用已有的 `pillStore`：

```typescript
// proactive:suggestion IPC → pillStore

function handleProactiveSuggestion(suggestion: ProactiveSuggestion) {
  pillStore.showPill({
    type: 'proactive',
    icon: SOURCE_ICON_MAP[suggestion.source],    // scan-eye / git-pull-request / folder / terminal / calendar
    iconColor: SOURCE_COLOR_MAP[suggestion.source],
    title: suggestion.title,
    description: suggestion.description,
    preview: suggestion.preview,
    sourceTag: SOURCE_TAG_MAP[suggestion.source], // "Screen Vision" / "GitHub MCP" / ...
    proactiveSuggestionId: suggestion.id
  })
}
```

**原型中 Pill 的 5 种 proactive 场景映射：**

| 原型按钮 | Source | Pill 样式 |
|----------|--------|-----------|
| "Pill: Screen Vision" → Fix TypeScript Error | `screen-vision` | `theme-code` + `scan-eye` 图标 |
| "Pill: GitHub MCP" → Review requested: UI Fixes | `github-watcher` | `theme-github` + `git-pull-request` 图标 |
| "Pill: Local FS" → Process 5 downloaded CSVs? | `fs-watcher` | `theme-system` + `file-spreadsheet` 图标 |
| (原型 terminal scenario) → Build failed. Fix it? | `process-watcher` | `theme-system` + `terminal-square` 图标 |
| Rollover Tasks | `daily-rollover` | `theme-calendar` + `calendar-clock` 图标 |

#### 3.11.9 去重与频率控制

防止同一问题反复弹出 Pill 骚扰用户：

```typescript
// 在 TriggerRouter 中
private recentSuggestions: Map<string, number> = new Map()
private COOLDOWN_MS = 300_000  // 5 分钟内同类建议不重复

private shouldSuppress(suggestion: ProactiveSuggestion): boolean {
  const key = `${suggestion.source}:${suggestion.title}`
  const lastShown = this.recentSuggestions.get(key)
  if (lastShown && Date.now() - lastShown < this.COOLDOWN_MS) return true
  this.recentSuggestions.set(key, Date.now())
  return false
}
```

**额外策略：**
- 用户 Dismiss 的建议，同类 24 小时内不再弹出
- 写入 `proactive_log` 表，用于历史回溯
- 后台 session 的 agent 收到 system prompt 说明："只在发现真正需要关注的问题时才返回建议"

### 3.12 Auto-Router 智能模型路由（全新）

利用 OpenClaw 已有的 per-message model 覆盖 + thinkingLevel variant 基建，在客户端加一层路由决策。

**新增文件：**
```
src/main/router/auto-router.ts              # 路由引擎（规则 + LLM 分类）
src/main/router/rules.ts                    # 规则定义
src/main/db/router.ts                       # 路由日志 CRUD
```

**核心架构：规则优先 + LLM 兜底**

```
用户输入 message
  │
  ▼
┌─────────────────────────────┐
│ Step 1: 规则快速匹配         │
│  消息长度 / 关键词 / 模式    │
│  → 能确定 tier? ────Yes────→ 直接返回 RouterDecision
│         │                    │
│         No                   │
│         ▼                    │
│ Step 2: LLM 分类（可选）     │
│  用最便宜模型分类意图         │
│  → 返回 tier                │
│         │                    │
│         ▼                    │
│ Step 3: Rate-limit 检查      │
│  选中模型在冷却中?           │
│  → 是：自动降级到同 tier      │
│         备选或低一档模型       │
│  → 否：使用选中模型           │
└──────────────┬──────────────┘
               ▼
         RouterDecision {
           tier, modelRef,
           thinkingLevel, reason
         }
               │
               ▼
         chat.send({ model: modelRef, thinkingLevel })
```

#### 3.12.1 规则引擎

```typescript
// src/main/router/rules.ts

interface RouterRule {
  name: string
  match: (ctx: RouteContext) => boolean
  tier: RouterTier
  confidence: 'high' | 'low'  // high = 直接决定，low = 仅作为 hint
}

interface RouteContext {
  message: string
  messageLength: number
  sessionHistory: number       // 当前 session 已有多少轮对话
  hasCodeBlock: boolean        // 消息中是否包含代码块
  hasImage: boolean            // 是否附带图片
  recentModel?: string         // 上一轮用的什么模型（保持一致性）
}

const RULES: RouterRule[] = [
  // ── 高置信规则：直接决定 ──

  // 附带图片 → 必须用 vision 模型（heavy tier 通常有 vision）
  {
    name: 'has-image',
    match: (ctx) => ctx.hasImage,
    tier: 'heavy',
    confidence: 'high'
  },
  // 极短消息（< 50 字，无代码）→ 轻量
  {
    name: 'short-simple',
    match: (ctx) => ctx.messageLength < 50 && !ctx.hasCodeBlock,
    tier: 'light',
    confidence: 'high'
  },
  // 超长消息或包含大段代码 → 重型
  {
    name: 'long-complex',
    match: (ctx) => ctx.messageLength > 2000 || (ctx.hasCodeBlock && ctx.messageLength > 500),
    tier: 'heavy',
    confidence: 'high'
  },

  // ── 低置信规则：作为 hint，可被 LLM 覆盖 ──

  // 关键词匹配：架构/重构/设计 → heavy
  {
    name: 'keyword-heavy',
    match: (ctx) => /架构|重构|设计方案|性能优化|安全审计|debug|分析.*原因/.test(ctx.message),
    tier: 'heavy',
    confidence: 'low'
  },
  // 关键词匹配：翻译/总结/解释 → light
  {
    name: 'keyword-light',
    match: (ctx) => /翻译|总结|解释一下|什么意思|帮我改|格式化/.test(ctx.message),
    tier: 'light',
    confidence: 'low'
  },
]

export function ruleBasedClassify(ctx: RouteContext): { tier: RouterTier; confident: boolean } | null {
  for (const rule of RULES) {
    if (rule.match(ctx)) {
      return { tier: rule.tier, confident: rule.confidence === 'high' }
    }
  }
  return null  // 规则无法判断
}
```

#### 3.12.2 LLM 分类器

```typescript
// src/main/router/auto-router.ts

async function llmClassify(
  message: string,
  classifyModel: string,
  gatewayClient: GatewayClient
): Promise<RouterTier> {
  // 用最便宜的模型做分类（glm-4-flash 免费 / qwen-turbo $0.04/M）
  const result = await gatewayClient.sendAgentMessage({
    sessionKey: '__router_classify__',
    model: classifyModel,
    message: `你是一个任务复杂度分类器。只返回一个词：heavy / medium / light。

分类标准：
- heavy：深度推理、长文档分析、架构设计、复杂 debug、涉及多文件重构
- medium：普通代码生成、功能实现、中等复杂度问答
- light：简单问答、格式化、翻译、解释概念

用户请求：${message.slice(0, 500)}

复杂度：`
  })
  // 解析返回，fallback 到 medium
  const text = extractTextFromResult(result).trim().toLowerCase()
  if (['heavy', 'medium', 'light'].includes(text)) return text as RouterTier
  return 'medium'
}
```

#### 3.12.3 路由主函数

```typescript
// src/main/router/auto-router.ts

export async function autoRoute(
  message: string,
  settings: RouterSettings,
  catalog: ModelConfig[],
  rateLimits: Record<string, { untilMs: number }>,
  gatewayClient: GatewayClient
): Promise<RouterDecision> {
  const ctx: RouteContext = {
    message,
    messageLength: message.length,
    sessionHistory: 0,  // 可扩展
    hasCodeBlock: /```[\s\S]*```/.test(message),
    hasImage: false,     // 从 params 传入
  }

  // Step 1: 规则匹配
  const ruleResult = ruleBasedClassify(ctx)
  let tier: RouterTier
  let reason: RouterDecision['reason']

  if (ruleResult?.confident) {
    // 规则高置信 → 直接用
    tier = ruleResult.tier
    reason = 'rule'
  } else if (settings.llmClassifyEnabled) {
    // 规则低置信或无结果 → LLM 分类
    tier = await llmClassify(message, settings.classifyModel || findCheapestModel(catalog), gatewayClient)
    reason = 'llm-classify'
  } else {
    // LLM 分类未开启 → 用规则 hint 或默认 medium
    tier = ruleResult?.tier || 'medium'
    reason = 'rule'
  }

  // Step 2: 取该 tier 对应的模型
  let { modelRef, thinkingLevel } = settings.tierMap[tier]

  // Step 3: Rate-limit fallback
  if (settings.costAwareFallback && rateLimits[modelRef]?.untilMs > Date.now()) {
    // 选中模型在冷却中 → 降级
    const fallback = findFallbackModel(tier, settings.tierMap, rateLimits)
    if (fallback) {
      modelRef = fallback.modelRef
      thinkingLevel = fallback.thinkingLevel
      reason = 'rate-limit-fallback'
    }
  }

  return { tier, modelRef, thinkingLevel, reason }
}

function findCheapestModel(catalog: ModelConfig[]): string {
  // 从 catalog 中找价格最低的模型用于分类
  // 优先 glm-4-flash (免费) → qwen-turbo → deepseek-chat
  const cheapOrder = ['glm-4-flash', 'glm-z1-flash', 'qwen-turbo', 'deepseek-chat']
  for (const prefix of cheapOrder) {
    const found = catalog.find(m => m.model.startsWith(prefix))
    if (found) return toGatewayModelRef(found.provider, found.model)
  }
  return catalog[0] ? toGatewayModelRef(catalog[0].provider, catalog[0].model) : ''
}
```

#### 3.12.4 接入点（改动极小）

只需改 `src/main/ipc.ts` 中 `AGENT_SEND` handler 的模型解析逻辑：

```typescript
// src/main/ipc.ts — AGENT_SEND handler (line ~175)

// 现有：
// const resolvedModel = params.model || fallbackModel

// 改为：
let resolvedModel: string | undefined
let resolvedThinkingLevel = params.thinkingLevel

if (params.model) {
  // 用户手动选了模型 → 尊重用户选择
  resolvedModel = params.model
} else if (settings.router?.enabled) {
  // Auto-Router 启用 → 自动路由
  const decision = await autoRoute(
    params.message,
    settings.router,
    settings.models.catalog,
    getModelRateLimits(),      // 从 renderer 同步过来的冷却记录
    gatewayClient
  )
  resolvedModel = decision.modelRef
  resolvedThinkingLevel = resolvedThinkingLevel || decision.thinkingLevel
  // 记录路由日志
  logRouterDecision(db, params.sessionKey, decision, params.message)
} else {
  // 都没有 → 用默认模型
  resolvedModel = fallbackModel
}
```

#### 3.12.5 UI 展示

**消息气泡上的模型 tag：**
- Auto-Router 路由的消息，tag 显示模型名 + 小图标 (如 `🔀 GLM-4.7-Flash`)
- 手动选择的消息，tag 只显示模型名
- `ChatMessage` 已有 `requestedModel` 字段，追加 `routedBy?: RouterDecision['reason']`

**Usage Dashboard 中的统计：**
- "28 via Auto-Router" — 从 `router_log` 表 COUNT WHERE reason != 'user-override'
- 可展示 tier 分布饼图：heavy 20% / medium 50% / light 30%
- 可展示 routing 省下的成本估算

**Settings > Models & API tab 中新增 Auto-Router 开关区域：**
```
┌────────────────────────────────────────────┐
│ Auto-Router                          [ON]  │
│                                            │
│ 🔴 Heavy    Claude Opus 4.6    ▼  high  ▼ │
│ 🟡 Medium   GLM-4.7            ▼  medium▼ │
│ 🟢 Light    GLM-4.7-Flash      ▼  off   ▼ │
│                                            │
│ ☑ LLM 分类（规则不确定时）                  │
│   分类模型: GLM-4-Flash (免费)      ▼      │
│ ☑ Rate-limit 自动降级                      │
└────────────────────────────────────────────┘
```

---

## 4. 数据流总览

### 4.1 Chat 对话流（现有，无变化）

```
用户输入
  → ChatInput.onSend()
  → chatStore.sendMessage()
  → window.electronAPI.agent.send({ sessionKey, message })
  → [Main] ipc.ts handler
  → [Main] gatewayClient.sendAgentMessage()
  → [Gateway] chat.send RPC
  → [Gateway] chat/agent events (streaming)
  → [Main] client.on('chat', event)
  → [Main] ipcMain.emit('agent:stream', event)
  → [Renderer] chatStore.handleAgentStream(event)
  → MessageItem re-render
```

### 4.2 Planner 日程流（新增）

```
读取：
  PlannerView mount
  → plannerStore.fetchEvents(today)
  → window.electronAPI.planner.listEvents(date)
  → [Main] db/planner.ts SELECT + calendar/index.ts readSystemCalendarEvents()
  → 合并结果返回

创建（用户手动）：
  Add Task 按钮
  → plannerStore.createTask(title)
  → window.electronAPI.planner.createEvent(data)
  → [Main] db/planner.ts INSERT
  → 返回新 event

创建（Agent 对话中）：
  Agent tool_use: create_calendar_event
  → exec.approval.requested event
  → Pill 弹出审批
  → 用户 Approve
  → exec.approval.resolve RPC
  → [Main] calendar/index.ts writeSystemCalendarEvent()
  → iCloud 同步到所有设备
  → tool_result 返回 agent
```

### 4.3 MemOS 记忆流（新增）

```
读取：
  MemosView mount
  → memosStore.fetchFacts()
  → window.electronAPI.memos.listFacts()
  → [Main] db/memos.ts SELECT
  → 返回 MemOSFact[]

Agent 自动提取：
  Agent 对话中识别到用户偏好
  → Agent tool_use: add_memory_fact
  → [Main] db/memos.ts INSERT
  → memosStore 刷新

注入 system prompt：
  用户发送消息前
  → [Main] 读取 memos_directive + 最近 facts
  → chat.inject({ role: 'system', content: directive + facts })
  → 再 chat.send({ message })
```

### 4.4 Usage 统计流（新增）

```
采集（自动）：
  Agent stream 完成
  → AgentStreamEvent.type === 'usage'
  → chatStore.handleAgentStream() 中 side effect
  → window.electronAPI.usage.record(usageData)
  → [Main] db/usage.ts INSERT

查询：
  UsageDashboard 打开
  → usageStore.fetchSummary(period)
  → window.electronAPI.usage.getSummary(start, end)
  → [Main] db/usage.ts 聚合查询
  → 返回 UsageSummary + UsageTimelinePoint[]
```

### 4.5 Proactive 主动智能流（新增）

```
触发（以 Screen Vision 为例）：
  定时器触发（每 30s）
  → [Main] ScreenVisionWatcher.capture()
  → desktopCapturer.getSources() 截屏
  → TriggerRouter.route({ source: 'screen-vision', context: { image, instruction } })
  → gatewayClient.sendAgentMessage({
      sessionKey: '__proactive_screen__',
      message: 截图 + 分析指令
    })
  → [Gateway] chat.send → Agent 使用 Vision 模型分析
  → Agent 返回建议（或 noAction）
  → [Main] TriggerRouter.parseSuggestion()
  → 去重检查 (shouldSuppress)
  → proactive:suggestion IPC → [Renderer]
  → pillStore.showPill(suggestion)
  → Pill 弹出

用户交互：
  ├── Approve → proactive:approve IPC
  │   → [Main] 执行对应操作（修复代码 / 合并 PR / 安装依赖...）
  │   → proactive_log UPDATE executed = 1
  │   → Pill 显示 "Done" → 自动消失
  │
  ├── Dismiss → proactive:dismiss IPC
  │   → proactive_log UPDATE dismissed = 1
  │   → 同类建议 24h 内不再弹出
  │   → Pill 消失
  │
  └── 展开 → 用户可修改指令后再执行
      → Pill expanded view → 用户输入修改
      → proactive:approve IPC + 自定义 payload

文件系统触发流：
  chokidar 检测到 ~/Downloads 新增 5 个 CSV
  → FsWatcher debounce 5s → flushChanges()
  → TriggerRouter.route({ source: 'fs-watcher', context: { changes, instruction } })
  → Agent 分析："用户下载了 5 个 CSV，建议合并"
  → Pill: "Process 5 downloaded CSVs?" [Approve] [Dismiss]

GitHub 触发流：
  定时器触发（每 60s）
  → GitHubWatcher.poll()
  → TriggerRouter.route → Agent 使用 GitHub MCP tool 查询
  → 发现新 PR review 请求
  → Pill: "Review requested: UI Fixes" [Review] [Dismiss]

进程错误触发流：
  Agent tool_result.isError === true
  → ProcessWatcher.onToolError({ name: 'bash', output: stderr })
  → TriggerRouter.route → Agent 分析错误
  → Pill: "Build failed. Fix it?" [Fix] [Dismiss]

每日 Rollover 触发流：
  App 启动
  → DailyRolloverWatcher.trigger()
  → SQLite 查询昨日未完成任务
  → 直接构造 suggestion（不需要 Agent）
  → Pill: "3 个未完成任务" [View in Planner] [Let AI prioritize]
```

### 4.6 Auto-Router 路由流（新增）

```
用户发送消息（未手动选择模型）：
  ChatInput.onSend({ message })
  → agent:send IPC → [Main] ipc.ts AGENT_SEND handler
  → 检查: params.model 存在? → 是: 跳过路由，直接用
  → 检查: settings.router.enabled? → 否: 用 fallbackModel
  → 是: autoRoute(message, settings, catalog, rateLimits)

    规则匹配阶段：
    → ruleBasedClassify(ctx)
    → 消息 < 50 字且无代码? → tier = 'light' (confident)
    → 消息 > 2000 字或含大段代码? → tier = 'heavy' (confident)
    → 关键词匹配 "架构/重构"? → tier = 'heavy' (hint, not confident)
    → 无匹配? → null

    LLM 分类阶段（规则不确定时）：
    → llmClassify(message, 'zai/glm-4-flash')
    → chat.send 到 __router_classify__ 后台 session
    → Agent 返回: "medium"
    → tier = 'medium'

    Rate-limit 检查：
    → tierMap[medium].modelRef = 'zai/glm-4.7'
    → rateLimits['zai/glm-4.7'] 在冷却中?
    → 是: findFallbackModel() → 降级到 'zai/glm-4.7-flash'
    → 否: 使用 'zai/glm-4.7'

  → RouterDecision { tier: 'medium', modelRef: 'zai/glm-4.7', reason: 'llm-classify' }
  → logRouterDecision(db, sessionKey, decision)  // 记录日志
  → chat.send({ model: 'zai/glm-4.7', thinkingLevel: 'medium', message })
  → 消息气泡显示: "🔀 GLM-4.7" tag
```

---

## 5. 文件结构变更

### 5.1 新增文件

```
src/
├── main/
│   ├── calendar/
│   │   └── index.ts              # macOS Calendar AppleScript 桥接
│   ├── db/
│   │   ├── index.ts              # SQLite 初始化 + migration
│   │   ├── planner.ts            # planner_events / planner_tasks CRUD
│   │   ├── memos.ts              # memos_facts / memos_directive CRUD
│   │   ├── usage.ts              # usage_records CRUD + 聚合查询
│   │   ├── proactive.ts          # proactive_log CRUD + 去重查询
│   │   └── router.ts             # router_log CRUD + 统计查询
│   ├── tray/
│   │   └── index.ts              # Tray 图标 + 菜单 + 状态同步
│   └── proactive/
│       ├── manager.ts            # ProactiveManager 统一调度器
│       ├── base-watcher.ts       # BaseWatcher 抽象基类
│       ├── screen-vision.ts      # 屏幕截图分析 (desktopCapturer)
│       ├── github-watcher.ts     # GitHub 事件轮询 (MCP)
│       ├── fs-watcher.ts         # 文件系统监听 (chokidar)
│       ├── process-watcher.ts    # 后台进程/终端错误监控
│       ├── daily-rollover.ts     # 每日任务 Rollover 检查
│       └── trigger-router.ts     # 触发事件 → Agent 后台 session 转发
│   └── router/
│       ├── auto-router.ts        # 路由主函数 + LLM 分类器
│       └── rules.ts              # 规则定义
│
├── renderer/
│   ├── views/
│   │   ├── PlannerView.tsx       # 日程规划视图
│   │   ├── SkillsView.tsx        # Skills Library 视图
│   │   ├── McpView.tsx           # MCP Servers 视图
│   │   └── MemosView.tsx         # MemOS 记忆视图
│   ├── stores/
│   │   ├── plannerStore.ts       # Planner 状态
│   │   ├── memosStore.ts         # MemOS 状态
│   │   ├── usageStore.ts         # Usage 统计状态
│   │   └── pillStore.ts          # Pill 通知状态（含 proactive 建议）
│   └── components/
│       ├── FloatingPill.tsx       # 浮动 Pill 通知组件
│       ├── UsageDashboard.tsx     # 用量统计 Modal
│       └── ViewNav.tsx            # 左侧视图导航栏
│
├── shared/
│   └── types.ts                   # += PlannerEvent, MemOSFact, UsageRecord, Proactive*, Router* 等
│   └── ipc-channels.ts           # += planner:*, memos:*, usage:*, tray:*, proactive:*, router:*
```

### 5.2 现有文件改动

```
src/renderer/layouts/MainLayout.tsx        # 重构: 集成 ViewNav + 多视图切换
src/renderer/panels/ChatPanel/index.tsx    # UI 对齐原型 + Quick Actions
src/renderer/components/ChatInput.tsx      # UI 对齐原型
src/renderer/components/MessageItem.tsx    # UI 对齐原型 (模型 tag 等)
src/renderer/views/SettingsView.tsx        # 精简为 5 tab（含 Proactive 配置 + Router 配置）
src/renderer/stores/settingsStore.ts       # currentView 类型扩展 + ProactiveSettings + RouterSettings
src/renderer/stores/chatStore.ts           # += usage 采集 side effect
src/renderer/App.tsx                       # += Pill 挂载 + 新 IPC listener (含 proactive:suggestion)
src/main/ipc.ts                            # += 新 IPC handler 注册 (含 proactive:* + router:*) + AGENT_SEND 路由逻辑
src/main/index.ts                          # += ProactiveManager 初始化和启动
src/main/gateway/client.ts                 # += tool_result.isError 时通知 ProcessWatcher
src/preload/index.ts                       # += 新 API 暴露
src/shared/ipc-channels.ts                # += 新 channel 常量
src/shared/types.ts                        # += 新类型定义
```

---

## 6. 开发阶段规划

### Phase 1: 基础设施（预计 2 天）

**目标：** 搭建新模块的底层支撑

- [ ] SQLite 数据层 (`src/main/db/`) — 建表、migration、CRUD
- [ ] 新增 IPC channels 和 types
- [ ] Preload 暴露新 API
- [ ] 视图导航系统 (`ViewNav` + `currentView` 扩展)

**验证：** `npm run typecheck` 通过，`npm run dev` 启动后视图切换正常

### Phase 2: Chat UI 重构（预计 1 天）

**目标：** Chat 视图样式对齐原型

- [ ] MainLayout 重构（顶栏 + 3列 + ViewNav）
- [ ] ChatPanel 样式对齐
- [ ] MessageItem 样式对齐（模型 tag、气泡风格）
- [ ] ChatInput 样式对齐 + Quick Actions 行

**验证：** 与原型视觉一致，对话功能不退化

### Phase 3: Planner 日程模块（预计 2 天）

**目标：** Daily Planner 完整可用

- [ ] PlannerView 视图
- [ ] plannerStore 状态管理
- [ ] 本地事件/任务 CRUD
- [ ] macOS 系统日历读取集成
- [ ] 系统日历写入（含权限处理）
- [ ] Agent tool 对接（create_calendar_event → Pill 审批 → 写入）

**验证：** 能显示今日日程，能创建任务，能同步系统日历，agent 对话中能触发日历写入

### Phase 4: MemOS 记忆模块（预计 1.5 天）

**目标：** MemOS 完整可用

- [ ] MemosView 视图
- [ ] memosStore 状态管理
- [ ] Fact CRUD + 搜索
- [ ] Core Directive 编辑 + 持久化
- [ ] Directive 注入到 chat.send 前的 system prompt
- [ ] Active Contexts 展示

**验证：** 能增删改查 facts，directive 能影响 agent 回复

### Phase 5: Skills + MCP 独立视图（预计 1 天）

**目标：** Skills 和 MCP 从嵌套位置独立为一级视图

- [ ] SkillsView（复用现有数据流 + 新 UI）
- [ ] McpView（复用现有数据流 + 新 UI）

**验证：** 功能与原有一致，UI 对齐原型

### Phase 6: Usage Dashboard + Auto-Router（预计 1.5 天）

**目标：** 用量统计可用 + 智能路由可用

- [ ] usageStore 状态管理
- [ ] chatStore += usage 事件自动采集
- [ ] UsageDashboard Modal 组件
- [ ] 聚合查询 + 柱状图渲染
- [ ] Auto-Router 规则引擎 (`src/main/router/rules.ts`)
- [ ] Auto-Router LLM 分类器 + 主函数 (`src/main/router/auto-router.ts`)
- [ ] ipc.ts AGENT_SEND handler 接入路由逻辑
- [ ] Rate-limit fallback 自动降级
- [ ] router_log 持久化 + Dashboard 中 "via Auto-Router" 统计
- [ ] Settings > Models & API tab 中 Auto-Router 开关 + tier 配置 UI
- [ ] MessageItem 模型 tag 显示路由来源标识

**验证：**
- 对话后 dashboard 数据更新，图表渲染正确
- 开启 Auto-Router → 短消息自动用 light 模型 → 长/复杂消息自动用 heavy 模型
- 手动选择模型时 Auto-Router 不干预
- rate-limited 模型自动降级到备选

### Phase 7: Pill + Tray（预计 2 天）

**目标：** 系统级交互完整

- [ ] FloatingPill 组件 + pillStore
- [ ] Pill 与 exec.approval.requested 对接
- [ ] Pill 展开/收起/执行/完成 全生命周期
- [ ] Pill 与 proactive:suggestion 对接（复用同一 Pill 组件）
- [ ] Tray 图标 + 状态同步
- [ ] Tray 右键菜单

**验证：** Agent tool calling 触发 Pill，Approve 后执行成功。Tray 图标反映 gateway 状态

### Phase 8: Proactive 主动智能（预计 2.5 天）

**目标：** 5 种 proactive 触发源全部可用

- [ ] BaseWatcher 抽象基类 + ProactiveManager 调度器
- [ ] TriggerRouter + 后台 session 管理
- [ ] Screen Vision (desktopCapturer + Vision 模型)
- [ ] GitHub Watcher (MCP 轮询)
- [ ] Filesystem Watcher (chokidar)
- [ ] Process Watcher (tool_result.isError 挂钩)
- [ ] Daily Rollover (SQLite 查询 + 启动检查)
- [ ] 去重与频率控制 (cooldown + dismiss 记录)
- [ ] Settings > Proactive tab（各触发源开关和参数配置）
- [ ] proactive_log 持久化

**验证：**
- 开启 Screen Vision → 截屏分析 → Pill 弹出代码错误建议
- GitHub MCP 连接状态下 → 轮询到新 PR → Pill 弹出 review 请求
- 往 ~/Downloads 放入多个 CSV → Pill 弹出合并建议
- Agent 执行 bash 失败 → Pill 弹出修复建议
- 创建昨天的未完成任务 → 重启 app → Pill 弹出 Rollover 提醒

### Phase 9: 联调与打磨（预计 1.5 天）

- [ ] 全流程联调：对话 → tool calling → Pill 审批 → 日历写入 → Usage 记录
- [ ] Proactive 全链路联调：触发 → Agent 分析 → Pill → Approve → 执行
- [ ] 主题切换（light/dark/system）全视图适配
- [ ] 键盘快捷键补全
- [ ] Edge case 处理（断连恢复、权限拒绝、空状态、截屏权限未授予）
- [ ] `npm run typecheck` + `npm run build` 通过

**总预估：~14.5 天**

---

## 7. 关键设计决策记录

| 决策 | 选择 | 理由 |
|------|------|------|
| 本地数据库 | better-sqlite3 | 已在依赖中，同步 API 简单可靠，单文件无额外配置 |
| macOS 日历集成 | osascript (AppleScript) | 零额外依赖，Electron 可直接调用，支持读写 |
| Pill 审批 vs 自动执行 | 所有系统操作需 Pill 审批 | 安全第一，防止 AI 误操作用户真实数据 |
| MemOS 存储位置 | 本地 SQLite（非 gateway） | Gateway 无 memory API，本地存储保证数据主权 |
| Usage 采集触发 | chatStore stream handler 中自动采集 | 无需用户操作，每次对话自动记录 |
| 视图导航 | Zustand state（非 react-router） | 延续现有架构，不引入路由库 |
| 日程数据架构 | 本地 SQLite 为主 + 系统日历只读同步 | 默认可用不依赖外部；写入系统日历需用户显式确认 |
| 日历服务选择 | macOS Calendar + 可选飞书 MCP | 国内可用，不依赖 Google 服务 |
| Proactive 触发架构 | Electron 主进程触发 + Gateway Agent 分析 | Gateway 无 proactive API，客户端作为触发器最灵活 |
| Proactive 后台 session | 独立 session（`__proactive_*` 前缀） | 不污染用户可见的对话历史，各 source 隔离 |
| Screen Vision 默认状态 | 默认关闭 | 屏幕截图涉及隐私，必须用户主动开启 |
| Proactive 去重策略 | 5 分钟 cooldown + dismiss 24h 屏蔽 | 避免反复弹出相同建议骚扰用户 |
| 文件监听 | chokidar | 已在依赖中，跨平台，支持 ignore pattern |

---

## 8. 开发注意事项

1. **UI 实现参照原型文件**：`openclaw-ultimate-desktop-v13.html` 是 UI 的唯一参考源。所有颜色、间距、字体、布局严格对齐原型。

2. **不破坏现有功能**：Chat、Session、Gateway、Settings 的现有逻辑必须保持可用。新增功能是增量式的。

3. **进程边界清晰**：SQLite / osascript / 文件操作只在 Main process。Renderer 只通过 IPC 调用。

4. **类型安全**：所有新增 wire types 放 `src/shared/types.ts`，IPC channels 放 `src/shared/ipc-channels.ts`。不使用 `any`。

5. **主题适配**：所有新视图必须同时支持 light 和 dark 模式。使用 CSS 变量（`var(--text-primary)` 等）。

6. **错误处理**：macOS 日历权限拒绝、Gateway 断连、SQLite 写入失败等场景都需要优雅处理，向用户展示 actionable 的错误提示。

7. **Proactive 隐私安全**：
   - Screen Vision 默认关闭，需用户主动在 Settings 中开启。
   - 截图数据不持久化，分析完即丢弃。
   - 所有 proactive 操作必须经过 Pill 用户确认，绝不自动执行。
   - 后台 session 不在 UI 中显示，不暴露分析过程给用户。
   - Settings 中每个触发源独立开关，用户可精细控制。

8. **Proactive 性能考量**：
   - Screen Vision 截图 + 模型分析是 CPU/网络密集操作，间隔不宜太短（最小 15 秒建议）。
   - chokidar 的 debounce 窗口防止文件系统 burst 事件触发过多 agent 请求。
   - GitHub 轮询使用条件请求（ETag / If-Modified-Since）减少无效 API 调用。
   - 后台 session 的 agent 使用轻量模型（如 flash/mini 系列）降低 cost。
