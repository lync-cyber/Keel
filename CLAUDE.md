# CLAUDE.md · Keel 项目进度跟踪

> 本文件是 Keel 项目的唯一进度真相源，供团队与 Claude 共同维护。
> 负责人：Alex　|　最近更新：2026-06-10

---

## 一、项目是什么

**Keel（龙骨）** —— 让非程序员零代码造出「可持续成长且不腐化」的真实应用。
骑在 Claude Code / Codex 之上，用**应用蓝图**作唯一真相源，用**确定性门禁**强制结构不漂移，代码全程归用户、可随时导出迁走。

竞争定位：对 Lovable/Bolt（起步快但成长期腐化）主打**不腐化**；对 Floot/Mocha（闭环锁定）主打**开放可迁出**；对 Spec Kit/Kiro（要求会写 spec、读 diff）主打**非程序员可用**。

**核心未验证假设 R1**：蓝图能否同时做到「机器可检」与「非程序员可读」。这是产品成立的前提，也是整个项目的 **go / no-go 闸门**。强制半边（机器可检）已被认为基础扎实；**可读半边尚未验证**，是当前最大风险。

---

## 二、当前阶段

**阶段：原型已就绪，R1 可读半边验证「待执行」（尚未跑测试）。**

关键路径上的唯一阻塞：**R1 可用性测试还没真正招募用户去跑。** 所有下游工作（执行引擎、嵌入式 pivot、技术接缝预研）都 gated 在这一闸门之后。

**本轮实施（2026-06-10）：蓝图「形态层」已对齐技术报告。** 收敛到 typed-rule 元模型、补上 R-IR 需求层与 `satisfies` 追溯链、为 Port 预留行为契约占位；新增 `docs/meta-model.md` 与 `docs/decisions/` ADR。已通过 schema 形态校验 + 引用完整性 + 需求覆盖率核验。并把三张原型重新指向规范实例、PRD G3 措辞按 ADR-0009 更新。执行层（编译器、模板引擎、行为门）仍 gated 在 R1 之后。详见 ADR-0007~0010。

---

## 三、资产清单（已产出）

| 文件 | 是什么 | 状态 |
|---|---|---|
| `docs/Keel-PRD.md` | 完整 PRD：13 条 FR、4 个主流程、附录 A 状态机/线框、附录 B 默认架构与宪法、附录 C 执行器接入 | ✅ 完成 |
| `docs/blueprint.example.yaml` | **唯一规范蓝图实例**「我的博客」v9：typed-rule 元模型 + R-IR 需求层 + `satisfies` 追溯 + 契约占位。已过 schema/引用/覆盖率核验 | ✅ 完成 |
| `docs/blueprint.schema.json` | 蓝图元模型机器门（Draft 2020-12，v0.2）：7 原语 + 类型化 rule DSL + 检查器映射 | ✅ 完成 |
| `docs/meta-model.md` | 原语规格（人工审查主入口）：逐原语字段 / plain 规则 / 编译目标 | ✅ 完成 |
| `docs/decisions/` | ADR 决策记录（D1–D6 + 本轮 0007–0010），一决策一文件 | ✅ 完成 |
| `demo/blueprint.yaml` | ⚠ 已弃用（收敛前字符串规则版 v7）；暂留供原型历史参照，见 ADR-0007 | 🗄 弃用 |
| `prototype-A-capability-map.html` | 原型 A · 能力地图（验证 H1：读懂有哪些能力、怎么连） | ✅ 自包含可渲染 |
| `prototype-B-diff-review.html` | 原型 B · 蓝图 diff 确认（验证 H2：读懂新增什么、影响谁、不动什么） | ✅ 自包含可渲染 |
| `prototype-C-health-warning.html` | 原型 C · 健康预警（验证 H3：读懂是什么结构问题、怎么修） | ✅ 自包含可渲染 |
| `usability-test-plan.md` | R1 可读半边可用性测试方案：5–8 人、A→B→C 出声思考、含甄别问卷与 go/no-go 阈值 | ✅ 写完，**待执行** |
| `embedded-market-analysis.html` | 嵌入式垂直方向竞品/市场分析（草案 v1，2026-06-08）：建议「集成执行层 + 自做治理可读」，滩头选 ESP32 | 📋 草案 v1 |
| `.claude/launch.json` | 本地起静态服务跑原型：`python -m http.server 4178` | ✅ |

**起原型预览**：在仓库根目录运行 `python -m http.server 4178`，浏览器开 `http://localhost:4178/prototype-A-capability-map.html`（B、C 同理）。

---

## 四、下一步（按优先级，关键路径在最前）

### P0 — 跑 R1 可用性测试（go/no-go 闸门，唯一阻塞项）
方案 `usability-test-plan.md` 可直接执行，原型已就绪。子步骤：
1. **定测试对象**：✅ 已定（D2，2026-06-09）= 用**现有 web/博客原型**测。三张原型已就绪、零额外平移工作，对核心可读假设最快证伪；嵌入式留作 R1 通过后的补测（见 P1）。
2. **招募 5–8 名 Persona A**（重来者）：用 §3.3 现成甄别问卷。招募有前置周期，**应尽早启动**。
3. **执行主持式出声思考**：固定顺序 A→B→C，逐题按通过标准计分，只有「无协助通过」才计通过。
4. **对照闸门判定**：任务①②③全部达通过线 = R1 可读半边成立（go）；任一命中重做线 = 对应呈现重做。任务①命中最严重（②③都建立在读懂地图之上）。

