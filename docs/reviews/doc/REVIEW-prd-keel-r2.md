---
id: "review-prd-keel-r2"
doc_type: review
author: reviewer
status: approved
deps: ["prd-keel"]
---
# REVIEW: PRD Keel r2（增量复审）

**被审文档**: prd-keel（主卷 prd-keel.md + 功能分卷一 prd-keel-f001-f008.md + 功能分卷二 prd-keel-f009-f016.md）
**审查日期**: 2026-06-11
**审查模式**: 增量复审（仅审查上轮 CRITICAL/HIGH 闭环状态 + 本轮修订变更部分全维度；其余维度标注 [previously-approved]）
**上轮报告**: REVIEW-prd-keel-r1

---

## Layer 1 结果摘要

| 文件 | Layer 1 结果 | 说明 |
|------|-------------|------|
| `prd-keel.md`（主卷） | PASS（WARN: ID 编号跨卷缺口，预期） | 上轮 FAIL（无 AC-NNN）已修复；AC-001/AC-002 占位引用通过检查器 |
| `prd-keel-f001-f008.md`（分卷一） | PASS | 无变化，延续上轮 PASS |
| `prd-keel-f009-f016.md`（分卷二） | PASS（WARN: 跨卷 ID 缺口，预期） | 新增 F-017 后 PASS；ID 缺口为跨卷预期行为 |

所有三卷均通过 Layer 1（exit 0）。WARN 条目均为多卷拆分结构下跨卷 ID 缺口，非真实错误。

---

## 上轮 HIGH 问题闭环核查

### R-001（HIGH）→ 已闭环

**修复内容核实**：主卷 `## 2. 功能需求` 段新增了两条占位引用：
```
- [ ] AC-001: （见 prd-keel-f001-f008 §2，F-001..F-008 各功能验收标准）
- [ ] AC-002: （见 prd-keel-f009-f016 §2，F-009..F-017 各功能验收标准）
```
并在引导语后追加了 `volume_delegation` 说明注释（"本卷为多卷 PRD 主卷，所有 AC 条目完整定义在功能分卷中（volume_delegation）"）。

**Layer 1 验证结果**：主卷由上轮 FAIL (exit 1) 转为本轮 **PASS**，R-001 正式闭环。

**NAV 一致性**：主卷 `[NAV]` 已同步更新，§2 引用改为"prd-keel-f009-f016（F-009..F-017）"，与 F-017 新增一致。

**结论**：完全闭环。

---

### R-002（HIGH）→ 已闭环（核实通过）

**修复内容核实**：主卷 §1.1「能力边界」段落现已包含：
> 「对可写出可执行判据（oracle: test/property）的约束，远期纳入行为门（契约派生单测/属性测试）；当前阶段只做字段预留（ADR-0009）。」

**对照 ADR-0009 核实**：ADR-0009 决定措辞为「**无 oracle → 人工；有 oracle → 远期设行为门**」，要求 PRD 措辞从「行为一律交人工」更新为体现「有 oracle → 远期设行为门」的正向承诺。当前 PRD §1.1 已完整呈现：
- 「无 oracle → 人工」：原有句「无可执行判据（oracle）的约束（产品意图、交互体验等）回退人工批准锚定」已保留。
- 「有 oracle → 远期设行为门」：新增句完整表达了正向承诺，并注明「当前阶段只做字段预留（ADR-0009）」与 ADR 后果一节一致。

**F-001 AC-006 一致性**：F-001 AC-006 写道「为远期行为门预留占位；当前只校验格式合法性，不执行行为测试」，与 §1.1 措辞一致。

**结论**：完全闭环，与 ADR-0009 一致。

---

## 本轮修订变更全维度审查

### R-003（MEDIUM）→ 已闭环

**修复内容核实**：F-003 AC-006 已改写为：
> 「门禁通过统一的结构化接口触发，检查结果为机器可解析的 JSON 格式，支持 LLM 读取结构化错误后自动重试；具体接口协议待架构阶段确定（[ASSUMPTION]）。」

原文中的 `cataforge skill run` 框架工具引用已移除，改为意图描述性语言，不再混入产品需求层。**结论**：完全闭环。

---

### R-004（MEDIUM）→ 已闭环（新增 F-017）

**修复内容核实**：分卷二新增 F-017 Escape Hatch（自由实现区），含 5 条 AC，优先级 P1。

