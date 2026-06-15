---
name: context
description: "统一上下文 I/O — 按需读取章节/实体、查询追溯关系、生成与写入、门禁校验。文档生命周期的单一入口；后端(知识图谱/文件)由框架按配置方案透明路由,调用方只表达意图。按操作分支见 references/。"
argument-hint: "<操作: navigate|generate|review|consistency|query>"
suggested-tools: Bash, Read, Glob, Grep
depends: []
disable-model-invocation: false
user-invocable: true
---

# 统一上下文 I/O (context)

文档生命周期的单一能力入口。读取、关系查询、生成/写入、校验都经 `cataforge context` / `cataforge docs` 表达**意图**;由哪个后端服务、用何种保真度,由框架按项目配置的上下文方案路由,调用方无需感知。后端能力**非对称**:读取/生成/校验按方案路由且在后端不可达时降级;关系追溯(query 分支)是图原生能力,需图后端就绪(无对等文件回退)。

## 能力边界
- 能做: 按需加载章节/实体、依赖与追溯查询、文档生成与写入、单文档与跨文档门禁校验
- 不做: 内容决策(由调用 Agent 负责)、代码审查(由 code-review 负责)、框架元资产审查(由 framework-review 负责)

## 输入规范
- 操作分支: `navigate` | `generate` | `review` | `consistency` | `query`
- 引用: `doc_id#§N[.item]`(如 `prd#§2.F-001`、`arch#§1`)
- 各分支的参数与命令见对应 reference 文件

## 输出规范
- 读取: 目标章节/实体内容(markdown 形式,后端无关)
- 查询: 依赖/追溯结果
- 生成/写入: authoring 落图 + 定稿导出路径
- 校验: 审查/一致性报告 + 三态结论

## 操作指令
按意图选分支,详见 reference:

1. **navigate** — 按需读取章节/实体、依赖展开、token 预算。见 [references/navigate.md](references/navigate.md)
2. **generate** — 取模板、authoring 落图、定稿导出、拆分。见 [references/generate.md](references/generate.md)
3. **review** — 单文档门禁(脚本检查 + 语义审查)。见 [references/review.md](references/review.md)
4. **consistency** — 跨文档一致性(覆盖矩阵、契约对齐)。见 [references/consistency.md](references/consistency.md)
5. **query** — 自然语言追溯查询。见 [references/query.md](references/query.md)

## Anti-Patterns
- 禁止: 一次性 Read 超过 200 行的整篇文档 — 按章节/条目加载是核心价值,全文 Read 吃掉数千 token 还引入无关噪声
- 禁止: 在调用面判断"该走哪个后端" — 后端选择由框架路由,prompt 里复述分发条件会随实现漂移
- 禁止: 跳过 generate 的定稿步骤直接退出 — 定稿负责持久化与事件写入,跳过会让后续读取/校验找不到新内容
- 避免: 同一会话重复加载同一章节 — 内容已在上下文,重复加载浪费 turn 和 token
## KG store 版本管理卫生 (kg-first)
本项目 `context.strategy=kg-first`，`.cataforge/kg/store/` 是被 git 跟踪的活 RocksDB 库（`.gitignore` 仅忽略 `LOG` / `LOCK` / `IDENTITY`，`MANIFEST-*` / `OPTIONS-*` / `CURRENT` / `*.log` WAL 保持跟踪以保证 fresh clone 可开库）。RocksDB 以读写模式 open 时会滚动 `MANIFEST-*` / `OPTIONS-*` / `CURRENT` / `*.log`，**即便只读命令**（`cataforge context status` / `kg query` / `kg reconcile` / `doctor` / `feedback`）也会触发，导致工作树出现「未提交变更」并打断 stop-hook / pre-commit 收尾。

判定与处置：

- **业务数据未变（纯 bookkeeping 滚动）**：`git status` 下 `.cataforge/kg/store/` 的改动**仅限** `CURRENT` / `MANIFEST-*` / `OPTIONS-*` / `*.log`，无 `.sst` 增删。此为 RocksDB 内部滚动，非真实图谱变更，**不要提交噪声**，按下式还原到已提交状态：

  ```bash
  git checkout HEAD -- .cataforge/kg/store/ && git clean -f .cataforge/kg/store/
  ```

- **真实图谱变更**：实体/关系变化时会出现 `.sst` 文件增删（多见于 `kg import` / `kg add` / `kg update` / `kg delete` 之后）。此类才 `git add .cataforge/kg/store/` 一并提交。

- **收尾顺序**：先完成所有 KG 写操作并提交 `.sst` 变更，再跑只读校验（`kg validate` / `reconcile` / `doctor`）；校验完用上式清掉滚动，确保交付时工作树干净。

- 根因在 CLI 以读写模式打开 store（只读路径未走 RocksDB read-only / secondary 模式），属上游 CataForge 范畴；本节是项目级运维约定，不改变 CLI 行为。
