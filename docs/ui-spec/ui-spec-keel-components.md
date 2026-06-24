---
id: "ui-spec-keel-components"
version: "0.1"
doc_type: ui-spec
author: ui-designer
status: approved
deps: ["prd-keel", "arch-keel", "research-ui-design-direction-keel"]
consumers: [tech-lead, developer]
volume: components
volume_type: components
split_from: "ui-spec-keel"
required_sections:
  - "## 2. 组件清单"
---
# UI Specification 分卷 — 组件清单: Keel

> 所有组件按 ui-spec-keel-theme（THEME-01）token 绑定主题；凡承载「界面内容」的字段一律取蓝图 `plain_*`，技术字段（类名/文件路径/堆栈/代码 diff）一律不上界面。核心组件（UC-001/002/003/004/007/009/018）已在 Penpot 文件「Keel」「Components」页落地验证。

[NAV]
- §2 组件清单 → UC-001..UC-020
[/NAV]

## 2. 组件清单

### UC-001: 健康灯 / 健康 Pill（HealthPill）
- **变体**: green（正常）, amber（小问题）, red（要处理）；尺寸 dot（顶栏/节点角标）/ pill（带文案）。
- **视觉差异**: 三态用 `color.ok|warn|alert` 实心 + 各自 `*-soft` 软底；pill 文案 `font.mono` 11–12px；red 态缓慢呼吸动画（`motion.slow` 300ms 循环，非急促）；amber/green 静止。健康色伴随图标（✓ / ! / ▲），不单独以色承载语义。
- **Props**: `{ state: 'green'|'amber'|'red', shape: 'dot'|'pill', label?: string, onClick?: fn }`
- **映射功能**: F-003 AC-003, F-004 AC-003, F-006 AC-004
- **交互说明**: red pill 可点击 → 进入 UC-009 问题详情与修复入口；顶栏总健康灯聚合全图最差态。

### UC-002: 能力节点卡（CapabilityNode）
- **变体**: default, hover（轻抬 + `shadow.raised`）, selected（`color.keel` 描边 + `color.keel-wash` 选中环）, escape-hatch（虚线描边 + UC-004「自由实现区」徽章）；status 维度叠加 planning/implementing/ready（UC-013）。
- **视觉差异**: 实体卡片 `color.surface` 底 + `radius.md` + `shadow.rest`；标题 `plain_name`（`font.sans` 500/15px）；副行 `plain_detail`（`color.muted` 13px，截断 2 行）；左上原语徽章（UC-004），右上健康角标（UC-001 dot）；hover 抬升、selected 描边。
- **Props**: `{ id, plainName, plainDetail, kind, health, status, isEscapeHatch, selected, onSelect }`（对应 API-011 MapNode）
- **映射功能**: F-006 AC-002/AC-005, F-010 AC-005
- **交互说明**: 单击选中 → 弹 UC-005 详情卡，并作为「指图说话」(point-and-prompt) 的意图绑定目标（F-009 AC-003）；只读，无内联编辑（F-006 AC-006）。

### UC-003: 关系连线（RelationEdge）
- **变体**: normal, diff-added（`color.diff-added`=keel 虚线）, diff-touched（`color.diff-touched` 紫）, diff-unchanged（`color.diff-unchanged` 淡化）, blast-highlight（影响预览高亮）。
- **视觉差异**: 自信墨色细线（`color.ink` 1.5px），带方向箭头；diff 态按上表换色 / 虚线；blast radius 时触及边加粗高亮、安全区边降透明。
- **Props**: `{ from, to, relationType, diffState?, highlighted? }`（对应 API-011 `edges: MapEdge[]`）。**[ASSUMPTION]**: API-011 `types:` 段当前仅定义 MapNode，MapEdge 字段未声明；本 Props 为 ui-spec 提出的 MapEdge 形态，待 arch API-011 补 `MapEdge: { id, from, to, relationType, diffState?, highlighted? }` 后对账（根因在 arch，R-004）。
- **映射功能**: F-006 AC-002, F-007 AC-001
- **交互说明**: 只读；hover 显示关系通俗说明（如「文章列表用到登录」），不暴露 relation 技术类型名。