**全维度审查（F-017 为新增内容，按完整维度审查）**：

**completeness（完整性）**：
- F-017 用户故事清晰描述了目标用户（「作为用户」）、场景（性能调优、复杂算法、UI 细节等图原语无法表达场景）与期望行为（标注自由实现区、附加意图说明、放宽图约束）。
- AC-001 ~ AC-005 覆盖：标注方式（AC-001）、门禁豁免范围（AC-002）、地图可见性（AC-003）、不可传播约束（AC-004）、导出物保留（AC-005）。边界条件覆盖完整。
- **新问题 [R2-001]**：AC-002 声明门禁豁免「依赖边界检查与架构 lint 检查，仅保留接口签名校验（Port 契约合法性）」，但未说明 `Constraint.severity: hard` 规则对 escape hatch 模块的处理策略——如果模块同时被打了 `escape_hatch: true` 且有显式 hard Constraint，该 Constraint 是否也被豁免？两者优先级关系不明，架构师无法据此设计门禁逻辑。见下方问题清单。

**consistency（一致性）**：
- F-017 备注引用了 `prd-ref.md §3`，与 r1 报告 R-004 建议方向一致。
- AC-002 说「跳过依赖边界检查与架构 lint 检查，仅保留接口签名校验」——与 F-003 AC-001 的三道门（编译门、契约测试门、架构 lint 门）对应关系是：跳过的是架构 lint 门，保留的是契约测试门中的 Port 接口签名部分；编译门（TS 类型检查）未明确说明是否跳过。这是潜在的歧义点，见下方 [R2-002]。
- 主卷 [NAV] 与分卷二 [NAV] 均已更新为包含 F-017，一致性良好。
- 术语表 `escape hatch` 条目与 F-017 内容一致（术语表定义"标注为'自由实现区'的模块，允许附加自然语言意图说明，绕过严格图原语表达"）。

**ambiguity（清晰度）**：
- AC-004「不可传播：escape hatch 模块的依赖方仍受完整图约束」表述清晰，边界明确。
- AC-001 中「intent 为空时系统拒绝接受该标注」的判定方式（编译期拒绝还是运行时拒绝）未指明，但这是架构阶段实现细节，PRD 层留[ASSUMPTION]可接受。

**feasibility（可行性）**：
- 「仅保留接口签名校验」的逻辑在已有 F-003 门禁框架下有实现路径（条件跳过），可行。
- AC-004「不可传播」约束需要门禁系统支持模块级标记传递语义，有一定实现复杂度，但未超出可行范围，备注已有[ASSUMPTION]兜底。

**convention（规范性）**：
- F-017 格式（用户故事、验收标准、优先级、备注）与其他 F-xxx 条目一致。
- AC 编号 AC-001 ~ AC-005 连续无缺口。
- 分卷二标题行已从"F-009..F-016"更新为"F-009..F-017"，分卷二 [NAV] 也已加入 F-017。

**security（安全性）**：
- 备注提及「滥用风险（将所有模块标为自由区）由产品层面的意图说明必填约束加以抑制」，且提及可追踪 escape hatch 占比作为架构健康信号，安全意识到位。

---

### R-005（MEDIUM）→ 已闭环

**修复内容核实**：
- `prd-keel-f001-f008.md` front matter：`deps: ["prd-keel"]` ✓
- `prd-keel-f009-f016.md` front matter：`deps: ["prd-keel"]` ✓

两分卷 deps 均已从 `[]` 改为 `["prd-keel"]`，文档依赖链完整。**结论**：完全闭环。

---

### R-006（MEDIUM）→ 已闭环

**修复内容核实**：F-010 AC-001 已追加执行模型分层说明：
> 「执行模型分层：F-003 负责本次改动的结构合规门（单点检查）；F-010 在 F-003 通过后追加跨能力回归校验（累积检查），两者串行执行，结果统一上报到能力地图健康状态。」

该说明完整定义了：F-003 与 F-010 的职责边界（单点 vs. 累积）、执行顺序（串行）、结果汇报路径（统一上报到能力地图健康状态）。架构师现可据此划分职责边界。**结论**：完全闭环。

---

## 本轮新增问题

### [R2-001] MEDIUM: F-017 AC-002 未定义 escape_hatch 与显式 hard Constraint 的优先级

