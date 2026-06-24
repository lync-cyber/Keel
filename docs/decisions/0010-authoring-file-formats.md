---
id: "adr-0010-authoring-file-formats"
doc_type: decision
author: Alex
status: approved
deps: []
---

# 0010. 概念阶段文件形态

状态：accepted（2026-06-10）

## 背景

概念阶段（无代码）要优化的是人工审查与迭代速度，不是机器强制。用一种格式硬扛三类不同产物会互相打架。CUE/Pkl 等的价值在「部署前校验、强类型」（机器门那侧），现在引入只增加仪式感、把半技术评审者挡在门外。

## 决定

把产物分三层、各用最省事的形态：

- **探索性**（决策、草图）：ADR Markdown（一决策一文件）+ Mermaid 内嵌。
- **真相源**（原语 + 实例）：`blueprint.schema.json` 元模型 + `blueprint.example.yaml`（YAML + 注释）。
- **机器门**（形态校验）：`blueprint.schema.json`，降级为不手读、只跑校验。

**现在不上 CUE/Pkl**；等到要编译强制时再评估。YAML 优于 TOML/KDL（深嵌套 + 注释 + 已在用）。

## 后果

- 新增 `docs/decisions/`（ADR 一决策一文件）。
- 一旦原语定稿，其视图从规范实例渲染，不另维护第二份真相。
- 升级路径：要强制时原语定义 → CUE 或保留 JSON Schema；图 → 由蓝图生成 Structurizr/D2。
