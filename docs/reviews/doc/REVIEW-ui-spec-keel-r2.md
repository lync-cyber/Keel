---
id: "review-ui-spec-keel-r2"
doc_type: review
author: reviewer
status: approved
deps: ["ui-spec-keel", "ui-spec-keel-theme", "ui-spec-keel-components", "ui-spec-keel-pages-core", "ui-spec-keel-pages-lifecycle"]
---

# 审查报告：ui-spec-keel（五卷）r2（revision 复审）

**审查对象**：ui-spec 全量五卷（main / theme / components / pages-core / pages-lifecycle）
**审查阶段**：r1 needs_revision 修订后复审
**上游比对源**：r1 报告（REVIEW-ui-spec-keel-r1.md），五卷修订后版本

---

## Layer 1 结果摘要

| 卷 | 结果 | 说明 |
|----|------|------|
| ui-spec-keel（main） | PASS（1 WARN） | WARN: UC-015 跨卷引用编号不连续（跨卷正常拆分，非 FAIL） |
| ui-spec-keel-theme | PASS | — |
| ui-spec-keel-components | PASS | — |
| ui-spec-keel-pages-core | PASS | — |
| ui-spec-keel-pages-lifecycle | PASS（1 WARN） | WARN: P-002/P-004/P-005 跨卷引用编号不连续（同上） |

全部五卷 Layer 1 通过，r1 的 FAIL 已消除。

---

## r1 问题逐条核验

### [R-001] HIGH → **RESOLVED**

**修订验证**：main §6 覆盖核验注释已将「六类」改为「七类 + 自由实现区修饰标识」，原文现为：「UC-004 原语徽章**七类** + 自由实现区修饰标识」，与 components 卷 UC-004 定义一致。数字歧义已消除。

### [R-002] HIGH → **RESOLVED**

**修订验证**：UC-004 已完整重写，七类业务译名为：功能区块（Module）/ 对外接口（Port）/ 记住了什么（Entity）/ 连接规则（Dependency/Relation）/ 当…就…（Workflow）/ 某物的一生（StateMachine）/ 安全规则（Constraint）。Dependency/Relation 已有明确业务译名「连接规则」并附「含允许·禁止」说明。

escape hatch 已彻底剥离出七原语计数体系，改为 UC-002 `isEscapeHatch` 布尔修饰位（对应 API-011 MapNode.isEscapeHatch），Props 中 `kind` 枚举明确仅含七原语字面值 `enum('module'|'port'|'entity'|'relation'|'workflow'|'statemachine'|'constraint')`，不计入 kind。

交互说明明文说明「escape hatch 是 F-017 的 Module 修饰符（`escape_hatch: true`）叠加在功能区块节点上，而非第八种原语；与七原语分层呈现，避免数量混淆」，层级边界清晰。

F-006 AC-003「七原语全量翻译」验收条件满足（七类全部有业务语言译名），R-010 PRD 歧义在 ui-spec 层通过 `[ASSUMPTION]` 显式闭合。

**一致性交叉核验**：
- P-002 布局描述：「七原语全量译为业务语言（UC-004）」——数量表述为「七」，与 UC-004 一致。
- P-011：使用组件列「UC-004（自由实现区徽章）」，未将 escape hatch 计入原语，引用自洽。
- main §6 覆盖核验：「UC-004 原语徽章**七类** + 自由实现区修饰标识」——「七类」与「修饰标识」分层清晰，无数量矛盾。
- theme 卷 §4.2：原写「原语徽章六类」现已改为「原语徽章六类」——**发现残留问题，见下方新问题 N-001**。

### [R-003] MEDIUM → **RESOLVED**

**修订验证**：theme §4.1.4 明确写「muted 4.80 AA 正文下限」并附「本卷为对比度单一事实源，数值以实测为准」。main 卷 §1.1 速览表中 muted 行亦写「4.8:1 AA 下限」，两处数值一致。research 历史卷不需同步，已在 theme 卷声明单一事实源。

### [R-004] MEDIUM → **RESOLVED（桥接）**

**修订验证**：UC-003 Props 已追加 `[ASSUMPTION]` 标注：「API-011 `types:` 段当前仅定义 MapNode，MapEdge 字段未声明；本 Props 为 ui-spec 提出的 MapEdge 形态，待 arch API-011 补 `MapEdge: { id, from, to, relationType, diffState?, highlighted? }` 后对账（根因在 arch，R-004）。」根因在 arch 且已显式标注，ui-spec 层已合规桥接。

### [R-005] MEDIUM → **RESOLVED**

