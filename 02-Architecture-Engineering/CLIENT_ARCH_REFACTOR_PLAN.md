# AutoClaw 客户端工程化改造方案（架构师视角）

## 1. 背景与目标

### 1.1 背景

当前项目已具备可用功能，但从最近登录链路排障过程看，存在明显的工程化短板：定位成本高、链路可观测性弱、模块耦合高、错误语义不统一。  
这类问题若不治理，会持续演化为“功能可跑但系统不可维护”的典型堆叠型技术债。

### 1.2 目标

本方案目标不是“重写”，而是分阶段将系统升级到可长期演进的工程状态：

1. 任何线上/预发问题都能在 5-15 分钟内完成首轮定位。
2. 新功能迭代可在既有边界内扩展，不再持续向大文件堆逻辑。
3. 错误具备统一语义（可读、可追踪、可分层处理）。
4. 回归风险可控（关键链路有自动化测试与发布门禁）。

### 1.3 非目标

1. 不在一轮改造中替换全部技术栈。
2. 不做“大爆炸式重构”。
3. 不以牺牲交付节奏为代价追求“完美架构”。

## 2. 现状诊断（基线）

### 2.1 能力内聚问题

1. 能力按“技术层”聚集多于按“业务能力”聚集，跨文件跳转成本高。
2. 同一能力在 renderer/main/shared 间缺少统一边界定义，容易出现隐式契约。
3. 认证、网关、会话等核心能力尚未形成稳定的“能力包（capability package）”。

### 2.2 架构分层问题

当前存在超大文件，已经明显超出可维护阈值：

1. `src/main/ipc.ts`：1631 行。
2. `src/main/gateway/client.ts`：1399 行。
3. `src/renderer/stores/chatStore.ts`：1249 行。

影响：

1. 变更冲突频繁，代码评审困难。
2. 局部改动容易引发跨域回归。
3. 边界责任模糊，出现“临时逻辑顺手加到大文件”。

### 2.3 可观测性问题

1. 日志出口分散：`process.stdout/stderr`、`console.*`、局部文件日志并存。
2. 日志结构不统一，机器可检索字段（`trace`、`stage`、`code`）并非全链路强制。
3. 关键链路缺少统一事件模型（如登录链路阶段事件）。

### 2.4 错误处理问题

1. 错误表示不统一：字符串、异常、返回对象混合使用。
2. 缺少标准错误分层：`renderer/main/backend/network/validation`。
3. 缺少错误分类策略：是否可重试、是否用户可见、是否需要告警。

### 2.5 测试与回归问题

1. 当前自动化测试覆盖薄弱：仅见 `src/main/auth/endpoints.test.ts`。
2. 关键流程（登录、刷新、网关重连）缺少稳定回归测试。
3. 发布前缺少强约束门禁（类型检查、单测、关键烟测）。

## 3. 目标架构原则

### 3.1 能力内聚优先（Feature-first）

以能力为组织单位，而不是继续按“技术类型”扩张。  
每个能力应包含：契约、状态、用例、适配器、视图。

### 3.2 分层清晰（Layered + Ports/Adapters）

推荐层次：

1. `UI Layer`：展示与用户交互。
2. `Application Layer`：用例编排（不关心具体 IO）。
3. `Domain Layer`：业务规则与模型。
4. `Infrastructure Layer`：IPC、网络、存储、网关适配。

### 3.3 可观测性内建（Observability by design）

1. 任何跨层调用必须携带 `trace_id`。
2. 关键事件必须结构化日志化。
3. 错误必须具备可机器处理元数据。

### 3.4 错误即协议（Error as contract）

所有业务/基础设施错误统一映射为标准错误对象，不再裸抛字符串。

## 4. 核心改造设计

### 4.1 能力内聚改造

建议引入能力目录（示意）：

```text
src/
  capabilities/
    auth/
      contract.ts
      domain/
      app/
      infra/
      ui/
    gateway/
    session/
```

关键动作：

1. 将登录相关 renderer/main/shared 逻辑收敛为 `auth` capability。
2. 为每个 capability 定义公开边界（exports），禁止跨能力直接读内部实现。
3. 建立“能力负责人”机制，避免无人维护。

### 4.2 架构分层改造

优先拆分超大文件：

1. `src/main/ipc.ts` 拆为：
   - `src/main/ipc/auth.handlers.ts`
   - `src/main/ipc/gateway.handlers.ts`
   - `src/main/ipc/settings.handlers.ts`
   - `src/main/ipc/index.ts`（仅注册路由）