### UC-004: 原语徽章（PrimitiveBadge）
- **变体**: 七类**原语**业务译名（一一对应 F-001 AC-001 七原语）——功能区块（Module）/ 对外接口（Port）/ 记住了什么（Entity）/ 连接规则（Dependency/Relation，含允许·禁止）/ 当…就…（Workflow）/ 某物的一生（StateMachine）/ 安全规则（Constraint）；另加 1 个**修饰标识**「自由实现区」（escape hatch，非原语，详见交互说明）。
- **视觉差异**: `font.mono` 11px 标签 + 类型图标；底为 `color.surface-sunken`；颜色克制，不抢健康/diff 语义色。「连接规则」徽章与 UC-003 连线同源——Dependency/Relation 在地图上既渲染为连线（UC-003）、又在图例以本徽章标注其业务含义。「自由实现区」修饰标识用 `color.brass` 描边以示「不受图约束保护」。
- **Props**: `{ kind: enum('module'|'port'|'entity'|'relation'|'workflow'|'statemachine'|'constraint') }`（七原语，对应 API-011 MapNode.kind）。自由实现区为独立修饰叠加，由 UC-002 节点的 `isEscapeHatch` 布尔位驱动（对应 API-011 MapNode.isEscapeHatch），**不计入七类 kind**。
- **映射功能**: F-001 AC-001, F-006 AC-003, F-017 AC-003
- **交互说明**: 纯展示；图例（legend）在地图面角落复用本组件解释七类原语 + 自由实现区标识。escape hatch 是 F-017 的 Module 修饰符（`escape_hatch: true`）叠加在功能区块节点上，而非第八种原语；与七原语分层呈现，避免数量混淆。
- **[ASSUMPTION]**: F-006 AC-003 的 PRD 例举仅列 5 个原语通俗名（缺 Port 与 Dependency/Relation）；本卷据 F-001 AC-001 七原语全集补全 Port=「对外接口」、Dependency/Relation=「连接规则」两译名，作为能力地图图例的业务语言定稿（R-010 上游 PRD 歧义在 ui-spec 层闭合）。

### UC-005: 能力详情卡（CapabilityDetailCard）
- **变体**: default, escape-hatch（附意图说明 + 「不受图约束保护」提示）。
- **视觉差异**: 选中节点后浮出（`color.surface` 卡 + `shadow.raised`）；含 plain 说明、所属功能区块、依赖关系（「用到 X / 被 Y 用到」通俗列举）、健康摘要（UC-001）；全程通俗语言。
- **Props**: `{ node, relations[], health, escapeHatchIntent? }`
- **映射功能**: F-006 AC-005, F-017 AC-003
- **交互说明**: 「针对这个区块提需求」按钮 → 把意图绑定到该节点回到对话面（F-009 AC-003）；red 健康摘要内「一键修复」直达 UC-009 / 调和。

### UC-006: 对话气泡（ChatBubble）
- **变体**: user, system, system-working（「正在修复…」状态行，UC-014 内嵌）, system-clarify（追问澄清）。
- **视觉差异**: user 右对齐 `color.keel-wash` 底；system 左对齐 `color.surface` 底；正文 `font.sans` 14/1.6；working 态左侧脉冲点 + 通俗文案，不显示日志/代码。
- **Props**: `{ role, content, state?, timestamp? }`
- **映射功能**: F-005 AC-001/AC-003, F-009 AC-002
- **交互说明**: 用户唯一的变更发起入口；系统回复以人话呈现，绝不输出堆栈/编译输出/lint 日志（F-003 AC-004）。

### UC-007: 蓝图 diff 卡 / 人话变更说明书（BlueprintDiffCard）
- **变体**: default（待确认）, with-blast（含影响预览 UC-008）, frozen（确认后契约冻结态）。
- **视觉差异**: 卡内分「新增（`color.diff-added` 虚框 chip）/ 修改（`color.diff-touched` 紫 chip）/ 触及边界」三组，每项通俗描述（如「添加了收藏功能，修改了用户信息区块」）；底部三按钮 确认 / 追加修改 / 拒绝（UC-018）。无代码 diff。
- **Props**: `{ added[], modified[], touchedBoundaries[], blastRadius?, onConfirm, onAppend, onReject }`（对应 API-005）
- **映射功能**: F-009 AC-001/AC-002/AC-005/AC-006
- **交互说明**: 确认 → 触发契约冻结门（F-009 AC-006），卡转 frozen 态并提示「契约已锁定，开始实现」；追加 → 回到对话生成新 diff；任何结构变更必经此卡（F-009 AC-007）。确认操作同时触发意图日志条目写入（F-009 AC-009，留痕原始意图文本 + 蓝图 diff，供 P-008 时光机追溯）。

### UC-008: 影响预览条（BlastRadiusBanner）
- **变体**: complete（依赖图完整无环）, degraded（有环/信息缺失，显式「影响范围可能不完整」）。
- **视觉差异**: 横条嵌于 UC-007 顶部；complete 态「会碰这 N 个区块：[名]，不会碰：[安全区名]」（碰到=`color.diff-touched`，安全=`color.muted`）；degraded 态 `color.warn` 提示横幅，不宣称零漏标。
- **Props**: `{ touched[], safe[], degraded: boolean }`（对应 API-005 blastRadius）
- **映射功能**: F-007 AC-002/AC-003, F-009 AC-004
- **交互说明**: 与地图联动——高亮地图上 touched 节点/边（UC-003 blast-highlight），淡化安全区。

