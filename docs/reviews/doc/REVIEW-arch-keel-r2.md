---
id: "review-arch-keel-r2"
doc_type: review
author: reviewer
status: approved
deps: ["arch-keel", "arch-keel-modules", "arch-keel-api", "arch-keel-data"]
---
# 架构文档增量复审报告: arch-keel r2

**被审对象**: arch-keel（主卷）、arch-keel-modules、arch-keel-api、arch-keel-data（version 0.1, status: draft）
**本轮性质**: 增量复审（task_type=revision），仅验证 r1 各问题的修复落实情况，并检查是否引入新问题
**上一轮**: REVIEW-arch-keel-r1.md（verdict: needs_revision，2 HIGH / 8 MEDIUM / 3 LOW）

---

## r1 问题逐条核验

| 编号 | 严重等级 | 标题 | 判定 | 备注 |
|------|---------|------|------|------|
| R-001 | HIGH | F-004 回归守护缺乏独立模块承接 | RESOLVED | M-002 内部组件新增 RegressionGuard，职责边界明确两阶段串行（结构合规门→回归守护），含 ADR-0009 约束行 |
| R-002 | HIGH | GateReport 无回归字段 | RESOLVED | API-002 GateReport 重构含 structural+regressionReport 两段；RegressionBreak 类型定义完整；§5.3 与 M-009 HealthOverlay 均同步引用区分两类 red |
| R-003 | MEDIUM | §5.1 漂移 ≤3s 口径缺发现延迟说明 | RESOLVED | §5.1 明确补充"≤3s 仅约束检测执行时长；通道外发现延迟不计入，由 M-003 ChangeWatcher [ASSUMPTION] 决定"，与 PRD F-002 AC-005 口径对齐 |
| R-004 | MEDIUM | F-018 AC-005 托管执行器预留无体现 | RESOLVED | M-008 新增"托管执行器预留（F-018 AC-005）"段；API-009 engineKind 枚举加 `hosted` variant，v1 空实现，注明透明切换契约覆盖此 variant |
| R-005 | HIGH | M-009 健康来源误标 M-011 | RESOLVED | M-009 依赖列表订正为 M-001 + M-002/M-003（经 E-005 缓存）+ M-011（仅意图节点状态）；新增「数据来源澄清」段，明确 M-011 不拥有也不写 HealthSnapshot |
| R-006 | MEDIUM | F-004 v1 行为门描述存在混同风险 | RESOLVED | M-002 职责边界显式加约束行："F-004 v1 回归守护仅含结构门与契约测试，行为门远期纳入（ADR-0009），不在 v1 范围内" |
| R-007 | MEDIUM | API-012/API-015 空 key request 写法不规范 | RESOLVED | 两处均改为 `request: none`，与其他无入参操作形式统一 |
| R-008 | MEDIUM | M-003 ChangeWatcher 进程边界未说明 | RESOLVED | M-003 ChangeWatcher 组件注释补充：运行在本地引擎服务进程(Node)，事件经 WebSocket 推送至 Web 壳，监听范围限 features/+蓝图文件 |
| R-009 | LOW | §1.4 React Flow 候选无风险说明 | RESOLVED | §1.4 技术栈表该行补充 [ASSUMPTION] 注：待 ui-spec 最终选型，须核验 React Flow v12 bundle size 对 ≤2s 加载影响，备选 D3 自绘 |
| R-010 | MEDIUM | M-007 ErrorClassifier 判定机制空白 | RESOLVED | M-007 ErrorClassifier 组件说明补充：经 frozenContractSetId 读 E-004，对 Port 签名/Entity schema 做结构比较；净变化→改变语义→上浮，否则→技术性→静默自愈；[ASSUMPTION] 精确比较算法待 dev 阶段细化 |
| R-011 | LOW | API-009 ContextBundle/AgentResult 无定义 | RESOLVED | API-009 types 段新增 ContextBundle（capabilityId/contract/directDepSignatures）与 AgentResult（success/gateReport?/output?）两类型骨架 |
| R-012 | LOW | arch-keel-data required_sections 格式 INFO | 无需修改（r1 已注明 INFO） | 确认：r1 报告已说明此条可作 INFO 参考，不要求修改，已核对文档无改动且 Layer 1 通过 |
| R-013 | MEDIUM | §5.2 localhost 通信安全未说明 | RESOLVED | §5.2 新增「Web 壳 ↔ 本地引擎服务通信约定」段：localhost HTTP + Origin 白名单防 CSRF；凭据明文不经前端；[ASSUMPTION] 具体鉴权方案待 dev-plan |

**RESOLVED: 12 条**（含 R-012 INFO 确认 1 条）
**UNRESOLVED: 0 条**

---

## 新增问题

增量复审未发现新增 CRITICAL、HIGH、MEDIUM 或 LOW 问题。

---

## 审查结论

**verdict**: approved

**问题统计（本轮）**:
- CRITICAL: 0
- HIGH: 0
- MEDIUM: 0
- LOW: 0

**一句话结论**: r1 全部 13 条问题（含 2 HIGH、8 MEDIUM、3 LOW）均已在修订版中有效落实，未发现新增问题，架构文档可进入下一阶段。
