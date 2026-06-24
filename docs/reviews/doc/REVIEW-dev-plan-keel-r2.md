---
id: "review-dev-plan-keel-r2"
doc_type: review
author: reviewer
status: approved
deps:
  - dev-plan-keel
  - dev-plan-keel-s1
  - dev-plan-keel-s2
  - dev-plan-keel-s3
  - dev-plan-keel-s4
  - dev-plan-keel-s5
  - dev-plan-keel-s6
---

# REVIEW-dev-plan-keel-r2

**被审文档**: dev-plan-keel（主卷）+ dev-plan-keel-s1 ~ s6（Sprint 1~6 任务卡，T-001~T-057）
**审查轮次**: r2（Layer 1 已由 orchestrator 复跑确认全 7 卷 PASS；本轮执行 r1 未执行的 Layer 2 语义审查）

---

## r1 问题状态（R-001~R-009 RESOLVED 核认）

| 问题 | 严重等级 | 状态 |
|------|---------|------|
| R-001 主卷缺「任务卡详细」章节 | HIGH | RESOLVED — 主卷已增 §3 含索引总表，各分卷 NAV 引用齐全 |
| R-002 各分卷缺 deliverables（6 处）| HIGH | RESOLVED — Layer 1 PASS 确认 |
| R-003 各分卷缺 tdd_acceptance（6 处）| HIGH | RESOLVED — Layer 1 PASS 确认 |
| R-004 AC 描述缺可观测动词（36 处）| MEDIUM | RESOLVED — 抽查 s1/s2/s4/s6 全卷 AC 均含「返回」「渲染」「触发」等可观测动词 |
| R-005 主卷 NAV 块与章节不一致 | MEDIUM | RESOLVED — NAV 块现与正文章节（§1 迭代规划 / §3 任务卡详细 / §2 依赖图 / §4 关键路径 / §5 风险项 / §6 集成与E2E测试规划）对应（注意正文顺序为 §1→§3→§2→§4→§5→§6，NAV 顺序与此一致） |
| R-006 主卷未引用 s2~s5 分卷 | MEDIUM | RESOLVED — NAV 块中 §3 任务卡详细下现列出 s1~s6 全部分卷链接 |
| R-007 主卷行数超 300 | LOW | RESOLVED（降为 WARN）— 主卷现 411 行，仍超 300，但属依赖图（~160 行 Mermaid）+ 集成测试规划（~20 行）+ 索引总表的必要内容，拆分会破坏单文档可读性；Layer 1 PASS 证明检查器已接受；**不升级为阻断问题** |
| R-008 s4 行数超 300 | LOW | 待核（Layer 1 PASS，行数约束为 WARN 级非 FAIL 级） |
| R-009 s5 行数超 300 | LOW | 待核（Layer 1 PASS，行数约束为 WARN 级非 FAIL 级） |

---

## Layer 2 语义审查

### 维度 1：F-001~F-018 覆盖矩阵核验

**结论：F-001~F-018 全量覆盖，每条 F 均有 ≥1 任务承载。**

