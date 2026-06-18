---
id: "review-arch-keel-r1"
doc_type: review
author: reviewer
status: approved
deps: ["arch-keel", "arch-keel-modules", "arch-keel-api", "arch-keel-data"]
---
# 架构文档审查报告: arch-keel r1

**被审对象**: arch-keel（主卷）、arch-keel-modules、arch-keel-api、arch-keel-data（version 0.1, status: draft）
**上游参考**: prd-keel v2.6（approved）、research-arch-tech-eval-stack（approved）、ADR-0003/0007/0008/0009/0011

---

## 问题列表

<!-- 维度1: completeness -->

### [R-001] HIGH: F-004 回归守护缺乏独立模块承接

- **category**: completeness
- **root_cause**: self-caused
- **描述**: 主卷 §2 模块总览表中未列出任何模块明确承接 F-004（回归守护）。M-002 职责描述提到「回归守护（F-004）= F-003 通过后追加的跨能力累积检查」，但 arch-keel-modules 分卷 M-002 的「实现需求」行仅列 `M-002 → prd#§2.F-003; F-004; F-017`，并未解释 F-004 在 M-002 内如何实现。PRD F-004 AC-001 明确要求回归守护「随能力增加累积行为检查，每次改动后运行」，且 AC-002 要求新改动导致既有能力检查失败时「硬阻断代码落地，并优先交由 F-005 自愈」，这涉及一套独立的跨能力检查编排逻辑。当前 M-002 的内部组件列表（RuleCompiler / CheckerAdapter / CheckerOrchestrator / GateResultAggregator / EscapeHatchResolver）无任何组件对应「跨能力累积检查」或「回归守护检查集管理」，实现路径不明确，tech-lead 无从据此设计。
- **建议**: 在 M-002 职责中显式说明回归守护的实现机制（例：在 GateResultAggregator 内维护能力级历史检查集，F-003 单点通过后追加跨能力回归检查），或新增 RegressionGuard 组件，并在 API-002 response 中增加 `regressionReport` 字段以区分当次结构门与回归守护结果。

---

### [R-002] HIGH: F-004 AC-001 要求的「F-003 与 F-004 串行、结果统一上报」在接口层未体现

- **category**: completeness
- **root_cause**: self-caused
- **描述**: PRD F-004 AC-001 明确说明「F-003 负责本次改动的结构合规门；F-004 在 F-003 通过后追加跨能力回归校验，两者串行执行，结果统一上报到能力地图健康状态」。当前 API-002 的 `GateReport` 结构为 `{ passed, hardViolations, softWarnings, durationMs }`，没有任何字段区分「本次结构门结果」与「跨能力回归守护结果」，也没有体现串行的两阶段语义。上游 M-009（HealthSnapshot）和 E-005 中的 `health` 字段只有 green/amber/red，无法区分是结构门触发的 red 还是回归守护触发的 red，导致能力地图健康状态的信息粒度不足（PRD F-004 AC-003 要求「被破坏的节点变红并带说明」，需区分来源）。
- **建议**: 在 `GateReport` 中增加 `regressionReport?: { capability: string, violations: GateError[] }[]` 字段或独立 `RegressionReport` 类型，使 M-009 能据此在地图节点上区分「结构违规」与「回归破坏」两类 red 状态并给出不同说明。

---

### [R-003] MEDIUM: §5.1 漂移增量 ≤3s 方案未覆盖通道外改动的发现延迟说明

- **category**: completeness
- **root_cause**: self-caused
- **描述**: 主卷 §5.1 性能方案中写道「漂移增量 ≤3s（F-002 AC-005）：检查器按改动文件集增量运行」，这描述的是检测执行时长。PRD F-002 AC-005 明确说明「本指标仅约束检测执行时长；通道外改动从发生到被系统感知的发现延迟不计入，由 AC-006 的发现时机选型决定」。arch-keel-modules M-003 的 ChangeWatcher 组件标注了 `[ASSUMPTION]`（「发现时机待细化，不削弱来源无关语义」），但主卷 §5.1 未同步体现此限制——读者会以为「通道外也能 ≤3s 感知」，实际 ≤3s 只保证检测本身，发现延迟未承诺。
- **建议**: 在 §5.1 漂移检测段尾补充说明：「≤3s 仅约束从系统开始执行检测到完成报告的时长；通道外改动的发现延迟（文件监听触发 vs 下次操作扫描）以 M-003 ChangeWatcher 的 [ASSUMPTION] 为准，不计入该指标」，与 PRD F-002 AC-005 口径对齐。

