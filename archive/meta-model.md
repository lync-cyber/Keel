---
id: "keel-meta-model"
doc_type: research
author: Alex
status: approved
deps: []
---

# Keel 蓝图元模型 · 原语规格（人工审查主入口）

> 本文件是「蓝图由哪些原语构成、每个原语是什么」的**人读规格**，供团队评审与演进。
> 机器校验在 `blueprint.schema.json`（Draft 2020-12，机器门，不拿来人工评审）。
> 规范实例见 `blueprint.example.yaml`（示例应用「我的博客」v9）。
> 语言：中文。最近更新：2026-06-10。

---

## 0. 这份文件的位置

概念阶段把三类产物分开，各用最省事的形态（决策 ADR-0010）：

| 层 | 产物 | 文件 | 评审方式 |
|---|---|---|---|
| 探索性 | 决策、草图 | `decisions/*.md`（ADR）、Mermaid 内嵌 | 读 Markdown |
| 真相源 | 原语规格 + 规范实例 | **本文件** + `blueprint.example.yaml` | 读本文件，对照实例 |
| 机器门 | 形态校验 | `blueprint.schema.json` | 不手读，跑校验器 |

一条铁律：原语**一旦定稿，其视图从规范实例渲染**，不另维护第二份真相（Keel 自身的「单一真相源 / 不腐化」）。

---

## 1. 设计目标

元模型用一组**正交原语**覆盖各种软件架构与模块交互形式，同时满足两个半边（核心假设 R1）：

- **机器可检半边**：每个 `Constraint` 的 `rule.type` 映射到一个确定性检查器；每条 `Requirement` 的 `acceptance` 将来降生为测试。
- **非程序员可读半边**：凡上界面的原语都必须带 `plain`（通俗表达）；技术字段一律不上界面。

对应技术报告「形式化输入 → 确定性变换规则 → 可验证输出」的三元式：蓝图是形式化输入，编译器是确定性规则，检查器/测试是可验证输出。

---

## 2. 原语总览

| 原语 | 一句话 | 必带 plain | 编译目标 |
|---|---|---|---|
| `meta` | 应用名 / 原型 / 版本 / 技术栈 | app_name 即 plain | — |
| **`Requirement`（R-IR）** | 一条需求：EARS 句式 + 可测验收准则 | ✔ | 验收/集成测试（需求覆盖率） |
| `Archetype` | 架构原型：词汇 + 允许关系 + 上图投影 + 约束集 | ✔ | 选原型即选初始宪法 |
| `Unit` | 有边界、拥有代码的构件（切片/层/服务/适配器…） | ✔ | 切片骨架 + 地图节点 |
| `Port` | 构件唯一对外接口；跨构件只认它 | ✔ | tsc + schema 校验；契约占位 |
| `Relation` | 两元素间一条有类型、有方向的边 | ✔ | 依赖图 + 地图连线 |
| `Entity` | 持久化数据形状 + 关系 | ✔ | db schema + 迁移校验 |
| `Constraint` | 对图的一个谓词 + 严重度（架构纪律） | ✔ | 一个确定性检查器 |

新增的 **`Requirement`** 与 `satisfies` 追溯链是 v0.2 相对 v0.1 的核心增量（决策 ADR-0008），补上了技术报告里「整套结构的第一锚点」。

---

## 3. 逐原语规格

### 3.1 `Requirement`（R-IR · 需求层）

报告 §05 阶段0 的形式化入口。LLM 仅把自然语言**归一**到此槽位，**不得发明需求**。

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | ✔ | 稳定标识，被 `satisfies` 回链。 |
| `statement` | | 受控 EARS 句式：`当<触发>，系统应<响应>`。近期最小版可省略，只写 plain + acceptance。 |
| `plain` | ✔ | 给非程序员的通俗需求。 |
| `acceptance[]` | ✔（≥1） | 可测验收准则，见下。 |
| `priority` | | `must / should / could`。 |
| `status` / `origin` | | 同通用枚举。 |

`acceptance`（一条验收准则）：

| 字段 | 必填 | 说明 |
|---|---|---|
| `id` | ✔ | 准则标识。 |
| `plain` | ✔ | 给人读的判定。 |
| `check` | | 判定方式描述。 |
| `oracle` | | `test / property / type / schema / human`。`human` = 无可执行 oracle，回退人工批准（报告 §05 诚实边界）。 |

> 行为门远期才落地（决策 ADR-0009）：现在 `oracle: test/property` 只是**标注**将来由谁把关，近期不实现测试基建。

最小例（取自规范实例）：

```yaml
- id: req-favorite
  statement: 当登录用户在文章详情页点击收藏时，系统应记录该用户对该文章的收藏…
  plain: 登录的人能把喜欢的文章收起来，之后能快速找回；没登录不能收藏。
  acceptance:
    - { id: req-favorite-a1, plain: 登录后点收藏，该文章出现在「我的收藏」里。, oracle: test }
    - { id: req-favorite-a3, plain: 同一篇重复点收藏不会产生两条记录。, oracle: property }
```

### 3.2 `satisfies`（追溯链 R→A）

`Unit` 与 `Constraint` 都可带 `satisfies: [需求 id]`，把架构回链到需求。

- **编译期校验（报告 §05 阶段1 门）**：每条 `Requirement` 至少被一个 `Unit` 或 `Constraint` satisfies；否则该需求「无人认领」，拒绝。
- 这条链让「需求覆盖率」可计算（报告 §05 阶段4），也让每个能力都能回答「我为什么存在」。
- 结构性密封不变量可不回链；但本实例里 `no-cross-feature-import` / `one-canonical-impl` 等都回链到 `req-no-corrupt`（不腐化需求），示范结构纪律也服务于一条显式需求。