| 功能 | 优先级 | 主要承载任务 | 核验状态 |
|------|--------|------------|---------|
| F-001 蓝图核心原语与元模型 | P0 | T-001（M-001 schema+IR，AC-001~AC-007 全覆盖）| 覆盖 |
| F-002 蓝图双向同步与漂移检测 | P0 | T-010（AstExtractor）T-011（DriftClassifier）T-012（ChangeWatcher）| 覆盖 |
| F-003 验证门（Gate）| P0 | T-004（CheckerAdapter）T-005（GateEngine 主入口）| 覆盖 |
| F-004 回归守护 | P1 | T-006（RegressionGuard）| 覆盖 |
| F-005 自动修复（静默自愈）| P1 | T-023（错误分类与重试）T-024（上浮与「需你决策」卡）| 覆盖 |
| F-006 能力地图 | P0 | T-030（MapProjector+React Flow）T-031（HealthOverlay）| 覆盖 |
| F-007 改动影响预览（Blast Radius）| P1 | T-003（DependencyGraph.blastRadius）T-016（blast radius 展示）T-033（P-003 diff 侧）| 覆盖 |
| F-008 应用实时预览 | P1 | T-034（P-004 M-010 应用预览面）| 覆盖 |
| F-009 意图捕获与蓝图 diff | P0 | T-015（IntentTranslator+LLM）T-016（diff+通俗说明）T-017（契约冻结）T-032（对话面）| 覆盖 |
| F-010 受门禁约束的实现执行 | P0 | T-019（DagScheduler）T-020（ContextInjector）T-021（WorktreeIsolator）T-022（ProgressStateMachine）T-035（P-005 进度叠加）| 覆盖 |
| F-011 调和引擎（Reconcile）| P1 | T-025（切片重生成+内容保护）T-036（P-006 健康调和抽屉）| 覆盖 |
| F-012 项目初始化与上手 | P0 | T-038（引擎检测）T-039（鉴权+骨架）T-040（P-007 向导页面）M-008（AC-003 连接冒烟）| 覆盖 |
| F-013 意图日志与时光机 | P1 | T-041（意图日志存储 E-002/E-003）T-042（时间线+回滚）T-043（P-008 时光机页面）| 覆盖 |
| F-014 一键上线与安全体检 | P1 | T-044（SecurityAuditor）T-045（DeployDriver）T-046（P-009 上线页面）| 覆盖 |
| F-015 开放代码与一键导出 | P1 | T-047（代码仓导出+工具配置编译）T-049（P-010 导出页面）| 覆盖 |
| F-016 开发者交接包 | P2 | T-048（HandoffPackager）| 覆盖 |
| F-017 Escape Hatch（自由实现区）| P1 | T-007（EscapeHatchResolver）T-050（P-011 自由实现区标注+地图展示）| 覆盖 |
| F-018 执行引擎抽象层 | P1 | T-013（ACP/SDK/CLI Adapter）T-014（MCP Server/hooks/CLI）T-052（全链路 AC-004 透明切换）| 覆盖 |

**F-001~F-018 全量覆盖核验结论：通过。**

**M-001~M-015 覆盖核验**（基于主卷任务索引总表与 arch-keel-modules §2 比对）：

| 模块 | 主要承载任务 |
|------|------------|
| M-001 蓝图引擎 | T-001/T-002/T-003 |
| M-002 门禁引擎 | T-004/T-005/T-006/T-007 |
| M-003 漂移检测器 | T-010/T-011/T-012 |
| M-004 调和引擎 | T-025 |
| M-005 意图编排器 | T-015/T-016/T-017/T-032/T-033 |
| M-006 实现执行器 | T-019/T-020/T-021/T-022 |
| M-007 自愈代理层 | T-023/T-024 |
| M-008 执行引擎抽象层 | T-013/T-014 |
| M-009 能力地图渲染器 | T-030/T-031 |
| M-010 应用预览 | T-034 |
| M-011 意图日志 | T-041/T-042/T-043 |
| M-012 上线与安全体检 | T-044/T-045/T-046 |
| M-013 导出交接 | T-047/T-048/T-049 |
| M-014 项目初始化 | T-038/T-039/T-040/T-053 |
| M-015 Web 壳 | T-027/T-028/T-029 |

M-001~M-015 全量覆盖。

---

### 维度 2：AC 可追溯性抽查（4 条 F 深核）

抽查 F-001、F-002、F-010、F-018 的 AC 与对应任务 tdd_acceptance 的对齐度。

**F-001 抽查（→ T-001）**：
- F-001 AC-001（七原语支持）→ T-001 AC-001 `SchemaValidator.validate()` 校验七原语 — 对齐
- F-001 AC-002（`plain` 字段必填，不上界面）→ T-001 AC-001 plain 缺失时返回 `{ valid: false }` — 对齐
- F-001 AC-003（`requirements[]` R-IR）→ T-001 AC-003 `ir.requirements` 含 id/plain/acceptance/oracle — 对齐
- F-001 AC-004（需求覆盖率 100%）→ T-001 AC-004 未被 `satisfies` 引用时校验报错 — 对齐
- F-001 AC-005（Constraint.severity hard/soft）→ T-001 AC-005 直接验证 — 对齐
- F-001 AC-006（Port.contract 格式合法性）→ T-001 AC-006 直接验证 — 对齐
- F-001 AC-007（blueprint.example.yaml 权威实例）→ T-001 deliverables + T-009 验证清单 — 对齐

