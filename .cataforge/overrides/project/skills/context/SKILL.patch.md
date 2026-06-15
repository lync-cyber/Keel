<!-- Section patch: a '## Title' matching a base section overrides it; a new title is appended. -->

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
