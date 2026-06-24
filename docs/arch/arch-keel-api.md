---
id: "arch-keel-api"
version: "0.1"
doc_type: arch
author: architect
status: approved
deps: ["prd-keel"]
consumers: [tech-lead, developer, devops]
volume: api
volume_type: api
split_from: "arch-keel"
required_sections:
  - "## 3. 接口契约"
---
# Architecture 分卷 — 接口契约: Keel

[NAV]
- §3 接口契约 → API-001..API-016
[/NAV]

## 3. 接口契约

> Keel 本体多为进程内 TS 模块契约（`kind: in-process`）；执行引擎接缝为 `kind: json-rpc`（ACP over stdio）；工具面为 `kind: mcp-tool`。每个 API 给出主操作的 `request`（含 type/required/desc）与 `response`（schema）；同模块次要操作在 `operations` 列出。错误统一 `GateError`（见 API-002）/ `KeelError { code, plainMessage }`。

### API-001: 蓝图读写与查询
```yaml
kind: in-process
module: M-001
request:           # 主操作 load
  path:    { type: string, required: true, desc: "蓝图 YAML 文件路径" }
response:
  ok:    { schema: "{ ir: BlueprintIR, valid: boolean, schemaErrors: SchemaError[] }" }
  error: { schema: "KeelError" }
operations:
  validate:    { in: "{ ir }", out: "{ valid, errors: SchemaError[] }" }
  query:       { in: "{ selector: Selector }", out: "{ nodes, edges }" }
  blastRadius: { in: "{ changedNodeIds: string[] }", out: "{ touched, safe, degraded }" }
  persist:     { in: "{ ir }", out: "{ path, version }" }
notes: ir 双层（business 投影 + implementation 全量）；persist 写 Git 可 diff YAML/JSON
```

### API-002: 门禁执行（结构化结果）— F-003 AC-006
```yaml
kind: in-process
module: M-002
request:
  changeset:    { type: "string[]", required: true, desc: "本次改动文件路径集" }
  scope:        { type: "enum(incremental|full)", required: true, desc: "增量或全量" }
  blueprintIR:  { type: BlueprintIR, required: true, desc: "期望态蓝图" }
response:
  ok:    { schema: "GateReport" }
  error: { schema: "KeelError" }
types:
  GateReport: "{ passed: boolean, structural: { hardViolations: GateError[], softWarnings: GateError[] }, regressionReport: RegressionBreak[], durationMs: number(≤5000) }"
  GateError:  "{ ruleId, checker: enum(dependency-cruiser|ts-morph|tsc|eslint|madge), severity: enum(hard|soft), locations: {file,line?}[], message, suggestedFix? }"
  RegressionBreak: "{ brokenCapability: string, plain: string, violations: GateError[] }"   # F-004: 本次改动破坏的既有能力 + 通俗说明
notes: 两阶段串行（F-004 AC-001）—— structural=本次结构门（F-003）；regressionReport=回归守护（F-004）跨能力检出的破坏，供 M-009 在地图上区分「结构违规 red」与「回归破坏 red」并带说明（F-004 AC-003）。passed=structural 无 hard 违规且 regressionReport 为空。message 机器可解析、对用户隐形（F-003 AC-004）；该 JSON 同经 API-010 工具面暴露
```

### API-003: 漂移检测
```yaml
kind: in-process
module: M-003
request:
  scope:   { type: "enum(incremental|full)", required: true, desc: "检测范围" }
  source:  { type: "enum(channel|external)", required: false, desc: "变更来源；external=通道外（F-002 AC-006），不影响检测覆盖" }
response:
  ok:    { schema: "DriftReport" }
  error: { schema: "KeelError" }
types:
  DriftReport: "{ items: DriftItem[], durationMs: number(增量≤3000) }"
  DriftItem:   "{ type: enum(boundary_break|duplicate_impl|orphan_code|contract_shape_mismatch), plain, locations:{file}[], resolution: enum(reconcile|update_blueprint) }"
```

### API-004: 调和切片
```yaml
kind: in-process
module: M-004
request:
  capabilityId:        { type: string, required: true, desc: "待调和切片能力 id" }
  frozenContractSetId: { type: string, required: true, desc: "锚定的冻结契约集合 id" }
response:
  ok:    { schema: "{ regenerated: boolean, gateReport: GateReport, regressionPassed: boolean, protectedContentPrompt?: string }" }
  error: { schema: "KeelError" }
notes: 含 escape_hatch / 未入图内容时返回 protectedContentPrompt 要求用户确认（F-011 AC-007）
```