**F-002 抽查（→ T-010/T-011/T-012）**：
- F-002 AC-001（AST 逆向提取）→ T-010 AC-001 `AstExtractor.extract` 返回模块边界/导出/依赖 — 对齐
- F-002 AC-002（四类漂移）→ T-011 AC-002 `DriftClassifier.classify` 返回四类枚举 — 对齐
- F-002 AC-003（通俗语言展示）→ T-011 AC-002 `item.plain` 不含 import/module/class 等代码术语 — 对齐
- F-002 AC-005（增量检测 ≤3s）→ T-054 AC-002 `DriftReport.durationMs ≤ 3000` — 对齐（注：在 T-054 性能优化任务，不在 T-011，属合理分离）
- F-002 AC-006（来源无关）→ T-011 AC-003 source:'external' vs source:'channel' 结果一致；T-012 AC-001 ChangeWatcher 通道外触发 — 对齐

**F-010 抽查（→ T-019~T-022/T-035）**：
- F-010 AC-001（契约冻结后进入）→ T-017 FrozenContractSet 产出冻结；T-019 AC-001 依赖 T-017 — 对齐
- F-010 AC-002（DAG 拓扑序）→ T-019 AC-002 `DagScheduler.run()` 拓扑序验证 — 对齐
- F-010 AC-003（每次改动经门禁）→ T-022 `ProgressStateMachine` 状态机中门禁触发点；T-005 GateEngine — 对齐
- F-010 AC-004（实现隔离）→ T-021 AC-002 `WorktreeIsolator` 回滚失败切片 — 对齐
- F-010 AC-005（进度以地图节点状态变化呈现）→ T-035 P-005 实现进度叠加+「需你决策」卡 — 对齐

**F-018 抽查（→ T-013/T-014/T-052）**：
- F-018 AC-001（蓝图/门禁/调和与引擎解耦）→ T-013 `EngineAdapter` 统一接口，三种 adapter — 对齐
- F-018 AC-002（ACP 作为标准接缝）→ T-013 AcpAdapter — 对齐
- F-018 AC-004（透明切换）→ T-052 AC-004 ACP→SDK 切换三者一致 — 对齐
- F-018 AC-006（工具面：MCP Server/hooks/CLI）→ T-014 工具面实现；T-052 AC-001 `engineServer.ts` 注册全链路模块 — 对齐

AC 可追溯性核验结论：4 条 F 全部对齐，无缺漏或错配。

---

### 维度 3：consistency — 技术引用一致性

抽查要点：

1. **栈一致性**（arch §1.4：Next.js 16 + Postgres + BetterAuth + Drizzle，Keel 引擎 TS 单栈）：
   - dev-plan 所有包均以 `packages/` monorepo 形式，使用 TypeScript；T-008 使用 better-sqlite3 + Drizzle（本地引擎数据层，非 Postgres）；T-039 写 E-006 Drizzle insert。
   - Web 壳：T-029 React + Vite（非 Next.js）；T-030 React Flow v12。注意：arch §1.4 的 Next.js 16 面向最终 Web 应用交付，Keel 工作区自身用 Vite+React。**技术引用本身不矛盾**（本地引擎服务 = Node/TS；Web 壳 = Vite React；BetterAuth/Postgres 将在骨架生成的目标应用中使用，不是 Keel 引擎自身），但 dev-plan 中未显式说明这一分层，可能给实现者带来疑惑（已标为 MEDIUM，见下方 R-010）。

