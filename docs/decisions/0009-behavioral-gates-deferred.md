# 0009. 行为门远期纳入，现在预留契约占位

状态：accepted（2026-06-10）

## 背景

技术报告的「验证门」有**行为半边**：契约派生单测 + 属性测试，用来拦单步逻辑幻觉（报告 §05 阶段3、图⑤左）。而 PRD G3 当前措辞是「门禁只保证结构、不保证功能，行为一律交人工」——两者正面冲突。

报告同时承认诚实边界：门只在能写 oracle 处有牙，写不出 oracle 的回退人工。区别在于：Keel 当前对**有 oracle 的地方也没设行为门**，而非「无 oracle 才交人工」。

## 决定

**行为门远期纳入**——在能写出可执行判据（oracle）的地方设契约派生单测/属性测试；写不出的回退人工。近期不实现测试基建，只做两件预留：

1. `Port` 加可选 `contract`（`pre`/`post`/`invariant`，子句 `plain` 必填、`formal` 可选），先留空/只写 plain，避免远期补行为门时翻修结构。
2. `acceptance.oracle` 标注 `test/property/type/schema/human`，预先区分「将来自动设门」与「回退人工」。

并微调 PRD G3 措辞：从「行为一律交人工」改为「**无 oracle → 人工；有 oracle → 远期设行为门**」。

## 后果

- schema v0.2 加 `clause` def 与 `Port.contract`；规范实例在 `core/auth.api`、`favorite.contract` 上示范占位。
- 行为门的实现 gated 在 R1 之后（远期），与执行引擎一同落地。
- 待办：PRD G3 文案更新（见 CLAUDE.md 待办）。
