---
id: "dev-plan-keel-s4"
version: "1.0.0"
doc_type: dev-plan
author: tech-lead
status: draft
deps: ["arch-keel", "ui-spec-keel"]
consumers: [developer, qa-engineer]
volume: sprint
volume_type: sprint
split_from: "dev-plan-keel"
required_sections:
  - "## 3. 任务卡详细"
---
# Development Plan 分卷 — Sprint 4: 工作区前端 + 能力地图 + 对话面

[NAV]
- §3 任务卡详细 → T-027..T-037 (Sprint 4)
[/NAV]

## 3. 任务卡详细

### T-027: 设计 Token 与主题系统（THEME-01）

- **目标**: 将 ui-spec-keel-theme THEME-01 全量 token 落为 CSS 变量 + TypeScript 常量，含颜色/排版/阴影/圆角/动效系统
- **模块**: M-015
- **接口**: —（设计 token 层，无 API 契约）
- **task_kind**: chore
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `tokens/index.css` 加载到 DOM，When 检查 CSS 变量，Then 含 `--color-paper`, `--color-keel`, `--color-ok`, `--color-warn`, `--color-alert`, `--font-serif`, `--font-sans`, `--font-mono`, `--shadow-rest`, `--shadow-raised`, `--radius-sm`, `--radius-md`, `--radius-lg`, `--motion-base`, `--motion-slow` 等 THEME-01 定义的全量 token [ui-spec-keel-theme]
  - [ ] AC-002: Given TypeScript `import { tokens } from '@keel/workspace/tokens'`，When `tokens.color.keel`，Then 等于 THEME-01 定义的 `#2b3f72` [ui-spec-keel-theme]
- **deliverables**:
  - [ ] `apps/workspace/src/tokens/index.css` — CSS 变量全集
  - [ ] `apps/workspace/src/tokens/index.ts` — TS 常量导出
- **context_load**:
  - ui-spec-keel-theme
  - ui-spec-keel#§0

---

### T-028: 基础原子组件（UC-001 / UC-004 / UC-013 / UC-015 / UC-018）

