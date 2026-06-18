---
id: "arch-keel-modules"
version: "0.1"
doc_type: arch
author: architect
status: approved
deps: ["prd-keel"]
consumers: [tech-lead, developer, devops]
volume: modules
volume_type: modules
split_from: "arch-keel"
required_sections:
  - "## 2. 模块划分"
---
# Architecture 分卷 — 模块划分: Keel

[NAV]
- §2 模块划分 → M-001..M-015
[/NAV]

## 2. 模块划分

> **实现需求** 行用 `M-xxx → prd#§2.F-yyy` 溯源格式（建 `cf:implements` 边）；每个 F-{NNN} 至少被一个 M-{NNN} 实现。

### M-001: 蓝图引擎 Blueprint Engine
- **实现需求**: M-001 → prd#§2.F-001
- **职责**: 蓝图图 IR 的唯一权威载体。加载/序列化 `blueprint.yaml`，按 `blueprint.schema.json` 做形态校验；维护双层图 IR（业务层 = `surfaces_on_map` 命中的原语投影，实现层 = 全量结构原语与契约）；提供图查询（节点/边/依赖/blast radius 基础算法）；以文本/JSON 双形态进 Git 可 diff。
- **对外接口**: API-001（蓝图读写/校验/查询）
- **依赖模块**: 无（最底层）
- **内部关键组件**: SchemaValidator(Ajv)、GraphModel(units/ports/relations/entities/constraints/workflows/statemachines 内存图)、BlueprintSerializer(YAML↔IR)、DependencyGraph(供 blast radius / no_orphan 复用)

### M-002: 门禁引擎 Gate Engine
- **实现需求**: M-002 → prd#§2.F-003；M-002 → prd#§2.F-004；M-002 → prd#§2.F-017
- **职责**: 把蓝图 `constraints[].rule.type` 映射为确定性检查器并编排执行；三道门（编译门 tsc / 契约测试门 Port 一致性 / 架构 lint 门）；硬违规阻断、软告警记录；输出机器可解析 JSON 错误供自愈与执行引擎重试；实现 escape hatch 豁免逻辑（仅豁免图结构类 + 架构 lint 软告警，保留编译门与 `hard` 安全约束）。
- **职责边界**: 门禁分**两阶段串行执行**（F-004 AC-001）：①**结构合规门（F-003）**——RuleCompiler+CheckerOrchestrator 对本次改动文件集做单点检查；②**回归守护（F-004）**——F-003 通过后，RegressionGuard 取该能力及其依赖方的累积检查集（按能力维护的历史 rule 集），跨能力复跑，检出「本次改动破坏既有能力」。两阶段结果由 GateResultAggregator 分区段统一上报（结构违规 vs 回归破坏，见 API-002 `regressionReport`）。**F-004 v1 回归守护仅含结构门与契约测试（Port 签名 / Entity schema 一致性），行为门（契约派生单测/属性测试）远期纳入（`ADR-0009`），不在 v1 范围内。** escape hatch（F-017）= 豁免规则（仅图结构类 + 架构 lint 软告警）。
- **对外接口**: API-002（门禁执行 → 结构化结果，含 regressionReport）
- **依赖模块**: M-001（读规则与图）
- **内部关键组件**: RuleCompiler(rule.type→checker 配置)、CheckerAdapter（dependency-cruiser / ts-morph / tsc / eslint / madge 各一）、CheckerOrchestrator(并行 + 增量调度)、**RegressionGuard(按能力维护累积检查集，F-003 通过后跨能力复跑回归校验)**、GateResultAggregator(分区段归一：结构门结果 + 回归守护结果)、EscapeHatchResolver

### M-003: 漂移检测器 Drift Detector
- **实现需求**: M-003 → prd#§2.F-002
- **职责**: 用 ts-morph 从代码 AST 逆向提取当前结构，与设计蓝图持续对账，检测四类漂移（破依赖边界 / 重复实现 / 孤立未接线 / 接口契约形态不符）；**来源无关**——无论改动经 Keel 通道还是通道外直接改文件均同等检测（F-002 AC-006）；以通俗语言产出漂移项；增量检测 ≤3s。
- **对外接口**: API-003（漂移检测 → 漂移报告）
- **依赖模块**: M-001（取期望态蓝图）, M-002（复用检查器做结构对账）
- **内部关键组件**: AstExtractor(ts-morph 逆向提取实际结构图)、BlueprintComparer(期望态 vs 实际态 diff)、DriftClassifier(四类归类 + plain 翻译)、ChangeWatcher(通道外改动发现：文件监听 / 项目打开扫描；**运行在本地引擎服务进程(Node)，事件经引擎服务 → Web 壳 WebSocket 推送；监听范围限 `features/` + 蓝图文件，避免全项目监听开销**；[ASSUMPTION] 发现时机(实时监听 vs 下次操作扫描)与精确进程边界待 dev-plan 确认，不削弱来源无关语义)

