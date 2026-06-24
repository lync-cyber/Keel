---
id: "adr-0002-test-web-prototype-first"
doc_type: decision
author: Alex
status: approved
deps: []
---

# 0002. 先测现有 web 原型（vs 嵌入式）

状态：accepted（2026-06-09，原 D2）

## 背景

R1 可读半边需要可用性测试。测试对象有两个候选：现有 web/博客原型，或嵌入式方向。后者需额外平移工作。

## 决定

**先用现有 web 原型测**。三张原型（A 能力地图 / B 蓝图 diff / C 健康预警）已就绪、零额外平移，对核心可读假设最快最省地证伪。嵌入式留作 R1 通过后的补测。

## 后果

- R1 测试关键路径不被嵌入式平移阻塞。
- 嵌入式市场分析（`docs/reference/embedded-market-analysis.html`）降为 R1 通过后的后续项。