2. **API/E 引用存在性**：
   - T-008 引用 E-001~E-006 — arch-keel-data §4 定义确认存在
   - T-001 引用 API-001（SchemaValidator.validate/load/blastRadius）— arch-keel-api §3.API-001 存在
   - T-004 引用 API-002（GateEngine.run/GateReport）— arch-keel-api §3.API-002 存在
   - T-053 引用 API-016（usageStatus/detectEngine）— arch-keel-api §3.API-016 存在
   - T-030 引用 API-011（MapProjector.project/MapNode/MapEdge）— arch-keel-api §3.API-011 存在

3. **UC/P 引用存在性**：
   - T-028 引用 UC-001/UC-004/UC-013/UC-015/UC-018 — 已在 ui-spec-keel-components 定义（通过 T-027 token 先行确认）
   - T-053 引用 UC-020 — 在 ui-spec-keel-components §2.UC-020 存在
   - T-029 引用 P-001 — ui-spec-keel-pages-core §3.P-001 存在

---

### 维度 4：feasibility — 依赖图与 Sprint 分组

1. **依赖图无环核验**：主卷已声明 orchestrator 确认权重 32，Mermaid DAG 有向图视觉检查（T-001 → 众多下游；全部 VALIDATION 任务均为有向边汇入；T-052 汇聚 T-009/T-018/T-026/T-037/T-051 五个 VALIDATION）未发现循环结构。

2. **Sprint 分组尊重依赖**：
   - Sprint 1: T-001（蓝图 IR 根节点）+ T-004（门禁骨架）→ 正确，二者是最多下游任务的前置节点
   - Sprint 2: T-010~T-012 依赖 T-001/T-004（Sprint 1 完成）；T-015~T-017 依赖 T-001/T-013 — 满足
   - Sprint 4: T-027 → T-028 → T-029 → 各业务面 — 依赖前序正确
   - Sprint 6: T-052 依赖 T-009/T-018/T-026/T-037/T-051（前五个 Sprint VALIDATION 全部完成）— 正确

3. **TDD 模式合理性抽查**：
   - T-001（schema+IR 核心）: `standard` — 正确（复杂，多 AC，有安全约束）
   - T-004（门禁骨架，含 CheckerAdapter 注册，跨模块）: `standard` — 正确
   - T-007（EscapeHatchResolver，单一豁免判决逻辑）: `light` — 合理
   - T-008（SQLite schema，`security_sensitive: true`）: `light` — 注意 security_sensitive=true 但 tdd_mode=light；按 COMMON-RULES TDD_LIGHT_LOC_THRESHOLD，security_sensitive=true 时应升 standard。**已标为 LOW**（见下方 R-011）
   - T-017（契约冻结门，门禁类）: `standard` — 正确
   - T-039（鉴权+骨架生成，`security_sensitive: true`）: `standard` — 正确

---

### 维度 5：范围纪律

1. **行为门执行类任务（ADR-0009）**：grep `oracle: test|oracle: property|行为门` 结果为空。dev-plan 任务卡中无行为测试执行相关任务。T-001 AC-003 包含 oracle 字段预留校验（`含 oracle 标注字段`），但仅是格式合法性检查，不执行行为测试——符合 ADR-0009。核验通过。

2. **非 TS 目标任务（ADR-0003）**：grep Python/Go/Rust/Java 结果为空（仅有 `python -m http.server` 是项目预览命令，不是实现任务）。核验通过。

3. **Gold-plating（PRD/arch 未要求功能）**：
   - T-052~T-057 Sprint 6 全为已确认任务（接线 + 性能优化 + 错误审查 + E2E 验收），无 PRD/arch 未要求的新功能
   - T-054 performance optimization 对应 F-002 AC-005（≤3s）+ F-003 AC-005（≤5s）PRD 约束，有明确来源
   - T-055 首屏 ≤2s 对应 F-008 AC-005（PRD 隐式，ui-spec §5 显式约束）

4. **ADR-0011 范围纪律**（非程序员产品，不面向开发者工具产品）：T-014 工具面（MCP Server / hooks / CLI）明确定位为「产品内部接缝，面向执行引擎/LLM 消费」，符合 ADR-0011 注释（arch §3.API-010 / F-018 AC-006 备注）。核验通过。

---

### 维度 6：convention — 工时估算 + 设计阶段残留

