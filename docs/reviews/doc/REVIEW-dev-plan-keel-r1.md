---
id: "review-dev-plan-keel-r1"
doc_type: review
author: reviewer
status: approved
deps:
  - dev-plan-keel
  - dev-plan-keel-s1
  - dev-plan-keel-s2
  - dev-plan-keel-s3
  - dev-plan-keel-s4
  - dev-plan-keel-s5
  - dev-plan-keel-s6
---

# REVIEW-dev-plan-keel-r1

**被审文档**: dev-plan-keel（主卷）+ dev-plan-keel-s1 ~ s6（Sprint 1~6 任务卡，T-001~T-057）

---

## Layer 1 自动检查摘要

| 卷 | 状态 | FAIL 项 | WARN 项 |
|----|------|---------|---------|
| dev-plan-keel（主卷）| FAIL | 缺少必填章节「任务卡详细」 | NAV块章节与实际章节不一致；行数361>300；主卷未引用s2~s5分卷 |
| dev-plan-keel-s1 | FAIL | 1任务缺deliverables；1任务缺tdd_acceptance | 5条AC无可观测动词 |
| dev-plan-keel-s2 | FAIL | 1任务缺deliverables；1任务缺tdd_acceptance | 6条AC无可观测动词 |
| dev-plan-keel-s3 | FAIL | 1任务缺deliverables；1任务缺tdd_acceptance | 6条AC无可观测动词 |
| dev-plan-keel-s4 | FAIL | 1任务缺deliverables；1任务缺tdd_acceptance | 行数316>300；4条AC无可观测动词 |
| dev-plan-keel-s5 | FAIL | 1任务缺deliverables；1任务缺tdd_acceptance | 行数387>300；9条AC无可观测动词 |
| dev-plan-keel-s6 | FAIL | 1任务缺deliverables；1任务缺tdd_acceptance | 6条AC无可观测动词 |

---

## Layer 2 语义审查

Layer 1 全部 FAIL（7/7 卷），按 Layer 1 调用协议短路，Layer 2 未执行。

---

## 问题列表

### [R-001] HIGH: 主卷缺少必填章节「任务卡详细」
- **category**: completeness
- **root_cause**: self-caused
- **描述**: dev-plan-keel 主卷未包含 Layer 1 强制检查的「任务卡详细」章节。该章节是 tech-lead 任务拆分的主体交付区，缺失导致 Layer 1 FAIL，且下游 tdd-engine 无法从主卷定位任务输入。
- **建议**: 在主卷添加「任务卡详细」章节（或明确声明全部任务卡在分卷 s1~s6，并于主卷加目录引用），确保 Layer 1 `has_task_detail_section` 检查通过。

### [R-002] HIGH: 各 Sprint 分卷各有 1 任务缺 `deliverables`（共 6 卷，6 处）
- **category**: completeness
- **root_cause**: self-caused
- **描述**: s1~s6 每卷各有 1 个任务卡缺少 `deliverables` 字段。`deliverables` 是 sprint-review Layer 1 `deliverables_exist` 检查的输入契约，缺失导致 Layer 1 FAIL，亦使 sprint-review 无法验证交付物是否落盘。
- **建议**: 逐一补全 s1~s6 中缺 `deliverables` 的任务卡，填写预期产出文件路径（相对路径，可为 src/ 或 docs/ 下的具体文件）。

### [R-003] HIGH: 各 Sprint 分卷各有 1 任务缺 `tdd_acceptance`（共 6 卷，6 处）
- **category**: completeness
- **root_cause**: self-caused
- **描述**: s1~s6 每卷各有 1 个任务卡缺少 `tdd_acceptance` 字段。该字段列出 AC-NNN 验收标准，是 TDD RED 阶段 test-writer 的唯一输入依据；缺失导致 test-writer 无 oracle 可依、Layer 1 FAIL。
- **建议**: 逐一补全 s1~s6 中缺 `tdd_acceptance` 的任务卡，按 `AC-NNN: <可观测行为描述>` 格式填写至少 1 条验收标准。

### [R-004] MEDIUM: AC 描述缺少可观测动词（s1:5条 / s2:6条 / s3:6条 / s4:4条 / s5:9条 / s6:6条，共 36 处）
- **category**: ambiguity
- **root_cause**: self-caused
- **描述**: 各分卷中合计 36 条 AC 描述未使用可观测动词（如「返回」「渲染」「抛出」「写入」「触发」等），仅描述状态或意图，无法直接作为测试 oracle。影响 TDD 可测性：test-writer 须自行解读 AC 语义，存在偏差风险。
- **建议**: 将 AC 改写为「当 X 时，系统/组件 Y 应 <可观测动词> Z」格式。例：「用户未登录时，访问 /dashboard 应重定向至 /login 并返回 302」。

### [R-005] MEDIUM: 主卷 NAV 块与实际章节不一致
- **category**: consistency
- **root_cause**: self-caused
- **描述**: 主卷文档头部导航块（NAV）所列章节与正文实际章节标题不一致，Layer 1 检测到漂移。读者依据 NAV 定位时会找不到对应章节，增加理解成本。
- **建议**: 同步 NAV 块与正文章节标题，确保两者一一对应。更新章节后先改 NAV，避免再次漂移。

### [R-006] MEDIUM: 主卷未引用 s2~s5 分卷
- **category**: completeness
- **root_cause**: self-caused
- **描述**: 主卷仅引用了部分 Sprint 分卷（s1、s6），s2~s5 未出现在主卷的分卷导航或引用列表中，导致读者无法从主卷完整追溯所有 Sprint 分卷。
- **建议**: 在主卷的分卷导航区补充对 s2~s5 的引用（文档路径或 doc_id 引用格式）。

### [R-007] LOW: 主卷行数 361 超过 `DOC_SPLIT_THRESHOLD_LINES`（300）
- **category**: convention
- **root_cause**: self-caused
- **描述**: dev-plan-keel 主卷 361 行，超出框架拆分阈值 300 行。长文档影响按需加载效率，且后续追加内容将进一步加大主卷体积。
- **建议**: 将主卷中可移至分卷的内容（如全局说明、技术约定段落）迁移到对应 Sprint 分卷或独立附录卷，使主卷精简至 ≤300 行。

### [R-008] LOW: s4 分卷行数 316 超过 `DOC_SPLIT_THRESHOLD_LINES`（300）
- **category**: convention
- **root_cause**: self-caused
- **描述**: dev-plan-keel-s4 行数 316，轻微超出阈值。
- **建议**: 精简 s4 内任务卡描述冗余文本，或将部分辅助说明移至注释/附录，使卷行数降至 ≤300。

### [R-009] LOW: s5 分卷行数 387 超过 `DOC_SPLIT_THRESHOLD_LINES`（300）
- **category**: convention
- **root_cause**: self-caused
- **描述**: dev-plan-keel-s5 行数 387，超出阈值较多（超出 29%）。s5 同时是 AC 无可观测动词最多的卷（9 条），两问叠加使该卷可维护性最低。
- **建议**: 优先处理 s5：先精简 AC 描述（修复 R-004 覆盖的 9 条），再评估是否需要将 s5 拆为 s5a/s5b 两个子卷。

---

## Verdict

**needs_revision** — Layer 1 全部 FAIL，存在 3 处 HIGH 级问题：主卷缺少必填章节「任务卡详细」（R-001），及 6 个 Sprint 分卷各有任务卡缺 `deliverables`（R-002）和 `tdd_acceptance`（R-003），这些字段是 TDD RED 阶段和 sprint-review 的输入契约，缺失将阻塞下游 test-writer 执行。
