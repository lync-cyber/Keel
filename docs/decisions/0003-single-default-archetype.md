---
id: "adr-0003-single-default-archetype"
doc_type: decision
author: Alex
status: approved
deps: []
---

# 0003. v1 只上 1 个默认架构原型

状态：accepted（2026-06-09，原 D3）

## 背景

附录 B.9 列了多个架构原型（特性切片、分层、六边形、内容站）。v1 是否一次上多个？

## 决定

v1 **只上 1 个默认原型**：特性切片 / 模块化单体（`modular-monolith`），由意图自动选中。先把闸门跑通再扩。

## 后果

- 元模型仍保留多原型能力（换 `unit_kinds / allowed_relation_kinds / constraints` 三旋钮即可扩），但实例只用一个。
- 规范实例 `blueprint.example.yaml` 内联 `modular-monolith` 原型。
