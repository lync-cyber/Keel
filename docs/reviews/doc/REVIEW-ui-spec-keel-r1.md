---
id: "review-ui-spec-keel-r1"
doc_type: review
author: reviewer
status: approved
deps: ["ui-spec-keel", "ui-spec-keel-theme", "ui-spec-keel-components", "ui-spec-keel-pages-core", "ui-spec-keel-pages-lifecycle"]
---

# 审查报告：ui-spec-keel（五卷）r1

**审查对象**：ui-spec 全量五卷（main / theme / components / pages-core / pages-lifecycle）
**审查阶段**：ui_design 质量门禁（Phase 3 → dev-plan 前置）
**上游比对源**：prd-keel-f001-f009、prd-keel-f010-f018、arch-keel（§1.4）、arch-keel-modules（M-005/M-009/M-010/M-014/M-015）、arch-keel-api（API-002/005/011/012/013/014/016）、research-ui-design-direction-keel（§0/§1）

---

## Layer 1 结果摘要

| 卷 | 结果 | 说明 |
|----|------|------|
| ui-spec-keel（main） | FAIL | 色彩 Token 定义表缺失（主卷委托给 theme 分卷，脚本未识别跨卷委托；见 R-001） |
| ui-spec-keel-theme | PASS | — |
| ui-spec-keel-components | PASS | — |
| ui-spec-keel-pages-core | PASS | — |
| ui-spec-keel-pages-lifecycle | PASS（1 WARN） | P-002/P-004/P-005 编号不连续（跨卷拆分正常，不视为缺陷） |

---

## 问题列表

### [R-001] HIGH: 主卷 §6 原语徽章数量与 components 卷自相矛盾

- **category**: consistency
- **root_cause**: self-caused
- **描述**: `ui-spec-keel`（main）§6 覆盖核验注释写「UC-004 原语徽章**六类**」，而 `ui-spec-keel-components` UC-004 明确定义「**七类**业务译名」，且 Props 使用 `enum(seven)`。两处数字不一致，会让 tech-lead/developer 在实现原语徽章时产生歧义（六种还是七种）。
- **建议**: 主卷 §6 覆盖核验注释中将「六类」统一改为「七类」，与 components 卷 UC-004 保持一致。

---

### [R-002] HIGH: UC-004 第七类用「自由实现区」替换了 Dependency/Relation，与 PRD F-001 AC-001 七原语定义不符

- **category**: consistency
- **root_cause**: self-caused
- **描述**: PRD F-001 AC-001 的七个原语为：Module/Unit、Interface/Port、Entity/DataModel、**Dependency/Relation（含允许/禁止规则）**、Workflow/Sequence、StateMachine、Constraint。UC-004 的七类为：功能区块/记住了什么/当…就…/某物的一生/安全规则/对外接口/自由实现区。其中「自由实现区（escape hatch）」并非蓝图原语，而是 F-017 定义的模块修饰符标注（`escape_hatch: true`）；而 **Dependency/Relation 原语没有对应的业务语言译名和徽章变体**。F-006 AC-003 明确要求「七原语**全量**翻译为业务语言」，其中 Dependency/Relation 的通俗译名（如「模块连接规则」）在能力地图图例说明层面缺失。UC-003 RelationEdge 可视化了关系连线，但没有相应的 PrimitiveBadge 变体覆盖 Dependency/Relation 原语的图例标注。
- **建议**: 在 UC-004 中补充 Dependency/Relation 的业务语言变体（例如「模块连接规则」）；将 EscapeHatch 从原语徽章变体中剥离（或保留作为 UC-002 节点的修饰叠加标识而非独立 PrimitiveBadge kind），并在组件卷说明两者的区别。同步修正 props `kind: enum(seven)` 的枚举内容。

---

### [R-003] MEDIUM: `color.muted` 对比度数值在 theme 卷（4.8:1）与 research 卷（4.6:1）不一致

- **category**: consistency
- **root_cause**: self-caused
- **描述**: `ui-spec-keel-theme` §4.1.1 与 §4.1.4 均写 muted 对 paper ≈4.8:1；`research-ui-design-direction-keel` §1.1 及注释写 ≈4.6:1。theme 卷是单一事实源，但与上游 research 卷数值出现分歧（同一色值 #6F6657 / #6f6657 对 #F1ECE1，实测对比度唯一）。任一开发者校对 WCAG AA 时会得到冲突数字。
- **建议**: 用工具（如 Stark/Contrast Checker）实测 `#6F6657` 对 `#F1ECE1` 的精确对比度，在 theme 卷更新为精确值并标注来源；research 卷为历史调研文档，theme 卷定稿后无需同步更新 research。