### UC-009: 健康问题卡（HealthIssueCard）
- **变体**: structural-violation（结构违规 red）, regression（回归破坏 red）, soft-warning（软告警 amber）。
- **视觉差异**: 列表条目，左 UC-001 状态点 + 通俗标题（如「收藏功能被实现了两遍，建议合并」），右「一键修复」按钮（UC-018 primary）；regression 与 structural 文案区分但同 red（对应 API-002 regressionReport）。
- **Props**: `{ issueType, plainTitle, plainDetail, affectedNodeId, onFix }`
- **映射功能**: F-002 AC-003, F-004 AC-003, F-011 AC-006
- **交互说明**: 「一键修复」→ 调和引擎（F-011），以受影响切片为单位重生成；若切片含 escape hatch 或未入图内容，先弹 UC-016 重做确认（F-011 AC-007）。

### UC-010: 需你决策卡（DecisionCard）
- **变体**: conflict（门禁反复硬阻断）, regression-tradeoff（新旧能力需求冲突）, untracked-rollback（回滚遇通道外改动）。
- **视觉差异**: 居中模态，阻断态 `color.alert` 顶边；通俗说明「这次改动破坏了 [能力名]」「违反什么 / 影响多大」；选项按钮：接受偏差更新蓝图 / 拒绝并重做（UC-018）。绝不显示技术错误。
- **Props**: `{ conflictKind, plainExplain, impact, options[], onDecide }`
- **映射功能**: F-010 AC-006, F-004 AC-002, F-013 AC-006
- **交互说明**: 仅在自动修复达上限（F-005 AC-004）或涉业务取舍时出现；决策留痕到意图日志（F-013 AC-001）。

### UC-011: 安全红绿灯卡（SecurityChecklistCard）
- **变体**: all-green（可上线）, has-red（阻断）, scanning（体检中）。
- **视觉差异**: 三类逐项红绿灯——认证配置 / 数据权限 / 密钥泄露（UC-001 pill 复用）；顶部安全分（`font.mono` 大字）；has-red 项展开通俗说明 + 修复入口；底「上线」按钮在 has-red 时禁用（UC-018 disabled）。
- **Props**: `{ score, checks: [{category, light: 'green'|'red', plain, fixEntry?}], canDeploy }`（字段名对齐 API-014 返回结构）；`canDeploy` 由前端推导 `checks.every(c => c.light === 'green')`，「修复」交互由 `fixEntry` 入口地址驱动；`scanning` 为 UI 层瞬态、非 API 字段（R-005）。
- **映射功能**: F-014 AC-002/AC-003
- **交互说明**: 全绿才放行部署（F-014 AC-003）；红灯「修复」可触发调和；体检结果不暴露扫描器原始输出。

### UC-012: 时间线条目（TimelineEntry）
- **变体**: change（普通变更）, decision（需你决策留痕）, deploy（上线动作）, current（当前所在点）。
- **视觉差异**: 竖向时间线节点 + 自然语言摘要（「添加了用户收藏文章功能」），时间 `font.mono` `color.muted`；不显示 commit 哈希/git 命令/代码 diff；current 态 `color.keel` 高亮。
- **Props**: `{ kind, plainSummary, timestamp, isCurrent, onRollback }`（plainSummary/timestamp 对应 API-013 timeline entries）。**[ASSUMPTION]**: `kind`（change/decision/deploy）待 arch API-013 timeline entries 补 `entryKind: enum(change|decision|deploy)` 字段；补全前由前端按条目内容推断变体（根因在 arch，R-006）。
- **映射功能**: F-013 AC-002/AC-004
- **交互说明**: 「回到这里」→ 时光机回滚（蓝图+代码+健康灯同步）；回滚前若有通道外改动先弹 UC-010 untracked-rollback 确认（F-013 AC-006）。

### UC-013: 进度状态标（StatusChip）
- **变体**: planning（规划中）, implementing（实现中）, ready（已就绪）。
- **视觉差异**: `font.mono` 11px chip；planning `color.muted` 描边、implementing `color.keel` + 微动效、ready `color.ok`；叠加于 UC-002 节点右下或独立列表。
- **Props**: `{ status: enum(three) }`
- **映射功能**: F-010 AC-002/AC-005
- **交互说明**: 实现期间随依赖拓扑序逐节点切换 planning→implementing→ready，用户据此观察进度而非看代码/终端。

### UC-014: 修复中状态条（RepairingStatus）
- **变体**: working（正在修复…）, progress（超阈值转可见进度）, abortable（含中止入口）。
- **视觉差异**: 内嵌对话或节点的细条，脉冲文案「正在修复…」（不无限期无反馈）；超 §3.1 NFR 阈值转进度条；右侧「中止」链接。
- **Props**: `{ state, progressPct?, onAbort }`
- **映射功能**: F-005 AC-004/AC-005
- **交互说明**: 中止后应用回到本次改动前完整可用状态（F-005 AC-005 / F-010 AC-004）；达自愈上限则升级 UC-010。

