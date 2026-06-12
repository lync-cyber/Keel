---
id: "adr-0008-add-rir-and-traceability"
doc_type: decision
author: Alex
status: approved
deps: []
---

# 0008. 近期加 R-IR 需求层 + 追溯链

状态：accepted（2026-06-10）

## 背景

对照技术报告的「需求→架构→代码」管线，蓝图缺**第一锚点**：报告 §05 阶段0 的需求 IR（R-IR，带 ID + 可测验收准则），以及阶段1/4 的 R→A 追溯与需求覆盖率。原 demo 直接从自然语言意图跳到架构 diff，跳过了需求层。

补 R-IR 是纯声明式工作，不依赖执行引擎，可在 R1 闸门内完成，且能顺带验证「验收准则/需求能否被非程序员读懂」，扩大 R1 证据面。

## 决定

**近期就在蓝图加 R-IR + 追溯链**，并入 R1 测试：

- 新增 `requirements[]` 原语：`id` + `plain` + `acceptance[]`（可测准则，带 `oracle` 标注）；`statement` 用 EARS 句式，最小版可省。
- `Unit` 与 `Constraint` 加 `satisfies: [需求 id]` 回链。
- 编译期校验：每条需求至少被一个能力/约束 satisfies（需求覆盖率）。

## 后果

- schema v0.2 新增 `requirement` / `acceptance` def 与 `satisfies` 字段。
- 规范实例 v9 含 5 条需求，全部被 satisfies 覆盖（已校验通过）。
- R1 测试方案增设一项：需求与验收准则的可读性。