---

### [R-004] MEDIUM: F-018 AC-005「预留托管执行器接入点」在模块架构中无任何体现

- **category**: completeness
- **root_cause**: self-caused
- **描述**: PRD F-018 AC-005 要求「预留托管执行器接入点（为日后 SaaS 托管形态预留，当前不实现）」。M-008 的 `EngineCapabilityProbe` 和三个 Adapter 组件（AcpAdapter / AgentSdkAdapter / HeadlessCliAdapter）没有提到此预留接入点，API-009 的 operations 也无对应设计。虽然预留不需完整实现，但 tech-lead 在设计 M-008 接口时需知道「何处留空以接托管执行器」——例如 AgentAdapter 是否应有 `HostedAgent` variant，API-009 是否需要 `engineKind: enum(acp|sdk|cli|hosted)` 字段。当前架构文档完全不提此预留，tech-lead 可能遗漏设计点。
- **建议**: 在 M-008 职责或 API-009 notes 中补充「F-018 AC-005 托管执行器接入点预留：EngineKind 枚举增加 `hosted` variant，当前为空实现；接口切换透明性（F-018 AC-004）覆盖此 variant」。

---

<!-- 维度2: consistency -->

### [R-005] HIGH: M-009 依赖 M-011「健康/意图状态来源」与数据模型不一致

- **category**: consistency
- **root_cause**: self-caused
- **描述**: arch-keel-modules 中 M-009 的依赖写「M-011（健康/意图状态来源）」，但 arch-keel-data §4 中健康状态快照的权威实体是 E-005（HealthSnapshot），其 `projectId + nodeId + health` 归属 M-002 门禁/漂移的输出，由 M-002 和 M-003 驱动写入；意图状态来自 E-002（IntentLogEntry），归属 M-011。M-009 的 `HealthOverlay` 组件若从 M-011 读取健康数据，则绕过了 M-002/M-003 门禁对 HealthSnapshot 的写入路径，与 E-005 的持久化机制相悖——M-011 不拥有也不直接写 HealthSnapshot，M-009 应依赖 M-002/M-003 的运行结果（经 E-005 缓存），而非 M-011。模块分卷中 M-011 的职责（意图日志/时间线/回滚）与健康状态无直接关系。
- **建议**: 修正 M-009 的依赖列表，将「健康/意图状态来源」拆分为：`M-001`（蓝图投影）、`M-002/M-003`（门禁/漂移结果→健康状态，经 E-005 缓存读取）；M-011 依赖可保留但仅用于「意图节点状态（规划中/实现中/就绪）来源」，并在 M-009 描述中澄清健康叠加与意图状态的不同数据来源。

---

### [R-006] MEDIUM: ADR-0009 要求「行为门远期，v1 只预留字段」，但 F-004「回归守护」的描述与行为门语义存在混同风险

- **category**: consistency
- **root_cause**: upstream-caused
- **描述**: ADR-0009 决定「行为门远期纳入，近期只做字段预留」。PRD F-004 AC-001 说「随能力增加累积行为检查」，AC-002 说「修复方案涉及业务取舍……则以通俗语言上浮」，PRD F-004 备注也明确「回归守护的行为检查在 v1 基于契约测试与结构检查，不包含端到端行为测试」。然而 arch-keel 对 F-004 的模块实现未加任何「v1 仅结构门，不含行为测试」的明确约定。arch-keel-modules M-002 职责边界仅说「回归守护（F-004）= F-003 通过后追加的跨能力累积检查」，但未限定该检查为「结构+契约测试，不含行为测试」。tech-lead 读此描述有可能误认为需要实现端到端行为测试，与 ADR-0009 和 PRD F-004 备注产生歧义。
- **建议**: 在 M-002 职责边界或 §1.2 架构风格的 ADR 引用处，增加一行约束：「F-004 v1 回归守护仅含结构门与契约测试（Port 签名/Entity schema 一致性），行为门（契约派生单测/属性测试）远期纳入（ADR-0009），不在 v1 范围内。」

---

### [R-007] MEDIUM: API-012 `getPreview` 请求参数 `{}` 写法不规范且语义不清

