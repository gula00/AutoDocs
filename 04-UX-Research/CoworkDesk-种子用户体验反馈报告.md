# CoworkDesk 种子用户体验反馈报告

**署名**: opus 4.6 体验的反馈报告
**测试日期**: 2026-02-21
**测试环境**: macOS, Electron 33.4.11, Protocol v3, Gateway Connected
**测试身份**: OpenClaw 214k+ stars 重度用户 / 日均 AI 工具使用 12h+

---

## 执行摘要

CoworkDesk 是一个有明确野心的 AI 桌面协作平台 -- 它想成为连接 OpenClaw Gateway 的官方桌面 GUI。**视觉层面已经接近 Raycast 级别**，5 套皮肤系统、OLED 纯黑背景、毛玻璃效果和语义化颜色都体现出了设计品味。**协议层面工程扎实**，WebSocket 客户端对 Gateway 协议的适配非常全面，tool call 去重、reconnect 退避、challenge-response 握手一应俱全。但作为一个要让 power user 从 OpenClaw TUI / Cursor 迁移过来的桌面应用，**当前版本存在多处功能空壳、交互死角和架构隐患**，距离"真正能干活"还有明显的差距。

**总体评分: 62 / 100**

---

## 评分表

| 维度 | 得分 | 权重 | 加权分 |
|------|------|------|--------|
| 首次印象与上手 | 7/10 | 15% | 1.05 |
| 视觉设计与审美 | 8/10 | 15% | 1.20 |
| 对话与流式体验 | 6/10 | 20% | 1.20 |
| Agent 透明度与 Tool Calls | 6/10 | 20% | 1.20 |
| 模型与 Provider 管理 | 5/10 | 10% | 0.50 |
| 会话与状态管理 | 4/10 | 10% | 0.40 |
| 连接与可靠性 | 7/10 | 5% | 0.35 |
| 键盘与效率 | 5/10 | 5% | 0.25 |
| **综合得分** | | | **6.15/10** |

---

## 各维度详细发现

### 维度 1: 首次印象与上手

**得分: 7/10**

**优势:**

- **0 配置即连接** -- 默认 `ws://127.0.0.1:18789` 自动连接 Gateway，不需要粘贴任何 URL。对有 OpenClaw Gateway 运行的用户来说，打开即用
- **空状态设计用心** -- 居中的 orb icon + "Main Thread" + "Describe your objective. The agent will execute your command sequence." 传达了产品定位
- **输入框自动聚焦** -- 连接成功后 textarea 自动 focus（`ChatInput.tsx:16`），placeholder 也从 "Waiting for Gateway..." 变为操作提示
- **第一分钟即有 wow 时刻** -- 发送消息后流式回复几乎立即出现，tool call 卡片动画弹出，有"AI 在工作"的感觉

**问题:**

- **[P1-严重]** Gateway 连接失败时，UI 仅显示红色 "Error" 状态灯和文字。没有展示**具体错误原因**和**下一步操作建议**。对新用户来说 "Error" 等于无信息。需要像 `ErrorCard` 一样在聊天区域展示可操作的错误提示（"Gateway 未运行？请先启动 OpenClaw: `openclaw gateway start`"）
- **[P2-中等]** 三栏布局中，左侧栏显示 "LOCAL WORKSPACE" 和 "SESSION METRICS" -- 但没有**会话列表**可供切换。作为对标 Claude Desktop 的产品，第一眼看不到历史对话的入口极其反直觉。实际上 `SessionSidebar` 组件里只渲染了工作区卡片和指标，根本没有会话列表 UI
- **[P2-中等]** 顶部 Model Selector 显示 "GLM-4.7" 但**点击无反应**。模型切换器完全是装饰性的 -- 没有 dropdown 实现。这让用户以为功能在但坏了
- **[P3-轻微]** "Uptime: 2h 14m" 是硬编码值（`SessionSidebar/index.tsx:103`），不是真实数据，属于原型残留

---

### 维度 2: 视觉设计与审美

**得分: 8/10**

**优势:**

