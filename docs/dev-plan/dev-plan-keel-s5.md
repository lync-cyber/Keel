---
id: "dev-plan-keel-s5"
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
# Development Plan 分卷 — Sprint 5: 生命周期功能（上手 + 时光机 + 上线 + 导出）

[NAV]
- §3 任务卡详细 → T-038..T-051 (Sprint 5)
[/NAV]

## 3. 任务卡详细

### T-038: 项目初始化：引擎检测与安装引导（M-014）

- **目标**: 实现 `EngineCapabilityProbe` 与 `EngineInstaller`（M-014）：检测本地执行引擎就绪性，未装时提供图形安装引导（无终端命令）
- **模块**: M-014
- **接口**: API-016 (`detectEngine`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 本地已安装 Claude Code CLI，When `EngineCapabilityProbe.detectEngine()`，Then 返回 `{ ready: true, kind: 'cli' }` [ARCH#§3.API-016 detectEngine / prd#§2.F-012 AC-001]
  - [ ] AC-002: Given 本地未安装任何执行引擎，When `EngineCapabilityProbe.detectEngine()`，Then 返回 `{ ready: false, kind: undefined, installGuide: '...' }`（`installGuide` 为图形化步骤描述文案，无终端命令字符串）[prd#§2.F-012 AC-001]
  - [ ] AC-003: Given 冒烟校验任务，When `EngineCapabilityProbe.smokeTest(kind: 'cli')`，Then 经引擎执行简单任务，返回 `{ passed: boolean }`；不成功时 `passed: false`（不抛异常）[prd#§2.F-012 AC-003]
- **deliverables**:
  - [ ] `packages/init/src/probe/EngineCapabilityProbe.ts`
  - [ ] `packages/init/src/installer/EngineInstaller.ts` — 图形引导步骤数据（无 shell 命令）
  - [ ] `packages/init/tests/engine-capability-probe.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-014
  - arch-keel-api#§3.API-016
  - prd-keel-f010-f018#§2.F-012

---

### T-039: 项目初始化：鉴权 + 骨架生成（M-014）

- **目标**: 实现 `AuthBroker`（OAuth + API Key 本地存储）与 `ScaffoldGenerator`（生成初始蓝图 + modular-monolith 骨架），返回 `blueprintPath` + `skeletonReady`
- **模块**: M-014
- **接口**: API-016 (`authenticate`, `scaffold`)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: true
- **tdd_acceptance**:
  - [ ] AC-001: Given `AuthBroker.authenticate({ method: 'oauth' })` 完成 OAuth 回调，When 调用完成，Then 返回 `{ authenticated: true }`；凭据存入系统密钥库（`authRef` 非明文），`EngineConnection.authRef` 写入 E-006（不含明文 token）[ARCH#§3.API-016 authenticate / arch-keel-data#§4.E-006 / prd#§2.F-012 AC-002]
  - [ ] AC-002: Given `AuthBroker.authenticate({ method: 'api_key' })`，When 用户输入 API Key，Then 返回 `{ authenticated: true }`；key 经最小权限校验后存系统密钥库，不写入任何文件或配置 [prd#§2.F-012 AC-002 / arch-keel#§5.2]
  - [ ] AC-003: Given `ScaffoldGenerator.scaffold({ intentOrTemplate: '博客应用：用户可发文章和评论' })`，When 调用，Then 返回 `{ blueprintPath: string（非空路径）, skeletonReady: true, archetype: 'modular-monolith' }`；对 `blueprintPath` 调用 `SchemaValidator.validate()` 返回 `{ valid: true }`；蓝图中每个 Module/Port/Entity 条目的 `plain` 字段均为非空字符串 [ARCH#§3.API-016 scaffold / prd#§2.F-012 AC-004/AC-005]
  - [ ] AC-004（production-path）: `packages/init/src/AuthBroker.ts` 写入 E-006 时调用 `dbClient.insert(engineConnections, { authRef: systemKeychainRef })`；明文 key 不通过 [ARCH#§4.E-006]
- **deliverables**:
  - [ ] `packages/init/src/auth/AuthBroker.ts`
  - [ ] `packages/init/src/scaffold/ScaffoldGenerator.ts`
  - [ ] `packages/init/src/scaffold/ArchetypeSelector.ts` — 自动选 modular-monolith
  - [ ] `packages/init/src/index.ts`
  - [ ] `packages/init/tests/auth-broker.test.ts`
  - [ ] `packages/init/tests/scaffold-generator.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-014
  - arch-keel-api#§3.API-016
  - arch-keel-data#§4.E-006
  - arch-keel#§5.2
  - prd-keel-f010-f018#§2.F-012

---

### T-040: 上手向导页面（P-007，M-014 + M-008）

- **目标**: 实现 `/onboarding` 全页向导：`EngineSetupCard`（UC-019）五步流程 + `UsageMeter`（UC-020）+ 结束后跳工作区；全程无终端命令
- **模块**: M-014
- **接口**: API-016
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 路由 `/onboarding`，When 页面加载，Then 展示 `UC-019 step="detect"` 检测中态；步进指示显示 5 步流程；无任何终端命令文本 [ui-spec-keel-pages-lifecycle#P-007 / prd#§2.F-012 AC-006]
  - [ ] AC-002: Given `detectEngine` 返回 `{ ready: false }`，When `UC-019` 渲染，Then `EngineSetupCard[data-step]` attribute 值为 `"install-guide"`；`getByTestId('install-guide-card')` DOM 文本内容不含字符串 `npm install`、`brew install`、`$`（shell 命令特征）[prd#§2.F-012 AC-001]
  - [ ] AC-003: Given 登录成功 + 冒烟通过 + 骨架生成完成，When 完成最后一步，Then 页面跳转至 `/project/:id` 三面工作区 [prd#§2.F-012 AC-004]
  - [ ] AC-004: Given `usageStatus` 返回 `{ planLabel: 'Pro', used: 120, quota: 1000, source: 'subscription' }`，When `UsageMeter` 渲染（顶栏角落），Then 展示「执行引擎侧 Pro · 120/1000」（措辞标明非 Keel 定价）[ARCH#§3.API-016 usageStatus / ui-spec-keel-components#UC-020]
- **deliverables**:
  - [ ] `apps/workspace/src/onboarding/OnboardingPage.tsx` — P-007 全页
  - [ ] `apps/workspace/src/onboarding/EngineSetupCard.tsx` — UC-019（5 步）
  - [ ] `apps/workspace/src/components/atoms/UsageMeter.tsx` — UC-020
  - [ ] `apps/workspace/src/onboarding/OnboardingPage.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-lifecycle#§3.P-007
  - ui-spec-keel-components#§2.UC-019
  - ui-spec-keel-components#§2.UC-020
  - prd-keel-f010-f018#§2.F-012
  - arch-keel-api#§3.API-016

---

### T-041: 意图日志存储（M-011，E-002 / E-003）

- **目标**: 实现 `IntentLogStore`（M-011）：写入 E-002 IntentLogEntry（原始意图文本 + blueprintDiff + timestamp）与 E-003 DecisionRecord；每次变更自动记录
- **模块**: M-011
- **接口**: API-013 (`logEntry`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `IntentLogStore.logEntry({ intentText: '加收藏功能', blueprintDiff, projectId })`，When 调用，Then SQLite E-002 表插入记录，`intentText`/`blueprintDiff`/`timestamp` 非空，返回 `{ entryId: string, gitRef?: string }` [ARCH#§3.API-013 / arch-keel-data#§4.E-002 / prd#§2.F-013 AC-001]
  - [ ] AC-002: Given `decision: DecisionCard` 参数（F-010 AC-006 留痕），When `IntentLogStore.logEntry({ ..., decision })`，Then E-003 DecisionRecord 插入与 entryId 关联的记录，含 `conflict`/`options`/`resolution` [arch-keel-data#§4.E-003 / prd#§2.F-013 AC-001]
  - [ ] AC-003: Given `IntentLogStore.timeline({ limit: 10 })`，When 查询，Then 返回最近 10 条 `{ entryId, plainSummary, timestamp, entryKind: 'change' }`，响应时间 ≤3s [ARCH#§3.API-013 operations timeline / prd#§2.F-013 AC-003]
- **deliverables**:
  - [ ] `packages/intent-log/src/IntentLogStore.ts`
  - [ ] `packages/intent-log/src/timeline/TimelineView.ts`
  - [ ] `packages/intent-log/src/index.ts`
  - [ ] `packages/intent-log/tests/intent-log-store.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-011
  - arch-keel-api#§3.API-013
  - arch-keel-data#§4.E-002
  - arch-keel-data#§4.E-003
  - prd-keel-f010-f018#§2.F-013

---

### T-042: 时间线视图与时光机回滚（M-011）

- **目标**: 实现 `TimelineView`（自然语言时间线展示，不含 git 哈希/代码 diff）与 `RollbackEngine`（蓝图+代码+健康灯同步恢复），含 `UntrackedChangeGuard`（回滚前通道外改动提示）
- **模块**: M-011
- **接口**: API-013 (`rollback`)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given E-002 中有 3 条历史条目，When `TimelineView.query({ limit: 10 })`，Then 返回 `{ entries: [{ entryId, plainSummary: '添加了用户收藏文章功能', timestamp, entryKind: 'change' }] }`；无 commit 哈希、git 命令、代码 diff [prd#§2.F-013 AC-002 / ARCH#§3.API-013 timeline]
  - [ ] AC-002: Given `RollbackEngine.rollback({ entryId })`（无通道外改动），When 调用，Then 蓝图文件恢复到该 entryId 时刻的 `blueprintDiff` 状态，E-005 HealthSnapshot 重算更新，返回 `{ restored: true }` [prd#§2.F-013 AC-004/AC-005]
  - [ ] AC-003: Given 回滚前 `UntrackedChangeGuard.detect()` 发现通道外改动（F-002 AC-006 检测），When `RollbackEngine.rollback({ entryId })`，Then 返回 `{ restored: false, untrackedChangePrompt: '有没记录在时间线上的改动，回到过去会丢掉它们，确定继续吗？' }`；不执行回滚（需用户二次确认）[prd#§2.F-013 AC-006 / ARCH#§3.API-013 rollback]
- **deliverables**:
  - [ ] `packages/intent-log/src/rollback/RollbackEngine.ts`
  - [ ] `packages/intent-log/src/rollback/UntrackedChangeGuard.ts`
  - [ ] `packages/intent-log/tests/rollback-engine.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-011
  - arch-keel-api#§3.API-013
  - prd-keel-f010-f018#§2.F-013

---

### T-043: 时光机页面（P-008，M-011）

- **目标**: 实现 `?panel=timeline` 右侧抽屉：`TimelineEntry`（UC-012）竖向时间线 + 回滚确认（UC-016 destructive）
- **模块**: M-011
- **接口**: API-013
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `timeline` 返回 3 条条目（entryKind: change/decision/deploy），When `TimelineDrawer` 渲染，Then 各条 `UC-012 TimelineEntry` 变体正确（change/decision/deploy）；current 点 `color.keel` 高亮；不展示 git 哈希 [ui-spec-keel-pages-lifecycle#P-008 / ui-spec-keel-components#UC-012]
  - [ ] AC-002: Given 用户点击某条目「回到这里」，When `onRollback` 触发，Then 展示 `UC-016 ConfirmModal kind="destructive" danger={true}`「确定回到这个时间点吗？」；确认后调用 `RollbackEngine.rollback({ entryId })` [prd#§2.F-013 AC-004 / ui-spec-keel-components#UC-016]
  - [ ] AC-003: Given `rollback()` 返回 `{ restored: false, untrackedChangePrompt: '...' }`，When `TimelineDrawer` 收到，Then 展示 `UC-010 DecisionCard kind="untracked-rollback"` 通俗提示；用户二次确认前不回滚 [prd#§2.F-013 AC-006 / ui-spec-keel-components#UC-010]
- **deliverables**:
  - [ ] `apps/workspace/src/timeline/TimelineDrawer.tsx` — P-008 抽屉（`?panel=timeline`）
  - [ ] `apps/workspace/src/timeline/TimelineEntry.tsx` — UC-012
  - [ ] `apps/workspace/src/timeline/TimelineDrawer.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-lifecycle#§3.P-008
  - ui-spec-keel-components#§2.UC-012
  - ui-spec-keel-components#§2.UC-016
  - prd-keel-f010-f018#§2.F-013

---

### T-044: 安全体检三类检查器（M-012 SecurityAuditor）

- **目标**: 实现 `SecurityAuditor`（M-012）三类检查器：认证配置（`authn_config`）/ 数据权限（`data_authz`）/ 密钥泄露（`secret_leak`）；输出 `checks[]` 红绿灯
- **模块**: M-012
- **接口**: API-014 (`securityAudit`)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: true
- **tdd_acceptance**:
  - [ ] AC-001: Given 目标应用代码含未受认证保护的受保护路由（fail-open 配置），When `SecurityAuditor.audit({ scope: 'repo' })`，Then 返回 `{ checks: [{ category: 'authn_config', light: 'red', plain: '发现未受保护的路由…', fixEntry: '...' }] }`；`score < 100` [ARCH#§3.API-014 / prd#§2.F-014 AC-002 / arch-keel#§3.2]
  - [ ] AC-002: Given 代码文件含硬编码字符串 `sk-abc123456789`（API Key 格式），When `SecurityAuditor.audit({ scope: 'repo' })`，Then `checks` 中存在 `{ category: 'secret_leak', light: 'red', plain: string（非空） }`；`result.secretsFound >= 1`；`result.score < 100` [arch-keel#§3.2 密钥泄露 / prd#§2.F-014 AC-002]
  - [ ] AC-003: Given 三类全绿（无认证漏洞、无越权路径、无密钥泄露），When `SecurityAuditor.audit(...)`，Then 返回 `{ score: 100, checks: [{ light: 'green' }, { light: 'green' }, { light: 'green' }] }` [prd#§2.F-014 AC-002]
  - [ ] AC-004（错误路径）: Given 审计无法访问代码目录，When `SecurityAuditor.audit(...)`，Then 返回 `KeelError { code: 'AUDIT_INACCESSIBLE', plainMessage: '...' }` [ARCH#§3.API-014 error]
- **deliverables**:
  - [ ] `packages/deploy/src/auditor/SecurityAuditor.ts` — 三类检查器编排
  - [ ] `packages/deploy/src/auditor/AuthnConfigChecker.ts`
  - [ ] `packages/deploy/src/auditor/DataAuthzChecker.ts`
  - [ ] `packages/deploy/src/auditor/SecretLeakChecker.ts`
  - [ ] `packages/deploy/src/auditor/SecurityScoreCard.ts`
  - [ ] `packages/deploy/src/index.ts`
  - [ ] `packages/deploy/tests/security-auditor.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-012
  - arch-keel-api#§3.API-014
  - arch-keel#§5.2
  - prd-keel-f010-f018#§2.F-014

---

### T-045: 一键上线部署驱动（M-012 DeployDriver）

- **目标**: 实现 `DeployDriver`（M-012）：安全体检全绿后触发构建+部署，返回可访问地址；上线动作记入意图日志（`entryKind: 'deploy'`）；红灯阻断
- **模块**: M-012
- **接口**: API-014 (`deploy`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `DeployDriver.deploy({ confirmedGreen: true })`，When 安全体检全绿已确认，Then 触发构建部署流程，返回 `{ deployed: true, url: 'https://...' }`；同时调用 `IntentLogStore.logEntry({ entryKind: 'deploy', ... })` [ARCH#§3.API-014 deploy / prd#§2.F-014 AC-001/AC-004]
  - [ ] AC-002: Given `confirmedGreen: false`（存在红灯），When `DeployDriver.deploy({ confirmedGreen: false })`，Then 返回 `{ deployed: false, blockedByRed: ['authn_config', 'secret_leak'] }`；不触发实际部署 [prd#§2.F-014 AC-003 / ARCH#§3.API-014 deploy]
- **deliverables**:
  - [ ] `packages/deploy/src/driver/DeployDriver.ts`
  - [ ] `packages/deploy/src/driver/DeployGate.ts` — 红灯阻断
  - [ ] `packages/deploy/tests/deploy-driver.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-012
  - arch-keel-api#§3.API-014
  - prd-keel-f010-f018#§2.F-014
  - arch-keel-modules#§2.M-011

---

### T-046: 上线+安全体检页面（P-009，M-012）

- **目标**: 实现 `?dialog=deploy` 居中模态：`SecurityChecklistCard`（UC-011）+ 上线按钮（has-red 时 disabled）+ 体检完成后 Toast
- **模块**: M-012
- **接口**: API-014
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `securityAudit` 返回 `{ checks: [{ category: 'authn_config', light: 'red', plain: '...', fixEntry: '...' }, { category: 'data_authz', light: 'green', ... }, { category: 'secret_leak', light: 'green', ... }] }`，When `DeployModal` 渲染，Then `UC-011 has-red` 变体：`authn_config` 展开说明 + 修复入口；「上线」按钮 disabled [ui-spec-keel-pages-lifecycle#P-009 / ui-spec-keel-components#UC-011]
  - [ ] AC-002: Given 体检全绿（`canDeploy: true`），When 用户点击「上线」，Then 调用 `DeployDriver.deploy({ confirmedGreen: true })`；成功后模态关闭 + `UC-015 Toast success "应用已上线：https://..."` [prd#§2.F-014 AC-001 / ui-spec-keel-components#UC-015]
  - [ ] AC-003: Given 部署失败，When `DeployModal` 收到错误，Then 展示通俗重试提示，不暴露部署日志/错误码 [ui-spec-keel-pages-lifecycle#P-009]
- **deliverables**:
  - [ ] `apps/workspace/src/deploy/DeployModal.tsx` — P-009 模态（`?dialog=deploy`）
  - [ ] `apps/workspace/src/deploy/SecurityChecklistCard.tsx` — UC-011
  - [ ] `apps/workspace/src/deploy/DeployModal.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-lifecycle#§3.P-009
  - ui-spec-keel-components#§2.UC-011
  - prd-keel-f010-f018#§2.F-014

---

### T-047: 代码仓导出与工具配置编译（M-013）

- **目标**: 实现 `RepoExporter` + `ToolConfigCompiler`（M-013）：导出标准 JS/TS 仓库，编译落地 dependency-cruiser / eslint / tsconfig / JSON Schema；`SecretStripper` 确保 `secretsFound === 0`
- **模块**: M-013
- **接口**: API-015 (主操作 `exportRepo`)
- **task_kind**: feature
- **tdd_mode**: standard
- **tdd_refactor**: auto
- **security_sensitive**: true
- **tdd_acceptance**:
  - [ ] AC-001: Given `RepoExporter.exportRepo()`，When 调用，Then 返回 `{ repoPath: '/tmp/keel-export-xxx', includesToolConfigs: true, secretsFound: 0 }`；导出目录含 `.dependency-cruiser.cjs` / `.eslintrc.json` / `tsconfig.json` / `blueprint.schema.json` [ARCH#§3.API-015 / prd#§2.F-015 AC-002]
  - [ ] AC-002: Given 原始代码含硬编码 `sk-abc123` 字符串，When `SecretStripper.strip(repoPath)`，Then 搜索导出目录 `grep -r 'sk-' <repoPath>` 返回空结果；`SecretStripper.strip()` 返回值 `secretsFound === 0` [prd#§2.F-015 AC-004 / ARCH#§3.API-015]
  - [ ] AC-003: Given 导出仓库，When 在无 Keel 环境执行 `npx dependency-cruiser packages/`，Then 结构违规被检出（工具配置有效）[prd#§2.F-015 AC-002/AC-003]
  - [ ] AC-004（production-path）: `packages/export/src/RepoExporter.ts` 调用 `toolConfigCompiler.compile(blueprintIR)` 生成工具配置文件，写入导出目录 [ARCH#§2.M-013]
- **deliverables**:
  - [ ] `packages/export/src/RepoExporter.ts`
  - [ ] `packages/export/src/config/ToolConfigCompiler.ts` — 蓝图约束→标准工具配置
  - [ ] `packages/export/src/security/SecretStripper.ts`
  - [ ] `packages/export/src/index.ts`
  - [ ] `packages/export/tests/repo-exporter.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-013
  - arch-keel-api#§3.API-015
  - arch-keel#§5.2
  - prd-keel-f010-f018#§2.F-015

---

### T-048: 交接包生成（M-013 HandoffPackager）

- **目标**: 实现 `HandoffPackager`（M-013）：生成 Markdown 交接包（蓝图→架构文档 + 意图日志→决策史 + 边界规则→上手指南 + 可选技术视图投影）
- **模块**: M-013
- **接口**: API-015 (`generateHandoff`)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `HandoffPackager.generateHandoff()`，When 调用，Then 返回 `{ handoffPath: '/tmp/keel-handoff-xxx', sections: ['arch_doc', 'decision_history', 'boundary_guide'] }`；`handoffPath` 目录含 `architecture.md` / `decisions.md` / `boundary-guide.md` 三个 Markdown 文件 [ARCH#§3.API-015 generateHandoff / prd#§2.F-016 AC-001/AC-002]
  - [ ] AC-002: Given `decisions.md` 文件路径，When 读取文件内容，Then 文件包含至少一段含 `intentText` 来源文本的段落（从 E-002 `intentText` 字段填充）；文件内容不含 40 位 git hash（`/[0-9a-f]{40}/` 匹配为空）；不含 `diff --git` 字符串；不含 `error:` 前缀的编译日志行 [prd#§2.F-016 AC-001/AC-002]
- **deliverables**:
  - [ ] `packages/export/src/handoff/HandoffPackager.ts`
  - [ ] `packages/export/src/handoff/ArchDocGenerator.ts` — 蓝图→架构文档
  - [ ] `packages/export/src/handoff/DecisionHistoryGenerator.ts` — 意图日志→决策史
  - [ ] `packages/export/src/handoff/BoundaryGuideGenerator.ts` — 规则→上手指南
  - [ ] `packages/export/tests/handoff-packager.test.ts`
- **context_load**:
  - arch-keel-modules#§2.M-013
  - arch-keel-api#§3.API-015
  - prd-keel-f010-f018#§2.F-016
  - arch-keel-modules#§2.M-011

---

### T-049: 导出交接页面（P-010，M-013）

- **目标**: 实现 `?dialog=export` 居中模态：导出仓库 + 生成交接包两个区块，含进度状态与完成下载入口
- **模块**: M-013
- **接口**: API-015
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given `ExportModal` 渲染，When 用户点击「导出代码仓库」，Then `getByTestId('export-progress')` 渲染（loading spinner 可见）；导出完成后 `getByTestId('download-repo-link')` 返回非 null 的 `<a>` 元素，DOM 文本含「脱离 Keel 亦可自我强制」[prd#§2.F-015 AC-005 / ui-spec-keel-pages-lifecycle#P-010]
  - [ ] AC-002: Given 用户点击「生成交接包」，When 完成，Then 展示 Markdown 交接包下载入口（architecture/decisions/boundary-guide）；文案强调「代码全程归你、可随时带走」[prd#§2.F-016 AC-002 / ui-spec-keel-pages-lifecycle#P-010]
- **deliverables**:
  - [ ] `apps/workspace/src/export/ExportModal.tsx` — P-010 模态（`?dialog=export`）
  - [ ] `apps/workspace/src/export/ExportModal.test.tsx`
- **context_load**:
  - ui-spec-keel-pages-lifecycle#§3.P-010
  - prd-keel-f010-f018#§2.F-015
  - prd-keel-f010-f018#§2.F-016

---

### T-050: 自由实现区标注 + 地图展示（P-011，M-001 + M-009）

- **目标**: 实现 escape hatch `Module` 标注流程（经意图流程写入 `escape_hatch: true` + `intent`）及地图上「自由实现区」显著标识（UC-004 brass 描边）与详情卡展示（UC-005 escape-hatch 变体）
- **模块**: M-001, M-009
- **接口**: API-011 (MapNode.isEscapeHatch)
- **task_kind**: feature
- **tdd_mode**: light
- **tdd_refactor**: skip
- **security_sensitive**: false
- **tdd_acceptance**:
  - [ ] AC-001: Given 蓝图中 Module `payment` 含 `escape_hatch: true` 且 `intent: '支付需要复杂状态机逻辑，暂不纳图'`，When `MapProjector.project(blueprintIR)`，Then 返回的 `nodes` 数组中 `id === 'payment'` 的节点满足 `node.isEscapeHatch === true` 且 `node.escapeHatchIntent === '支付需要复杂状态机逻辑，暂不纳图'` [ARCH#§3.API-011 types.MapNode.isEscapeHatch / prd#§2.F-017 AC-003]
  - [ ] AC-002: Given `MapNode.isEscapeHatch: true`，When `CapabilityNode` 渲染，Then 节点 CSS class 包含 `escape-hatch-border`（虚线描边）；`getByTestId('escape-hatch-badge')` 返回非 null DOM（brass 标识存在）；普通 Module 节点（`isEscapeHatch: false`）该 badge 不存在 [prd#§2.F-017 AC-003 / ui-spec-keel-components#UC-002/UC-004]
  - [ ] AC-003: Given escape hatch Module 点击后 `UC-005 CapabilityDetailCard` 弹出，When 渲染，Then 展示 `escapeHatchIntent: '支付需要复杂状态机逻辑…'` 及「该区域不受图约束保护」提示文案 [prd#§2.F-017 AC-003 / ui-spec-keel-components#UC-005]
  - [ ] AC-004: Given 意图流程标注 escape hatch 时 `intent` 为空字符串，When `ContractFreezer.freeze()` 或 `SchemaValidator.validate(ir)` 执行，Then 返回 `{ valid: false, schemaErrors: [{ path: '/modules/payment/intent', message: '自由实现区必须填写意图说明' }] }`；不执行冻结，E-004 表中无新记录 [prd#§2.F-017 AC-001]
- **deliverables**:
  - [ ] `apps/workspace/src/map/nodes/EscapeHatchBadge.tsx` — brass 描边修饰组件
  - [ ] `apps/workspace/src/map/detail/CapabilityDetailCard.tsx` — UC-005（含 escape-hatch 变体）
  - [ ] `apps/workspace/src/map/detail/CapabilityDetailCard.test.tsx`
- **context_load**:
  - arch-keel-modules#§2.M-001
  - arch-keel-modules#§2.M-009
  - arch-keel-api#§3.API-011
  - ui-spec-keel-components#§2.UC-004
  - ui-spec-keel-components#§2.UC-005
  - ui-spec-keel-pages-lifecycle#§3.P-011
  - prd-keel-f010-f018#§2.F-017

---

### T-051: [VALIDATION] 生命周期功能：上手 + 时光机 + 上线 + 导出

- **目标**: 用户手动验证 Sprint 5 生命周期功能产出
- **task_kind**: validation
- **模块**: M-011, M-012, M-013, M-014
- **tdd_acceptance**: N/A（validation 任务不走 TDD，由 orchestrator 展示验证清单后用户手动确认）
- **deliverables**: N/A（validation 任务不产出代码文件，由前置任务 T-038~T-050 的 deliverables 覆盖）
- **验证清单**:
  - [ ] `/onboarding` 向导全程无终端命令文本；`UC-019 smoke-ok` 绿勾在冒烟通过后出现
  - [ ] `UsageMeter` 展示措辞明确「执行引擎侧消耗」而非「Keel 用量」
  - [ ] 添加一个意图并确认后，`?panel=timeline` 抽屉显示「添加了…功能」自然语言摘要（无 git 哈希/代码 diff）
  - [ ] 时光机回滚：选择历史点 → 确认模态（destructive）→ 回滚后蓝图和代码一致态，健康灯同步刷新
  - [ ] 回滚前有通道外改动时，`UC-010 untracked-rollback` 先出现，用户二次确认后才回滚
  - [ ] `SecurityAuditor` 在测试目标应用（含 fail-open 路由）产出 `authn_config: red`；修复后重跑 `green`
  - [ ] 存在红灯时「上线」按钮 disabled；全绿时可点，点击后 Toast 显示可访问地址
  - [ ] `RepoExporter` 导出目录含 `.dependency-cruiser.cjs`；在无 Keel 环境下 `npx dependency-cruiser` 可运行且检出结构违规
  - [ ] 导出完成提示文案含「脱离 Keel 亦可自我强制」
  - [ ] escape hatch Module 在地图上显示虚线描边 + brass 标识；详情卡展示 intent 文案；`intent` 为空时流程拒绝
- **前置任务**: [T-038, T-039, T-040, T-041, T-042, T-043, T-044, T-045, T-046, T-047, T-048, T-049, T-050]
- **context_load**:
  - prd-keel-f010-f018#§2.F-012
  - prd-keel-f010-f018#§2.F-013
  - prd-keel-f010-f018#§2.F-014
  - prd-keel-f010-f018#§2.F-015
  - prd-keel-f010-f018#§2.F-016
  - prd-keel-f010-f018#§2.F-017
