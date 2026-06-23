---
id: "research-ui-design-direction-keel"
version: "0.1.0"
doc_type: research
author: ui-designer
status: draft
deps: ["prd-keel", "arch-keel"]
---

# UI 设计方向与设计 Token 锁定 — Keel（ui_design 阶段，Step 1+2 产出）

> 本 note 是 ui_design 阶段 inline 承载 ui-designer 的发散澄清与设计方向/Token 决策留痕（Inline Role Execution Protocol Step 2）。
> 用户已确认方向：**采纳原型 A/B/C 信息架构与交互骨架，但演进视觉**（借 /frontend-design 强化），**用 Penpot MCP 建设计系统**，ui-spec **全量覆盖 F-001~F-018**。
> 下游：本 note 的 §1 Token 表是 `ui-spec-keel-theme` 卷与 Penpot 设计系统的单一事实源。

## 0. 设计方向（Calm Cartography / 沉静海图 · 暖调蓝图）

- **产品调性**: 稳固 · 克制 · 可信（像一份郑重的「应用宪章 / 海图」），而非花哨工具的廉价感。
- **目标用户场景**: 「重来者」非程序员——用过 Lovable/Bolt/v0 撞过「越改越乱」的腐化墙，看不懂代码，核心情绪诉求是**确定感与掌控感**。视觉要传达「结构稳固、看得懂、拿得走」，桌前长时间使用。
- **核心隐喻**: Keel = 龙骨；能力地图 = 你应用的海图/蓝图。
- **视觉策略**:
  1. **暖纸底色**取代原型冷灰 `#f5f6f8` → `#f1ece1`，给蓝图以「可信赖绘图纸」的温度。
  2. **航海藏青主色 `#2b3f72`**取代泛滥 SaaS 蓝 `#2563eb`——藏青=深海/龙骨/郑重；diff「新增」归入主色，与「会碰到」的紫拉开。
  3. **三声部排版**：衬线「宪章」标题（Noto Serif SC）+ 等宽「制图标签」（IBM Plex Mono 跑图例/状态/数字）+ 无衬线正文（Noto Sans SC）。
  4. **饱和色只留给语义**：健康三态（绿/琥珀/红，情绪信号层）+ diff（新增/会碰到/不动）；其余一律暖中性，规避「彩虹式 AI slop」。
  5. **画布极淡蓝图网格纹理**；节点是有暖投影的实体卡片；连线是自信墨色细线。
- **与原型的关系**: 信息架构（顶栏健康灯 + 三面工作区 + 只渲染 `plain_*`）与交互骨架**保留**；视觉语言**演进**（暖纸/藏青/三声部/纹理）。原型 A/B/C 仍是 R1「可读半边」可用性测试的被测对象。

## 1. 设计 Token（锁定 — theme 卷 + Penpot 的单一事实源）

### 1.1 色彩
| Token | 值 | 用途 | 设计意图 |
|---|---|---|---|
| `color.paper` | `#f1ece1` | 画布/页面底 | 暖纸，替冷灰，给可信温度 |
| `color.surface` | `#faf7f0` | 面板/卡片底 | 比 paper 略亮，分层 |
| `color.surface-sunken` | `#ece6d9` | 凹陷区/分组底 | 比 paper 略暗，承托样张/分组 |
| `color.ink` | `#221f1a` | 主文字/标题 | 暖墨近黑，对 paper ≈14:1 (AAA) |
| `color.muted` | `#6f6657` | 次要文字/提示 | 暖taupe，对 paper ≈4.6:1 (AA 正文) |
| `color.line` | `#e4ddcf` | 边框/分隔 | 暖中性细线 |
| `color.line-strong` | `#d8cfbd` | 强调边框/输入 | hover/聚焦态边框 |
| `color.keel` | `#2b3f72` | 主色：选中/聚焦/主按钮/新增 | 航海藏青，结构与确定感 |
| `color.keel-bright` | `#3a55a0` | 交互态主色（hover/active） | 提亮藏青 |
| `color.keel-wash` | `#e9ecf5` | 主色软底/选中环 | 低饱和藏青洗 |
| `color.brass` | `#b3792f` | 点缀：龙骨线/wordmark/聚焦环 | 黄铜暖点，**非语义**，极少量 |
| `color.ok` | `#2f8f5b` | 健康正常 | 绿（情绪信号层） |
| `color.ok-soft` | `#e6f4ec` | 正常软底/pill | — |
| `color.warn` | `#c8841a` | 健康小问题 | 暖琥珀 |
| `color.warn-soft` | `#fbf1da` | 小问题软底 | — |
| `color.alert` | `#cc4636` | 健康要处理 | 暖朱 |
| `color.alert-soft` | `#fbe5e1` | 要处理软底 | — |
| `color.diff-added` | `{color.keel}` | diff 新增（虚线） | 新结构=主色 |
| `color.diff-touched` | `#7c54c4` | diff 会碰到 | 紫，与新增拉开 |
| `color.diff-touched-soft` | `#efe7fb` | 会碰到软底 | — |
| `color.diff-unchanged` | `#efe9dd` | diff 不动（淡化） | 降透明/暖灰 |

> 对比度：`ink`/`muted`/`keel` 对 `paper` 均 ≥4.5:1（muted 4.6，正文下限）；健康色 × soft 底配对在 theme 卷逐对核验 WCAG AA。
> 主题：v1 以**浅色暖纸为唯一规范主题**；深色主题不在 v1 范围（如需，theme 卷另立 dark set，健康/diff 语义保持）。