- **category**: consistency
- **root_cause**: self-caused
- **描述**: API-012 主操作 `getPreview` 的 request 字段写为 `{}: { type: void, required: false, desc: "无入参" }`，与其他 API 的 request 格式不一致（其他接口 request 描述具名字段）。同样，API-015 主操作也写了 `{}: { type: void, required: false }`。这种写法在 YAML 中是空字符串作 key（`""`），语义上不规范，会给 tech-lead 和代码生成工具带来解析歧义。
- **建议**: 无入参的操作统一写为 `request: none` 或 `request: {}（无字段）` 加注释，与有入参的写法形式区分，避免 key 为空字符串的歧义表达。

---

<!-- 维度3: feasibility -->

### [R-008] MEDIUM: M-003 ChangeWatcher 文件监听方案在 Web 壳 + 本地引擎服务形态下的可行性未说明

- **category**: feasibility
- **root_cause**: self-caused
- **描述**: M-003 的 ChangeWatcher 组件「通道外改动发现：文件监听 / 项目打开扫描」标注了 `[ASSUMPTION]`。tech-eval §4 说明「v1 先做 Web 壳，桌面外壳延后」，而 Web 壳阶段 Keel 以 Web 壳 + 本地引擎服务形态运行（参考 §5.2：「Web 壳阶段经本地引擎服务持有」）。文件系统监听（`fs.watch` / `chokidar`）在本地引擎服务（Node 进程）中可行，但架构文档未说明该监听在哪个进程中运行、如何与 Web 壳的前端通信（推测经本地引擎服务的 HTTP/WebSocket），也未说明性能含义（全项目文件监听 vs 仅 `features/` 目录）。若这一细节不在架构层确认，tech-lead 实现时可能走错路径或产生不必要的跨进程通信复杂度。
- **建议**: 在 M-003 ChangeWatcher 组件注释中补充：「文件监听运行在本地引擎服务进程（Node），事件经引擎服务→Web 壳 WebSocket 推送；监听范围建议限制到 `features/` + 蓝图文件，避免全项目监听开销。[ASSUMPTION] 具体进程边界待 dev-plan 阶段确认。」

---

### [R-009] LOW: §1.4 技术栈表中「能力地图渲染」库标注「待 ui-spec 细化」，但依赖关系中无对 React Flow 候选的风险说明

- **category**: feasibility
- **root_cause**: self-caused
- **描述**: §1.4 技术栈表中「能力地图渲染」一行标注「React Flow 候选，待 ui-spec 细化」，生命周期标为「—」。React Flow 是专注图可视化的库，其许可证（MIT）和 bundle 大小（约 200KB gzip 后）对 ≤2s 工作区加载（F-008 AC-005）有一定影响，且 React Flow v12 引入了新的 API 表面；若 ui-spec 阶段选型时未核验，可能在 dev 阶段才发现兼容性或性能问题。当前技术栈表对此无任何风险标注或 fallback 说明。
- **建议**: 在该行的「备注/选型理由」列补充「[ASSUMPTION] 待 ui-spec 阶段做最终选型确认：需核验 React Flow v12 bundle size 对 ≤2s 加载目标的影响，备选 D3.js 自绘方案（体积更小但开发成本更高）」，使 ui-spec 阶段有明确 checklist。

---

<!-- 维度4: ambiguity -->

### [R-010] MEDIUM: M-007 ErrorClassifier「是否改变已确认蓝图语义」作为不变判据，但架构层未说明判定机制

- **category**: ambiguity
- **root_cause**: self-caused
- **描述**: M-007 ErrorClassifier 的职责说「不变判据=是否改变已确认蓝图语义」，对应 PRD F-005 AC-002。但架构文档未说明「已确认蓝图语义」是什么数据载体（E-004 FrozenContractSet？还是完整蓝图 IR？），也未说明 ErrorClassifier 如何访问它（读 FrozenContractSet 的 ports/entitySchemas 字段？）。tech-lead 需要知道：①「改变蓝图语义」的具体判定是与 FrozenContractSet 做结构 diff 还是更粗粒度的语义比较；②访问路径是 M-007 直接读 E-004，还是经 M-005 暴露一个查询接口。当前架构对此完全空白。
- **建议**: 在 M-007 的 ErrorClassifier 组件说明中补充：「判定依据：与 FrozenContractSet（E-004）的 ports/entitySchemas 做结构比较；若修复后 Port 签名或 Entity schema 相对冻结版本有净变化（增删字段/改类型）则视为改变蓝图语义→上浮；否则视为纯技术性修复→静默自愈。访问路径：M-007 通过 API-006 返回的 frozenContractSetId 直接读 E-004。[ASSUMPTION] 精确比较算法待 dev 阶段细化。」