### API-005: 意图→蓝图 diff + blast radius
```yaml
kind: in-process
module: M-005
request:
  intentText:  { type: string, required: true, desc: "用户自然语言意图" }
  boundNodeId: { type: string, required: false, desc: "point-and-prompt 绑定的地图节点 id" }
response:
  ok:    { schema: "{ blueprintDiff: BlueprintDiff, plainExplanation: string, blastRadius: {touched,safe,degraded}, durationMs: number(≤10000) }" }
  error: { schema: "KeelError" }
notes: blueprintDiff 经确认/追加修改/拒绝（F-009 AC-005）后才进 API-006
```

### API-006: 契约冻结
```yaml
kind: in-process
module: M-005
request:
  blueprintDiff: { type: BlueprintDiff, required: true, desc: "已确认的蓝图 diff" }
response:
  ok:    { schema: "{ frozenContractSetId: string, complete: boolean, clarificationNeeded?: string }" }
  error: { schema: "KeelError" }
notes: 冻结 Port 签名 + Entity schema 为只读锚点；不完整退回澄清（F-009 AC-006）
```

### API-007: 实现执行
```yaml
kind: in-process
module: M-006
request:
  frozenContractSetId: { type: string, required: true, desc: "实现锚定的冻结契约集合 id" }
response:
  ok:    { schema: "{ sliceResults: {capabilityId, status: enum(ready|failed|aborted)}[], gateReports: GateReport[] }" }
  error: { schema: "KeelError" }
notes: DAG 拓扑序；隔离 worktree；失败回滚到改动前完整态（F-010 AC-004）；进度经 API-011 投影到地图节点状态
```

### API-008: 自愈
```yaml
kind: in-process
module: M-007
request:
  gateErrors:          { type: "GateError[]", required: true, desc: "门禁输出的结构化错误" }
  frozenContractSetId: { type: string, required: true, desc: "锚定上下文" }
response:
  ok:    { schema: "{ resolved: boolean, attempts: number(≤3), escalation?: DecisionCard }" }
  error: { schema: "KeelError" }
types:
  DecisionCard: "{ conflict, plainImpact, options: {id, plain}[] }"
notes: 业务决策类冲突或达重试上限 → escalation（F-010 AC-006）；技术性错误静默自愈
```

### API-009: 执行引擎抽象（统一接缝）
```yaml
kind: json-rpc                   # ACP over stdio；SDK/CLI adapter 同契约
module: M-008
request:           # 主操作 applyEdit
  edit:       { type: EditRequest, required: true, desc: "执行引擎产生的一次编辑请求" }
  engineKind: { type: "enum(acp|sdk|cli|hosted)", required: false, desc: "目标执行器类型；hosted 为 F-018 AC-005 预留 variant，v1 空实现" }
response:
  ok:    { schema: "{ applied: boolean, gateReport: GateReport }" }
  error: { schema: "KeelError" }
operations:
  mediatePermission: { in: "{ action: AgentAction }", out: "{ decision: enum(allow|deny), reason? }" }
  prompt:            { in: "{ task: string, context: ContextBundle }", out: "{ result: AgentResult }" }
types:
  ContextBundle: "{ capabilityId: string, contract: FrozenContractSet, directDepSignatures: PortSignature[] }"   # F-010 精准上下文：仅自身契约 + 直接依赖签名
  AgentResult:   "{ success: boolean, gateReport?: GateReport, output?: string }"
notes: ACP / Agent SDK / CLI / (预留)hosted 各 adapter 实现同契约；切换对上层透明（F-018 AC-004），hosted variant v1 空实现（F-018 AC-005）。ContextBundle 由 M-006 ContextInjector 装配，[ASSUMPTION] PortSignature 细化待 dev-plan
```

### API-010: 工具面（执行引擎可消费）— F-018 AC-006
```yaml
kind: mcp-tool                   # 并行候选：引擎 hooks / CLI 命令
module: M-008
request:           # 主工具 keel.gate.check
  changeset: { type: "string[]", required: true, desc: "待检查文件集" }
response:
  ok:    { schema: "GateReport" }
  error: { schema: "KeelError" }
operations:
  keel.blueprint.read: { in: "{ selector? }", out: "BlueprintIR" }
  keel.map.state:      { in: "{}", out: "{ nodes: {id, health, status}[] }" }
  keel.drift.report:   { in: "{}", out: "DriftReport" }
notes: 执行引擎在实现/调和中必须能以结构化方式调用门禁并读 JSON 结果（衔接 API-002）
```