- **皮肤系统出色** -- 5 套主题全面覆盖了主流审美偏好：Raycast（热红）、Dracula（冷紫）、Neon Noir（赛博蓝粉）、Nord（北极冰蓝）、Monokai（经典绿）。每套都有完整的 CSS 变量映射（`themes.css`），不是简单换个 accent color
- **真黑背景** -- Raycast 和 Neon 皮肤的 `--bg: #000000`，OLED 友好。作为 OpenClaw 用户，我打开就感觉"这人懂"
- **Ambient lighting** -- `body::before` 的 radial-gradient glow 效果精妙（`global.css:84-91`），每套皮肤有定制的光晕位置和颜色
- **Glass morphism** -- 侧栏 `backdrop-filter: blur(40px) saturate(150%)`，设置面板 `blur(40px) saturate(180%)`，导航栏 `blur(24px)` -- 层次分明
- **Typography 一致** -- 正文使用 `-apple-system, SF Pro`，代码使用 `ui-monospace, SF Mono`，数字使用 `tabular-nums`（在 duration 显示中）
- **微动画有生命感** -- 消息 `slideUp` 动画（`cubic-bezier(0.2, 0.8, 0.2, 1)`），卡片 hover `translateY(-1px)` + 阴影变化，streaming cursor 闪烁
- **Mac 原生感** -- `titleBarStyle: 'hiddenInset'`，traffic lights 位置精确（`x: 16, y: 16`），topnav padding 80px 安全区
- **颜色是语义信号** -- `--success: #4ade80`（绿=连接成功/工具完成），`--danger: #ff6363`（红=错误/停止），`--warn: #f5a623`（黄=连接中）

**问题:**

- **[P2-中等]** themes.css 对 `--shadow-sm` / `--shadow-float` 等变量只在 raycast 主题中定义了完整的 inset highlights。Dracula、Nord、Monokai 的 `--shadow-sm` 未定义，导致这些皮肤下的卡片少了 premium inner lighting 效果
- **[P2-中等]** 设置面板中 Neon Noir 主题下选中的 tab 背景是 `var(--accent1)` 即蓝色 `#3b82f6`，搭配黑色文字 `color: #000` 的对比度在某些蓝色主题下偏低
- **[P3-轻微]** `PreviewPanel` 仍使用 `background: #fff` 硬编码白色背景（`PreviewPanel/index.tsx:8`），和暗色主题完全冲突。虽然面板未在布局中显示，但如果启用将是严重的视觉断裂
- **[P3-轻微]** `global.css:3-6` 引用了 `@tailwind base/components/utilities` 但 `package.json` 没有 tailwindcss 依赖，这些 at-rules 在生产环境会被忽略但在开发日志中可能产生警告
- **[P3-轻微]** 代码块语言标签颜色方案非常详细（20+ 语言专属颜色，`chat.css:998-1024`），这是超预期的品质细节

---

### 维度 3: 对话与流式体验

**得分: 6/10**

**优势:**

- **Smart Scroll** -- `useSmartScroll` hook 实现了"跟随新内容 + 用户上滚时暂停"的逻辑（`ChatPanel/index.tsx:11-39`），包括 ResizeObserver 监听子元素高度变化
- **mergeStreamText 去重** -- `chatStore.ts:6-24` 的文本合并逻辑很巧妙，处理了 Gateway 发送全量快照的场景（`incoming.startsWith(prev)` check）和增量块的最大重叠合并
- **Streaming Cursor** -- 闪烁的蓝色光标 + 旋转 asterisk 指示器，视觉上清晰地标识"AI 在工作"
- **Markdown 渲染完整** -- react-markdown + remark-gfm + rehype-highlight 的组合，标题、列表、表格、链接、行内代码、代码块全覆盖
- **extractText 递归提取** -- `CodeBlock.tsx:10-19` 安全地从 React children 中递归提取纯文本，解决了 streaming 时 rehype-highlight 生成 `<span>` 节点的问题

**问题:**

