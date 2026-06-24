---
id: "ui-spec-keel-theme"
version: "0.1"
doc_type: ui-spec
author: ui-designer
status: approved
deps: ["arch-keel", "research-ui-design-direction-keel"]
consumers: [tech-lead, developer]
volume: theme
volume_type: theme
split_from: "ui-spec-keel"
required_sections:
  - "## 4. 主题方案"
---
# UI Specification 分卷 — 主题方案: 沉静海图 / 暖调蓝图

[NAV]
- §4 主题方案 → THEME-01
[/NAV]

## 4. 主题方案

### THEME-01: Calm Cartography（沉静海图 · 暖调蓝图）

本卷是 Keel 设计系统 design token 的单一事实源，与 Penpot 文件「Keel」的 design tokens 一一对应（6 集 54 token + library 21 colors / 8 typographies，见「Design System」页验证板）。设计方向锚点见 research-ui-design-direction-keel。

- **产品调性**: 稳固 · 克制 · 可信——像一份郑重的「应用宪章 / 海图」，而非花哨工具的廉价感。
- **目标情绪**: 给「重来者」（非程序员）以确定感与掌控感——传达「结构稳固、看得懂、拿得走」。
- **核心隐喻**: Keel = 龙骨；能力地图 = 应用的海图 / 蓝图。
- **饱和色纪律**: 饱和色只留给语义（健康三态 + diff），其余一律暖中性，规避「彩虹式 AI slop」。

#### 4.1 视觉语言

##### 4.1.1 色彩（Penpot token 集: Color）

中性（暖纸基底）:

| token | 值 | 用途 |
|---|---|---|
| color.paper | #F1ECE1 | 画布/页面底（暖纸，替冷灰给可信温度） |
| color.surface | #FAF7F0 | 面板/卡片底（比 paper 略亮，分层） |
| color.surface-sunken | #ECE6D9 | 凹陷区/分组底（承托样张/分组） |
| color.ink | #221F1A | 主文字/标题（暖墨近黑，对 paper ≈13.9:1 AAA） |
| color.muted | #6F6657 | 次要文字/提示（暖 taupe，对 paper ≈4.8:1 AA 正文） |
| color.line | #E4DDCF | 边框/分隔（暖中性细线） |
| color.line-strong | #D8CFBD | 强调边框/输入 hover/聚焦态 |

品牌（航海藏青 + 黄铜点缀）:

| token | 值 | 用途 |
|---|---|---|
| color.keel | #2B3F72 | 主色：选中/聚焦/主按钮/diff 新增（航海藏青，对 paper ≈8.7:1） |
| color.keel-bright | #3A55A0 | 交互态主色（hover/active） |
| color.keel-wash | #E9ECF5 | 主色软底/选中环 |
| color.brass | #B3792F | 点缀：龙骨线/wordmark/聚焦环（非语义，极少量） |

健康三态（情绪信号层，各带 soft 软底）:

| token | 值 | 用途 |
|---|---|---|
| color.ok / color.ok-soft | #2F8F5B / #E6F4EC | 健康正常（绿） |
| color.warn / color.warn-soft | #C8841A / #FBF1DA | 健康小问题（暖琥珀） |
| color.alert / color.alert-soft | #CC4636 / #FBE5E1 | 健康要处理（暖朱） |

diff（改动预览）:

| token | 值 | 用途 |
|---|---|---|
| color.diff-added | {color.keel} | 新增（虚线，新结构=主色） |
| color.diff-touched / -soft | #7C54C4 / #EFE7FB | 会碰到（紫，与新增拉开） |
| color.diff-unchanged | #EFE9DD | 不动（淡化暖灰） |

##### 4.1.2 排版 — 三声部（Penpot token 集: Typography）

| token | 值 | 嗓音 |
|---|---|---|
| font.serif | "Noto Serif SC", serif | 「宪章」标题 / 郑重声明（500/700） |
| font.sans | "Noto Sans SC", system-ui, sans-serif | 正文 / UI（400/500） |
| font.mono | "IBM Plex Mono", monospace | 制图标签 / 状态 / 数字 / 图例（400/500） |

字号阶梯（fontSizes）: display 30 / h1 21 / h2 17 / h3 15 / body 14 / body-sm 13 / label 12 / caption 11（px）。
字重（fontWeights）: regular 400 / medium 500 / bold 700。
行高（dimension）: 正文 line.body 1.6 / 标题 line.title 1.35。

library typographies（已落 Penpot）: Display(Serif 30/700) · Heading/H1(Serif 21/700) · Heading/H2(Serif 17/500) · Heading/H3(Sans 15/500) · Body/Base(Sans 14/400) · Body/Small(Sans 13/400) · Mono/Label(Mono 12/500) · Mono/Caption(Mono 11/400)。

##### 4.1.3 间距 / 圆角 / 阴影 / 动效

- 间距（Spacing，基数 4px）: space.1..space.12 = 4 / 8 / 12 / 16 / 20 / 24 / 32 / 48。
- 圆角（Radius）: sm 8（按钮/chip）· md 12（节点/卡片）· lg 14（面板/canvas-wrap）· pill 999。
- 阴影（Shadow，暖色调两级）: rest `0 1px 2px rgba(20,18,12,.05)` · raised `0 6px 16px rgba(20,18,12,.12)`。
- 动效（Motion）: fast 120 / base 200 / slow 300（ms）。地图节点错峰落位（staggered，像海图被绘出）；红色健康 pill 缓慢呼吸（非急促）；hover 轻抬。

##### 4.1.4 对比度与无障碍

- ink/muted/keel 对 paper 均 ≥4.5:1（实测 #221F1A / #6F6657 / #2B3F72 on #F1ECE1：ink 13.9 AAA、keel 8.7、muted 4.80 AA 正文下限）。本卷为对比度单一事实源，数值以实测为准。
- 健康色 ok/warn/alert 对 paper 为 3.4–4.0:1：仅用于 pill/图标/大字与各自 soft 底配对，不用作 paper 上的小字正文；soft 底配对在组件卷逐对核验。
- brass 为非语义点缀（对 paper 3.1），仅用于 wordmark/龙骨线/聚焦环等极少量场景。
- v1 以浅色暖纸为唯一规范主题；深色主题不在 v1 范围（如需另立 dark set，健康/diff 语义保持）。

#### 4.2 适配组件

组件清单与主题覆写规则见分卷 ui-spec-keel-components（UC-XXX）。健康 pill 三态、diff chips、能力节点卡、原语徽章七类（+ 自由实现区修饰标识）、ghost/primary 按钮等核心组件的 token 绑定已在 Penpot「Components」页落地验证。

#### 4.3 关键页面应用

各屏的主题视觉应用见分卷 ui-spec-keel-pages-core（P-XXX，核心屏 F-002~F-011）与 ui-spec-keel-pages-lifecycle（P-XXX，生命周期屏 F-012~F-018）。三面工作区的顶栏健康灯、能力地图画布（极淡蓝图网格 + 暖投影实体卡片 + 墨色连线）、对话/预览面均以本卷 token 为唯一视觉来源。
