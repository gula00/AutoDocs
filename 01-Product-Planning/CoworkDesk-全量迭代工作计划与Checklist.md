# CoworkDesk 全量迭代工作计划与 Checklist（A-J）

> 目标：把三份种子用户反馈中的全部优化项统一纳入执行盘，不漏项、不跳项。
> 范围：稳定性、安全、可观测、会话工作流、模型管理、透明度、效率、UI/可访问性、技术债。

---

## 一、执行策略（先救命，再增强）

- Phase 0（D1-D3）：P0 全清 + 黑箱等待止血
- Phase 1（W1）：核心可用（会话/模型/thinking/token）
- Phase 2（W2）：效率与透明度（Cmd+K/快捷键/工具语义化）
- Phase 3（W3-W4）：差异化能力（Terminal/MCP/附件/搜索）+ 技术债收口

---

## A. 稳定性与可靠性

### A1. 立即修复
- [ ] 修复 main 进程 `EPIPE` 崩溃弹窗（替换危险 `console.log` 输出路径）
- [ ] main 入口增加 `stdout/stderr` 错误防护和 uncaught 错误兜底
- [ ] 建立请求状态机：`Queued -> Planning -> Executing -> Streaming -> Done/Failed`
- [ ] 将 `Sending request...` 静态文案替换为阶段性可视反馈

### A2. 用户可控
- [ ] Stop 在各阶段均可中断，且 500ms 内反馈“已停止”
- [ ] Failed 阶段给可执行建议（重试、检查网关、检查鉴权）
- [ ] 重试保留上次输入与上下文，不强制用户重打

### A3. 连接与认证
- [ ] 在 UI 区分并展示 `gateway-token-missing` / `device-token-mismatch`
- [ ] 导入 `~/.openclaw` 后重连改为稳态流程（去除脆弱固定延时）
- [ ] 连续 50 次请求稳定性回归（无卡死、无崩溃弹窗）

**验收标准**
- [ ] “仅显示 Sending request 无后续状态”比例 < 1%
- [ ] 崩溃率 < 0.5%

---

## B. 安全与数据主权

### B1. 凭证安全
- [ ] API Key 真正落 Keychain/safeStorage（优先）
- [ ] 若未完成加密，所有文案改为“本地文件存储”并明确风险

### B2. 用户可验证主权
- [ ] 新增 `Data & Privacy` 面板
- [ ] 显示本地存储路径、网络外发边界、日志范围
- [ ] 提供遥测开关（默认值与说明清晰）
- [ ] 提供一键清理凭证、备份、恢复入口

### B3. 导出安全
- [ ] 导出前显示敏感字段提示
- [ ] 导出文件格式说明（JSON/MD）和用途说明

**验收标准**
- [ ] 用户 3 步内可完成备份导出
- [ ] 设置页可直接看到“存哪里、发哪里、怎么关”

---

## C. Token/成本可观测

### C1. 数据链路打通
- [ ] 每条消息展示 `prompt/completion/total tokens`
- [ ] 会话总 tokens 与估算 cost 实时累计
- [ ] usage 缺失时显示 `usage unavailable`，禁止回退为 0

### C2. 成本可信
- [ ] provider 单价来源可追溯（配置版本/时间）
- [ ] 显示估算依据（模型、输入输出量、价格表）

### C3. 一致性
- [ ] 会话总量与消息求和误差 < 2%
- [ ] 90%+ 消息具备可见 token 数据

**验收标准**
- [ ] Session Metrics 与消息级 usage 一致
- [ ] 成本统计在多 provider 场景稳定

---

## D. 会话与工作流

### D1. Session Rail（最小可用）
- [ ] 左侧会话列表渲染（sessions + localSessions）
- [ ] 点击切换会话，支持回到 Main Thread
- [ ] 会话搜索（按标题、最近内容）

### D2. 管理能力
- [ ] 新建、重命名、删除（带确认）
- [ ] 归档/置顶
- [ ] 20+ 会话下首屏检索 < 200ms

### D3. 资产化
- [ ] 导出 Markdown/JSON
- [ ] 导入最小可用
- [ ] 会话分支（branch）

### D4. 历史准确性
- [ ] 历史 tool call 状态按真实结果恢复，不再全部 running

**验收标准**
- [ ] 会话可检索、可切换、可管理、可导出

---

## E. 对话体验与 Agent 透明度

### E1. Thinking 与可追溯
- [ ] thinking blocks 展开/折叠
- [ ] 工具调用展示语义标签（覆盖 `exec`、`run_command` 等）

### E2. 正确关联与并发
- [ ] `tool_use/tool_result` 通过 id 精确配对
- [ ] 并发 tool calls 不串台

### E3. 进度与时间
- [ ] 长操作显示中间进度和最近活动
- [ ] 消息时间戳显示
- [ ] Stop 按钮增强可见性（位置、尺寸、状态）

### E4. 错误模型优化
- [ ] 错误从文本拼接改为结构化字段
- [ ] ErrorCard 统一错误信息和建议动作

### E5. 操作增强
- [ ] 消息编辑/重新生成
- [ ] Activity 可查看历史 turn，不仅最近一轮

**验收标准**
- [ ] 用户能清楚看到“正在做什么、做了多久、为何失败”