### 3.3 `Archetype`（架构原型）

一套「`unit_kinds` 构件词汇 + `allowed_relation_kinds` 允许关系 + `node_kinds` 上图投影 + `internal_layers` 内部分层 + `constraints` 约束集（含密封不变量）」的捆绑。换架构 = 换这几个旋钮。v1 仅上 `modular-monolith`（决策 D3 / ADR-0003）。

### 3.4 `Unit`（构件）

有边界、拥有代码的命名节点。关键字段：`kind`（由 archetype 定义词汇）、`plain`（节点副标题短句）、`plain_detail`（详情面板补充段落）、`boundary`（代码边界，供检查器界定归属）、`owns`（独占的横切职责标签，配合 `single_canonical`）、`ports`（挂出的对外接口）、`surfaces_on_map`（是否升为地图节点——投影策略，并非所有 Unit 都上图）、`satisfies`。能力地图只渲染 `plain_name` / `plain` / `plain_detail`，技术字段不上界面。

### 3.5 `Port`（对外接口）+ 行为契约占位

构件唯一被允许触达的表面；跨构件交互只认 Port。`shape` 声明类型/schema 引用（编译为 tsc + schema 校验）。

`contract`（**行为契约占位**，决策 ADR-0009）：

| 字段 | 说明 |
|---|---|
| `pre[]` | 前置条件 |
| `post[]` | 后置条件 |
| `invariant[]` | 不变量 |

每个子句（`clause`）`plain` 必填、`formal` 可选（将来编译为 zod refinement / TS 断言）。**近期留空或只写 plain**，仅作结构预留，使蓝图不必在远期补行为门时翻修。例：

```yaml
- id: favorite.contract
  of: favorite
  contract:
    invariant:
      - { plain: 同一用户对同一文章最多一条收藏。, formal: "unique(Favorite.userId, Favorite.articleId)" }
```

### 3.6 `Relation`（关系）

两元素间一条有类型（`relationKind`）、有方向（`from`→`to`）的边，是模块间所有交互形式的统一载体。`via` 指定必经的 Port（封装：跨构件须走正门）。`relationKind`：`imports / calls / emits / consumes / reads / writes / implements / routes_to / uses`。

### 3.7 `Entity`（数据实体）

持久化数据形状 + 关系（`has_many / has_one / belongs_to`）。`db/schema` 为数据模型唯一真相源，变更走迁移。

### 3.8 `Constraint`（约束）+ rule DSL

对「Unit/Port/Relation 这张图」的一个谓词 + 严重度。每个 `rule.type` 映射到一个确定性检查器：

| `rule.type` | 含义 | 编译目标检查器 |
|---|---|---|
| `forbid_relation` | 禁止某形状的关系（如 feature 间 imports） | dependency-cruiser |
| `allowed_relation_kinds` | 两类构件间只允许某些关系类型 | dependency-cruiser |
| `direction` | 关系沿指定次序单向、不可跨级（`scope` 区分构件内/构件间） | dependency-cruiser |
| `via_port` | 跨构件关系必须经对方 Port | dependency-cruiser + eslint |
| `access_only_via` | 对某类目标的访问只能经统一封装（如 core/db） | dep-cruiser + eslint 白名单 |
| `single_canonical` | 某横切职责全应用仅一处规范实现 | ts-morph 重复/相似扫描 |
| `no_outbound_except` | 某类构件除指定出口外不得有出向关系 | dependency-cruiser |
| `no_orphan` | 不得有孤立未接线构件 | madge / ts-morph 可达性 |

`severity`：结构性 = `hard`（阻断），风格类 = `soft`（告警）（决策 D4 / ADR-0004）。`sealed: true` = 密封不变量，覆盖层不可触及。映射不到静态检查的显式落到 `NEEDS_DECISION`。

---

## 4. 与技术报告管线的对应

| 报告管线（§05） | 本元模型承载 |
|---|---|
| 阶段0 需求 → R-IR | `requirements`（+ acceptance/oracle） |
| 阶段1 R-IR → 架构 IR（A-IR） | `units / ports / relations / entities` + `satisfies` 追溯 |
| 阶段2 模板引擎生成骨架/契约 | 由 `Unit.boundary` / `Port` 确定性脚手架（**编译器待实现**，P1） |
| 阶段3 契约 → 函数体（装箱槽位 + 行为门） | `Port.contract` 占位（**行为门远期**，ADR-0009） |
| 阶段4 集成与系统验证 + 需求覆盖率 | `satisfies` 覆盖率（已可算）；集成测试待实现 |
| 编译：蓝图 → 确定性检查器配置 | `Constraint.rule.type → checker`（**编译器待实现**，P1） |

近期已落地的是「形态」层（原语、追溯、占位）；标注 **待实现** 的执行层 gated 在 R1 闸门之后（见 `CLAUDE.md`）。

---

## 5. 校验方式

```
# 形态校验：实例是否符合元模型
python -c "import yaml,json,jsonschema; \
  jsonschema.Draft202012Validator(json.load(open('docs/blueprint.schema.json'))) \
  .validate(yaml.safe_load(open('docs/blueprint.example.yaml')))"
```

除 schema 形态校验外，规范实例还应通过（已在本次实施中跑通）：

- **引用完整性**：`satisfies` / `via` / `Port.of` / `Relation.from,to` 全部解析到已声明 id。
- **需求覆盖率**：每条 `Requirement` 至少被一个 `Unit`/`Constraint` satisfies。

> 注：本机 jsonschema 需 ≥4.18 以支持 Draft 2020-12（`pip install -U jsonschema`）。
