---
id: "adr-0004-gate-severity-tiers"
doc_type: decision
author: Alex
status: approved
deps: []
---

# 0004. 门禁严重度：结构=硬，风格=软

状态：accepted（2026-06-09，原 D4；沿用 PRD 附录 B.7）

## 背景

门禁若一律硬阻断，会把 LLM 逼到空转或绕路（风险 R2）。需要分级。

## 决定

- **硬阻断（hard）**：破依赖边界、重复实现、契约不符等结构性违规。
- **软告警（soft）**：命名、目录、轻微冗余等风格类问题，记入项目健康，稍后调和。

## 后果

- 元模型 `Constraint.severity` 枚举为 `hard / soft`。
- 状态机中硬违规触发 `GATE_BLOCKED`，软问题流过 `VERIFYING`（黄）。