- **[P1-严重]** **Thinking blocks 不可展开** -- `MessageItem.tsx:47-54` 中 thinking block 有 "Thinking..." / "Thoughts" 标签但**注释里写着 "If expanded, show it" 就没了**。thinking 内容完全不可查看，对使用 Claude extended thinking 的用户来说这是核心功能缺失
- **[P1-严重]** **Token 计数永远为 0** -- 左侧栏 SESSION METRICS 的 Tokens 始终显示 0、Est. Cost 显示 $0.000。`ChatMessage.tokens` 在流式处理中从未被填充（`chatStore.ts` 的 handleAgentStream 中没有任何 token 数据的提取逻辑）。Gateway 的 `AgentRunResult` 有 tokens 字段但从未被处理
- **[P2-中等]** **Stop 按钮的位置不够明显** -- 在流式输出时，Stop 按钮替换 Send 按钮出现在输入框内，但因为输入框在页面底部且按钮尺寸仅 30x30px，在长对话中容易被忽视。Cursor 和 Claude Desktop 都使用更大的独立 Stop 按钮
- **[P2-中等]** **消息没有时间戳** -- 每条消息有 `timestamp` 字段但 UI 中没有渲染。对于长会话调试来说，不知道消息是什么时候发送的
- **[P2-中等]** **User avatar "我" 但系统语言可切换** -- 用户头像硬编码为中文 "我"，但设置中有 `language: 'zh-CN' | 'en-US'` 选项。切到英文界面后 "我" 就很突兀
- **[P3-轻微]** **Error 嵌入在 content 中** -- 错误通过 `\n\n**Error:** ${message}` 拼接到消息内容尾部（`chatStore.ts:321`），然后由 `MessageItem.tsx:14-19` 用正则提取。这种 in-band 错误编码脆弱且可能导致误判（如果 AI 自己输出了类似格式的文本）

---

### 维度 4: Agent 透明度与 Tool Calls

**得分: 6/10**

**优势:**

- **Tool Call 内联显示** -- tool calls 直接在聊天消息内展示为可折叠卡片，位于文本内容上方（`MessageItem.tsx:57-63`），不需要切换到单独面板
- **语义化图标和标签** -- `ToolCallBlock.tsx` 的 TOOL_META 映射为 20+ 个常用工具定义了专属图标和提取 key。`Read` 显示 FileText 图标 + 文件路径，`Bash` 显示 Terminal 图标 + 命令内容 -- 这比 Claude Desktop 的展示方式更好
- **实时 Activity Timeline** -- 右侧边栏的 Activity 面板实时更新 tool call 列表，running 状态有脉冲动画，completed 状态显示绿点和精确耗时
- **Artifacts 追踪** -- Artifacts tab 自动追踪所有 Write/Edit/apply_patch 操作产生的文件，支持点击用系统默认程序打开
- **执行耗时精确** -- `ToolCallBlock.tsx:35-39` 计算并显示每个 tool call 的毫秒/秒级耗时

**问题:**

- **[P1-严重]** **Tool call 展示缺少人类可读描述** -- 当前 tool header 显示 `exec` 或工具原始名称。比如 `>_ exec 70ms`。理想的展示应该是 `>_ 执行命令: ls -la | head -5 70ms`。虽然 `getSemanticLabel` 有提取逻辑，但对 `exec` 这个名称没有匹配（TOOL_META 中有 `Bash` 和 `run_command` 但没有 `exec`），导致语义标签为空
- **[P2-中等]** **Tool output 展示过于简陋** -- 展开 tool call 后，output 用 `<pre>` 标签渲染，没有语法高亮、行号或截断逻辑。对于返回大量内容的 tool（如 Read 整个文件），120px max-height 的滚动区域体验极差
- **[P2-中等]** **tool_use 和 tool_result 的 runId 关联** -- `chatStore.ts:296-317` 中 tool_result 的匹配策略是"找最后一个 running 状态的 tool call"。这在并发 tool calls 时可能导致错误匹配。应该通过 toolCallId 精确匹配
- **[P2-中等]** **右侧 Activity 只显示最新 turn** -- `ContextSidebar/index.tsx:166-178` 的 `recentToolCalls` 只取最后一个用户消息之后的 tool calls。历史 turn 的 activity 无法查看
- **[P3-轻微]** **tool-body 背景色硬编码** -- `global.css:248` `.tool-body { background: #0d0f14 }` 在 Nord/Monokai 等非纯黑主题下显得突兀，应使用 CSS 变量