---

## F. 模型与 Provider 管理

### F1. 顶部切换可用化
- [ ] 顶部模型选择器可点击
- [ ] 会话内即时切换并反馈（toast + 徽标）

### F2. 低摩擦配置
- [ ] 减少“进设置四步切模型”路径
- [ ] 常用 provider 预设完善（OpenAI/DeepSeek/Qwen/Gemini/Ollama 等）

### F3. 配置可靠性
- [ ] Add Model 流程避免双层 modal 阻塞
- [ ] Push Config 显示成功/失败/回滚

**验收标准**
- [ ] 用户在主界面 1-2 步切模型成功

---

## G. 关键功能补全（空壳转可用）

### G1. 面板能力
- [ ] Terminal 面板接入真实输出流
- [ ] Preview 面板主题兼容且可预览常见文件

### G2. 输入增强
- [ ] 文件附件/图片上传
- [ ] Web 搜索按钮真实可用
- [ ] System Prompt 自定义

### G3. 生态扩展
- [ ] MCP 最小可用（连接、启停、状态、错误反馈）
- [ ] Skills 管理入口（可发现、可开关）

**验收标准**
- [ ] 关键入口无“有图标无功能”

---

## H. 键盘效率与可发现性

### H1. 高频快捷键
- [ ] `Cmd+K` 命令面板（会话搜索/模型切换/常用动作）
- [ ] `Cmd+/` 快捷键帮助面板

### H2. 可访问性
- [ ] 所有 icon button 增加 tooltip + aria-label
- [ ] tooltip 中展示对应快捷键
- [ ] 主界面 Tab 顺序可完整导航

### H3. 行为修正
- [ ] 修正 `Cmd+A` 拦截，避免影响可选中内容区

**验收标准**
- [ ] 核心操作 100% 可键盘完成

---

## I. UI / 视觉 / 响应式 / 国际化

### I1. 响应式与布局
- [ ] 修复右侧 Activity/Artifacts 压缩
- [ ] 修复设置模态内容截断

### I2. 主题一致性
- [ ] 所有主题补齐阴影变量（`--shadow-sm` / `--shadow-float`）
- [ ] 修复 Neon Noir 选中态对比度
- [ ] `.tool-body` 背景去硬编码改变量
- [ ] PreviewPanel 去白底硬编码

### I3. 细节品质
- [ ] 输入区工具按钮 hover 可感知
- [ ] Environment 区域信息价值重构
- [ ] 空状态文案国际化（zh-CN/en-US）
- [ ] 用户头像文案跟随语言

**验收标准**
- [ ] 标准窗口和窄窗口都可用、可读、可点

---

## J. 工程质量与技术债

### J1. 类型与规范
- [ ] 清理关键路径 `any`，补齐类型
- [ ] 统一 CommonJS `require` 到 ES import（如 uuid）

### J2. 状态与内存
- [ ] 断连时清理 `runSessionMap` / `emittedToolIds` 等映射
- [ ] 修复 `setSettings` 浅合并导致字段丢失问题（改深合并）

### J3. 原型残留清理
- [ ] 清理硬编码 ping、uptime 等伪数据
- [ ] 清理未使用 Tailwind at-rules 或补齐依赖策略
- [ ] 建立回归清单：连接/流式/会话/模型/导出/快捷键

**验收标准**
- [ ] `npm run typecheck` 和 `npm run build` 通过
- [ ] 无高优先级回归

---

## 里程碑与发布门禁

### M1（D3）- 可继续使用
- [ ] A+B+C 的 P0 项全部完成
- [ ] 不再出现 EPIPE 弹窗
- [ ] Token/Cost 不再恒 0

### M2（W1）- 能干活
- [ ] D+F+E 核心项完成
- [ ] 会话/模型/thinking/错误反馈可用

### M3（W2）- 高效率
- [ ] H + E 透明度增强完成
- [ ] 命令面板和快捷键体系上线

### M4（W4）- 可托付
- [ ] G + I + J 收口
- [ ] 全链路回归通过，进入稳定发布

---

## 每周固定检查（执行纪律）

- [ ] 周一：确认本周目标与验收指标
- [ ] 周三：中期 demo（必须跑真实场景）
- [ ] 周五：回归 + 指标复盘 + 下周风险清单
- [ ] 每日：新增 issue 必须标注“是否阻断 M1/M2”

---

## 建议看板分组（可直接建 Epic）

- [ ] Epic A：Stability & Reliability
- [ ] Epic B：Security & Privacy
- [ ] Epic C：Usage & Cost Observability
- [ ] Epic D：Session Workflow
- [ ] Epic E：Agent Transparency
- [ ] Epic F：Model Switching
- [ ] Epic G：Feature Completion
- [ ] Epic H：Keyboard Productivity
- [ ] Epic I：UI/Responsive/i18n
- [ ] Epic J：Tech Debt & Quality

---

## 最后检查（上线前必须全绿）

- [ ] 没有“看起来有但点不动”的控件
- [ ] 没有“看起来安全但实际不安全”的文案
- [ ] 没有“统计面板存在但永远 0”的伪可观测
- [ ] 没有“长等待无解释”的黑箱阶段
- [ ] 没有“无法找回历史会话”的生产力断点