### UC-015: 通知 Toast（Toast）
- **变体**: success, info, warn, error-soft（仍通俗，不暴露技术错误）。
- **视觉差异**: 右下浮出，`shadow.raised`，`motion.base` 进出；warn `color.warn-soft` 底；自动消散，可手动关。
- **Props**: `{ level, plainMessage, duration?, action? }`
- **映射功能**: F-008 AC-002（预览刷新提示）, F-014 AC-001（上线完成返回地址）
- **交互说明**: 用于非阻断反馈（预览已刷新 / 已上线，含可访问地址链接）；阻断类信息走 UC-010 模态而非 Toast。

### UC-016: 确认模态 / 抽屉（ConfirmModal / Drawer）
- **变体**: modal-confirm（居中阻断确认）, drawer-right（右侧抽屉：时光机/健康）, destructive（回滚/重做覆盖，`color.alert` 强调）。
- **视觉差异**: 模态居中 + 遮罩；抽屉右贴边滑入（`motion.base`）；destructive 主按钮 `color.alert`；标题 `font.serif`，正文通俗。
- **Props**: `{ kind, title, body, confirmLabel, danger?, onConfirm, onClose }`
- **映射功能**: F-011 AC-007, F-013 AC-004/AC-006
- **交互说明**: 承载 escape hatch 重做确认、回滚确认、通道外改动提示等「先提示后执行」流程；遵守 §4 query 参数化以支持后退关闭。

### UC-017: 空状态（EmptyState）
- **变体**: preview-unavailable（预览不可用）, map-empty（地图空/未生成骨架）, first-run（首个项目引导）。
- **视觉差异**: 居中插画/线稿 + 通俗文案 + 单一引导按钮；`color.muted` 文案，绝不显示技术错误页（F-008 AC-003）。
- **Props**: `{ kind, plainMessage, ctaLabel?, onCta }`
- **映射功能**: F-008 AC-003, F-012 AC-004
- **交互说明**: 预览构建中显示「应用正在搭建，稍候即可预览」类通俗状态；引导用户回到对话面发起首个意图。

### UC-018: 按钮（Button）
- **变体**: primary（`color.keel` 实心）, ghost（描边/透明底）, danger（`color.alert`）, disabled（opacity 降 + 禁交互）；尺寸 sm/md。
- **视觉差异**: `radius.sm`；hover→`color.keel-bright`/抬升；active 微缩；disabled 去阴影、`color.muted` 文字；焦点环 `color.brass`。
- **Props**: `{ variant, size, label, icon?, disabled?, onClick }`
- **映射功能**: 通用原子，贯穿全屏。
- **交互说明**: primary 一屏一主操作；危险操作（回滚/拒绝重做）用 danger 并配 UC-016 二次确认。

### UC-019: 引擎安装 / 登录卡（EngineSetupCard）
- **变体**: detect（检测中）, install-guide（图形安装引导）, login（OAuth 订阅登录 / API Key 兼容）, smoke-ok（冒烟校验通过）。
- **视觉差异**: 向导步骤卡（`font.serif` 标题 + 步进指示）；安装引导为图形步骤，绝无终端命令；login 优先「订阅登录」大按钮、次级「用 API Key」；smoke-ok 绿勾。
- **Props**: `{ step, engineStatus, authMode, onInstall, onLogin }`（对应 API-016）
- **映射功能**: F-012 AC-001/AC-002/AC-003/AC-006, F-018 AC-002
- **交互说明**: 桌面图形界面仅用于安装与登录，不用于驱动；后台经 ACP/SDK/CLI 连接并冒烟校验连通。

### UC-020: 用量 / 订阅徽标（UsageMeter）
- **变体**: subscription（订阅计划+额度）, apikey（Key 额度+消耗）, unknown（引擎侧未提供）。
- **视觉差异**: 顶栏角落紧凑徽标，`font.mono` 数字 + 细环进度；点击展开明细；措辞标明「执行引擎侧消耗」与 Keel 定价无关。
- **Props**: `{ planLabel, used, quota, source }`（数据由引擎侧接口提供）。**[ASSUMPTION]**: API-016 当前仅定义 scaffold/detectEngine/authenticate，无用量查询操作；M-014 内部 UsageProbe 的契约待 arch 补 API-016 `usageStatus` 操作（返回 `{ planLabel, used, quota, source }`）后对账（根因在 arch，R-007）。
- **映射功能**: F-012 AC-007
- **交互说明**: 仅作执行引擎消耗透明度展示（定价延后 ADR-0005）；无 Keel 自身计费入口。