---

### [R-004] MEDIUM: UC-003（RelationEdge）Props 引用「API-011 MapEdge」，但 API-011 仅定义 MapNode 类型，MapEdge 类型字段缺失

- **category**: consistency
- **root_cause**: upstream-caused
- **描述**: `ui-spec-keel-components` UC-003 Props 注释写「对应 API-011 MapEdge」，但 `arch-keel-api` API-011 的 `types:` 段只定义了 `MapNode` 结构，`MapEdge` 虽然出现在 response schema `{ nodes: MapNode[], edges: MapEdge[] }` 中，但无类型定义（字段名/类型均未声明）。UC-003 使用的 `{ from, to, relationType, diffState?, highlighted? }` 字段无法在 arch 侧验证是否符合契约。
- **建议**: 在 arch-keel-api API-011 的 `types:` 段补充 `MapEdge: "{ id, from, to, relationType, diffState?: enum(...), highlighted?: boolean }"` 定义；此问题根因在 arch 卷，ui-spec 可在 UC-003 Props 下追加 `[ASSUMPTION]` 标注说明待 arch 补全。

---

### [R-005] MEDIUM: UC-011（SecurityChecklistCard）Props 字段名与 API-014 返回结构不吻合

- **category**: consistency
- **root_cause**: self-caused
- **描述**: `ui-spec-keel-components` UC-011 Props 定义为 `{ score, items: [{category, state, plainDetail, onFix}], canDeploy }`；而 `arch-keel-api` API-014 返回 `{ score, checks: [{category, light: enum(green|red), plain, fixEntry?}] }`。具体差异：
  - `items` vs `checks`（字段名不同）
  - `state` vs `light`（字段名不同，且 UC-011 的 state 类型未声明枚举值）
  - `plainDetail` vs `plain`（字段名不同）
  - `onFix: function` vs `fixEntry?: string`（语义不同：UI 层回调 vs API 层入口地址）
  - `canDeploy: boolean` 在 API-014 中无对应字段（需前端从 checks 推导）

  实现者若直接按 UC-011 Props 对接 API-014 响应会遇到字段名映射错误。
- **建议**: 统一字段命名。建议 UC-011 Props 对齐 API-014 字段名（`checks` / `light` / `plain`），前端推导 `canDeploy = checks.every(c => c.light === 'green')`；或在 UC-011 Props 旁注明「字段需经适配层映射自 API-014」。

---

### [R-006] MEDIUM: UC-012（TimelineEntry）Props 中的 `kind` 字段在 API-013 timeline 返回结构中没有对应

- **category**: consistency
- **root_cause**: upstream-caused
- **描述**: `ui-spec-keel-components` UC-012 有四种变体（change / decision / deploy / current）并在 Props 中声明 `kind` 字段；`arch-keel-api` API-013 `timeline` 操作返回 `{ entries: {entryId, plainSummary, timestamp}[] }`，无 `kind` 字段。实现者无法从 API-013 响应中确定 entry 类型（变更/决策/上线）以渲染不同变体。
- **建议**: 在 API-013 timeline entries 返回结构中补充 `entryKind: enum(change|decision|deploy)` 字段；或在 UC-012 Props 下注明 kind 需从其他字段（如 entry 内容）推断，并说明推断规则。此根因在 arch 卷，ui-spec 需追加 `[ASSUMPTION]` 标注。

---

### [R-007] MEDIUM: UC-020（UsageMeter）和 P-007 引用 API-016 提供用量数据，但 API-016 无 usageMeter 操作

- **category**: completeness
- **root_cause**: upstream-caused
- **描述**: `ui-spec-keel-components` UC-020 注释「数据由引擎侧接口提供，API-016」；`ui-spec-keel-pages-lifecycle` P-007 引用「顶栏 UC-020 显引擎侧订阅/用量（F-012 AC-007）」。但 `arch-keel-api` API-016 只定义了三个操作（`scaffold`、`detectEngine`、`authenticate`），无任何 usage/subscription 查询操作。M-014 内部有 `UsageProbe` 组件，但其契约未暴露到 API-016。UC-020 Props `{ planLabel, used, quota, source }` 所需数据无对应 API 契约。
- **建议**: 在 arch-keel-api API-016（或单独 API 条目）补充 `usageStatus` 操作及返回结构；ui-spec 的 UC-020 Props 引用更新到对应 API 操作。此根因在 arch 卷，ui-spec 需追加 `[ASSUMPTION]` 标注。