### M-004: 调和引擎 Reconcile Engine
- **实现需求**: M-004 → prd#§2.F-011
- **职责**: 以受影响切片为单位、以该切片冻结契约为锚，驱动执行引擎整片重生成 `features/<capability>/`（非就地修补、无状态、有界）；重生成受门禁约束（M-002）+ 回归守护通过才落地；内容保护——含 `escape_hatch` 或未入图行为的切片须先提示用户确认。
- **对外接口**: API-004（调和切片）
- **依赖模块**: M-001, M-002, M-006（复用受门禁实现机制）, M-008（驱动引擎）, M-007（重生成失败走自愈/上浮）
- **内部关键组件**: SliceBoundaryResolver、ContractAnchor(取冻结契约)、RegenerationDriver、ContentProtectionGuard(escape hatch / 未入图内容检测)

### M-005: 意图编排器 Intent Orchestrator
- **实现需求**: M-005 → prd#§2.F-009；M-005 → prd#§2.F-007
- **职责**: 把自然语言意图（可绑定选中地图节点，point-and-prompt）经 LLM 结构化生成为蓝图 diff（新增/修改哪些原语、触及边界、定义契约）；产出「地图 diff + 通俗解释」人话变更说明书；计算并展示 blast radius（F-007 复用 M-001 依赖图）；确认后冻结本次涉及的 Port 签名与 Entity schema 为只读锚点（冻结本身是一道门，不完整则退回澄清）。
- **对外接口**: API-005（意图→diff + blast radius）, API-006（契约冻结）
- **依赖模块**: M-001（图与依赖）, M-008（LLM 经执行引擎/直连）
- **内部关键组件**: IntentTranslator(LLM 结构化生成，注入最小上下文)、BlueprintDiffer、PlainExplainer(diff→人话)、BlastRadiusCalculator(降级显式提示)、ContractFreezer

### M-006: 实现执行器 Implementation Executor
- **实现需求**: M-006 → prd#§2.F-010
- **职责**: 契约冻结通过后进入实现；按蓝图依赖关系做 DAG 拓扑序调度，逐切片实现（被依赖者先就绪）；每模块仅注入自身契约 + 直接依赖签名的精准上下文；在隔离 worktree 内「实现—校验—合并」，失败/中止回滚到改动前完整可用状态（实现隔离）；每次文件改动经门禁（M-002），硬违规交自愈（M-007）；进度以地图节点状态机（规划中→实现中→已就绪）呈现，不暴露代码/终端。
- **对外接口**: API-007（实现执行）
- **依赖模块**: M-001, M-002, M-007, M-008（驱动引擎）, M-005（取冻结契约）
- **内部关键组件**: DagScheduler(拓扑序)、ContextInjector(精准最小上下文)、WorktreeIsolator、SliceImplDriver、ProgressStateMachine

### M-007: 自愈代理层 Auto-Repair Layer
- **实现需求**: M-007 → prd#§2.F-005
- **职责**: 消费门禁（M-002）输出的结构化错误，驱动执行引擎静默自愈；错误分类——纯技术性（唯一结构性修复路径、不改变已确认蓝图语义）静默自愈，业务决策类（需多方案取舍或改变已确认蓝图语义）上浮；同一错误 ≤3 次重试（可配），达上限升级「需你决策」卡片；自愈期间用户只见「正在修复…」，提供可见进度与可中止入口；不改变已确认蓝图 diff 语义。
- **对外接口**: API-008（自愈）
- **依赖模块**: M-002（消费错误）, M-008（驱动引擎）
- **内部关键组件**: ErrorClassifier(技术性 vs 业务决策；**判定依据：经 API-006 返回的 frozenContractSetId 直接读 E-004 FrozenContractSet，将修复后的 Port 签名/Entity schema 与冻结版本做结构比较——若有净变化(增删字段/改类型)→视为改变已确认蓝图语义→上浮；否则→纯技术性→静默自愈**；[ASSUMPTION] 精确比较算法待 dev 阶段细化)、RetryController(上限 + 计数)、SelfHealDriver、EscalationEmitter(→「需你决策」卡片)、RepairProgressReporter