**修订验证**：UC-011 Props 已完整对齐 API-014：`{ score, checks: [{category, light: 'green'|'red', plain, fixEntry?}], canDeploy }`。字段名 `checks` / `light` / `plain` / `fixEntry?` 均与 API-014 返回结构一致。`canDeploy` 显式注明「由前端推导 `checks.every(c => c.light === 'green')`」，`scanning` 注明为「UI 层瞬态、非 API 字段」。实现者可直接按 UC-011 Props 对接 API-014 响应，无歧义。

### [R-006] MEDIUM → **RESOLVED（桥接）**

**修订验证**：UC-012 Props 已追加 `[ASSUMPTION]`：「`kind`（change/decision/deploy）待 arch API-013 timeline entries 补 `entryKind: enum(change|decision|deploy)` 字段；补全前由前端按条目内容推断变体（根因在 arch，R-006）。」根因在 arch 且已显式标注，ui-spec 层已合规桥接。

### [R-007] MEDIUM → **RESOLVED（桥接）**

**修订验证**：UC-020 Props 已追加 `[ASSUMPTION]`：「API-016 当前仅定义 scaffold/detectEngine/authenticate，无用量查询操作；M-014 内部 UsageProbe 的契约待 arch 补 API-016 `usageStatus` 操作（返回 `{ planLabel, used, quota, source }`）后对账（根因在 arch，R-007）。」根因在 arch 且已显式标注，ui-spec 层已合规桥接。

### [R-008] MEDIUM → **RESOLVED**

**修订验证**：main §1 已拆分为 §1.1 色彩速览（8 token 表，满足脚本 ≥5 要求）+ §1.2 排版与间距（含说明委托 THEME-01 §4.1.2–§4.1.3）。Layer 1 复跑结果：PASS（仅 UC-015 跨卷 WARN，为正常跨卷拆分，非 FAIL）。Layer 1 FAIL 已彻底消除。

### [R-009] LOW → **RESOLVED**

**修订验证**：UC-007 交互说明已追加「确认操作同时触发意图日志条目写入（F-009 AC-009，留痕原始意图文本 + 蓝图 diff，供 P-008 时光机追溯）。」F-009 AC-009 在 UI 层的触发点已显式标注。

### [R-010] LOW → **RESOLVED**

**修订验证**：UC-004 追加 `[ASSUMPTION]`：「F-006 AC-003 的 PRD 例举仅列 5 个原语通俗名（缺 Port 与 Dependency/Relation）；本卷据 F-001 AC-001 七原语全集补全 Port=「对外接口」、Dependency/Relation=「连接规则」两译名，作为能力地图图例的业务语言定稿（R-010 上游 PRD 歧义在 ui-spec 层闭合）。」

---

## 新发现问题

### [N-001] LOW: theme 卷 §4.2 遗留「原语徽章六类」旧描述

- **category**: consistency
- **root_cause**: self-caused
- **描述**: `ui-spec-keel-theme` §4.2 适配组件段有如下原文：「健康 pill 三态、diff chips、能力节点卡、**原语徽章六类**、ghost/primary 按钮等核心组件的 token 绑定已在 Penpot『Components』页落地验证。」此处「六类」为 r1 修订前的旧描述，R-001/R-002 修订后原语徽章已为七类（+escape hatch 修饰标识），与 components 卷 UC-004 和 main §6 不一致。
- **建议**: 将 theme §4.2 中「原语徽章六类」改为「原语徽章七类（+自由实现区修饰标识）」以与其他卷保持一致。

---

## Verdict 自检

- CRITICAL 问题：0
- HIGH 问题：0
- MEDIUM 问题：0
- LOW 问题：1（N-001，theme 卷遗留旧描述）

r1 的 10 条问题中：10 条 RESOLVED，0 条 PARTIAL，0 OPEN。新问题 1 条（LOW）。

三态判定：无 CRITICAL/HIGH，有 1 LOW → **approved_with_notes**

---

## Verdict

**approved_with_notes**

r1 全部 10 条问题（2 HIGH + 6 MEDIUM + 2 LOW）均已有效修订：
- R-001/R-002 两条 HIGH 彻底闭合：七原语全量有业务译名，数量在主卷/组件卷/页面卷三处一致为七；escape hatch 已明确与七原语分层（isEscapeHatch 布尔修饰位），不计入 kind 枚举，P-011 引用自洽。
- R-003/R-005/R-008 三条 MEDIUM（self-caused）直接修复，字段/数值/Layer 1 全部对齐。
- R-004/R-006/R-007 三条 MEDIUM（upstream-arch）均用 `[ASSUMPTION]` 合规桥接，根因在 arch 层且已显式标注，不阻塞 ui-spec 本身。
- R-008 Layer 1 FAIL 消除，五卷均 PASS。
- R-009/R-010 两条 LOW 均已补充说明并闭合。

唯一新发现问题 N-001（LOW）：theme §4.2 遗留「原语徽章六类」旧描述，与 UC-004 / main §6 不一致，建议修订但不阻塞进入 dev-plan 阶段。