### API-011: 能力地图视图模型
```yaml
kind: in-process
module: M-009
request:
  focusNodeId:    { type: string, required: false, desc: "焦点子图中心节点" }
  aggregateLevel: { type: number, required: false, desc: "层级聚合深度" }
response:
  ok:    { schema: "{ nodes: MapNode[], edges: MapEdge[], renderMs: number(≤1000) }" }
  error: { schema: "KeelError" }
types:
  MapNode: "{ id, plainName, plainDetail, kind: enum(module|port|entity|relation|workflow|statemachine|constraint), health: enum(green|amber|red), status, isEscapeHatch: boolean }"
  MapEdge: "{ id, from, to, relationType, diffState?: enum(added|touched|unchanged), highlighted?: boolean }"
notes: 仅 surfaces_on_map=true；只读（F-006 AC-006）；红色节点带问题详情与修复入口；kind 对应 F-001 AC-001 七原语，isEscapeHatch 为 Module 修饰位（非第八原语）
```

### API-012: 应用预览
```yaml
kind: in-process
module: M-010
request: none      # 主操作 getPreview 无入参
response:
  ok:    { schema: "{ url: string, state: enum(ready|building|unavailable), plainStatus: string }" }
  error: { schema: "KeelError" }
operations:
  refresh: { in: "{}", out: "{ refreshed: boolean }" }   # 门禁通过后自动触发
```

### API-013: 意图日志与时光机
```yaml
kind: in-process
module: M-011
request:           # 主操作 logEntry
  intentText:    { type: string, required: true, desc: "原始意图文本" }
  blueprintDiff: { type: BlueprintDiff, required: true, desc: "对应蓝图 diff" }
  decision:      { type: DecisionCard, required: false, desc: "「需你决策」留痕（如有）" }
response:
  ok:    { schema: "{ entryId: string, gitRef?: string }" }
  error: { schema: "KeelError" }
operations:
  timeline: { in: "{ limit? }", out: "{ entries: {entryId, plainSummary, timestamp, entryKind: enum(change|decision|deploy)}[] }" }   # 查询 ≤3s；entryKind 驱动时间线条目变体
  rollback: { in: "{ entryId }", out: "{ restored: boolean, untrackedChangePrompt? }" }       # 通道外改动先提示（F-013 AC-006）
```

### API-014: 上线与安全体检
```yaml
kind: in-process
module: M-012
request:           # 主操作 securityAudit
  scope: { type: "enum(repo|export)", required: true, desc: "体检范围" }
response:
  ok:    { schema: "{ score: number, checks: {category: enum(authn_config|data_authz|secret_leak), light: enum(green|red), plain, fixEntry?}[] }" }
  error: { schema: "KeelError" }
operations:
  deploy: { in: "{ confirmedGreen: boolean }", out: "{ deployed: boolean, url?, blockedByRed?: string[] }" }   # 红灯阻断（F-014 AC-003）
```

### API-015: 导出与交接
```yaml
kind: in-process
module: M-013
request: none      # 主操作 exportRepo 无入参（导出当前项目）
response:
  ok:    { schema: "{ repoPath: string, includesToolConfigs: boolean, secretsFound: number(须为0) }" }
  error: { schema: "KeelError" }
operations:
  generateHandoff: { in: "{}", out: "{ handoffPath: string, sections: enum(arch_doc|decision_history|boundary_guide)[] }" }   # Markdown（F-016）
```

### API-016: 项目初始化
```yaml
kind: in-process
module: M-014
request:           # 主操作 scaffold
  intentOrTemplate: { type: string, required: true, desc: "应用意图自由输入或起步模板 id" }
response:
  ok:    { schema: "{ blueprintPath: string, skeletonReady: boolean, archetype: 'modular-monolith' }" }
  error: { schema: "KeelError" }
operations:
  detectEngine: { in: "{}", out: "{ ready: boolean, kind?: enum(acp|sdk|cli), installGuide? }" }
  authenticate: { in: "{ method: enum(oauth|api_key) }", out: "{ authenticated: boolean }" }   # 本地存储最小权限
  usageStatus:  { in: "{}", out: "{ planLabel: string, used: number, quota: number, source: enum(subscription|api_key|unknown) }" }   # M-014 UsageProbe 暴露：引擎侧订阅/用量（F-012 AC-007），仅消耗透明度
```