### 1.2 排版（三声部）
| Token | 值 | 用途 |
|---|---|---|
| `font.serif` | `"Noto Serif SC", serif` | 标题/「宪章」嗓音（500/700） |
| `font.sans` | `"Noto Sans SC", system-ui, sans-serif` | 正文/UI（400/500） |
| `font.mono` | `"IBM Plex Mono", monospace` | 制图标签/状态/数字/图例（400/500） |

字号层级（建议，theme 卷定稿）：display 30 / h1 21 / h2 17 / h3 15 / body 14 / body-sm 13 / label-mono 11–12 / caption 11。行高正文 1.6、标题 1.35。

### 1.3 间距 / 圆角 / 阴影 / 动效
- 间距基数 4px：`4/8/12/16/20/24/32/48`。
- 圆角：`sm 8`（按钮/chip）· `md 12`（节点/卡片）· `lg 14`（面板/canvas-wrap）· `pill 999`。
- 阴影（暖色调，两级）：rest `0 1px 2px rgba(20,18,12,.05)`；hover/raised `0 6px 16px rgba(20,18,12,.12)`。
- 动效：120/200/300ms；地图节点错峰落位（staggered，像海图被绘出）；红色健康 pill 缓慢呼吸（非急促）；hover 轻抬。

## 2. ui-spec 分卷计划（全量覆盖 F-001~F-018）
| 卷 | 内容 | 覆盖 |
|---|---|---|
| `ui-spec-keel`（main） | §0 设计方向 · §4 信息架构/导航 · §5 响应式 · 屏↔功能映射总表 | 全局 |
| `ui-spec-keel-theme` | §1 设计系统 Token（本 note §1 定稿） | F-001/F-006 |
| `ui-spec-keel-components` | §2 组件清单 UC-NNN（健康灯·能力节点·连线·原语徽章·详情卡·对话气泡·diff 卡·问题卡·需你决策卡·安全红绿灯·时间线条目·Toast·模态·抽屉·空状态…） | 全功能 |
| `ui-spec-keel-pages-core` | §3 核心屏：三面工作区外壳(M-015/F-008) · 能力地图(F-006) · 对话+diff 确认+影响预览(F-007/F-009) · 实现进度(F-010) · 应用预览(F-008/M-010) · 健康调和(F-002~F-005/F-011) | F-002~F-011 |
| `ui-spec-keel-pages-lifecycle` | §3 生命周期屏：新项目上手+引擎安装(F-012/F-018) · 意图日志时光机(F-013) · 一键上线+安全体检(F-014) · 导出交接(F-015/F-016) · escape hatch(F-017) | F-012~F-018 |

### 待 ui-spec 决策（[ASSUMPTION] 标记，arch §1.4 已点名）
- **能力地图渲染选型**：React Flow v12（候选）vs D3 自绘。须核验 React Flow bundle size 对 F-008 AC-005「主工作区首屏 ≤2s」的影响；备选 D3（体积小、开发成本高）。在 `ui-spec-keel-pages-core` 能力地图屏给出选型 + 理由。

## 3. Penpot 接入状态（root cause + 修复）
- **现象**: `execute_code` 报 `No userToken found in session context. Multi-user mode requires authentication.`
- **根因**: `cataforge penpot deploy` 把 penpot-mcp 容器以 `node index.js --multi-user` 拉起（单用户才是镜像默认）。多用户模式下，MCP 服务靠 query 参数 `?userToken=<TOKEN>` 配对 Plugin(WS) 与 MCP 客户端(HTTP stream)；但 `cataforge` 给 Claude 注册的是裸 `http://localhost:9001/mcp/stream`（无 token），Claude 会话恒为 `userTokenFp=<none>`，与 Plugin 的 token 配不上 → 拒绝。
- **userToken 语义**: 任意共享串（`tokenFingerprint` 仅日志用；`clientsByToken[fullToken]` 全串配对）；Plugin 浏览器会话提供真鉴权，token 只做两条管道的配对键。
- **采纳修复（用户选 B）**: 保留 multi-user，给 Claude 的 MCP 注册 URL 加 `?userToken=<TOKEN>`（与 Plugin 用的同一串），重启 Claude 会话。
- **备选修复 A**: compose 给 penpot-mcp 加 `command: [node, index.js]`（去 `--multi-user`）→ 仅重启该容器、不重启 Claude；缺点：下次 `cataforge penpot deploy` 还原 `--multi-user`。

## 4. 下一步（resume guidance，供重启后续）
1. 确认 Penpot MCP `execute_code` 可用（探针：列 pages + 字体 Noto Serif SC/Noto Sans SC/IBM Plex Mono 是否可用）。
2. 在 Penpot 建设计系统：design tokens（本 note §1）→ library colors/typographies → 核心组件（健康 pill/能力节点/ghost 按钮/diff chips/原语徽章）→ 关键画板（三面工作区 idle、diff 确认、健康预警调和）。
3. 授权 ui-spec 五卷（§2 计划），逐屏映射 F-xxx，经 context finalize 定稿。
4. 写入边界自检（git diff vs ui-designer allowed_paths: docs/ui-spec/、docs/research/）→ 派 reviewer 跑 doc-review 门禁。