---

### 维度 5: 模型与 Provider 管理

**得分: 5/10**

**优势:**

- **多 Provider 支持** -- 默认 catalog 包含 6 个模型：Claude Opus 4.6、Claude Sonnet 4.5、GPT-5.2、o3 Pro、GLM-5、DeepSeek R1（`types.ts:234-242`）
- **Add Model 流程完整** -- AddModelModal 支持 7 个 provider 选项（含 Ollama 本地和自定义），有 API Key 密码遮蔽、Base URL 配置
- **Push Config to Gateway** -- "Apply Config to Gateway" 按钮通过 `config.patch` RPC 将模型配置推送到 Gateway

**问题:**

- **[P0-致命]** **Model Selector 不工作** -- 顶部导航栏中央的模型选择器 `<div class="model-selector">` 只是一个静态 div，**没有 onClick 处理**（`MainLayout.tsx:37-39`）。用户无法在对话中切换模型，只能去 Settings > Models & API 修改默认模型。对于习惯了 ChatGPT/Claude 即时切换模型的用户来说，这是核心功能缺失
- **[P1-严重]** **API Key 存储声称用 macOS Keychain 但实际是 JSON 文件** -- Settings 中显示 "Your API key is stored securely in your macOS Keychain" 但实际存储在 `settings.json` 纯文本文件中（`store/settings.ts`）。这是误导性的安全声明
- **[P2-中等]** **切换模型需要 4 步** -- 打开设置 -> 切到 Models & API -> 在下拉框选择 -> Apply Config to Gateway。应该可以在聊天界面一键切换
- **[P2-中等]** **Gateway Token 显示为 Import from ~/.openclaw 按钮** -- 这很好，但按钮在 importGatewayAuth 后会自动 disconnect + reconnect（`SettingsView.tsx:161-162`），中间 400ms 的 setTimeout 不够稳定

---

### 维度 6: 会话与状态管理

**得分: 4/10**

**优势:**

- **Cmd+N 新建会话** -- 快捷键实际可用，创建带随机 ID 的新 session key
- **Session history 恢复** -- `chatStore.ts:354-418` 的 `loadSessionHistory` 从 Gateway 加载历史消息，包括 tool calls 的解析
- **Gateway 会话同步** -- `fetchSessions()` 从 Gateway 获取 session 列表

**问题:**

- **[P0-致命]** **没有会话列表 UI** -- `SessionSidebar` 组件根本没有渲染会话列表！只有工作区卡片和指标。用户无法切换或查看历史会话。`sessionStore.ts` 有完整的 `sessions` 和 `localSessions` 数据，但前端没有渲染。这是最严重的功能缺失
- **[P1-严重]** **新建会话后无法回到 Main Thread** -- 创建新会话后，只能通过刷新页面回到默认的 `agent:main:main` 会话
- **[P1-严重]** **deleteSession 和 renameLocalSession 无 UI 触发入口** -- store 中有完整的删除和重命名逻辑，但没有 UI 按钮调用它们
- **[P2-中等]** **历史消息中的 tool calls 状态永远是 running** -- `parseHistoryEntry()` 创建 tool calls 时 `status` 硬编码为 `'running'`（`chatStore.ts:64`），即使工具已完成

---

### 维度 7: 连接与可靠性

**得分: 7/10**

**优势:**

- **自动重连** -- `GatewayClient` 实现了指数退避重连（1.5x factor，最大 30s，最多 20 次尝试）
- **连接状态可视化** -- 绿色脉冲点 + "Gateway Connected" + "12ms" ping 展示
- **Auth 容错** -- device token mismatch 时自动清除 stale token 并重连（`ipc.ts:108-129`）
- **Challenge-Response 握手** -- 支持 Gateway 的 `connect.challenge` nonce 机制，ED25519 签名认证
- **断连清理** -- disconnect 时正确清理所有 pending requests（`client.ts:158-163`）