### P1 — 待 R1 通过后才解锁
- 执行引擎接入（ACP / Agent SDK），FR-1、FR-5、FR-13。
- 「蓝图 → 标准工具配置（dependency-cruiser/eslint/tsconfig）」编译落地，坐实可迁出。
- 若走嵌入式：技术接缝预研「蓝图 → Zephyr devicetree/Kconfig + PlatformIO」。

### P2 — 治理/打磨
- 测试中暴露的高频误解 → 定向修订原型呈现。
- ✅ PRD G3 措辞更新（2026-06-10，ADR-0009）：改为「无 oracle→人工；有 oracle→远期设行为门」。
- ✅ 三张原型重新指向唯一规范实例 `docs/blueprint.example.yaml`（2026-06-10，ADR-0007）；新增 `plain_detail` 并入 Unit 原语承载详情面板文本，layout/diff/health 标注为视图层。
- 远期：原型内嵌 JSON 由「手工投影」升级为「从规范实例自动生成」（ADR-0010 的视图生成方向）。

---

## 五、决策日志（2026-06-09 拍板）

| 编号 | 决策 | 结论 | 说明 |
|---|---|---|---|
| D1 | 项目负责人 | ✅ **Alex** | 由 Alex 对 go/no-go 拍板并负责路线（PRD 原 `负责人 TBD` 已同步更新）。 |
| D2 | 测试方向：web vs 嵌入式 | ✅ **先测现有 web 原型** | 三张原型已就绪、零平移工作，对核心可读假设最快最省证伪；嵌入式留作 R1 通过后补测。 |
| D3 | v1 架构原型集合 | ✅ **只上 1 个默认** | 仅「特性切片/模块化单体」，由意图自动选中；先把闸门跑通再扩。 |
| D4 | 门禁严重度分级 | ✅ **结构=硬，风格=软** | 破依赖边界/重复实现/契约不符=硬阻断；命名/目录/轻微冗余=软告警。沿用 PRD 附录 B.7。 |
| D5 | 无头 token 池定价模型 | ⏸ **暂缓到 R1 通过后** | 后期项，闸门未过不投入定价。 |
| D6 | 正式产品命名 | ✅ **就用 Keel** | 龙骨隐喻贴合「不腐化承重结构」，已全程使用，不改。 |
| D7 | 蓝图收敛（2026-06-10） | ✅ **收敛到 typed-rule 元模型** | demo 字符串规则版弃用，唯一规范实例 `docs/blueprint.example.yaml`。详见 ADR-0007。 |
| D8 | R-IR 时机（2026-06-10） | ✅ **近期加 R-IR + 追溯链** | 蓝图补 `requirements` + `satisfies`，并入 R1 测试。详见 ADR-0008。 |
| D9 | 行为门（2026-06-10） | ✅ **远期纳入，现预留契约占位** | `Port.contract` 占位；PRD G3 措辞待改。详见 ADR-0009。 |
| D10 | 文件形态（2026-06-10） | ✅ **Markdown 规格 + YAML 实例 + ADR + Mermaid** | 不上 CUE/Pkl。详见 ADR-0010。 |

> 决策细节统一落在 `docs/decisions/`（ADR），本表只留摘要。

---

## 六、约定

- **唯一真相源**：`docs/blueprint.example.yaml`（规范实例，符合 `docs/blueprint.schema.json`；`demo/blueprint.yaml` 已弃用）。原型界面只渲染 `plain_*` 字段，技术字段一律不上界面。任何「界面内容」改动须先核对是否 100% 来自蓝图、未从代码逆推。
- **不腐化原则**：本项目自身的文档也遵循「单一真相源、不重复」——进度只记在本文件，避免散落多处。
- 文档语言：中文。

## 文档导航

- 导航索引: `docs/.doc-index.json`（机器索引，所有 Agent 通过 `cataforge docs load` 查询；缺失时运行 `cataforge docs index` 重建）
- 通用规则: .claude/rules/COMMON-RULES.md
- 子代理协议: .claude/rules/SUB-AGENT-PROTOCOLS.md
- 编排协议: .cataforge/agents/orchestrator/ORCHESTRATOR-PROTOCOLS.md (orchestrator专属)
- 状态码Schema: .cataforge/schemas/agent-result.schema.json
- 加载原则: 按任务需要通过 `cataforge docs load` 加载相关章节，不全量加载

## 框架机制

- Agent编排: orchestrator 通过 agent-dispatch skill 激活子代理
- DEV阶段: orchestrator 通过 tdd-engine skill 编排 RED/GREEN/REFACTOR 三个子代理（独立上下文）
- Skill调用: Agent按SKILL.md步骤式指令执行工作流
- 状态持久化: 项目指令文件（CLAUDE.md/AGENTS.md）§项目状态 + docs/ 目录
- 子代理通信: 通过文件系统(docs/和src/)传递产出物路径
- 运行时: 由 framework.json runtime.platform 决定（deploy 自动适配）
- **写权限**: 项目指令文件 §项目状态 由 orchestrator 独占写入；其他Agent只写 docs/ 或 src/ 下的产出文件
- 统一配置 `.cataforge/framework.json`:
  - `upgrade.source` — 远程升级源配置。升级时保留用户已配置值，仅补充新字段
  - `upgrade.state` — 本地升级状态。升级时始终保留
  - `features` — 功能注册表。升级时全量覆盖
  - `migration_checks` — 迁移检查声明。升级时全量覆盖
