---
id: "review-prd-keel-r8"
doc_type: review
author: reviewer
status: approved
deps: ["prd-keel", "prd-keel-f001-f009", "prd-keel-f010-f018"]
---

# REVIEW: PRD Keel — 资深 PM + 需求分析专家全量审计与修复（r8）

**被审文档**:
- docs/prd/prd-keel.md（主卷，审计时 v2.5 → 修复后 v2.6）
- docs/prd/prd-keel-f001-f009.md（分卷 1，v2.5 → v2.6）
- docs/prd/prd-keel-f010-f018.md（分卷 2，v2.5 → v2.6）
- docs/blueprint.schema.json（KG 元模型，0.2-draft → 0.3-draft）
- docs/blueprint.example.yaml（KG 规范实例，v9 → v10）

**上轮报告**: REVIEW-prd-keel-r7.md（amendment 增量审查；verdict approved_with_notes，R-001 MEDIUM 已于 v2.5 正文修复，本轮不再重提）

**审查模式**: 全量审计（research/doc-lookup 通读 3 卷 PRD + KG 元模型 + KG 实例 + r1–r7 审查史；research/web-search 取 ACP 成熟度与"非程序员读架构图"两条参照基准；req-analysis 六维拆解）。**本轮非纯审查：所有 CRITICAL/HIGH 已在同一变更集内修复**，故 verdict=approved。

**用户决策（4 项，经 AskUserQuestion 确认）**:
1. P-01 → schema 补 Workflow/StateMachine 最小占位 `$def`（保留七原语）
2. P-05 → 不引入 MVP 分版，仅在 §4.3 澄清 R1 验证集
3. P-09 → F-014 安全清单归 architecture 阶段，§3.2 升格为可测 NFR
4. 范围 → PRD 三卷 + KG 两文件全修 + 落盘报告，提交至 `claude/prd-kg-analysis-g18dj5`

---

## 总体评估

PRD 成熟度高：ADR 可追溯、`[ASSUMPTION]` 纪律严明、P0 分级有论证、`oracle` 诚实边界、KG 元模型与 PRD 同步共设。常规缺陷（AC 主观化、漏写 NFR、写实现细节）基本不存在。短板集中在三处非常规处并已修复：(1) PRD↔KG 元模型漂移（schema 缺 PRD 承诺的两个原语）；(2) KG 规范实例两处实打实缺陷（核心约束静默失效 / 约束语义错位）；(3) 头号价值"不腐化"无成功指标、MVP 语义混淆。

**参照基准（web-search）**：ACP 已 Linux Foundation 治理、2026-03 有 25+ agent 支持、stdio 本地稳定 → F-018 押注合理且有 SDK 回退兜底；"非程序员读架构图"几无实证先例（研究多面向开发者/学生）→ 反向确认 R1 是真正高风险假设，§1.3 验证设计需收紧（已在 §4.3 澄清最小验证集）。

---

## 问题清单与修复对账