### M-008: 执行引擎抽象层 Engine Abstraction
- **实现需求**: M-008 → prd#§2.F-018；M-008 → prd#§2.F-012
- **职责**: 蓝图/门禁/调和与具体执行引擎解耦；以 ACP（JSON-RPC/stdio）为标准接缝，Claude Code / Codex 经 adapter 接入，门禁映射为 ACP 权限中介 + 编辑后检查；原生回退 Agent SDK / 无头 CLI；切换对上层透明（固定蓝图+任务下产出/门禁判定/地图最终态一致）；预留托管执行器接入点；以工具面（MCP Server / hooks / CLI）把 Keel 核心能力暴露给执行引擎消费（F-018 AC-006）。
- **职责边界**: F-012 仅承接其中「引擎检测/连接/冒烟」部分，初始化主流程在 M-014。
- **托管执行器预留（F-018 AC-005）**: EngineKind 枚举预留 `hosted` variant（API-009 `engineKind`），v1 为空实现（不接 SaaS 托管），但透明切换契约（F-018 AC-004）已覆盖该 variant，tech-lead 设计 adapter 接口时须留此扩展位。
- **对外接口**: API-009（统一执行引擎抽象）, API-010（工具面 MCP/hooks/CLI）
- **依赖模块**: M-002（工具面暴露门禁）, M-001（工具面暴露蓝图读取）, M-009（工具面暴露地图状态）, M-003（工具面暴露漂移报告）
- **内部关键组件**: AcpAdapter、AgentSdkAdapter、HeadlessCliAdapter、PermissionMediator(门禁映射)、ToolSurface(MCP Server/hooks/CLI)、EngineCapabilityProbe(检测/冒烟)

### M-009: 能力地图渲染器 Capability Map
- **实现需求**: M-009 → prd#§2.F-006
- **职责**: 100% 从蓝图 `plain_*` 字段渲染（不从代码逆推）；七原语全量翻译为业务语言（功能区块/记住了什么/当…就…/某物的一生/安全规则）；叠加健康状态（绿/黄/红，红色可点进问题详情与修复入口）；只读（结构变更一律走 M-005 意图流程，无直接编辑入口）；大图层级聚合 + 任务焦点子图；渲染 ≤1s。
- **对外接口**: API-011（蓝图→地图视图模型）
- **依赖模块**: M-001（蓝图投影）, M-002/M-003（门禁/漂移结果 → 健康状态，经 E-005 HealthSnapshot 缓存读取；健康叠加的权威来源，**非 M-011**）, M-011（仅取意图节点状态：规划中/实现中/已就绪，及修复入口跳转）
- **数据来源澄清**: HealthOverlay 读 E-005（由 M-002/M-003 写入），节点 status（规划/实现/就绪）来自 M-011/M-006 进度；二者分属不同数据来源，M-011 不拥有也不写 HealthSnapshot。
- **内部关键组件**: MapProjector(IR business 层→视图模型，仅 surfaces_on_map)、PlainRenderer、HealthOverlay(区分结构违规 red 与回归破坏 red，对应 API-002 regressionReport)、HierarchyAggregator(折叠/焦点子图)

### M-010: 应用预览 App Preview
- **实现需求**: M-010 → prd#§2.F-008
- **职责**: 主工作区提供目标应用实时运行界面预览，可直接交互；门禁与回归守护通过后自动刷新到最新（无需手动）；预览不可用（骨架未生成/构建中）时展示通俗状态说明，不出现技术错误页。
- **对外接口**: API-012（预览控制/状态）
- **依赖模块**: M-006（实现完成事件触发刷新）, M-015（同屏挂载）
- **内部关键组件**: PreviewServer(目标应用 dev server 代理)、AutoRefreshTrigger、PreviewStateView(降级文案)

### M-011: 意图日志与时光机 Intent Log & Time Machine
- **实现需求**: M-011 → prd#§2.F-013
- **职责**: 每次变更自动记录意图日志条目（原始意图文本、蓝图 diff、时间戳；「需你决策」决策同样留痕）；提供非程序员可读的自然语言演化时间线（不暴露 commit 哈希/git 命令/代码 diff）；时光机一键回滚到任意时间点（蓝图+代码+健康灯同步恢复）；回滚前若检测到时间线未记录的通道外改动，先通俗提示并要求确认（不静默丢弃）。
- **对外接口**: API-013（日志记录/时间线查询/回滚）
- **依赖模块**: M-001（蓝图快照）, M-003（通道外改动检测）, M-008（驱动引擎执行回滚的代码恢复）
- **内部关键组件**: IntentLogStore(条目持久化，映射 git commit [ASSUMPTION])、TimelineView(自然语言)、RollbackEngine(蓝图+代码状态恢复)、UntrackedChangeGuard