**问题:**

- **[P2-中等]** **ping "12ms" 是硬编码** -- `MainLayout.tsx:53` 的 ping 值 `12ms` 是写死的，不是真实测量值。误导用户
- **[P2-中等]** **authKind 错误没有在 UI 中展示** -- `handleDisconnect` 在 `device-token-mismatch` 或 `gateway-token-missing` 时只 console.error，用户在 UI 上只看到红色 "Error"，不知道是认证问题
- **[P3-轻微]** **connectTimer 的 750ms 延迟** -- `queueConnect()` 在 WebSocket open 后等 750ms 再发 connect frame（`client.ts:238`），这个延迟似乎是为了等 challenge event 但会影响首次连接速度

---

### 维度 8: 键盘与 Power User 效率

**得分: 5/10**

**优势:**

- **Cmd+N** -- 新建会话
- **Cmd+,** -- 打开/关闭设置
- **Esc** -- 关闭设置 / 停止流式输出
- **Enter** -- 发送消息，**Shift+Enter** -- 换行
- **focus-visible** -- 键盘导航时有 2px accent color outline

**问题:**

- **[P1-严重]** **没有 Cmd+K 命令面板** -- 这是 Raycast/Linear/Cursor 级别工具的标配。应该能快速搜索会话、切换模型、执行命令
- **[P2-中等]** **没有快捷键列表/帮助** -- 无法发现可用的快捷键。没有 tooltip、没有 help modal、没有 Cmd+?
- **[P2-中等]** **Tab 导航不完整** -- Tab 键在设置面板中的表单元素间可以移动，但主界面中的 sidebar items 和 tool call blocks 不在 tab order 中
- **[P3-轻微]** **Cmd+A 被拦截** -- `App.tsx:72-79` 在非 input 区域时阻止 Cmd+A 的默认行为。虽然是为了防止全选 UI 元素，但也阻止了用户在 code block 等可选中区域的全选

---

## 竞品差距分析

| 功能 | Cursor | Claude Desktop | CoworkDesk | 结论 |
|------|--------|---------------|------------|------|
| 模型切换 | 顶部一键切换 | 每次对话前选择 | 模型选择器不工作 | **严重落后** |
| 会话管理 | 左侧完整列表 + 搜索 | 左侧带分组的历史 | 没有会话列表 UI | **致命缺失** |
| Thinking blocks | 展开/折叠 | 完整展示 | 标签存在但内容不可见 | **功能空壳** |
| Tool calls | 内联 + activity log | 基础展示 | 内联 + 语义图标 + timeline | **领先 Claude** |
| 代码块 | 语法高亮 + 多操作 | 高亮 + Copy | 高亮 + Copy + 20+ 语言色标 | **优秀** |
| 快捷键 | Cmd+K + 完整体系 | 基础快捷键 | 3 个快捷键，无命令面板 | **明显不足** |
| 皮肤主题 | 仅暗色 | 明暗两色 | 5 套完整皮肤 | **明显领先** |
| Token/成本 | 实时显示 | 无 | UI 存在但数据为 0 | **虚假功能** |
| 终端面板 | 内嵌终端 | 无 | 空壳（硬编码提示文字） | **未实现** |
| 文件预览 | 内嵌预览 | 无 | 空壳（Ant Design Empty） | **未实现** |

---

## 优先级建议

### 必须修复 (P0-P1) -- 阻止用户继续使用

1. **P0: 实现会话列表 UI** -- 在左侧栏渲染 `sessions` 和 `localSessions`，支持点击切换、右键删除/重命名。这是聊天应用的基础功能
2. **P0: 让模型选择器工作** -- 顶部 model-selector 加 dropdown，展示 catalog 中的模型，点击切换当前对话使用的模型
3. **P1: 实现 thinking blocks 展开/折叠** -- 已有数据（`message.thinking`），只需加一个 `expanded` state 和渲染 thinking 内容的 UI
4. **P1: 填充 token 数据** -- 从 Gateway 的 `done` 事件中提取 token 使用量，更新 `ChatMessage.tokens`，让 Session Metrics 成为真实数据
5. **P1: 移除虚假 UI 信息** -- "12ms" ping、"2h 14m" uptime、"macOS Keychain" 声明 -- 要么实现要么删掉

