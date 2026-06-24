---
id: "dev-plan-keel-s2"
version: "1.0.0"
doc_type: dev-plan
author: tech-lead
status: draft
deps: ["arch-keel", "ui-spec-keel"]
consumers: [developer, qa-engineer]
volume: sprint
volume_type: sprint
split_from: "dev-plan-keel"
required_sections:
  - "## 3. 任务卡详细"
---
# Development Plan 分卷 — Sprint 2: 漂移检测 + 意图捕获 + 执行引擎接缝

[NAV]
- §3 任务卡详细 → T-010..T-018 (Sprint 2)
[/NAV]

## 3. 任务卡详细

### T-010: AST 逆向提取（AstExtractor）

- **目标**: 用 ts-morph 从代码 AST 逆向提取实际结构图（模块边界 / 导出接口 / 依赖关系），产出 `ActualStructureGraph`
- **模块**: M-003
- **接口**: API-003 (内部，供 T-011 对账使用)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given TypeScript 项目含 `features/auth/index.ts` 导出 `AuthService` 并 import `features/user/UserRepo`，When `AstExtractor.extract(changeset)` 扫描这两个文件，Then 返回 `ActualStructureGraph { modules: ['auth', 'user'], edges: [{from:'auth', to:'user', kind:'import'}] }` [ARCH#§2.M-003]
  - [ ] AC-002: Given 同一功能在 `features/auth/AuthService.ts` 与 `features/profile/AuthHelper.ts` 均实现（代码相似度超阈值 [ASSUMPTION] 待 dev 阶段校准），When `AstExtractor.extract(fullScope)`，Then 返回图中标记两处为 `duplicate_candidate: true` [prd#§2.F-002 AC-002]
  - [ ] AC-003: Given 文件不存在或 ts 解析报错，When `AstExtractor.extract(['non-existent.ts'])`，Then 返回 `KeelError { code: 'AST_PARSE_ERROR', plainMessage: '...' }`，不崩溃 [ARCH#§3.API-003 error]
- **deliverables**:
  - [ ] `packages/drift/src/extractor/AstExtractor.ts` — ts-morph 逆向提取实现
  - [ ] `packages/drift/src/types/ActualStructureGraph.ts` — 实际结构图类型
  - [ ] `packages/drift/tests/ast-extractor.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-003
  - arch-keel-api#§3.API-003
  - arch-keel#§1.4
  - prd-keel-f001-f009#§2.F-002

---

### T-011: 蓝图对账与漂移分类

- **目标**: 实现 `BlueprintComparer` 与 `DriftClassifier`，将期望态（蓝图 IR）vs 实际态（ActualStructureGraph）做 diff，产出分类后的 `DriftReport`（四类漂移 + plain 说明）
- **模块**: M-003
- **接口**: API-003
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 蓝图声明 `auth → user`（允许）而 AST 图含 `payment → user`（未声明），When `DriftDetector.detect({ scope: 'incremental', source: 'channel' })`，Then 返回 `DriftReport { items: [{ type: 'boundary_break', plain: '支付模块越界访问了用户模块', locations: [...], resolution: 'reconcile' }], durationMs ≤ 3000 }` [ARCH#§3.API-003 / prd#§2.F-002 AC-002/AC-003]
  - [ ] AC-002: Given 四类漂移同时存在（`boundary_break` / `duplicate_impl` / `orphan_code` / `contract_shape_mismatch`），When `DriftClassifier.classify(comparerDiff)`，Then 返回 `DriftItem[]` 长度为 4，且 `items.map(i => i.type)` 包含全部四个枚举值；每条 `item.plain` 字段非空且不含 `import`、`module`、`class` 等代码术语 [prd#§2.F-002 AC-003]
  - [ ] AC-003: Given 相同的 changeset 和 blueprintIR，When 分别以 `source: 'external'` 和 `source: 'channel'` 调用 `DriftDetector.detect(...)`，Then 两次返回的 `DriftReport.items` 数组长度相同、各条目 `type` 字段集合相同（`source` 字段不影响检测结果，两次 `durationMs ≤ 3000`）[prd#§2.F-002 AC-006 / ARCH#§3.API-003]
- **deliverables**:
  - [ ] `packages/drift/src/comparer/BlueprintComparer.ts` — 期望态 vs 实际态 diff
  - [ ] `packages/drift/src/classifier/DriftClassifier.ts` — 四类归类 + plain 翻译
  - [ ] `packages/drift/src/DriftDetector.ts` — API-003 主入口（含 durationMs 计量）
  - [ ] `packages/drift/src/index.ts`
  - [ ] `packages/drift/tests/drift-detector.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-003
  - arch-keel-api#§3.API-003
  - prd-keel-f001-f009#§2.F-002

---

### T-012: ChangeWatcher 通道外改动发现

- **目标**: 实现 `ChangeWatcher`，监听 `features/` + 蓝图文件变化（「项目打开扫描」路径 [ASSUMPTION] 初版），检测通道外改动并经 WebSocket 推送到 Web 壳
- **模块**: M-003
- **接口**: API-003 (内部触发 DriftDetector)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given Keel 工作区启动时，When 在 `features/` 目录直接修改一个 TS 文件（通道外），Then `ChangeWatcher.onExternalChange` 触发，`DriftDetector.detect({ source: 'external' })` 被调用，结果经 WebSocket 事件 `{ type: 'drift:updated', report: DriftReport }` 推送到 Web 壳 [prd#§2.F-002 AC-006 / ARCH#§2.M-003]
  - [ ] AC-002: Given ChangeWatcher 正在监听，When 监听路径下 5 秒内无任何文件变化，Then `onExternalChange` 回调调用次数为 0，WebSocket 不广播任何 `drift:updated` 事件（通过 spy 计数验证调用次数 === 0）[ARCH#§2.M-003]
- **deliverables**:
  - [ ] `packages/drift/src/watcher/ChangeWatcher.ts` — 文件监听 + 外部改动识别
  - [ ] `packages/drift/src/watcher/DriftEventBroadcaster.ts` — WebSocket 推送（接入本地引擎服务 WS）
  - [ ] `packages/drift/tests/change-watcher.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-003
  - prd-keel-f001-f009#§2.F-002
- **实现提示**: [ASSUMPTION] 初版用「项目打开扫描」（chokidar watch 在引擎服务启动时触发全量扫描，改动后增量）；实时监听为后续优化；监听范围限 `features/` + 蓝图文件，避免全项目监听开销

---

### T-013: 执行引擎抽象层（ACP / SDK / CLI Adapter）

- **目标**: 实现 `EngineAbstraction`（M-008）：AcpAdapter / AgentSdkAdapter / HeadlessCliAdapter 三种 adapter 实现统一接口 `EngineAdapter`，含 `PermissionMediator` 与 `EngineCapabilityProbe`
- **模块**: M-008
- **接口**: API-009
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: true
- **tdd_acceptance**:
  - [ ] AC-001: Given `AcpAdapter` 配置（JSON-RPC over stdio），When 调用 `adapter.applyEdit({ edit, engineKind: 'acp' })`，Then 向引擎发送正确 JSON-RPC 请求并返回 `{ applied: boolean, gateReport: GateReport }`；`engineKind: 'hosted'` 时返回空实现响应（v1 预留，不报错）[ARCH#§3.API-009 / prd#§2.F-018 AC-002/AC-005]
  - [ ] AC-002: Given 相同 `edit` 参数分别传给 `AcpAdapter`（`engineKind: 'acp'`）和 `AgentSdkAdapter`（`engineKind: 'sdk'`），When 两者均调用 `adapter.applyEdit(...)`，Then 两次返回的 `gateReport.passed` 值相同、`gateReport.structural.hardViolations.length` 相同（门禁判定结果一致）；`applied` 字段均为 `boolean` [prd#§2.F-018 AC-004]
  - [ ] AC-003: Given `PermissionMediator.mediatePermission({ action: { type: 'write_file', path: 'features/auth/index.ts' } })`，When 该文件变更不触发门禁违规，Then 返回 `{ decision: 'allow', reason: undefined }`；若违规，Then 返回 `{ decision: 'deny', reason: 'gate violation: ...' }` [ARCH#§3.API-009 mediatePermission / prd#§2.F-018 AC-002]
  - [ ] AC-004（production-path）: `packages/engine-adapter/src/index.ts` 导出 `EngineAbstraction`，`packages/intent/src/IntentTranslator.ts` 通过 `engineAbstraction.prompt(...)` 调用 LLM [ARCH#§2.M-008]
  - [ ] AC-005（错误路径）: Given adapter 连接引擎超时（>30s），When `adapter.applyEdit(...)`，Then 返回 `KeelError { code: 'ENGINE_TIMEOUT', plainMessage: '执行引擎无响应...' }` [ARCH#§3.API-009 error]
- **deliverables**:
  - [ ] `packages/engine-adapter/src/adapters/AcpAdapter.ts`
  - [ ] `packages/engine-adapter/src/adapters/AgentSdkAdapter.ts`
  - [ ] `packages/engine-adapter/src/adapters/HeadlessCliAdapter.ts`
  - [ ] `packages/engine-adapter/src/EngineAbstraction.ts` — 统一接口 + adapter 选择
  - [ ] `packages/engine-adapter/src/PermissionMediator.ts`
  - [ ] `packages/engine-adapter/src/EngineCapabilityProbe.ts` — 检测/冒烟
  - [ ] `packages/engine-adapter/src/index.ts`
  - [ ] `packages/engine-adapter/tests/engine-abstraction.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-008
  - arch-keel-api#§3.API-009
  - prd-keel-f010-f018#§2.F-018
  - arch-keel#§5.2

---

### T-014: 工具面（MCP Server / Hooks / CLI）

- **目标**: 实现 `ToolSurface`（M-008）：以 MCP Server 暴露 `keel.gate.check` / `keel.blueprint.read` / `keel.map.state` / `keel.drift.report` 四个工具，供执行引擎消费
- **模块**: M-008
- **接口**: API-010
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given MCP Server 启动，When 执行引擎调用 `keel.gate.check({ changeset: ['file.ts'] })`，Then 返回 `GateReport` JSON（调用内部 `GateEngine.run(...)`）[ARCH#§3.API-010 / prd#§2.F-018 AC-006]
  - [ ] AC-002: Given 执行引擎调用 `keel.blueprint.read({ selector: { kind: 'module' } })`，Then 返回 `BlueprintIR`（或投影子集）JSON [ARCH#§3.API-010 operations]
- **deliverables**:
  - [ ] `packages/engine-adapter/src/toolsurface/ToolSurface.ts` — MCP tool handler 注册
  - [ ] `packages/engine-adapter/src/toolsurface/McpServer.ts` — MCP Server 启动
  - [ ] `packages/engine-adapter/tests/tool-surface.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-008
  - arch-keel-api#§3.API-010
  - prd-keel-f010-f018#§2.F-018

---

### T-015: 意图翻译器（IntentTranslator + LLM 调用）

- **目标**: 实现 `IntentTranslator`（M-005）：将用户自然语言意图经执行引擎 LLM 结构化生成为 `BlueprintDiff`，注入最小上下文（选中节点 + 一跳邻域），`plainExplanation` 无代码术语
- **模块**: M-005
- **接口**: API-005
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `intentText: '加一个用户能收藏文章的功能'` 且 `boundNodeId: 'article-module'`，When `IntentTranslator.translate({ intentText, boundNodeId })`（含 mock LLM 返回）,Then 返回 `{ blueprintDiff: BlueprintDiff { added: [...], modified: [...] }, plainExplanation: string（无类名/文件路径）, blastRadius, durationMs ≤ 10000 }` [ARCH#§3.API-005 / prd#§2.F-009 AC-001/AC-002]
  - [ ] AC-002: Given LLM 返回不完整 JSON（缺 `blueprintDiff.added`），When `IntentTranslator.translate(...)`，Then 返回 `KeelError { code: 'INTENT_PARSE_ERROR', plainMessage: '没能理解这个需求，请换个说法...' }` [ARCH#§3.API-005 error]
  - [ ] AC-003: Given `boundNodeId: null`（无 point-and-prompt），When `translate({ intentText, boundNodeId: null })`，Then 正常产出 `BlueprintDiff`，不依赖绑定节点 [prd#§2.F-009 AC-003]
  - [ ] AC-004（production-path）: `packages/intent/src/IntentTranslator.ts` 中调用 `engineAbstraction.prompt({ task, context: ContextBundle })` 完成 LLM 请求 [ARCH#§2.M-005 / ARCH#§2.M-008]
- **deliverables**:
  - [ ] `packages/intent/src/translator/IntentTranslator.ts`
  - [ ] `packages/intent/src/translator/ContextBuilder.ts` — 最小上下文装配（选中节点 + 一跳邻域）
  - [ ] `packages/intent/src/translator/PlainExplainer.ts` — diff → 通俗说明
  - [ ] `packages/intent/tests/intent-translator.test.ts`（含 LLM mock）
- **context_load**:
  - arch-keel-modules#§2.M-005
  - arch-keel-api#§3.API-005
  - prd-keel-f001-f009#§2.F-009
  - arch-keel#§5.1

---

### T-016: 蓝图 Diff + 通俗说明 + Blast Radius 展示

- **目标**: 实现 `BlueprintDiffer` 和 `BlastRadiusCalculator`（M-005）；`BlueprintDiff` 含 added/modified/touchedBoundaries；blast radius 降级时显式 `degraded: true` [ASSUMPTION]
- **模块**: M-005
- **接口**: API-005 (完整返回)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `BlueprintDiff` 含新增 Module `favorites`，When `BlueprintDiffer.diff(prevIR, nextIR)`，Then 返回 `{ added: [{ id: 'favorites', ... }], modified: [], touchedBoundaries: [...] }` [ARCH#§3.API-005]
  - [ ] AC-002: Given 依赖图无环且完整，When `BlastRadiusCalculator.calculate({ changedNodeIds })`，Then 返回 `{ touched: [...], safe: [...], degraded: false }`；有环时 `degraded: true` [prd#§2.F-007 AC-003 / ARCH#§3.API-005]
  - [ ] AC-003: Given `blastRadius.degraded === true`，When `<BlastRadiusBanner degraded={true} />` 渲染，Then DOM 中出现含 `color.warn` 样式的警告横幅文本「影响范围可能不完整」；`degraded={false}` 时该横幅 DOM 节点不存在 [prd#§2.F-007 AC-003 / ui-spec-keel-components#UC-008]
- **deliverables**:
  - [ ] `packages/intent/src/differ/BlueprintDiffer.ts`
  - [ ] `packages/intent/src/blast/BlastRadiusCalculator.ts`
  - [ ] `packages/intent/tests/blueprint-differ.test.ts`
  - [ ] `packages/intent/tests/blast-radius.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-005
  - arch-keel-api#§3.API-005
  - prd-keel-f001-f009#§2.F-007
  - prd-keel-f001-f009#§2.F-009

---

### T-017: 契约冻结门（FrozenContractSet + E-004）

- **目标**: 实现 `ContractFreezer`（M-005）：确认后冻结 Port 签名 + Entity schema 为只读锚点（E-004），不完整退回澄清；产出 `frozenContractSetId`
- **模块**: M-005
- **接口**: API-006
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `BlueprintDiff` 含完整 Port 签名与 Entity schema，When `ContractFreezer.freeze({ blueprintDiff })`，Then 返回 `{ frozenContractSetId: string, complete: true, clarificationNeeded: undefined }`；在 SQLite E-004 表插入记录，`ports` 与 `entitySchemas` 字段非空 [ARCH#§3.API-006 / prd#§2.F-009 AC-006]
  - [ ] AC-002: Given `BlueprintDiff` 中 Port `favorites.api` 缺少 `shape.type_ref`（不完整），When `ContractFreezer.freeze({ blueprintDiff })`，Then 返回 `{ complete: false, clarificationNeeded: '收藏接口的参数类型还没定义清楚，请补充…' }` [ARCH#§3.API-006 notes / prd#§2.F-009 AC-006]
  - [ ] AC-003: Given `frozenContractSetId` 已写入 E-004 表（`ports = '{"api":{"type":"rest"}}'`），When 后续蓝图 IR 修改该 Port 签名并再次调用 `ContractFreezer.freeze(...)`，Then 从 E-004 `SELECT ports WHERE id = frozenContractSetId` 返回原始 JSON 字符串不变（仍为 `'{"api":{"type":"rest"}}'`），不被覆盖 [ARCH#§4.E-004]
  - [ ] AC-004（production-path）: `packages/intent/src/freezer/ContractFreezer.ts` 调用 `dbClient.insert(frozenContractSets, ...)` 写入 E-004 [ARCH#§4.E-004]
- **deliverables**:
  - [ ] `packages/intent/src/freezer/ContractFreezer.ts`
  - [ ] `packages/intent/tests/contract-freezer.test.ts`（含 SQLite in-memory 测试）
- **context_load**:
  - arch-keel-modules#§2.M-005
  - arch-keel-api#§3.API-006
  - arch-keel-data#§4.E-004
  - prd-keel-f001-f009#§2.F-009

---

### T-018: [VALIDATION] 漂移检测 + 意图捕获核心流程

- **目标**: 用户（或测试负责人）手动验证 Sprint 2 核心产出
- **task_kind**: validation
- **模块**: M-003, M-005, M-008
- **tdd_acceptance**: N/A（validation 任务不走 TDD，由 orchestrator 展示验证清单后用户手动确认）
- **deliverables**: N/A（validation 任务不产出代码文件，由前置任务 T-010~T-017 的 deliverables 覆盖）
- **验证清单**:
  - [ ] 在 `features/payment/index.ts` 中直接 import `features/auth/` 中未声明的内部模块（通道外改动），`ChangeWatcher` 触发，`DriftDetector` 产出 `DriftReport` 含 `boundary_break` 类型
  - [ ] `DriftReport.items[0].plain` 为中文通俗描述（不含「import」「module」等代码术语）
  - [ ] 输入意图文本「加收藏功能」，`IntentTranslator` 在 mock LLM 下 ≤10s 产出 `BlueprintDiff`，`plainExplanation` 不含类名或文件路径
  - [ ] `BlastRadiusCalculator` 在有环依赖图下返回 `degraded: true`，无环时 `degraded: false`
  - [ ] 完整 Port 签名的 `BlueprintDiff` 调用 `ContractFreezer.freeze()` 后，SQLite E-004 表有记录，`frozenContractSetId` 非空
  - [ ] 不完整 Port（缺 `shape.type_ref`）的 diff 调用冻结后返回 `complete: false` 且 `clarificationNeeded` 为中文提示
  - [ ] `EngineCapabilityProbe.detect()` 在有 Claude Code CLI 的环境返回 `{ ready: true, kind: 'cli' }`；无引擎时返回 `{ ready: false, installGuide: '...' }`
- **前置任务**: [T-010, T-011, T-012, T-013, T-014, T-015, T-016, T-017]
- **context_load**:
  - arch-keel-modules#§2.M-003
  - arch-keel-modules#§2.M-005
  - arch-keel-modules#§2.M-008
  - prd-keel-f001-f009#§2.F-002
  - prd-keel-f001-f009#§2.F-009
