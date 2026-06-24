# Keel

## 项目信息

- 技术栈: 原型阶段 — 静态 HTML 原型 + YAML 蓝图实例 + JSON Schema 门禁；执行引擎技术栈待 architecture 阶段选型
- 运行时: claude-code
- 框架版本: 0.14.0
  <!-- 由 cataforge deploy 自动盖入已安装包版本。SemVer: MAJOR=不兼容变更, MINOR=新功能, PATCH=修复 -->
- 语言定位: 中文框架（提示词/文档/交互用中文；代码/变量/CLI参数用英文）
- 执行模式: standard
  <!-- 可选值: standard | agile-lite | agile-prototype。矩阵见 COMMON-RULES §执行模式矩阵。模式切换由 orchestrator §Mode Routing Protocol 路由 -->
- 阶段配置:
  - ui_design: 保留（蓝图可视化界面是核心交付物，原型 A/B/C 即 UI 雏形）
  - testing: 保留
  - deployment: 保留
- model 继承: AGENT.md 中 `model: inherit` 继承父会话模型；可用 `model: <model-id>` 覆盖

- 项目定位: Keel（龙骨）— 让非程序员零代码构建「可持续成长且不腐化」的应用：应用蓝图作唯一真相源 + 确定性门禁防结构漂移，代码全程归用户、可随时导出迁走
- 项目负责人: Alex（对 go/no-go 拍板并负责路线）

## 执行环境 (Bootstrap 时由 `cataforge setup env-block` 填入)

<!-- 本节在 Bootstrap 步骤中生成。每次会话都会作为项目指令加载，
     权重高于 hook 注入的 additionalContext。项目生命周期内保持稳定。 -->
- 无自动检测到的标准包管理器（请根据实际技术栈手动填写）
- Shell: 默认使用 Git Bash（POSIX 语法，Claude Code 中走 Bash 工具），不使用 PowerShell；脚本与命令示例一律按 bash 语法书写
- 原型预览: 仓库根目录运行 `python -m http.server 4178`，浏览器开 `http://localhost:4178/demo/prototype-A-capability-map.html`（B、C 同理）

## 项目状态 (orchestrator专属写入区，其他Agent禁止修改)

- 当前阶段: ui_design
- 上次完成: architect — arch-keel 4 卷（main/modules/api/data）approved（doc-review r2，r1 的 2 HIGH+8 MED+3 LOW 全部 RESOLVED）；技术栈锁定 Next.js16+Postgres+BetterAuth+Drizzle、Keel 引擎 TS 单栈、ACP-first 执行引擎；M-001~M-015 覆盖全 F-001~F-018。两处 KG 阶段过严门（doc-review has_test 覆盖、doc-consistency ac-traceability）经 reviewer-calibration 驳回/降级，已归档 docs/feedback/
- 下一步行动: ui_design 进行中（inline 承载 ui-designer）。设计方向+Token 见 docs/research/ui-design-direction-keel-20260623.md（沉静海图/暖调蓝图，藏青#2b3f72+暖纸#f1ece1+三声部排版；全量覆盖 F-001~F-018；ui-spec 五卷 main/theme/components/pages-core/pages-lifecycle）。**Penpot 设计系统已建成并验证**（2026-06-24，文件「Keel」）：6 集 54 design token + library 21 colors/8 typographies + 核心组件（能力节点已 componentize、健康pill/ghost按钮/diff chips/原语徽章），「Design System」「Components」两页导出渲染确认。**整屏 mockup 折叠进 ui-spec**：Penpot MCP 深层嵌套整屏叶子内容不持久/不渲染（用户 2026-06-24 确认调整），三面工作区等屏由 ui-spec pages 卷文档精确定义、引用设计系统+静态原型A/B/C。**ui-spec 进度**：theme 卷已授权注册（docs/ui-spec/ui-spec-keel-theme.md，validate OK；hybrid 授权流程=Write docs/→`cataforge context ingest`→`index`，非 write-doc）。**剩**：main/components/pages-core/pages-lifecycle 四卷 → task #6 reviewer doc-review 门禁。Penpot 动笔前须验文件身份、见记忆 penpot-mcp-setup
- 已完成阶段: [requirements, architecture]
- 当前Sprint: —
- 文档状态:
  - prd: approved
  - arch: approved
  - ui-spec: 未开始
  - dev-plan: 未开始
  - test-report: 未开始
  - deploy-spec: 未开始
  <!-- changelog 由 devops 产出但不纳入门禁追踪 -->
- Learnings Registry: (compacted; archive in .cataforge/learnings/registry-archive.md)
  <!-- 上限：framework.json#claude_md_limits.learnings_registry_max_entries；超限运行 `cataforge claude-md compact` -->

## 文档导航

- 导航索引: `docs/.doc-index.json`（机器索引，所有 Agent 通过 `cataforge context read` 查询；缺失时运行 `cataforge context index` 重建）
- 通用规则: .claude/rules/COMMON-RULES.md
- 子代理协议: .claude/rules/SUB-AGENT-PROTOCOLS.md
- 编排协议: .cataforge/agents/orchestrator/ORCHESTRATOR-PROTOCOLS.md (orchestrator专属)
- 状态码Schema: .cataforge/schemas/agent-result.schema.json
- 加载原则: 按章节/条目粒度按需通过 `cataforge context read` 加载，不全量加载

## 全局约定

- 命名: 文件/文档小写 kebab-case；代码命名规范待 architecture 阶段技术栈定型后补充
- Commit: Conventional Commits（feat:/fix:/docs: 等前缀）
- 分支: 主干（main）直推，无 PR 流程
- 设计工具: penpot
  <!-- 由 cataforge deploy 从 framework.json#project.design_tool 盖入。切换用 `cataforge setup --with-penpot`，勿手改本行 -->
  <!-- 可选值: none | penpot。penpot 时启用 Penpot MCP 集成 -->

- 人工审查检查点: [pre_dev, pre_deploy]
  <!-- 详见 COMMON-RULES §MANUAL_REVIEW_CHECKPOINTS -->
- 文档类型命名: 小写 kebab-case（prd、arch、dev-plan、test-report、ui-spec、deploy-spec…），含工具参数和产出文件名
- 效率原则:
  - 最小传递: Agent间传递doc_id#section引用，非全文
  - 不确定时调研: 调用research skill，不猜测
  - 选择题优先: 需要用户输入时优先提供选项
  - 长文拆分: 文档超 `DOC_SPLIT_THRESHOLD_LINES` 行时按doc-gen拆分策略分卷
- 业务约定（Keel 特有）:
  - 唯一真相源: `docs/blueprint.example.yaml`（规范蓝图实例，符合 `docs/blueprint.schema.json`；`demo/blueprint.yaml` 已弃用）
  - 原型界面只渲染 `plain_*` 字段，技术字段一律不上界面；「界面内容」改动须核对 100% 来自蓝图、未从代码逆推
  - 文档语言: 中文

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