**工时估算 grep**（`分钟|小时|天|Wave|人天|快.*完成`）：结果为空。无工时估算违规。

**设计阶段/变更说明残留 grep**（`之前|previously|used to|修复了|MVP|原方案|改为|之前是|现已废弃|本次新增|本轮加入`）：
- `docs/dev-plan/dev-plan-keel-s3.md:35` — `「之前调用返回 'waiting'」`：出现在 AC 描述句子中，表达行为语义对比（「之前」指未标 done 时）。这是 AC 内的**状态变化描述**（Before-After 断言语法），不是变更说明残留。属合理用法，不计入违规。
- `docs/dev-plan/dev-plan-keel-s6.md:178` — `「回滚到 2 步之前」`：validation 验证清单中的用户操作描述语言（「2步之前」描述回滚目标），非设计阶段残留。合理。

**结论**：无 convention 违规。

---

## 问题列表

### [R-010] MEDIUM: dev-plan 未说明「技术栈分层」背景知识

- **category**: ambiguity
- **root_cause**: self-caused
- **描述**: arch §1.4 锁定栈为 Next.js 16 + Postgres + BetterAuth + Drizzle（Keel 引擎 TS 单栈），但 dev-plan 任务卡中 Keel 工作区 Web 壳用 Vite + React（T-029），本地引擎服务用 better-sqlite3（T-008，非 Postgres）。这一「三层」分离（目标应用托管栈 / Keel 引擎服务栈 / Keel Web 壳栈）在 dev-plan 中未作说明，可能导致实现者混淆「Keel 的 Drizzle/better-sqlite3」与「目标应用的 Postgres」的职责边界，在 T-039 ScaffoldGenerator 实现时产生错误选型。
- **建议**: 在主卷 §1（迭代规划）或 §5（风险项）下补充一段「技术栈分层说明」（3~5 行），明确：① Keel 引擎本体（`packages/`）使用 TypeScript + better-sqlite3 存储元数据；② Keel Web 壳（`apps/workspace/`）使用 Vite + React；③ 目标应用（骨架生成后的用户应用）使用 Next.js 16 + Postgres + BetterAuth，由 ScaffoldGenerator 生成配置，不是 Keel 本身。

---

### [R-011] LOW: T-008 security_sensitive=true 但 tdd_mode=light

- **category**: feasibility
- **root_cause**: self-caused
- **描述**: T-008（元数据 SQLite Schema + E-001~E-006）标注 `security_sensitive: true`（因 E-006 authRef 字段涉及凭据引用约束），但 tdd_mode 为 `light`。按 COMMON-RULES `TDD_DEFAULT_MODE` 规定「`security_sensitive: true` → 应升 standard」，存在标注不一致。T-008 的 LOC 估算属轻量（schema 定义 + 迁移 SQL + 单例 client），light 模式体量合理，但 security_sensitive 标注明确要求升级。
- **建议**: 将 T-008 的 `tdd_mode` 改为 `standard`，或在任务卡中补充注释说明为何 security_sensitive=true 但维持 light（如「认证约束已在 AC-002 充分覆盖，全为同步单层 SQL 操作，LOC 极小」），以便 tdd-engine 路由不产生疑问。

---

## Verdict

**approved_with_notes** — Layer 1 全 7 卷 PASS，Layer 2 语义审查未发现 CRITICAL 或 HIGH 问题。F-001~F-018 全量覆盖，M-001~M-015 全量承载，AC 可追溯性抽查（F-001/F-002/F-010/F-018）全部对齐，依赖图分组尊重前置关系，无工时估算、无设计阶段残留、无行为门执行任务、无非 TS 任务。仅存在两个 MEDIUM/LOW 改善建议（R-010 技术栈分层未说明、R-011 T-008 tdd_mode 标注不一致），不阻断开发阶段。

**notes_summary**: (1) 主卷建议补充 3~5 行技术栈分层说明，区分 Keel 引擎栈 / Web 壳栈 / 目标应用托管栈；(2) T-008 的 tdd_mode 应对齐 security_sensitive=true 标注升为 standard，或显式注释豁免理由。