- **目标**: 实现健康 Pill（UC-001）、原语徽章（UC-004）、进度状态标（UC-013）、Toast（UC-015）、按钮（UC-018）五个原子组件
- **模块**: M-015
- **接口**: —
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `<HealthPill state="red" shape="pill" label="需处理" />`，When 渲染，Then 展示 `color.alert` 底色 + 呼吸动画（CSS animation `--motion-slow` 循环）+ `▲` 图标；`shape="dot"` 时无文案 [ui-spec-keel-components#UC-001]
  - [ ] AC-002: Given `<PrimitiveBadge kind="module" />`，When 渲染，Then 展示「功能区块」标签 + `color.surface-sunken` 底；`kind="constraint"` 时展示「安全规则」；七类全量覆盖 [ui-spec-keel-components#UC-004]
  - [ ] AC-003: Given `<Button variant="primary" label="确认" disabled={true} />`，When 渲染，Then opacity 降低、pointer-events none；焦点环使用 `color.brass` [ui-spec-keel-components#UC-018]
  - [ ] AC-004: Given `<StatusChip status="implementing" />`，When 渲染，Then 展示 `color.keel` 色 + 微动效；`status="ready"` 时 `color.ok` [ui-spec-keel-components#UC-013]
  - [ ] AC-005: Given `<Toast level="success" plainMessage="已上线" />`，When 挂载，Then 右下角显示，`duration` 毫秒后自动消散；`level="error-soft"` 时展示通俗错误（不含堆栈）[ui-spec-keel-components#UC-015]
- **deliverables**:
  - [ ] `apps/workspace/src/components/atoms/HealthPill.tsx` + `.test.tsx`
  - [ ] `apps/workspace/src/components/atoms/PrimitiveBadge.tsx` + `.test.tsx`
  - [ ] `apps/workspace/src/components/atoms/StatusChip.tsx` + `.test.tsx`
  - [ ] `apps/workspace/src/components/atoms/Toast.tsx` + `.test.tsx`
  - [ ] `apps/workspace/src/components/atoms/Button.tsx` + `.test.tsx`
- **context_load**:
  - ui-spec-keel-components#§2.UC-001
  - ui-spec-keel-components#§2.UC-004
  - ui-spec-keel-components#§2.UC-013
  - ui-spec-keel-components#§2.UC-015
  - ui-spec-keel-components#§2.UC-018

---

### T-029: 三面工作区外壳（P-001，M-015）

- **目标**: 实现 React + Vite Web 壳的三面布局（对话/地图/预览），含顶栏、响应式断点适配、面间拖拽调宽、ShellAdapter 接缝
- **模块**: M-015
- **接口**: —（前端外壳，消费 M-009/M-010/M-005）
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 宽度 ≥1440px（comfortable 断点），When 工作区加载，Then 三面并列渲染（对话 ~360px 左 / 地图弹性中 / 预览 ~420px 右）；面间 `1px color.line` 分隔线 [ui-spec-keel-pages-core#P-001 / ui-spec-keel#§5]
  - [ ] AC-002: Given 宽度 768~1023px（narrow 断点），When 渲染，Then 顶部 segmented control 切换对话/地图/预览单面显示 [ui-spec-keel#§5]
  - [ ] AC-003: Given 三面首次挂载（含骨架屏状态），When `performance.now()` 计时，Then 到达骨架屏渲染完成 ≤2s（地图面板采用 `React.lazy` 懒加载，不阻塞主线程）[prd#§2.F-008 AC-005 / ui-spec-keel-pages-core#P-001]
  - [ ] AC-004（production-path）: `apps/workspace/src/App.tsx` 渲染 `<WorkspaceLayout>` 组件，路由 `/project/:id` 挂载此布局 [ui-spec-keel#§4]
- **deliverables**:
  - [ ] `apps/workspace/src/layouts/WorkspaceLayout.tsx` — 三面布局 + 响应式
  - [ ] `apps/workspace/src/layouts/TopBar.tsx` — 顶栏（项目名 + 健康灯 + 用量）
  - [ ] `apps/workspace/src/App.tsx` — React Router 路由配置
  - [ ] `apps/workspace/src/layouts/ShellAdapter.ts` — Web↔桌面可替换接缝
  - [ ] `apps/workspace/src/layouts/WorkspaceLayout.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-core#§3.P-001
  - ui-spec-keel#§4
  - ui-spec-keel#§5
  - arch-keel-modules#§2.M-015
  - prd-keel-f001-f009#§2.F-008

---

### T-030: 能力地图渲染面（React Flow，P-002，M-009）

- **目标**: 实现 M-009 `MapProjector` + React Flow v12 渲染面，从 `BlueprintIR.business` 投影 MapNode/MapEdge，含七原语 + 节点选中 + 层级聚合 + 只读约束
- **模块**: M-009
- **接口**: API-011
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `BlueprintIR` 含 20 个 `surfaces_on_map=true` 节点，When `MapProjector.project(blueprintIR)`，Then 返回 `{ nodes: MapNode[], edges: MapEdge[], renderMs ≤ 1000 }`；`MapNode.plainName` 来自蓝图 `plain_name` 字段（非代码类名）[ARCH#§3.API-011 / prd#§2.F-006 AC-001/AC-008]
  - [ ] AC-002: Given React Flow 渲染 20 节点地图，When 用户点击节点，Then `nodesDraggable=false`（节点不可拖动）+ `nodesConnectable=false`（不可连线）+ 点击触发 `onNodeSelect(nodeId)` 回调（只读约束）[prd#§2.F-006 AC-006 / ui-spec-keel-pages-core#P-002]
  - [ ] AC-003: Given `MapNode.kind: 'module'`，When 渲染，Then `UC-002 CapabilityNode` 展示 `UC-004 PrimitiveBadge kind="module"`（「功能区块」标签）；七类 kind 各自正确 badge [ui-spec-keel-components#UC-002 / ARCH#§3.API-011 types.MapNode.kind]
  - [ ] AC-004: Given 左下角图例区，When 渲染，Then 展示七类原语 + 健康三态图例（复用 `UC-004` + `UC-001`） [prd#§2.F-006 AC-003 / ui-spec-keel-pages-core#P-002]
  - [ ] AC-005（production-path）: `apps/workspace/src/map/CapabilityMap.tsx` 中通过 `mapProjector.project(blueprintIR)` 获取视图数据，并以 `<ReactFlow nodes={...} edges={...} />` 渲染 [ARCH#§2.M-009]
- **deliverables**:
  - [ ] `packages/blueprint/src/map/MapProjector.ts` — IR business 层投影（供后端）
  - [ ] `apps/workspace/src/map/CapabilityMap.tsx` — React Flow 渲染（`React.lazy` 懒加载）
  - [ ] `apps/workspace/src/map/nodes/CapabilityNode.tsx` — 自定义节点（UC-002）
  - [ ] `apps/workspace/src/map/edges/RelationEdge.tsx` — 自定义边（UC-003）
  - [ ] `apps/workspace/src/map/Legend.tsx` — 七类图例（UC-004）
  - [ ] `apps/workspace/src/map/CapabilityMap.test.tsx`
- **context_load**:
  - arch-keel-modules#§2.M-009
  - arch-keel-api#§3.API-011
  - ui-spec-keel-pages-core#§3.P-002
  - ui-spec-keel-components#§2.UC-002
  - ui-spec-keel-components#§2.UC-003
  - prd-keel-f001-f009#§2.F-006

---

### T-031: 健康叠加 + 节点 Status（M-009 HealthOverlay）

- **目标**: 实现 `HealthOverlay`（M-009）：从 E-005 HealthSnapshot 读取健康状态叠加到地图节点（绿/黄/红），区分 structural-violation red 与 regression red；节点 status 来自 M-006/M-011 进度
- **模块**: M-009
- **接口**: API-011 (health/status 叠加)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given E-005 中 nodeId `auth` 的 `health: 'red'`（来自 M-002 结构违规），When `HealthOverlay.apply(nodes, healthSnapshots)`，Then 返回 `MapNode` 中 `auth.health === 'red'`；`red` 节点 `UC-001 dot` 显示 `color.alert` 呼吸动画 [ARCH#§3.API-011 / arch-keel-data#§4.E-005]
  - [ ] AC-002: Given M-002 `GateReport.regressionReport` 非空（回归破坏），When `HealthOverlay` 读取，Then 受影响节点 `health: 'red'` 且 `UC-009 issueType: 'regression'`（区别于 structural-violation）[prd#§2.F-004 AC-003 / ARCH#§3.API-002 types.RegressionBreak]
  - [ ] AC-003: Given 节点 `favorites` 处于 `implementing` 进度状态（M-006 ProgressStateMachine），When 地图渲染，Then `CapabilityNode[data-id='favorites']` 内部存在 `<StatusChip data-status="implementing">` 节点（`queryByTestId('status-chip-favorites')` 返回 DOM 元素且 `dataset.status === 'implementing'`）[prd#§2.F-010 AC-005 / ui-spec-keel-components#UC-013]
- **deliverables**:
  - [ ] `apps/workspace/src/map/overlays/HealthOverlay.ts` — E-005 健康快照读取 + 叠加
  - [ ] `apps/workspace/src/map/overlays/StatusOverlay.ts` — 进度状态叠加
  - [ ] `apps/workspace/src/map/overlays/HealthOverlay.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-009
  - arch-keel-api#§3.API-011
  - arch-keel-data#§4.E-005
  - ui-spec-keel-components#§2.UC-009
  - prd-keel-f001-f009#§2.F-004

---

### T-032: 对话面 + 意图输入（P-003 对话侧，M-005）

- **目标**: 实现对话面（左面板）：ChatBubble（UC-006）消息流 + 底部输入框，支持 point-and-prompt（先选节点再输入），调用 `IntentTranslator`，期间显示 `working` 态
- **模块**: M-005, M-015
- **接口**: API-005 (调用入口)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 用户在输入框输入意图文本并提交，When `ChatPanel` 处理提交，Then 显示 user 气泡（`ChatBubble role="user"`）+ system working 气泡（`role="system" state="working"`，脉冲文案「正在理解你的意图…」）；不显示代码/日志 [ui-spec-keel-components#UC-006 / prd#§2.F-009 AC-002]
  - [ ] AC-002: Given 用户先在地图选中节点 `auth-module`（point-and-prompt），When 在输入框提交意图，Then `IntentTranslator.translate({ intentText, boundNodeId: 'auth-module' })` 被调用 [prd#§2.F-009 AC-003 / ui-spec-keel-pages-core#P-003]
  - [ ] AC-003: Given `IntentTranslator` 返回 `KeelError`，When 对话面收到错误，Then 展示 `ChatBubble state="system-clarify"` 通俗追问（「没能理解，换个说法？」），不显示技术错误 [prd#§2.F-003 AC-004]
  - [ ] AC-004（production-path）: `apps/workspace/src/chat/ChatPanel.tsx` 调用 `intentService.translate(intentText, boundNodeId)` 触发后端 API-005 [ARCH#§2.M-005]
- **deliverables**:
  - [ ] `apps/workspace/src/chat/ChatPanel.tsx` — 消息流 + 输入框
  - [ ] `apps/workspace/src/chat/ChatBubble.tsx` — UC-006 实现
  - [ ] `apps/workspace/src/chat/ChatPanel.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-core#§3.P-003
  - ui-spec-keel-components#§2.UC-006
  - arch-keel-modules#§2.M-005
  - prd-keel-f001-f009#§2.F-009

---

### T-033: 蓝图 Diff 卡与 Blast Radius 展示（P-003 diff 侧）

- **目标**: 实现 `BlueprintDiffCard`（UC-007）+ `BlastRadiusBanner`（UC-008）居中模态；确认触发契约冻结；地图联动高亮 blast radius 节点
- **模块**: M-005, M-009
- **接口**: API-005 / API-006
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `BlueprintDiff { added: [favorites], modified: [user], touchedBoundaries: [article] }` 与 `blastRadius { touched: [user,article], safe: [payment], degraded: false }`，When `BlueprintDiffCard` 渲染，Then 展示 added（`color.diff-added` 虚框 chip）/ modified（`color.diff-touched` 紫 chip）/ touchedBoundaries 三组；`BlastRadiusBanner` 显示「会碰这 2 个区块…不会碰：支付」[ui-spec-keel-components#UC-007 / UC-008]
  - [ ] AC-002: Given `blastRadius.degraded: true`，When `BlastRadiusBanner` 渲染，Then `degraded` 变体显示 `color.warn` 提示横幅「影响范围可能不完整」，不宣称零漏标 [prd#§2.F-007 AC-003 / ui-spec-keel-components#UC-008]
  - [ ] AC-003: Given 用户点击「确认」按钮，When `onConfirm()` 触发，Then 调用 `ContractFreezer.freeze(blueprintDiff)`；成功后 diff 卡转 `frozen` 态，展示「契约已锁定，开始实现」；同时意图日志条目写入（F-009 AC-009）[prd#§2.F-009 AC-006/AC-009 / ui-spec-keel-components#UC-007]
  - [ ] AC-004: Given 地图面加载，When `BlastRadiusBanner` 中 `touched: ['user', 'article']`，Then `CapabilityNode[data-id='user']` 与 `CapabilityNode[data-id='article']` 具有 CSS class `blast-highlight`；`CapabilityNode[data-id='payment']` 具有 CSS class `blast-dimmed`（opacity 降低）[prd#§2.F-007 AC-001 / ui-spec-keel-components#UC-003]
- **deliverables**:
  - [ ] `apps/workspace/src/diff/BlueprintDiffCard.tsx` — UC-007
  - [ ] `apps/workspace/src/diff/BlastRadiusBanner.tsx` — UC-008
  - [ ] `apps/workspace/src/diff/DiffModal.tsx` — 居中模态容器
  - [ ] `apps/workspace/src/diff/DiffModal.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-core#§3.P-003
  - ui-spec-keel-components#§2.UC-007
  - ui-spec-keel-components#§2.UC-008
  - prd-keel-f001-f009#§2.F-007
  - prd-keel-f001-f009#§2.F-009

---

### T-034: 应用预览面（P-004，M-010）

- **目标**: 实现 `PreviewPanel`（M-010）：目标应用 dev server 代理的 iframe 嵌入；自动刷新（门禁通过后）；降级空态 UC-017
- **模块**: M-010
- **接口**: API-012
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given API-012 `getPreview()` 返回 `{ url: 'http://localhost:3000', state: 'ready', plainStatus: '应用已就绪' }`，When `PreviewPanel` 渲染，Then 展示 iframe src 为该 url，顶栏显示 `ready` 状态 [ARCH#§3.API-012 / prd#§2.F-008 AC-001]
  - [ ] AC-002: Given API-012 返回 `{ state: 'unavailable', plainStatus: '应用正在搭建，稍候即可预览' }`，When `PreviewPanel` 渲染，Then 展示 `UC-017 EmptyState kind="preview-unavailable"` 通俗文案，无技术错误页 [prd#§2.F-008 AC-003 / ui-spec-keel-components#UC-017]
  - [ ] AC-003: Given 门禁通过事件（WebSocket `{ type: 'gate:passed' }`），When `PreviewPanel` 收到，Then 自动调用 `API-012.refresh()`，iframe 刷新；弹 `UC-015 Toast success "预览已更新"` [prd#§2.F-008 AC-002 / ui-spec-keel-components#UC-015]
- **deliverables**:
  - [ ] `apps/workspace/src/preview/PreviewPanel.tsx`
  - [ ] `apps/workspace/src/preview/PreviewStateView.tsx` — 降级态
  - [ ] `apps/workspace/src/preview/PreviewPanel.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-core#§3.P-004
  - arch-keel-modules#§2.M-010
  - arch-keel-api#§3.API-012
  - prd-keel-f001-f009#§2.F-008

---

### T-035: 实现进度叠加 + 「需你决策」卡（P-005）

- **目标**: 实现进度状态叠加（`UC-013 StatusChip` 地图叠加）、`RepairingStatus`（UC-014）与 `DecisionCard` 模态（UC-010）
- **模块**: M-006, M-007
- **接口**: API-007 / API-008
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given WebSocket 事件 `{ type: 'progress:updated', capabilityId: 'favorites', status: 'implementing' }`，When `ProgressOverlay` 处理事件，Then 地图节点 `favorites` 展示 `UC-013 StatusChip status="implementing"` [prd#§2.F-010 AC-005 / ui-spec-keel-components#UC-013]
  - [ ] AC-002: Given WebSocket 事件 `{ type: 'repair:progress', state: 'working' }`，When `RepairingStatus` 展示，Then 显示 `state="working"` 脉冲文案「正在修复…」；不出现技术错误 [prd#§2.F-005 AC-001 / ui-spec-keel-components#UC-014]
  - [ ] AC-003: Given `DecisionCard { conflictKind: 'conflict', plainExplain: '这次改动破坏了收藏功能…', options: [...] }`，When `DecisionCard` 模态渲染，Then 居中阻断模态 + `color.alert` 顶边 + 通俗说明 + 两选项按钮；无技术错误文本 [prd#§2.F-010 AC-006 / ui-spec-keel-components#UC-010]
- **deliverables**:
  - [ ] `apps/workspace/src/progress/ProgressOverlay.tsx` — 地图叠加 StatusChip
  - [ ] `apps/workspace/src/progress/RepairingStatus.tsx` — UC-014
  - [ ] `apps/workspace/src/progress/DecisionCard.tsx` — UC-010 阻断模态
  - [ ] `apps/workspace/src/progress/ProgressOverlay.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-core#§3.P-005
  - ui-spec-keel-components#§2.UC-010
  - ui-spec-keel-components#§2.UC-014
  - prd-keel-f010-f018#§2.F-010
  - prd-keel-f001-f009#§2.F-005

---

### T-036: 健康调和抽屉（P-006）

- **目标**: 实现健康调和抽屉（P-006）：`HealthIssueCard`（UC-009）问题清单 + 一键修复（触发调和）+ 内容保护确认（UC-016）+ 双向裁决入口
- **模块**: M-002, M-003, M-004
- **接口**: API-003 / API-004
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `DriftReport { items: [{ type: 'boundary_break', plain: '收藏功能被实现了两遍…', ... }] }`，When `HealthDrawer` 渲染，Then 展示 `UC-009 HealthIssueCard issueType="structural-violation"` 含通俗标题 + 「一键修复」按钮；`regression` 类型时 `issueType="regression"`（文案区分同 red）[prd#§2.F-002 AC-003 / ui-spec-keel-components#UC-009]
  - [ ] AC-002: Given 用户点击「一键修复」且切片含 escape hatch，When `onFix` 触发，Then 弹 `UC-016 ConfirmModal kind="destructive"` 提示「这块有你的自由实现区，重做可能丢失」；用户确认后才调用 `ReconcileEngine.reconcile(...)` [prd#§2.F-011 AC-006/AC-007 / ui-spec-keel-components#UC-016]
  - [ ] AC-003: Given 调和完成（`DriftReport` 不再含该漂移项），When `HealthDrawer` 刷新，Then DOM 中该 `UC-009 HealthIssueCard` 组件被卸载（`queryByTestId('issue-card-boundary_break')` 返回 `null`）；对应地图节点 `MapNode.health` 值变为 `'green'` [prd#§2.F-011 AC-004]
  - [ ] AC-004: Given 调和问题，When 用户点击「更新蓝图承认实际」选项，Then `HealthDrawer` 收起（DOM 中抽屉面板隐藏），`DiffModal` 组件挂载（`queryByTestId('diff-modal')` 非 null），等待用户确认 diff（F-002 AC-004 双向裁决路径）[prd#§2.F-002 AC-004]
- **deliverables**:
  - [ ] `apps/workspace/src/health/HealthDrawer.tsx` — P-006 主容器（`?panel=health` 路由）
  - [ ] `apps/workspace/src/health/HealthIssueCard.tsx` — UC-009
  - [ ] `apps/workspace/src/health/ConfirmModal.tsx` — UC-016
  - [ ] `apps/workspace/src/health/HealthDrawer.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-core#§3.P-006
  - ui-spec-keel-components#§2.UC-009
  - ui-spec-keel-components#§2.UC-016
  - prd-keel-f001-f009#§2.F-002
  - prd-keel-f010-f018#§2.F-011

---

### T-037: [VALIDATION] 工作区前端核心三面 + 意图→diff→确认流程

- **目标**: 用户手动验证 Sprint 4 前端产出
- **task_kind**: validation
- **模块**: M-009, M-015
- **tdd_acceptance**: N/A（validation 任务不走 TDD，由 orchestrator 展示验证清单后用户手动确认）
- **deliverables**: N/A（validation 任务不产出代码文件，由前置任务 T-027~T-036 的 deliverables 覆盖）
- **验证清单**:
  - [ ] 宽度 ≥1440px 时三面并列渲染，面间可拖拽调宽；宽度 <1024px 时 segmented control 切换可用
  - [ ] 20 节点 `BlueprintIR` 地图渲染完成在 1s 内（`MapProjector.project()` + React Flow 渲染）
  - [ ] 健康 red pill 有呼吸动画，green/amber 静止；`▲` 图标与色彩不单独承载语义
  - [ ] 七类 `PrimitiveBadge` 全部展示正确业务译名（功能区块/对外接口/记住了什么/连接规则/当…就…/某物的一生/安全规则）
  - [ ] 对话面输入意图 → 系统显示 working 气泡（脉冲文案，无日志）→ diff 卡弹出
  - [ ] `BlastRadiusBanner degraded=true` 时显示「影响范围可能不完整」（不宣称零漏标）
  - [ ] 确认 diff 卡 → 触发契约冻结 → 卡转 frozen 态 → 地图节点逐个进入 implementing/ready 进度
  - [ ] 健康调和抽屉：含 escape hatch 切片一键修复弹出确认模态，不静默执行
  - [ ] 预览面在 `state: 'unavailable'` 时展示通俗空态，无技术错误页
- **前置任务**: [T-027, T-028, T-029, T-030, T-031, T-032, T-033, T-034, T-035, T-036]
- **context_load**:
  - ui-spec-keel-pages-core#§3
  - ui-spec-keel-components#§2
  - prd-keel-f001-f009#§2.F-006
  - prd-keel-f001-f009#§2.F-009
  - prd-keel-f010-f018#§2.F-010