| # | 位置 | 类型 | 问题（原文定位） | 严重度 | 修复（resolution） |
|---|------|------|------|--------|------|
| P-01 | F-001 AC-001 × schema $defs × F-006 AC-003 | consistency | PRD 七原语含 Workflow/StateMachine，`blueprint.schema.json` 完全无此二原语 | 高 | schema 增 `$defs.workflow`/`$defs.statemachine` + top-level `workflows[]`/`statemachines[]` 最小占位（0.3-draft）；F-001 备注对齐；实例新增 `wf-read-flow`/`sm-article-life` 示范 |
| P-02 | blueprint.example.yaml `data-via-core` | feasibility | sealed/hard 约束 `target: {kind: store}` 匹配不到任何 unit→静默失效 | 高 | `unit_kinds` 增 `store`，新增 `db`(kind store) 单元 + r12/r13；jsonschema 校验通过，约束实际绑定（已验证）|
| P-03 | F-011 × F-017 × ADR-0009 | consistency | 整片重生成会丢失 escape-hatch 手工区/未入图行为，无保护 AC | 高 | F-011 增 AC-007（重生成前内容保护+用户确认）、AC-008（行为等价回归校验）|
| P-04 | §1.3 × F-002/F-003 | measurability | "不腐化"无纵向指标；门禁指标混淆拦截(block)/检测(detect) | 高 | §1.3 拆为通道内拦截率 100% + 全量检测率 100% + 新增"结构不腐化"纵向指标 |
| P-05 | §4.3 × ADR-0002 × P0 集合 | priority | MVP 语义混淆（R1 静态原型验证 vs P0 全量构建环）| 高 | §4.3 澄清 R1 最小验证集=F-001/006/009 可读层，与全量 P0 解耦（不引入 MVP 分版）|
| P-06 | blueprint.example.yaml `naming-convention` | convention | 约束 plain（命名）与 rule（no_orphan）语义不符，checker 错配 | 中 | 改名 `no-orphan-feature`、plain 与 `checker: madge` 对齐 no_orphan 语义 |
| P-07 | F-001 AC-004 | ambiguity | "覆盖率 100%"=结构链接率，被误读为验收通过 | 中 | AC-004 增口径说明：覆盖率=结构 satisfies 链接，≠ acceptance 已自动验证 |
| P-08 | F-002 AC-002 | ambiguity | "契约不符"未限定 v1 仅 shape 级 | 中 | AC-002 限定为 Port 签名/Schema 形态层；行为契约漂移远期 |
| P-09 | F-014 备注 × 阶段配置 | completeness | 引用不存在的"安全规范阶段" | 中 | §3.2 升格三类安全项为带红灯判据+验收的 NFR；F-014 备注改归 architecture 阶段 |
| P-10 | F-005 × §3.1 | completeness | "正在修复…"无时长上限/中止入口 | 中 | F-005 增 AC-005（可见进度+可中止）；§3.1 增自愈等待 NFR 行 |
| P-11 | F-002 备注 × req-no-corrupt-a1 | completeness | single_canonical 仅声明 auth，重复实现检测覆盖面窄 | 中 | F-002 备注补两条互补路径（single_canonical + ts-morph 通用扫描）|
| P-12 | F-018 AC-004 | ambiguity | "切换透明"无可测定义 | 中 | AC-004 增可测判据（固定任务下三输出一致）|
| P-13 | F-007 AC-003 | consistency | "漏标率 0"绝对值与降级（有环/缺失）冲突 | 低 | AC-003 限定"完整无环图"时为 0；降级须显式提示不完整 |
| P-14 | §3.1 | measurability | 含 LLM 的 diff 目标无依据/未标 provisional | 低 | diff 行加 [ASSUMPTION]+依赖条件；漂移行补计时口径 |
| P-15 | F-013 × F-002 AC-006 | completeness | 回滚 vs 通道外编辑交互未定义 | 低 | F-013 增 AC-006（回滚前提示未留痕改动并确认）|
| P-16 | §6 术语表 × 计数口径 | consistency | 原语计数口径两处不一 | 低 | §6 增"原语集口径"条目；图 IR 双层语义补全 |

---

## 验证

- `blueprint.example.yaml` 经 jsonschema 校验通过 schema 0.3-draft；`data-via-core` 约束确认实际绑定 `store` 单元。
- `cataforge context ingest`（16 entities / 19 sections）→ `index`（25 docs）→ `validate`（0 orphans/stale/xref/alias/id）→ `reconcile`（no drift）→ `doctor`（all checks passed）。

---

## 审查结论

**verdict**: approved

5 条 HIGH（P-01~P-05）、7 条 MEDIUM、4 条 LOW 全部在本变更集内修复并经机器校验。PRD↔KG 漂移消除、KG 实例两处缺陷修复并验证、核心价值与 R1 验证的可衡量性补齐。无遗留 CRITICAL/HIGH，可进入 architecture 阶段。

**遗留待 architecture 阶段承接的 [ASSUMPTION]**（非阻塞）：Workflow/StateMachine 完整语义、ts-morph 相似度阈值、三类安全检查器规则集、自愈等待阈值调参、diff 生成性能实测校准。
