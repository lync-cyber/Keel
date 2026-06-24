---
id: "dev-plan-keel-s6"
version: "1.0.0"
doc_type: dev-plan
author: tech-lead
status: approved
deps: ["arch-keel", "ui-spec-keel"]
consumers: [developer, qa-engineer]
volume: sprint
volume_type: sprint
split_from: "dev-plan-keel"
required_sections:
  - "## 3. 任务卡详细"
---
# Development Plan 分卷 — Sprint 6: 集成打通 + 性能调优 + 错误处理完善

[NAV]
- §3 任务卡详细 → T-052..T-057 (Sprint 6)
[/NAV]

## 3. 任务卡详细

### T-052: 全链路接线：意图→diff→冻结→执行→健康

- **目标**: 完成 Keel 核心链路的全量进程内接线（container 注册 / event handler / lifecycle hook），确保 M-001~M-015 所有模块在本地引擎服务进程中正确连接，从意图到地图健康叠加端到端可运行
- **模块**: 全链路（M-001~M-015）
- **接口**: 全链路
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001（production-path）: `apps/workspace/src/server/engineServer.ts` 启动本地引擎服务，注册 `gateEngine` / `driftDetector` / `intentOrchestrator` / `reconcileEngine` / `autoRepairLayer` / `intentLogStore` / `deployDriver` / `repoExporter`，所有实例通过 DI 容器（或工厂函数）注入 [ARCH#§6]
  - [ ] AC-002（production-path）: `apps/workspace/src/server/websocket.ts` 注册 WebSocket handler，将后端 `progress:updated` / `drift:updated` / `repair:progress` 事件转发到前端 [ARCH#§2.M-015]
  - [ ] AC-003: Given 用户在对话面输入意图文本，When 从意图到 diff 到契约冻结到执行到地图健康刷新走一遍完整流程，Then 地图节点从 `planning` → `implementing` → `ready`，预览面自动刷新，时间线新增条目 [prd#§2.F-009/F-010/F-013]
  - [ ] AC-004: Given 切换执行引擎（ACP → SDK），When 固定蓝图 + 固定加能力任务，Then 蓝图产出、门禁判定结果、地图最终状态三者一致（F-018 AC-004 透明切换）[prd#§2.F-018 AC-004]
  - [ ] AC-005: Given 通道外直接修改 `features/auth/index.ts` 制造 boundary_break，When `ChangeWatcher` 触发检测，Then 地图 `auth` 节点变 red + 健康调和抽屉显示漂移条目 [prd#§2.F-002 AC-006 全链路]
- **deliverables**:
  - [ ] `apps/workspace/src/server/engineServer.ts` — 本地引擎服务启动与模块注册
  - [ ] `apps/workspace/src/server/websocket.ts` — WebSocket event bus
  - [ ] `apps/workspace/src/server/container.ts` — 依赖注入容器/工厂
  - [ ] `apps/workspace/src/server/routes/intentRoutes.ts` — 意图相关 HTTP 路由
  - [ ] `apps/workspace/src/server/routes/gateRoutes.ts` — 门禁查询路由
  - [ ] `apps/workspace/tests/integration/full-chain.test.ts` — 全链路集成测试
- **context_load**:
  - arch-keel#§6
  - arch-keel#§7
  - arch-keel-modules#§2.M-015
  - prd-keel-f010-f018#§2.F-018

---

### T-053: 用量徽标与引擎侧消耗透明（UC-020，M-014）

- **目标**: 在工作区顶栏常驻 `UsageMeter`（UC-020），接入 API-016 `usageStatus` 展示引擎侧订阅/用量；措辞明确与 Keel 定价无关
- **模块**: M-014
- **接口**: API-016 (`usageStatus`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given API-016 `usageStatus()` 返回 `{ planLabel: 'Pro', used: 500, quota: 2000, source: 'subscription' }`，When 顶栏 `UsageMeter` 渲染，Then 显示「执行引擎侧 Pro · 500/2000」文案；点击展开明细，措辞包含「与 Keel 定价无关」[ARCH#§3.API-016 usageStatus / ui-spec-keel-components#UC-020 / prd#§2.F-012 AC-007]
  - [ ] AC-002: Given `source: 'unknown'`（引擎侧未提供用量），When `UsageMeter` 渲染，Then `getByTestId('usage-meter')` DOM 不含进度环元素（`queryByTestId('usage-ring')` 为 null）；`getByTestId('usage-meter').textContent` 含「引擎未提供用量信息」[ui-spec-keel-components#UC-020]
- **deliverables**:
  - [ ] `apps/workspace/src/layouts/TopBar.tsx` 中集成 `UsageMeter`（接入 API-016 `usageStatus`）
  - [ ] `apps/workspace/src/components/atoms/UsageMeter.test.tsx`
- **context_load**:
  - ui-spec-keel-components#§2.UC-020
  - arch-keel-api#§3.API-016
  - prd-keel-f010-f018#§2.F-012

---

### T-054: 增量检测性能优化（M-002/M-003 ≤3s / ≤5s）

- **目标**: 针对 M-002 门禁（≤5s）与 M-003 漂移检测（≤3s）做增量优化：仅扫受影响切片 + 一跳依赖；检查器并行编排；结果聚合
- **模块**: M-002, M-003
- **接口**: API-002 / API-003
- **task_kind**: fix
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given changeset 含 1 个文件（`features/auth/AuthService.ts`），When `GateEngine.run({ changeset, scope: 'incremental', blueprintIR })`，Then `GateReport.durationMs ≤ 5000`（performance.now 测量，在 CI 中验证）[prd#§2.F-003 AC-005 / arch-keel#§5.1]
  - [ ] AC-002: Given changeset 含 1 个文件，When `DriftDetector.detect({ scope: 'incremental' })`，Then `DriftReport.durationMs ≤ 3000` [prd#§2.F-002 AC-005 / arch-keel#§5.1]
  - [ ] AC-003: Given 独立 rule 的检查器（如 dependency-cruiser 与 madge 无互相依赖），When `CheckerOrchestrator` 调度，Then 两者 `Promise.allSettled` 并行执行（通过 spy 验证并发启动时机，两个 check 调用之间间隔 <100ms）[arch-keel#§5.1]
- **deliverables**:
  - [ ] `packages/gate/src/orchestrator/CheckerOrchestrator.ts`（增量范围计算 + 并行执行优化）
  - [ ] `packages/drift/src/DriftDetector.ts`（增量范围 + durationMs 精确计量）
  - [ ] `packages/gate/tests/performance/gate-perf.test.ts`
  - [ ] `packages/drift/tests/performance/drift-perf.test.ts`
- **context_load**:
  - arch-keel#§5.1
  - arch-keel-modules#§2.M-002
  - arch-keel-modules#§2.M-003
  - prd-keel#§3.1
  - prd-keel-f001-f009#§2.F-003
  - prd-keel-f001-f009#§2.F-002

---

### T-055: 工作区首屏性能优化（≤2s，React Flow 懒加载）

- **目标**: 验证并确保三面工作区首屏 ≤2s；React Flow `React.lazy` 懒加载不阻塞对话面；若未达标则实施 bundle 拆分或 D3 替换决策
- **模块**: M-015, M-009
- **接口**: —（性能）
- **task_kind**: fix
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 三面工作区 `/project/:id` 路由，When 浏览器首次加载（无缓存），Then `performance.now()` 从导航开始到「骨架屏渲染完成」（三面框架可见）≤2s；通过 Playwright 或 Lighthouse CI 验证 [prd#§2.F-008 AC-005 / arch-keel#§5.1]
  - [ ] AC-002: Given 对话面首次可交互时间（TTI），When 测量（三面同时挂载），Then 对话面 input 可用时间不超过地图面 `React.lazy` chunk 加载完成时间（验证懒加载隔离效果：对话面 TTI 不应依赖地图 chunk）[ui-spec-keel#§5 首屏性能约束]
- **deliverables**:
  - [ ] `apps/workspace/src/map/CapabilityMap.tsx` — 确认 `React.lazy` 包裹生效
  - [ ] `apps/workspace/vite.config.ts` — code split 配置（确保地图 chunk 独立）
  - [ ] `apps/workspace/tests/performance/workspace-perf.test.ts`（Playwright）
- **context_load**:
  - arch-keel#§5.1
  - ui-spec-keel#§5
  - ui-spec-keel-pages-core#§3.P-001
  - ui-spec-keel-pages-core#§3.P-002
  - prd-keel-f001-f009#§2.F-008

---

### T-056: 错误降级 + 通俗化全面审查

- **目标**: 全面检查并修复所有用户可见界面中可能暴露技术错误（堆栈/编译输出/lint 日志/git 命令/代码 diff）的路径；确保 F-003 AC-004 / F-008 AC-003 / F-005 AC-003 等通俗化约束的完整落地
- **模块**: 全链路
- **接口**: —（横切面质量审查）
- **task_kind**: fix
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `GateError.message === "Type 'string' is not assignable to type 'number'"` 经 M-007 上浮到 UI，When 渲染 `DecisionCard` 或 `ChatBubble`，Then `getByTestId('decision-explain').textContent` 不含 `'is not assignable'` 子字符串；`textContent.length > 5`（通俗中文描述非空）[prd#§2.F-003 AC-004 / F-005 AC-003]
  - [ ] AC-002: Given 预览代理服务崩溃（HTTP 502），When `PreviewPanel` 收到失败响应，Then `getByTestId('preview-unavailable')` 返回非 null DOM，文本含「稍后重试」；DOM 不含 `502`、`stack trace`、`Error:` 等技术字符串 [prd#§2.F-008 AC-003]
  - [ ] AC-003: Given 安全体检发现 `secret_leak`，When `SecurityChecklistCard` 渲染该检查项，Then `getByTestId('check-secret_leak').textContent` 不含 `/` 路径分隔符（不含源码文件路径）；文本含「硬编码」或「密钥」等通俗词汇 [prd#§2.F-014 / arch-keel#§5.3]
- **deliverables**:
  - [ ] 错误降级修复散布于相关组件（按 AC 点修改对应文件）
  - [ ] `apps/workspace/tests/error-ux/error-ux.test.ts` — E2E 错误场景通俗化验证
- **context_load**:
  - arch-keel#§5.3
  - prd-keel-f001-f009#§2.F-003
  - prd-keel-f001-f009#§2.F-005
  - prd-keel-f001-f009#§2.F-008
  - prd-keel-f010-f018#§2.F-014

---

### T-057: [VALIDATION] 完整产品端到端验收

- **目标**: 用户（Alex）对 Keel v1 完整产品进行端到端验收，覆盖 F-001~F-018 全功能及所有 NFR 指标
- **task_kind**: validation
- **模块**: M-001~M-015
- **tdd_acceptance**: N/A（validation 任务不走 TDD，由 orchestrator 展示验证清单后用户手动确认）
- **deliverables**: N/A（validation 任务不产出代码文件，由前置任务 T-052~T-056 的 deliverables 覆盖）
- **验证清单**:
  **核心流程 F-001~F-009**:
  - [ ] `blueprint.example.yaml` schema 校验通过；七原语全量；plain 字段必填校验有效
  - [ ] 通道外直接修改 `features/` 目录下文件，30s 内（或下次打开）DriftReport 检测到漂移（F-002 AC-006）
  - [ ] 制造 hard 违规（dependency-cruiser 边界违反），门禁阻断代码落地；soft 告警放行并记录健康（F-003）
  - [ ] 添加新能力后回归守护检测：手动破坏既有能力对应文件，门禁阻断 + 地图被破坏节点变 red（F-004）
  - [ ] 门禁触发 ≥2 次相同错误，地图显示「正在修复…」（不见堆栈）；第 3 次后升级「需你决策」卡（F-005）
  - [ ] 20 节点地图渲染 ≤1s；七原语全部展示业务译名；地图只读（无编辑入口）（F-006）
  - [ ] 意图输入后 blast radius 展示「会碰」/「不会碰」通俗说明；依赖图有环时显示「影响范围可能不完整」（F-007）
  - [ ] 三面工作区首屏 ≤2s；门禁通过后预览面自动刷新（F-008）
  - [ ] 意图文本 → 10s 内 diff 卡出现（mock LLM）；确认 → 契约冻结；时间线新增条目（F-009）

  **执行与调和 F-010~F-011**:
  - [ ] 契约冻结后进入实现；地图节点按依赖顺序进入 implementing/ready；全程无代码/终端（F-010）
  - [ ] 健康调和：漂移条目「一键修复」→ 切片重生成 → 漂移消失 → 节点转绿（F-011）

  **生命周期 F-012~F-018**:
  - [ ] 新项目上手：全程无终端命令（F-012 AC-006）
  - [ ] 时光机：回滚到 2 步之前，蓝图/代码/健康灯一致恢复；有通道外改动时先提示（F-013）
  - [ ] 一键上线：认证 fail-open → 红灯阻断；修复后全绿 → 部署 → Toast 显示地址（F-014）
  - [ ] 导出：`npx dependency-cruiser` 在无 Keel 环境下检出违规（F-015）；交接包含三文档（F-016）
  - [ ] escape hatch Module：虚线描边 + brass 标识 + intent 说明；调和前提示保护（F-017）
  - [ ] 切换执行器（ACP → SDK）：蓝图/门禁/地图结果一致（F-018 AC-004）

  **NFR 指标全过关**:
  - [ ] 门禁单次 ≤5s、漂移增量 ≤3s、地图渲染 ≤1s、首屏 ≤2s、diff ≤10s、时间线查询 ≤3s
  - [ ] 所有用户界面零原始技术错误暴露
- **前置任务**: [T-052, T-053, T-054, T-055, T-056]
- **context_load**:
  - prd-keel#§1
  - prd-keel#§3
  - prd-keel-f001-f009#§2
  - prd-keel-f010-f018#§2
