---
id: research-arch-tech-eval-stack
doc_type: research
author: architect
status: approved
deps: ["prd-keel"]
---

# Tech-Eval 调研记录 — Keel 架构技术选型（2026-06-18）

> 架构阶段选型的版本新鲜度核验与对比矩阵。验证来源：web-search（2026-06 当前）。
> 决策权威落 `arch` §1.4；本文为可追溯调研痕迹。

## 1. 目标应用锁定技术栈（生成代码 + 一键上线目标）

用户选定方向：**Next.js + Postgres + 开源自托管认证**（理由：F-015 导出不锁定、可独立运行）。

| 组件 | 选型 | 当前稳定（2026-06） | 选型理由 | 重新评估条件 |
|------|------|--------------------|----------|--------------|
| 全栈框架 | **Next.js 16 (App Router)** | 16.2.x；Turbopack 默认、React 19.2、min Node 20、Pages Router 进维护期 | 全栈一体、React 生态、Server Actions 契合受控生成；App Router 为新项目官方默认 | Next.js 出现破坏性 17 大版本、或 RSC 模型重大变更 |
| 数据库 | **PostgreSQL** | 成熟 | 关系型、可自托管、schema 即 Entity 真相源；导出物用标准迁移工具即可独立运行 | 项目出现强文档型/图型数据需求（当前博客类场景无） |
| 认证 | **Better Auth**（替代初选 Auth.js） | 活跃稳定 | 见 §1.1 | 见 §1.1 |
| ORM/迁移 | **Drizzle**（候选，待 §2 细化时定） | 活跃 | TS 原生、schema 即代码、迁移可导出；与 Better Auth 的 schema 集成良好 | — |
| 部署目标 | 锁定托管（Vercel 或自托管 Node 容器，F-014 细化时定） | — | 锁定收窄解空间保证生成质量（§4.1）；导出物不绑定托管商 | — |

### 1.1 认证组件：Auth.js vs Better Auth（关键修正）

| 维度 | Auth.js v5 (NextAuth) | **Better Auth（推荐）** |
|------|----------------------|------------------------|
| 成熟度 | 生产可用但**至今仍 beta 标签**，API 仍在收敛 | 活跃、面向新项目定位清晰 |
| 维护者取向 | **维护者现把新项目导向 Better Auth** | 官方/社区主推 |
| 开源/自托管 | 开源 | 开源、TS 原生、框架无关 |
| 数据归属 | 适配器接外部存储 | **认证表直接落项目自有 Postgres** → 导出后零外部依赖 |
| 对 F-015 不锁定 | 中 | **高**（导出物自带全部认证逻辑与表） |
| 对 F-014 安全体检 | 可做认证配置检查 | 可做；schema 在库内更易做权限/越权检查 |

**结论**：采用 **Better Auth**。它比 Auth.js 更好满足用户选 Next.js+Postgres 开源栈的初衷（不锁定、可导出独立运行），且规避 Auth.js 永久 beta 风险。**此为对用户初选的 tech-eval 驱动修正，需用户确认后写入 §1.4。**

## 2. 执行引擎接入（F-018）

PRD F-018 AC-002 既定 ACP-first，本调研验证其成立：

- **ACP（Agent Client Protocol）**：协议稳定版 `1`，基于 JSON-RPC over stdio；2026-01 ACP Registry 上线，Zed / JetBrains 内建。
- **Claude Code** 经 `claude-agent-acp` adapter 接入；**Codex** 经 `codex-acp` adapter 接入 —— 对应 F-018「Claude Code 与 Codex 经 adapter 接入」。
- **回退**：Agent SDK / 无头 CLI（F-018 AC-003）提供更完整 hooks/subagents/sessions。
- **工具面**（F-018 AC-006）：Keel 核心能力以 MCP Server / 引擎 hooks / CLI 暴露，与 ACP 权限中介并列；具体组合 §架构细化时定。

**结论**：标准接缝 = ACP（权限中介 + 编辑后检查映射门禁）；原生回退 = Agent SDK/CLI；切换对上层透明（F-018 AC-004）。

## 3. 门禁检查器工具链（F-003 rule.type → checker）

schema `constraint.checker` 枚举（dependency-cruiser/eslint/ts-morph/tsc/schema/madge）全部当前可用：

| checker | 当前状态（2026-06） | 映射的 rule.type |
|---------|--------------------|------------------|
| dependency-cruiser | 17.4.x，活跃 | forbid_relation / allowed_relation_kinds / direction / via_port / access_only_via / no_outbound_except |
| ts-morph | 成熟活跃 | single_canonical（重复/相似实现扫描）；F-002 AST 逆向提取 |
| tsc | 随 TS 发布 | 编译门（F-003 AC-001） |
| eslint | 成熟 | via_port / access_only_via 的 import 白名单辅助 |
| madge | 维护中 | no_orphan（可达性） |

**结论**：门禁引擎以这套标准工具链为后端，导出物自带同款配置即可脱离 Keel 自我强制（F-015 AC-002）。Keel 核心引擎采用 **TypeScript 单栈**，与目标栈同语言、直接在进程内复用上述工具。

## 4. Keel 自身形态（用户选定）

- **桌面外壳延后**：v1 先做 Web 壳（复用现有静态原型 demo A/B/C 演进），F-012 桌面安装体验后置（与 §4.3 R1 优先一致）。
- 影响：F-012 AC-001/AC-003「检测本地执行引擎/后台连接」的桌面部分在 v1 以 Web 壳 + 本地引擎桥接的最小形态承载，完整桌面安装引导列为后续项；架构需保留外壳可替换接缝（Web 壳 → 后续 Tauri/Electron 不影响引擎核心）。

## Sources

- Next.js 版本：https://nextjs.org/blog/next-16-2 ，https://github.com/vercel/next.js/releases
- Auth.js / Better Auth：https://blog.logrocket.com/best-auth-library-nextjs-2026/ ，https://authjs.dev/getting-started/migrating-to-v5
- ACP：https://agentclientprotocol.com/get-started/agents ，https://zed.dev/blog/acp-registry ，https://github.com/cola-io/codex-acp
- 工具链：https://www.npmjs.com/package/dependency-cruiser ，https://github.com/pahen/madge
