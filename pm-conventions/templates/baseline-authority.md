# Baseline 更新权限矩阵

> 所有涉及 Baseline 修改的 PM skill，执行更新前必须读取本文件确认权限范围。
> 更新操作步骤仍遵循 pm-conventions §3.3。

| Skill | 允许更新范围 | 触发条件 | 上限约束 |
|-------|------------|---------|---------|
| pm-baseline-gen | 全量（创建/重建） | 生成 Baseline | 无 |
| pm-prd-update Phase 2 | 全量（核心规则） | 用户明确要求"更新 Baseline" | 无 |
| pm-prd-review Phase 2 | 局部修复 | 评审发现 Baseline 自身问题 | ≤2 条；超出回退 pm-prd-update |
| pm-proto-review Phase 2 | 仅交互层（UI 行为/布局） | 交互评审发现问题 | 不涉及业务逻辑；业务逻辑标记"需 pm-prd-update 处理" |
| pm-baseline-mapping | 仅"需求文档位置"列 | PRD 生成/更新后 | 不改前三列和规则内容 |