### M-012: 上线与安全体检 Deploy & Security Audit
- **实现需求**: M-012 → prd#§2.F-014
- **职责**: 一键上线入口：安全体检通过后自动构建并部署到锁定托管环境，返回可访问地址（不暴露终端/部署配置）；安全体检三类红绿灯（认证配置 / 数据权限 / 密钥泄露，最小基线见主卷 §3.2）+ 安全分；红灯阻断上线并逐项给通俗说明与修复入口（可触发调和）；上线动作记入意图日志、可回滚。
- **对外接口**: API-014（上线 + 安全体检）
- **依赖模块**: M-002（复用检查器框架跑安全检查器）, M-011（上线留痕）, M-008（部署经引擎/CI）
- **内部关键组件**: SecurityAuditor(三类检查器：authn-config / data-authz / secret-scan)、SecurityScoreCard(红绿灯+分)、DeployDriver(锁定托管)、DeployGate(红灯阻断)

### M-013: 导出与交接 Export & Handoff
- **实现需求**: M-013 → prd#§2.F-015；M-013 → prd#§2.F-016
- **职责**: 随时导出结构服从蓝图的标准 JS/TS 仓库，含编译落地的标准工具配置（dependency-cruiser / eslint / tsconfig / JSON Schema），脱离 Keel 仍可用标准工具自我强制结构；导出物无 Keel 专有运行时依赖、无凭据/内部地址硬编码；一键生成开发者交接包（蓝图→架构文档 + 意图日志→决策史 + 边界规则→上手指南，Markdown，独立可读，可含技术视图投影）。
- **对外接口**: API-015（导出 + 交接包）
- **依赖模块**: M-001（蓝图→工具配置编译 + 架构文档）, M-011（意图日志→决策史）, M-002（规则集→工具配置）
- **内部关键组件**: RepoExporter、ToolConfigCompiler(蓝图 constraints→dependency-cruiser/eslint 配置)、SecretStripper、HandoffPackager(架构文档/决策史/上手指南 + 技术视图投影)

### M-014: 项目初始化 Project Init
- **实现需求**: M-014 → prd#§2.F-012
- **职责**: 检测本地执行引擎（Claude Code/Codex 桌面应用/CLI/ACP adapter）就绪性，未装则图形安装引导（无终端命令）；鉴权优先应用订阅登录 OAuth，兼容 API Key，本地存储最小权限；经 ACP/SDK/CLI 后台连接 + 冒烟校验；用户描述意图或选模板 → 自动选定 `modular-monolith` 原型 → 生成初始蓝图与最小可运行骨架（地图可完整渲染）；展示执行引擎侧订阅/用量（仅消耗透明度）。
- **对外接口**: API-016（初始化/引擎检测/鉴权/骨架生成）
- **依赖模块**: M-008（引擎检测/连接）, M-001（生成初始蓝图）, M-006（生成骨架）, M-009（首次渲染）
- **内部关键组件**: EngineInstaller(图形引导)、AuthBroker(OAuth/API Key 本地存储)、ArchetypeSelector(自动选 modular-monolith)、ScaffoldGenerator、UsageProbe

### M-015: 工作区外壳 Workspace Shell
- **实现需求**: M-015 → prd#§2.F-008
- **职责**: 主工作区同屏提供对话、能力地图、应用预览三面（用户可完全只用对话面完成构建，地图与预览为增强理解）；Web 壳承载（v1），保留外壳可替换接缝（后续 Tauri/Electron 不影响引擎核心）；工作区初始加载 ≤2s。三面布局/响应式细节由 ui-spec 定义。
- **职责边界**: 与 M-009/M-010 的关系为「承载容器 vs 面内容」；M-015 不重复地图/预览逻辑。
- **对外接口**: 无独立对外 API（前端外壳，消费 M-009/M-010/M-005 接口）
- **依赖模块**: M-009（地图面）, M-010（预览面）, M-005（对话→意图）
- **内部关键组件**: WorkspaceLayout(三面)、ChatPanel、ShellAdapter(Web↔桌面可替换接缝)
