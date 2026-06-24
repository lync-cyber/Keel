---
id: "dev-plan-keel-s3"
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
# Development Plan 分卷 — Sprint 3: 实现执行 + 自愈 + 调和

[NAV]
- §3 任务卡详细 → T-019..T-026 (Sprint 3)
[/NAV]

## 3. 任务卡详细

### T-019: DAG 调度器（DagScheduler）

- **目标**: 实现 `DagScheduler`（M-006）：按蓝图依赖关系拓扑序调度切片实现，被依赖切片先就绪
- **模块**: M-006
- **接口**: API-007 (内部调度)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 蓝图 IR 含依赖链 A→B→C（B 依赖 A，C 依赖 B），When `DagScheduler.schedule(frozenContractSetId)`，Then 返回执行顺序 `[['A'], ['B'], ['C']]`（拓扑层级，同层可并行） [ARCH#§2.M-006 / prd#§2.F-010 AC-002]
  - [ ] AC-002: Given 依赖图有环（A→B→A），When `DagScheduler.schedule(...)`，Then 返回 `KeelError { code: 'DEPENDENCY_CYCLE', plainMessage: '能力之间存在循环依赖，需要人工处理...' }` [ARCH#§2.M-006]
  - [ ] AC-003: Given 拓扑序执行中 B 完成，When `DagScheduler.markDone('B')`，Then `DagScheduler.getStatus('C')` 返回 `'ready_to_execute'`（之前调用返回 `'waiting'`）[prd#§2.F-010 AC-002]
- **deliverables**:
  - [ ] `packages/executor/src/scheduler/DagScheduler.ts` — Kahn 拓扑排序 + 状态管理
  - [ ] `packages/executor/tests/dag-scheduler.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-006
  - arch-keel-api#§3.API-007
  - prd-keel-f010-f018#§2.F-010

---

### T-020: 精准上下文注入（ContextInjector）

- **目标**: 实现 `ContextInjector`（M-006）：每个切片实现仅注入自身契约 + 直接依赖签名（`ContextBundle`），不注入全图
- **模块**: M-006
- **接口**: API-009 `ContextBundle`
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 切片 `favorites`（直接依赖 `article` 和 `user`），When `ContextInjector.build({ capabilityId: 'favorites', frozenContractSetId })`，Then 返回 `ContextBundle { capabilityId: 'favorites', contract: FrozenContractSet(favorites), directDepSignatures: [PortSignature(article), PortSignature(user)] }`；不含 `article` 的间接依赖 [ARCH#§3.API-009 types.ContextBundle / prd#§2.F-010 AC-002]
  - [ ] AC-002: Given 切片无依赖（孤立 Module），When `ContextInjector.build({ capabilityId: 'standalone' })`，Then 返回 `ContextBundle { directDepSignatures: [] }` [ARCH#§3.API-009]
- **deliverables**:
  - [ ] `packages/executor/src/context/ContextInjector.ts`
  - [ ] `packages/executor/tests/context-injector.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-006
  - arch-keel-api#§3.API-009
  - prd-keel-f010-f018#§2.F-010

---

### T-021: Worktree 隔离与回滚（WorktreeIsolator）

- **目标**: 实现 `WorktreeIsolator`（M-006）：单切片实现在隔离 worktree 内「实现—校验—合并」，失败或中止时回滚到改动前完整可用状态
- **模块**: M-006
- **接口**: API-007 (隔离机制)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 开始实现切片 `favorites`，When `WorktreeIsolator.begin('favorites')` 创建隔离 worktree，Then `git worktree list` 输出新增一行临时路径记录；主工作区 `git status` 输出 `nothing to commit, working tree clean`（状态不变）[ARCH#§2.M-006 / prd#§2.F-010 AC-004]
  - [ ] AC-002: Given 切片实现在隔离 worktree 中通过门禁，When `WorktreeIsolator.commit('favorites')`，Then 主工作区 `git status` 显示新文件（`features/favorites/` 目录存在）；`git worktree list` 不再包含该临时路径（临时 worktree 已删除）[ARCH#§2.M-006]
  - [ ] AC-003: Given 切片实现在隔离 worktree 中失败（门禁拦截），When `WorktreeIsolator.rollback('favorites')`，Then `git worktree list` 不再含该临时路径；主工作区 `git status` 输出 `nothing to commit, working tree clean`；`fs.existsSync('features/favorites/')` 为 `false` [prd#§2.F-010 AC-004]
  - [ ] AC-004: Given 并发实现切片 A 与 B，When 两个 `WorktreeIsolator.begin(...)` 调用，Then `git worktree list` 输出两个不同路径；当 `rollback('B')` 执行后，`worktree list` 仍包含 A 的路径（A 独立路径不被删除），`features/A/` 目录存在而 `features/B/` 不存在 [prd#§2.F-010 AC-004]
- **deliverables**:
  - [ ] `packages/executor/src/isolator/WorktreeIsolator.ts` — git worktree 管理
  - [ ] `packages/executor/tests/worktree-isolator.test.ts`（需 git 测试 fixture）
- **context_load**:
  - arch-keel-modules#§2.M-006
  - arch-keel-api#§3.API-007
  - prd-keel-f010-f018#§2.F-010
- **实现提示**: 使用 `simple-git` 库的 worktree 命令；测试需临时 git 仓库 fixture（`tmp` + `git init`）

---

### T-022: 实现进度状态机（ProgressStateMachine）

- **目标**: 实现 `ProgressStateMachine`（M-006）：维护每个切片节点的状态（planning → implementing → ready），并向前端广播状态变化事件
- **模块**: M-006
- **接口**: API-007 (进度投影)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 切片 `favorites` 进入调度队列，When `ProgressStateMachine.transitionTo('favorites', 'implementing')`，Then 状态变为 `implementing`；广播 `{ type: 'progress:updated', capabilityId: 'favorites', status: 'implementing' }` WebSocket 事件 [ARCH#§2.M-006 / prd#§2.F-010 AC-005]
  - [ ] AC-002: Given 切片 `favorites` 完成（门禁通过），When `transitionTo('favorites', 'ready')`，Then 状态变为 `ready`；广播 `status: 'ready'` 事件 [prd#§2.F-010 AC-002/AC-005]
- **deliverables**:
  - [ ] `packages/executor/src/progress/ProgressStateMachine.ts`
  - [ ] `packages/executor/tests/progress-state-machine.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-006
  - prd-keel-f010-f018#§2.F-010

---

### T-023: 自愈层错误分类与重试（M-007 核心）

- **目标**: 实现 `ErrorClassifier` 与 `RetryController`（M-007）：消费 `GateError[]`，分类为「纯技术性」（静默自愈）/ 「业务决策类」（上浮），同一错误 ≤3 次重试
- **模块**: M-007
- **接口**: API-008
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `GateError` 含 `rule: 'import_path_wrong'`（类型不符，唯一修复路径），When `ErrorClassifier.classify(gateError, frozenContractSetId)`（读 E-004 与修复后签名对比），Then 返回 `{ category: 'technical', canSilentlyFix: true }`；`SelfHealDriver` 驱动引擎静默修复 [ARCH#§2.M-007 / prd#§2.F-005 AC-002]
  - [ ] AC-002: Given `GateError` 修复会改变已冻结 Port 签名（E-004 `ports` 与修复后签名净变化非空），When `ErrorClassifier.classify(...)`，Then 返回 `{ category: 'business_decision', canSilentlyFix: false }` → 上浮 DecisionCard [ARCH#§2.M-007 / prd#§2.F-005 AC-002]
  - [ ] AC-003: Given 同一 `{ ruleId, locations }` 错误连续出现，When `RetryController.shouldRetry(errorKey)`，Then 第 1/2/3 次返回 `true`；第 4 次返回 `false`（达上限） [ARCH#§2.M-007 / prd#§2.F-005 AC-004]
  - [ ] AC-004（production-path）: `packages/repair/src/AutoRepairLayer.ts` 在 `packages/executor/src/SliceImplDriver.ts` 内通过 `autoRepairLayer.repair(gateErrors, frozenContractSetId)` 调用 [ARCH#§2.M-006 / ARCH#§2.M-007]
- **deliverables**:
  - [ ] `packages/repair/src/classifier/ErrorClassifier.ts`
  - [ ] `packages/repair/src/retry/RetryController.ts` — 错误 key 映射 + 计数
  - [ ] `packages/repair/src/SelfHealDriver.ts` — 调用引擎驱动静默修复
  - [ ] `packages/repair/src/AutoRepairLayer.ts` — 主入口（API-008）
  - [ ] `packages/repair/src/index.ts`
  - [ ] `packages/repair/tests/error-classifier.test.ts`
  - [ ] `packages/repair/tests/retry-controller.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-007
  - arch-keel-api#§3.API-008
  - arch-keel-data#§4.E-004
  - prd-keel-f001-f009#§2.F-005

---

### T-024: 自愈上浮与「需你决策」卡片

- **目标**: 实现 `EscalationEmitter` 与 `RepairProgressReporter`（M-007）：达重试上限或业务决策类错误时产出 `DecisionCard`；自愈期间报告可见进度
- **模块**: M-007
- **接口**: API-008 (`escalation: DecisionCard`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `RetryController` 返回 `shouldRetry: false`（达上限），When `EscalationEmitter.escalate(gateError, frozenContractSetId)`，Then 返回 `DecisionCard { conflict: '...（通俗）', plainImpact: '...', options: [{ id: 'accept_deviation', plain: '接受偏差并更新蓝图' }, { id: 'reject', plain: '拒绝并重做' }] }` [ARCH#§3.API-008 types.DecisionCard / prd#§2.F-010 AC-006]
  - [ ] AC-002: Given 自愈进行中，When `RepairProgressReporter.report({ state: 'working' })`，Then 广播 `{ type: 'repair:progress', state: 'working', plainMessage: '正在修复…' }` WebSocket 事件；不含堆栈/编译日志 [prd#§2.F-005 AC-003/AC-005]
- **deliverables**:
  - [ ] `packages/repair/src/escalation/EscalationEmitter.ts`
  - [ ] `packages/repair/src/reporter/RepairProgressReporter.ts`
  - [ ] `packages/repair/tests/escalation-emitter.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-007
  - arch-keel-api#§3.API-008
  - prd-keel-f001-f009#§2.F-005
  - prd-keel-f010-f018#§2.F-010

---

### T-025: 调和引擎（切片重生成 + 内容保护）

- **目标**: 实现 `ReconcileEngine`（M-004）：以受影响切片为单位整片重生成，受门禁约束，含 `ContentProtectionGuard`（escape hatch / 未入图内容检测）
- **模块**: M-004
- **接口**: API-004
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 切片 `favorites` 漂移，When `ReconcileEngine.reconcile({ capabilityId: 'favorites', frozenContractSetId })`，Then 驱动引擎重生成 `features/favorites/`，门禁通过后返回 `{ regenerated: true, gateReport: { passed: true }, regressionPassed: true }` [ARCH#§3.API-004 / prd#§2.F-011 AC-001/AC-003]
  - [ ] AC-002: Given 切片 `payment` 含 `escape_hatch: true`，When `ReconcileEngine.reconcile({ capabilityId: 'payment', ... })`，Then `ContentProtectionGuard` 检测到 escape hatch，返回 `{ regenerated: false, protectedContentPrompt: '这块有你的自由实现区，重做可能丢失，确定继续吗？' }` [prd#§2.F-011 AC-007 / ARCH#§3.API-004]
  - [ ] AC-003: Given 调和完成（`reconcile()` 返回 `{ regenerated: true }`），When 重新运行 `DriftDetector.detect(...)` 检测原漂移项，Then `DriftReport.items.filter(i => i.type === '调和前漂移类型').length === 0`（该漂移条目不再出现）[prd#§2.F-011 AC-004]
  - [ ] AC-004（production-path）: `packages/reconcile/src/ReconcileEngine.ts` 调用 `engineAbstraction.prompt(...)` 驱动重生成，并调用 `gateEngine.run(...)` 做调和后门禁验证 [ARCH#§2.M-004]
- **deliverables**:
  - [ ] `packages/reconcile/src/ReconcileEngine.ts`
  - [ ] `packages/reconcile/src/boundary/SliceBoundaryResolver.ts`
  - [ ] `packages/reconcile/src/protection/ContentProtectionGuard.ts`
  - [ ] `packages/reconcile/src/anchor/ContractAnchor.ts` — 读 FrozenContractSet
  - [ ] `packages/reconcile/src/index.ts`
  - [ ] `packages/reconcile/tests/reconcile-engine.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-004
  - arch-keel-api#§3.API-004
  - prd-keel-f010-f018#§2.F-011
  - prd-keel-f010-f018#§2.F-017

---

### T-026: [VALIDATION] 实现执行 + 自愈 + 调和端到端流程

- **目标**: 用户（或测试负责人）手动验证 Sprint 3 核心产出
- **task_kind**: validation
- **模块**: M-004, M-006, M-007
- **tdd_acceptance**: N/A（validation 任务不走 TDD，由 orchestrator 展示验证清单后用户手动确认）
- **deliverables**: N/A（validation 任务不产出代码文件，由前置任务 T-019~T-025 的 deliverables 覆盖）
- **验证清单**:
  - [ ] 调用 `DagScheduler.schedule()` 在依赖链 A→B→C 场景下返回 `[['A'], ['B'], ['C']]`（拓扑层级正确）
  - [ ] `WorktreeIsolator.begin()` 创建临时 worktree 后，主工作区 git status clean；`rollback()` 后临时 worktree 消失、主工作区仍 clean
  - [ ] 同一 `GateError`（含相同 ruleId + location）连续触发 4 次，`RetryController` 第 4 次返回 `shouldRetry: false`
  - [ ] `EscalationEmitter.escalate()` 产出 `DecisionCard`，`conflict` 与 `plainImpact` 为中文通俗描述，无堆栈/编译日志
  - [ ] `RepairProgressReporter` 广播事件 `plainMessage: '正在修复…'`（不含技术错误）
  - [ ] 切片含 `escape_hatch: true` 时 `ReconcileEngine.reconcile()` 返回非空 `protectedContentPrompt`
  - [ ] 切片不含 escape hatch 的正常调和：`DriftReport` 调和后清除对应漂移条目，`regressionPassed: true`
- **前置任务**: [T-019, T-020, T-021, T-022, T-023, T-024, T-025]
- **context_load**:
  - arch-keel-modules#§2.M-004
  - arch-keel-modules#§2.M-006
  - arch-keel-modules#§2.M-007
  - prd-keel-f010-f018#§2.F-010
  - prd-keel-f010-f018#§2.F-011
  - prd-keel-f001-f009#§2.F-005