- **category**: ambiguity
- **root_cause**: self-caused（新增功能条目在边界场景的交互语义未完整定义）
- **描述**: F-017 AC-002 声明 escape hatch 模块门禁「跳过依赖边界检查与架构 lint 检查，仅保留接口签名校验」。但 F-001 AC-005 定义了 `Constraint.severity: hard` 触发硬阻断。当一个模块同时具有 `escape_hatch: true` 与显式 `Constraint`（尤其是 hard Constraint）时，两者的优先级未定义：escape hatch 豁免是否也豁免该模块上的 hard Constraint？这将影响安全类 Constraint（如密钥隔离、权限边界）在 escape hatch 模块上的执行语义。架构师无法在没有 PRD 指引的情况下做出正确的设计决策。
- **建议**: 在 F-017 AC-002 或备注中补充交互语义说明，例如：「escape hatch 豁免仅适用于图结构类检查（依赖边界、架构 lint）；`Constraint.severity: hard` 的约束（含安全类规则）不受豁免，仍强制执行」；如果设计意图相反（escape hatch 也豁免 hard Constraint），则需在此处明确并在备注中说明风险。

---

### [R2-002] LOW: F-017 AC-002 未明确编译门（TS 类型检查）对 escape hatch 模块是否适用

- **category**: ambiguity
- **root_cause**: self-caused（门禁豁免范围描述遗漏了 F-003 三道门中的编译门）
- **描述**: F-003 AC-001 定义了三道门：编译门（TS 类型检查）、契约测试门（Port 接口一致性）、架构 lint 门（依赖方向、分层、重复）。F-017 AC-002 仅说「跳过依赖边界检查与架构 lint 检查，仅保留接口签名校验」，未提及编译门。从语义推断编译门应保留（类型安全与图约束无关，escape hatch 不应豁免 TS 编译），但 PRD 层面对此未明确，存在歧义。
- **建议**: 在 F-017 AC-002 末尾补充一句「编译门（TS 类型检查）仍完整适用于 escape hatch 模块」，消除歧义；或在备注中说明「三道门中仅架构 lint 门被豁免，编译门与接口签名校验保留」。

---

## [previously-approved] 维度

以下维度在上轮 REVIEW-prd-keel-r1 中无 CRITICAL/HIGH 问题，本轮未发生相关变更，标注 [previously-approved] 不重复审查：

| 维度 | 上轮结论 | 本轮状态 |
|------|---------|---------|
| feasibility（可行性） | 无重大问题 | [previously-approved](REVIEW-prd-keel-r1) |
| security（安全性） | §3.2 覆盖完整 | [previously-approved](REVIEW-prd-keel-r1) |
| 非功能需求 §3 整体 | 无问题 | [previously-approved](REVIEW-prd-keel-r1) |
| F-001..F-009、F-011..F-016 各功能条目 | 除 R-003/R-006 均无 CRITICAL/HIGH | [previously-approved](REVIEW-prd-keel-r1)；R-003/R-006 已本轮闭环 |

**上轮 LOW 问题保留说明**：
- R-007（LOW：F-004/F-005 P0 占比）：本轮未修复，问题仍成立，保留为 LOW 备注。
- R-008（LOW：F-013 AC-003 阈值位置）：本轮未修复，问题仍成立，保留为 LOW 备注。

两条 LOW 均不影响本轮判定。

---

## 综合判定

**上轮 HIGH × 2 均已闭环。本轮新增问题均为 MEDIUM/LOW，无 CRITICAL/HIGH。**

| 级别 | 数量 | 条目 |
|------|------|------|
| CRITICAL | 0 | — |
| HIGH | 0 | — |
| MEDIUM | 1 | R2-001（F-017 AC-002 与 hard Constraint 优先级未定义）|
| LOW | 3 | R2-002（F-017 编译门豁免范围未明确）、R-007（保留）、R-008（保留）|

**verdict**: approved_with_notes

**notes_summary**:
1. R2-001（MEDIUM）：F-017 escape hatch 豁免语义与 hard Constraint 的交互关系未定义，建议在进入架构阶段前在 PRD 中明确，以免架构师误解安全类约束是否受豁免。
2. R2-002（LOW）：编译门是否对 escape hatch 适用未明确，一句补充即可消除歧义。
3. R-007 / R-008（LOW，沿用上轮）：P0 占比与 F-013 AC-003 阈值位置问题，产品团队可在适当时机自行评估是否处理。