### 应该改进 (P2) -- 影响 power user 体验

6. **P2: Gateway 连接失败的可操作错误提示** -- 在聊天区域用 ErrorCard 展示原因和下一步
7. **P2: Cmd+K 命令面板** -- 即使初版只支持搜索会话和切换模型
8. **P2: Tool call 人类可读描述** -- 扩展 TOOL_META 覆盖 `exec` 等工具名，显示命令内容摘要
9. **P2: 消息时间戳** -- 在消息头部或 hover 时显示发送时间
10. **P2: 统一 CSS 变量** -- `--shadow-sm`/`--shadow-float` 在所有皮肤中定义，`.tool-body` 背景用变量

### 锦上添花 (P3) -- 提升品质感

11. **P3: 清理原型残留** -- 移除硬编码值、未使用的 Tailwind imports、`PreviewPanel` 的白色背景
12. **P3: Textarea 自动调整高度** -- 当前的 `scrollHeight` 逻辑在输入/删除时可能闪烁，考虑用 CSS `field-sizing: content`
13. **P3: 快捷键提示** -- 在按钮 tooltip 中显示快捷键（如 Settings 按钮显示 "Cmd+,"）
14. **P3: 国际化** -- "我" 头像 / ErrorCard "错误" title 应该跟随语言设置

---

## 代码质量审查发现

### 架构优点

- **进程隔离清晰** -- main/preload/renderer/shared 的四层分离严格遵守
- **contextBridge 最小化** -- preload 只暴露了必要的 API，没有 raw Node 泄露
- **Zustand 使用规范** -- store 分离合理，selector 稳定（`EMPTY_MESSAGES` 常量避免了重新渲染）
- **GatewayClient 事件去重** -- `emittedToolIds` Map 有效防止了 tool call 重复显示

### 代码隐患

- **`any` 使用** -- `ToolCallBlock.tsx:6` 的 `{ icon: any }` 和 `CodeBlock.tsx:63` 的 `code({...}: any)` 违反了 strict 模式精神
- **Gateway client 内存泄漏风险** -- `runSessionMap` 和 `emittedToolIds` 在 `handleDisconnect` 中没有被清理（`client.ts:1134-1185`），长时间运行后可能累积大量废弃映射
- **`require('uuid')` in settings** -- `store/settings.ts:151` 使用了 `require('uuid')` 而不是顶部的 ES import，不一致且可能引起 tree-shaking 问题
- **setSettings 浅合并** -- `store/settings.ts:115-126` 只做一层浅合并，如果传入 `{ appearance: { skin: 'nord' } }` 会丢掉 `fontSize`、`theme`、`language` 等同层字段

---

## OpenClaw 重度用户最终裁决

作为一个每天用 OpenClaw + Cursor 12 小时的 power user，我会这样评价 CoworkDesk：

**它有品味，但还不能干活。**

视觉上，它是我见过的最好看的 AI 桌面应用原型之一 -- 5 套皮肤的质量超过了大多数生产级产品。tool call 的语义化展示和 Activity Timeline 比 Claude Desktop 做得更好。Gateway 协议的工程实现非常扎实。

但作为一个要替代 `openclaw --chat` 或 Cursor 的日常工具，**我不会切换过来**。原因很简单：

1. 我无法切换到之前的对话（没有会话列表）
2. 我无法在对话中切换模型（模型选择器是装饰）
3. 我看不到 thinking（空壳 UI）
4. 终端面板和预览面板都是占位符

如果团队能在下一个 sprint 解决 P0/P1 问题（特别是会话列表和模型切换），我愿意把它作为 OpenClaw Gateway 的首选桌面入口。视觉基础已经到位，需要的是把功能填满。

**一句话**: 它像一辆内饰豪华但缺少方向盘的概念车 -- 好看，但开不走。
