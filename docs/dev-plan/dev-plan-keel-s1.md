---
id: "dev-plan-keel-s1"
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
# Development Plan 分卷 — Sprint 1: 蓝图核心 + 门禁骨架

[NAV]
- §3 任务卡详细 → T-001..T-009 (Sprint 1)
[/NAV]

## 3. 任务卡详细

### T-001: 蓝图 Schema 与图 IR 核心

- **目标**: 实现 `blueprint.schema.json`（符合 JSON Schema 2020-12）与 `BlueprintIR` 内存图模型，支持七原语 + R-IR + 约束，提供 Ajv 校验入口
- **模块**: M-001
- **接口**: API-001
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `docs/blueprint.example.yaml` 读入内存，When 调用 `SchemaValidator.validate(ir)`，Then 返回 `{ valid: true, schemaErrors: [] }`；若缺少 `plain` 字段，Then 返回 `{ valid: false, schemaErrors: [{ path, message }] }` [ARCH#§2.M-001 / API-001]
  - [ ] AC-002: Given `BlueprintIR` 含 3 个节点（A: `surfaces_on_map=true`、B: `surfaces_on_map=true`、C: `surfaces_on_map=false`），When 访问 `ir.business` 投影，Then `ir.business.nodes.length === 2` 且节点 C 不在返回集合中；当访问 `ir.implementation` 时，`ir.implementation.nodes.length === 3`（含 C）[ARCH#§2.M-001]
  - [ ] AC-003: Given 蓝图 YAML 含 `requirements[]` 条目，When 解析后调用 `ir.requirements`，Then 每条含 `id`、`plain`、`acceptance[]`（含 `oracle` 标注字段）[ARCH#§2.M-001 / prd#§2.F-001 AC-003]
  - [ ] AC-004: Given `requirements[]` 中某条 req 未被任何 `Unit.satisfies` 引用，When 调用 `SchemaValidator.validate(ir)`，Then 覆盖率校验返回 error，错误含该 req.id [prd#§2.F-001 AC-004]
  - [ ] AC-005: Given `Constraint.severity` 设为 `hard`，When 序列化为 IR，Then `constraint.severity === 'hard'`；设为 `soft` 则 `=== 'soft'`；其他值，Then schema 校验拒绝（SchemaError）[prd#§2.F-001 AC-005]
  - [ ] AC-006: Given `Port` 携带可选 `contract`（含 `pre/post/invariant`，子句含 `plain`，可选 `formal`），When schema 校验，Then 格式合法时通过；子句 `plain` 为空则拒绝 [prd#§2.F-001 AC-006]
  - [ ] AC-007（错误路径）: Given `BlueprintIR` 为 `null` 或缺少必填 `meta.app_name`，When 调用 `SchemaValidator.validate(ir)`，Then 返回 `{ valid: false, schemaErrors: [{ path: '/meta/app_name', ... }] }`，不抛 uncaught exception [ARCH#§3.API-001]
- **deliverables**:
  - [ ] `packages/blueprint/src/schema/blueprint.schema.json` — JSON Schema 2020-12（七原语 + R-IR + Constraint）
  - [ ] `packages/blueprint/src/ir/BlueprintIR.ts` — 双层 IR 类型定义（business/implementation 投影）
  - [ ] `packages/blueprint/src/validator/SchemaValidator.ts` — Ajv 实例 + validate + 覆盖率校验
  - [ ] `packages/blueprint/tests/schema.test.ts` — schema 校验 + 七原语 + plain 必填 + requirements 覆盖率
- **context_load**:
  - arch-keel-modules#§2.M-001
  - arch-keel-api#§3.API-001
  - arch-keel-data#§4
  - prd-keel-f001-f009#§2.F-001
- **实现提示**: `docs/blueprint.example.yaml` 为权威实例，schema 以其为测试固件；Ajv 2020-12 模式需 `import Ajv from 'ajv/dist/2020'`；双层 IR 用 getter 懒计算 business 投影，不重复存储

---

### T-002: 蓝图加载 / 序列化 / 持久化

- **目标**: 实现 YAML↔IR 双向序列化（BlueprintSerializer）及 Git 可 diff 的持久化写入
- **模块**: M-001
- **接口**: API-001 (`load`, `persist`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `docs/blueprint.example.yaml` 文件路径，When 调用 `BlueprintSerializer.load(path)`，Then 返回 `{ ir: BlueprintIR, valid: boolean, schemaErrors: [] }`，且 `ir.meta.app_name` 等于 YAML 中 `meta.app_name` 值 [ARCH#§3.API-001 load]
  - [ ] AC-002: Given `BlueprintIR` 对象，When 调用 `BlueprintSerializer.persist({ ir, path })`，Then 在 `path` 写入合法 YAML 文件；重新 load 后 IR 与原对象语义等价 [ARCH#§3.API-001 persist]
  - [ ] AC-003: Given YAML 文件包含语法错误（缺闭合括号 / 非法缩进），When 调用 `load(path)`，Then 返回 `KeelError { code: 'BLUEPRINT_PARSE_ERROR', plainMessage: string（含非空错误描述） }`，不抛 uncaught exception；`typeof result.ir === 'undefined'` [ARCH#§3.API-001 error]
- **deliverables**:
  - [ ] `packages/blueprint/src/serializer/BlueprintSerializer.ts` — load/persist 实现
  - [ ] `packages/blueprint/tests/serializer.test.ts` — load/persist/error 路径
- **context_load**:
  - arch-keel-modules#§2.M-001
  - arch-keel-api#§3.API-001
- **实现提示**: 使用 `js-yaml` 库；persist 后文件行尾需保证 Git 可 diff（无随机排序 key）；错误统一包装为 `KeelError`

---

### T-003: 蓝图查询与 Blast Radius 基础

- **目标**: 实现 `GraphModel` 节点/边查询与 blast radius 依赖图算法（DependencyGraph），供 M-005 blast radius 计算复用
- **模块**: M-001
- **接口**: API-001 (`query`, `blastRadius`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `BlueprintIR` 含 3 个 Unit（A→B→C 依赖链），When 调用 `GraphModel.query({ selector: { kind: 'module' } })`，Then 返回 `{ nodes: [A,B,C], edges: [A→B, B→C] }` [ARCH#§3.API-001 query]
  - [ ] AC-002: Given Unit B 在依赖链 A→B→C 中，When 调用 `DependencyGraph.blastRadius({ changedNodeIds: ['B'] })`，Then 返回 `{ touched: ['B','C'], safe: ['A'], degraded: false }` [ARCH#§3.API-001 blastRadius]
  - [ ] AC-003: Given 依赖图有环（A→B→C→A），When 调用 `blastRadius({ changedNodeIds: ['A'] })`，Then 返回 `{ touched: ['A','B','C'], safe: [], degraded: true }`；不抛异常，`degraded: true` 表示需显式提示 [prd#§2.F-007 AC-003 / ARCH#§2.M-001]
- **deliverables**:
  - [ ] `packages/blueprint/src/graph/GraphModel.ts` — query 实现
  - [ ] `packages/blueprint/src/graph/DependencyGraph.ts` — blast radius + 环检测
  - [ ] `packages/blueprint/tests/graph.test.ts` — 查询 + blast radius + 有环降级
- **context_load**:
  - arch-keel-modules#§2.M-001
  - arch-keel-api#§3.API-001
  - prd-keel-f001-f009#§2.F-007

---

### T-004: 门禁引擎骨架与检查器 Adapter

- **目标**: 实现 `RuleCompiler`（rule.type → checker 配置）与五个 `CheckerAdapter`（dependency-cruiser / ts-morph / tsc / eslint / madge），构建可扩展的 adapter 注册机制
- **模块**: M-002
- **接口**: API-002
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `Constraint` 含 `rule.type: 'no_outbound_except'`，When 调用 `RuleCompiler.compile(constraint)`，Then 返回 dependency-cruiser adapter 的配置对象，含正确 `forbidden[].from`/`to` 规则 [ARCH#§2.M-002]
  - [ ] AC-002: Given `ChangerAdapter` 注册 tsc adapter，When 对含 TS 类型错误的文件集调用 `tsc.check(changeset)`，Then 返回 `GateError[]`，每条含 `{ ruleId, checker: 'tsc', severity: 'hard', locations: [{file,line}], message }` [ARCH#§3.API-002 types.GateError]
  - [ ] AC-003: Given 未注册的 rule.type（如 `unknown_rule`），When 调用 `RuleCompiler.compile(constraint)`，Then 返回 `KeelError { code: 'RULE_UNKNOWN', plainMessage: '...' }` [ARCH#§3.API-002 error]
- **deliverables**:
  - [ ] `packages/gate/src/compiler/RuleCompiler.ts` — rule.type → checker 配置
  - [ ] `packages/gate/src/adapters/DependencyCruiserAdapter.ts`
  - [ ] `packages/gate/src/adapters/TsMorphAdapter.ts`
  - [ ] `packages/gate/src/adapters/TscAdapter.ts`
  - [ ] `packages/gate/src/adapters/EslintAdapter.ts`
  - [ ] `packages/gate/src/adapters/MadgeAdapter.ts`
  - [ ] `packages/gate/src/adapters/CheckerAdapterRegistry.ts`
  - [ ] `packages/gate/tests/rule-compiler.test.ts`
  - [ ] `packages/gate/tests/adapters.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-002
  - arch-keel-api#§3.API-002
  - arch-keel#§1.4
- **实现提示**: 每个 adapter 实现统一接口 `{ check(changeset, config): GateError[] }`；dependency-cruiser 以 `runWithOptions` 编程式调用，不依赖 CLI；预留 adapter 注册扩展点为 Map，避免 switch 硬编码

---

### T-005: 门禁两阶段执行与结构化输出

- **目标**: 实现 `CheckerOrchestrator`（并行 + 增量调度）与 `GateResultAggregator`，完成 API-002 `GateReport` 输出；两阶段串行（structural + regressionReport 分区段）
- **模块**: M-002
- **接口**: API-002 (主操作)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given changeset 含违反 `dependency-cruiser` 规则的文件，When 调用 `GateEngine.run({ changeset, scope: 'incremental', blueprintIR })`，Then 返回 `GateReport { passed: false, structural: { hardViolations: [{ ruleId, checker: 'dependency-cruiser', severity: 'hard', ... }], softWarnings: [] }, regressionReport: [], durationMs }` 且 `durationMs ≤ 5000` [ARCH#§3.API-002 / prd#§2.F-003 AC-005]
  - [ ] AC-002: Given changeset 含 soft 软告警（`severity: 'soft'`）但无 hard 违规，When 调用 `GateEngine.run(...)`，Then 返回 `GateReport { passed: true, structural: { hardViolations: [], softWarnings: [...] }, regressionReport: [] }` [prd#§2.F-003 AC-003]
  - [ ] AC-003: Given changeset 含编译错误（tsc），When 调用 `GateEngine.run(...)`，Then 返回 `GateReport` 中 `structural.hardViolations` 含 `{ checker: 'tsc', severity: 'hard', ... }`；`GateReport` JSON 可机器解析（`JSON.parse(JSON.stringify(report))` 无异常）[prd#§2.F-003 AC-006]
  - [ ] AC-004（production-path）: `packages/gate/src/index.ts` 导出 `GateEngine` 并在 `packages/engine-adapter/src/toolsurface/ToolSurface.ts` 中调用 `gateEngine.run(...)` 处理 `keel.gate.check` 工具请求 [ARCH#§3.API-010 / ARCH#§7]
- **deliverables**:
  - [ ] `packages/gate/src/orchestrator/CheckerOrchestrator.ts` — 并行调度 + 增量范围
  - [ ] `packages/gate/src/aggregator/GateResultAggregator.ts` — structural + regressionReport 分段聚合
  - [ ] `packages/gate/src/GateEngine.ts` — 主入口，两阶段串行
  - [ ] `packages/gate/src/index.ts` — 模块导出
  - [ ] `packages/gate/tests/gate-engine.test.ts` — 三道门 + 增量 + 结构化输出
- **context_load**:
  - arch-keel-modules#§2.M-002
  - arch-keel-api#§3.API-002
  - prd-keel-f001-f009#§2.F-003
  - prd-keel-f001-f009#§2.F-004
- **实现提示**: 并行用 `Promise.allSettled` 跑独立 rule checker；增量 = 只传改动文件 + 一跳依赖；`durationMs` 用 `performance.now()` 计算；`GateReport` 类型严格对应 arch API-002 定义

---

### T-006: 回归守护（RegressionGuard）

- **目标**: 实现 `RegressionGuard`：F-003 结构门通过后，跨能力复跑累积检查集，检出「本次改动破坏既有能力」
- **模块**: M-002
- **接口**: API-002 (`regressionReport` 字段)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 已有能力 A（累积检查集 C_A）且本次改动通过 structural 门，When `RegressionGuard.run({ changeset, accumulatedChecks: { A: C_A }, blueprintIR })`，Then 对 C_A 重跑；若 changeset 破坏 A 的检查，返回 `RegressionBreak[] [{ brokenCapability: 'A', plain: '...', violations: GateError[] }]` [ARCH#§3.API-002 types.RegressionBreak]
  - [ ] AC-002: Given 改动未破坏任何既有能力，When `RegressionGuard.run(...)`，Then 返回空数组 `[]`；`GateReport.passed = true` 且 `regressionReport = []` [ARCH#§3.API-002]
  - [ ] AC-003: Given `RegressionGuard` 检测到破坏（`RegressionBreak[] = [{ brokenCapability: 'A', ... }]`），When `GateResultAggregator` 合并结果，Then `GateReport.passed` 值为 `false` 且 `GateReport.regressionReport.length >= 1`（即使 `structural.hardViolations` 为空，`passed` 仍返回 `false`）[ARCH#§3.API-002 notes / prd#§2.F-004 AC-001]
- **deliverables**:
  - [ ] `packages/gate/src/regression/RegressionGuard.ts` — 累积检查集维护 + 跨能力复跑
  - [ ] `packages/gate/src/regression/CapabilityCheckStore.ts` — 按能力存储历史规则集
  - [ ] `packages/gate/tests/regression-guard.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-002
  - arch-keel-api#§3.API-002
  - prd-keel-f001-f009#§2.F-004

---

### T-007: Escape Hatch 豁免逻辑

- **目标**: 实现 `EscapeHatchResolver`：标注 `escape_hatch: true` 的 Module 跳过依赖边界检查与架构 lint，保留编译门与 `hard` 安全约束
- **模块**: M-002
- **接口**: API-002 (内部豁免决策)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given Module M 标注 `escape_hatch: true`，When `EscapeHatchResolver.shouldSkip({ moduleId: 'M', ruleType: 'dependency_boundary' })`，Then 返回 `true`（跳过）；When `ruleType: 'tsc_compile'`，Then 返回 `false`（不跳过） [prd#§2.F-017 AC-002 / ARCH#§2.M-002]
  - [ ] AC-002: Given `Constraint.severity: 'hard'` 且该 Constraint 属安全类规则，When `EscapeHatchResolver.shouldSkip({ severity: 'hard', isSecurityRule: true })`，Then 返回 `false`（不豁免）[prd#§2.F-017 AC-002 / ARCH#§2.M-002]
- **deliverables**:
  - [ ] `packages/gate/src/escape-hatch/EscapeHatchResolver.ts`
  - [ ] `packages/gate/tests/escape-hatch.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-002
  - prd-keel-f010-f018#§2.F-017

---

### T-008: 元数据 SQLite Schema + E-001~E-006

- **目标**: 定义 Keel 本体元数据的 SQLite 表结构（E-001~E-006），提供迁移脚本与 Drizzle schema 类型
- **模块**: —（数据层，供 M-011/M-005/M-014 使用）
- **接口**: E-001~E-006
- **task_kind**: chore
- **tdd_mode**: standard
- **tdd_refactor**: skip
- **security_sensitive**: true
- **tdd_acceptance**:
  - [ ] AC-001: Given 空 SQLite 数据库，When 运行迁移脚本，Then `SELECT name FROM sqlite_master WHERE type='table'` 返回含 `projects` / `intent_log_entries` / `decision_records` / `frozen_contract_sets` / `health_snapshots` / `engine_connections` 六张表；`SELECT name FROM sqlite_master WHERE type='index'` 返回各表 index 条目非空 [ARCH#§4.E-001~E-006]
  - [ ] AC-002: Given `EngineConnection.authRef` 字段，When 插入值为 `'sk-abc123'` 的记录，Then 数据库抛出约束错误（`SQLITE_CONSTRAINT`）或应用层校验返回 `KeelError { code: 'INVALID_AUTH_REF' }`，记录不落表；When 插入 `'keychain://keel/engine'` 时正常插入且 `SELECT authRef` 返回该值 [ARCH#§4.E-006 / arch-keel#§5.2 security]
- **deliverables**:
  - [ ] `packages/blueprint/src/db/schema.ts` — Drizzle schema（六表）
  - [ ] `packages/blueprint/src/db/migrations/0001_init.sql` — SQLite DDL
  - [ ] `packages/blueprint/src/db/client.ts` — better-sqlite3 单例 + 迁移运行
  - [ ] `packages/blueprint/tests/db-schema.test.ts` — 迁移 + authRef 约束
- **context_load**:
  - arch-keel-data#§4
  - arch-keel#§5.2
- **实现提示**: 使用 `better-sqlite3`（本地同步 API，适合 Node 本地引擎服务）；Drizzle ORM for SQLite；`authRef` 设计为不透明字符串，实际值由调用方填系统密钥库引用

---

### T-009: [VALIDATION] 蓝图核心与门禁骨架

- **目标**: 用户（或测试负责人）手动验证 Sprint 1 核心产出：蓝图加载 + schema 校验 + 三道门可执行 + escape hatch 豁免
- **task_kind**: validation
- **模块**: M-001, M-002
- **tdd_acceptance**: N/A（validation 任务不走 TDD，由 orchestrator 展示验证清单后用户手动确认）
- **deliverables**: N/A（validation 任务不产出代码文件，由前置任务 T-001~T-008 的 deliverables 覆盖）
- **验证清单**:
  - [ ] `docs/blueprint.example.yaml` 通过 `SchemaValidator.validate()`，返回 `valid: true`
  - [ ] 在 `blueprint.example.yaml` 中删除某 Module 的 `plain` 字段后重新校验，返回 `valid: false` 且 `schemaErrors` 包含该字段路径
  - [ ] 向 `blueprint.example.yaml` 加入一条 `requirements[]` 条目但不添加 `satisfies` 引用，校验报覆盖率错误
  - [ ] 运行 `GateEngine.run({ changeset: ['packages/blueprint/src/ir/BlueprintIR.ts'], scope: 'incremental', blueprintIR })`，返回 `GateReport` 含 `durationMs ≤ 5000`
  - [ ] 在测试文件中将某 Module 标注 `escape_hatch: true` 并制造依赖边界违规，验证 `EscapeHatchResolver` 使检查跳过
  - [ ] 运行迁移后确认六张 SQLite 表存在，`authRef` 不接受 `sk-` 前缀字符串
- **前置任务**: [T-001, T-002, T-003, T-004, T-005, T-006, T-007, T-008]
- **context_load**:
  - arch-keel-modules#§2.M-001
  - arch-keel-modules#§2.M-002
  - prd-keel-f001-f009#§2.F-001
  - prd-keel-f001-f009#§2.F-003