---

### [R-011] LOW: API-009 `prompt` 操作的 `ContextBundle` 类型无定义

- **category**: ambiguity
- **root_cause**: self-caused
- **描述**: API-009 operations 中有 `prompt: { in: "{ task: string, context: ContextBundle }", out: "{ result: AgentResult }" }`，但 `ContextBundle` 和 `AgentResult` 两个类型在 API 卷或数据卷中均无定义。这是执行引擎与 Keel 交互的核心上下文注入接口（F-010 要求「每模块仅注入自身契约 + 直接依赖签名」），类型不明会让 M-006 ContextInjector 的实现缺乏约束。
- **建议**: 在 API-009 的 `types` 段中至少给出 `ContextBundle` 的最小结构（如 `{ capabilityId, contract: FrozenContractSet, directDepSignatures: PortSignature[] }`）和 `AgentResult` 的骨架（`{ success, gateReport?, output? }`），或在 notes 中引用「见 M-006 ContextInjector 规格，待 dev-plan 阶段细化 [ASSUMPTION]」。

---

<!-- 维度5: convention -->

### [R-012] LOW: arch-keel-data 分卷 frontmatter 中 `required_sections` 与实际章节编号不一致

- **category**: convention
- **root_cause**: self-caused
- **描述**: arch-keel-data 的 frontmatter `required_sections` 写的是 `"## 4. 数据模型"`，但文档正文的实际 NAV 标注的是「§4 数据模型 → §4.1 实体关系, E-001..E-006」，而非独立的「## 4.」。这与 arch-keel-modules 和 arch-keel-api 的 required_sections 写法（`## 2. 模块划分` / `## 3. 接口契约`）形式上一致，但三卷 required_sections 中的节号与整体 arch 分章节号是否对齐，未通过 context index 验证会产生跳过风险。低严重等级，因为 Layer 1 Pass 已验证，但建议统一。
- **建议**: 各分卷 required_sections 保持现有格式不变（`"## 4. 数据模型"`），无需修改；但若后续 cataforge context index 出现「required section 缺失」警告，对照节标题前缀（`##` vs `###`）检查是否需要调整。此条可作 INFO 参考。

---

<!-- 维度6: security -->

### [R-013] MEDIUM: §5.2 安全方案中「Web 壳阶段经本地引擎服务持有」凭据的传输路径未说明

- **category**: security
- **root_cause**: self-caused
- **描述**: §5.2 凭据段写道「Web 壳阶段经本地引擎服务持有，桌面阶段迁系统密钥库」，但未说明 Web 壳（浏览器）如何访问本地引擎服务持有的凭据，以及本地引擎服务与浏览器之间的通信安全（localhost HTTP vs HTTPS，CORS 策略）。如果浏览器前端通过不加密的 localhost HTTP 请求触发执行引擎操作，CSRF 攻击面存在，且如果凭据需要从本地引擎服务回传给浏览器（如 OAuth Token 用于显示订阅状态），则传输安全性需要明确约定。E-006 EngineConnection 中 `authRef` 字段注释「指向系统密钥库，不存明文」，但这是在本地引擎服务侧的 SQLite 中，与浏览器层的访问控制是两个不同问题。
- **建议**: 在 §5.2 或 §5.4 配置管理中补充：「本地引擎服务与 Web 壳的通信约定：localhost HTTP（无需 TLS，因仅本机访问）+ Origin 白名单（仅允许本 Web 壳来源，防 CSRF）；凭据明文不经前端传输，前端仅持 session token / 工作区状态，凭据操作全部经本地引擎服务代理。[ASSUMPTION] 具体鉴权方案待 dev-plan 阶段设计。」

---

## 审查结论

**verdict**: needs_revision

**问题统计**：
- CRITICAL: 0
- HIGH: 2（R-001 F-004 回归守护无实现路径、R-002 GateReport 缺回归守护字段）
- MEDIUM: 8（R-003、R-004、R-005、R-006、R-007、R-008、R-010、R-013）
- LOW: 3（R-009、R-011、R-012）

**一句话结论**: 架构文档整体质量较高，技术栈选型有调研依据，NFR 方案可信，F-001~F-018 均有模块承接；存在 2 条 HIGH（F-004 回归守护无实现路径、GateReport 接口不支持分来源健康状态），须修复后方可进入 dev-plan。

---