2. `src/main/gateway/client.ts` 拆为：
   - 连接生命周期
   - 协议编解码
   - 事件分发
   - 重连策略
3. `src/renderer/stores/chatStore.ts` 拆为：
   - session state
   - message stream state
   - action/usecase dispatcher

约束：

1. 单文件软上限建议 500 行，硬上限 800 行。
2. 任何新逻辑禁止继续写入超 1000 行历史文件。

### 4.3 日志与可观测性改造

定义统一日志模型（建议字段）：

```json
{
  "ts": "ISO8601",
  "level": "DEBUG|INFO|WARN|ERROR",
  "service": "renderer|main|gateway",
  "capability": "auth|session|gateway",
  "event": "auth.login.start",
  "trace_id": "xxx",
  "stage": "renderer|main|backend",
  "code": "optional",
  "message": "optional",
  "payload": {}
}
```

落地动作：

1. 封装统一 logger（禁止业务直接 `console.*` / `process.stdout.*`）。
2. 接口/IPC入口强制注入 trace（已有基础，需全能力推广）。
3. 增加敏感字段脱敏器（手机号、token、key）。
4. 建立日志级别策略和采样策略。

### 4.4 整体错误处理改造

定义统一错误类型（建议）：

```ts
type AppError = {
  code: string
  message: string
  stage: 'renderer' | 'main' | 'backend' | 'network'
  traceId: string
  retriable: boolean
  userVisible: boolean
  cause?: unknown
}
```

治理规则：

1. 只在边界层做错误翻译（infra -> app）。
2. UI 层只消费标准错误，不解析底层异常字符串。
3. 用户文案与诊断信息分离：  
   `user_message`（可读） + `debug_message`（日志可查）。

### 4.5 工程治理与防腐化机制

1. 引入架构规则检查（ESLint import boundary）。
2. 增加 ADR（Architecture Decision Record）目录，重大改造先记决策。
3. 变更必须附“影响能力/回滚方式/观测点”。
4. 发布门禁固定化：`typecheck + lint + test + build`。

## 5. 分阶段实施路线图

### 5.1 Phase 0（1 周）：止血与可观测性

1. 全链路 trace_id 强制透传（renderer -> preload -> ipc -> api）。
2. 登录、刷新、网关重连统一 stage 标注。
3. 建立统一日志包装器并替换关键链路直写日志。

验收：

1. 关键链路失败都能给出 `trace + stage + code`。
2. 任一登录失败 10 分钟内可定位到层级。

### 5.2 Phase 1（1-2 周）：结构化拆分

1. 拆分 `ipc.ts`。
2. 拆分 `gateway/client.ts`。
3. 拆分 `chatStore.ts`。

验收：

1. 超 1000 行核心文件清零。
2. 新增功能不需要改动“总控大文件”。

### 5.3 Phase 2（1-2 周）：测试体系建立

1. capability 级单测（auth/gateway/session）。
2. 登录链路集成测试（成功、验证码错误、网络异常、IPC异常）。
3. CI 门禁启用 `npm run test`。

验收：

1. 关键链路测试覆盖率达可接受基线（建议 >60% 起步）。
2. 每次改登录链路均可自动回归。

### 5.4 Phase 3（2 周）：长期治理

1. 架构约束自动化（边界/目录规则）。
2. ADR 与技术债看板制度化。
3. 发布与回滚操作手册标准化。

## 6. 优先级改造清单（建议）

### 6.1 P0（立即）

1. 统一错误对象与 stage 语义。
2. 统一日志模型与 trace 透传。
3. 登录/刷新/发码链路补足关键事件埋点。

### 6.2 P1（近期）

1. 拆分三大超大文件。
2. 能力目录化（从 auth/gateway 先行）。
3. 建立最小回归测试集。

### 6.3 P2（中期）

1. 架构规则自动化检查。
2. ADR 流程与 Owner 机制。
3. 可观测性指标（失败率、重试率、超时率）仪表化。

## 7. 验收指标（建议）

1. 平均故障定位时间（MTTD）下降 50% 以上。
2. 登录链路回归引入缺陷率显著下降。
3. 核心文件平均变更冲突次数下降。
4. 每次发布前自动化门禁通过率稳定提升。

## 8. 结论

当前项目最大风险不是“功能不够”，而是“系统复杂度增长速度超过治理速度”。  
按上述路线推进，可以在不牺牲交付速度的前提下，把项目从“靠经验排障”升级为“靠系统可观测和架构边界稳定演进”。
