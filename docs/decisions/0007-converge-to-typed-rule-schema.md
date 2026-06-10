# 0007. 蓝图收敛到 typed-rule 元模型

状态：accepted（2026-06-10）

## 背景

存在两套蓝图且不一致：

- `demo/blueprint.yaml`：`dependency_rules` 是 `formal`/`plain` **字符串** + 一个 severity，非可执行规则。
- `docs/blueprint.schema.json` + `blueprint.example.yaml`：6 原语 + **类型化 constraint DSL**（`forbid_relation` / `direction` / `via_port` / `access_only_via` / `single_canonical` / `no_orphan` …）+ 检查器映射，更接近技术报告的形式化 A-IR。

两套并存违反 Keel 自己的「单一真相源」。

## 决定

**收敛到 typed-rule 元模型**。`blueprint.example.yaml` 升为唯一规范实例；`demo/blueprint.yaml` 的字符串规则版弃用（标注指向规范实例）。

## 后果

- 规范实例：`docs/blueprint.example.yaml`（v9）。
- `demo/blueprint.yaml` 顶部加弃用指向；三张原型内嵌 JSON 的重新指向列为后续（R1 演示不受影响——原型各自内嵌副本，运行不破）。
- 后续所有原语/字段都写在收敛后的单一 schema 上，避免再分叉。