---

### [R-008] MEDIUM: Layer 1 FAIL — 主卷色彩 Token 定义表缺失未通过脚本检查，需解决脚本误报问题

- **category**: convention
- **root_cause**: self-caused
- **描述**: `cataforge skill run doc-review -- ui-spec ui-spec-keel.md` 返回 FAIL：「设计系统缺少色彩 Token 定义表（standard 模式至少需要 5 个 Token）」。主卷在 §1 明确委托到 `ui-spec-keel-theme` 分卷（「完整 token 见 ui-spec-keel-theme（THEME-01）」），是合理的跨卷拆分设计，Layer 1 脚本未识别此模式。虽然问题在脚本，但 Layer 1 FAIL 状态会阻塞 CI 流程。
- **建议**: 在主卷 frontmatter 添加 `token_volume: ui-spec-keel-theme` 或等效字段，让 Layer 1 脚本识别跨卷委托并跳过该检查；或联系 framework 更新 doc-review 脚本增加分卷委托识别逻辑。短期解决方案：在主卷 §1 添加 Layer 1 可识别的最小 token 列表占位（如 5 个主色引用），指向 theme 卷定义。

---

### [R-009] LOW: F-009 AC-009（每次变更绑定意图日志条目）在 ui-spec 无显式 UI 映射标注

- **category**: completeness
- **root_cause**: self-caused
- **描述**: PRD F-009 AC-009 要求「每次变更绑定意图日志条目（含原始意图文本与对应蓝图 diff），支持后续追溯（F-013）」。P-003 和 UC-007 覆盖了 diff 确认流程，P-008 覆盖了意图日志显示（F-013）。但没有组件或页面显式标注「F-009 AC-009」映射——意图绑定日志的动作触发点（diff 确认后的日志写入）在 UI 层无明确标注。此为后端自动行为，但 UI 层应在 P-003 或 UC-007 交互说明中说明「确认后触发日志写入（F-009 AC-009）」以确保完整性追溯。
- **建议**: 在 P-003 布局描述或 UC-007 交互说明中追加「确认操作同时触发意图日志条目写入（F-009 AC-009）」的说明。

---

### [R-010] LOW: F-006 AC-003 PRD 自身七原语枚举仅列出五个（缺 Port 和 Dependency/Relation 译名），需标注上游原语计数口径澄清

- **category**: ambiguity
- **root_cause**: upstream-caused
- **描述**: PRD F-006 AC-003 说「七原语**全量**翻译为业务语言」，但例举仅给出 5 个（Module/Entity/Workflow/StateMachine/Constraint），未给 Port（对外接口）和 Dependency/Relation 的通俗译名。ui-spec UC-004 七类中包含 Port（对外接口）但用 EscapeHatch 替代了 Dependency/Relation（详见 R-002）。PRD 的原语枚举不完整使 ui-spec 实现者难以确认七类的完整映射。
- **建议**: 在 prd-keel F-006 AC-003 的例举中补全 Port 的通俗名及 Dependency/Relation 的通俗名（如「模块连接规则」）；或在 ui-spec-keel components UC-004 旁添加注释说明与 F-001 AC-001 的对应关系（考虑 escape_hatch 是修饰符而非独立原语）。

---

## Verdict

**needs_revision**

存在 2 个 HIGH 问题（R-001、R-002）：
- R-001：main 卷与 components 卷原语徽章数量（六类 vs 七类）内部矛盾，会导致实现时出现歧义。
- R-002：UC-004 七原语业务译名中用 EscapeHatch 替代了 F-001 AC-001 规定的 Dependency/Relation 原语，致 F-006 AC-003「七原语全量翻译」验收条件未满足，且 Dependency/Relation 在整个 ui-spec 中无业务语言对应。

MEDIUM 问题（R-003 到 R-008）均为字段对齐和跨文档一致性问题，部分根因在 arch 卷（R-004、R-006、R-007），修复时可用 `[ASSUMPTION]` 标注桥接，不需要 arch 先行完成；R-005、R-003、R-008 为 ui-spec 自身可直接修复项。

LOW 问题（R-009、R-010）均为补充说明类，不阻塞实现。

**修复优先级**：R-001 和 R-002（HIGH）为 revision 必须处理项；R-005（UC-011 Props vs API-014 字段名对齐）建议同步修复避免前后端实现分歧。
