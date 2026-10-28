# 会话隔离与模型使用规则

> 来源：pm-conventions §11.5

除 pm-workflow（只读导航）外，每个 PM skill **应**在独立会话中执行：
1. 每个 skill 大量读取文件和多轮确认，单会话上下文不足以支撑连续执行
2. 不同 skill 使用不同模型（Opus/Sonnet），交互模式下同一会话无法切换模型
3. 独立会话强制从文件重新读取，避免确认偏误

**各 Skill 会话与模型策略**：

| Skill | 会话策略 | 模型 | 后续引导（流程终止后追加） |
|-------|---------|:----:|------------------------|
| pm-req-analysis | 独立会话 | Opus | `/pm-baseline-gen`，附分析报告路径 |
| pm-baseline-gen | 独立会话 | Opus | 涉及前端：`/pm-proto-gen`；不涉及：`/pm-prd-draft`，附 Baseline 路径 |
| pm-prd-draft | 独立会话 | Opus | ⚠️ 请在新会话中执行 `/pm-prd-review`（Sonnet 即可） |
| pm-prd-update | 独立会话 | Opus | ⚠️ 请在新会话中执行 `/pm-prd-review`（Sonnet 即可） |
| pm-proto-gen | 独立会话 | Sonnet | Baseline 模式：`/pm-proto-review`；PRD 模式：无 |
| pm-proto-review | 独立会话 | Sonnet | 探索性：`/pm-prd-draft`；精确（有问题）：`/pm-prd-update` |
| pm-prd-review | 独立会话 | Sonnet | 有阻塞项：`/pm-prd-update`；无阻塞项：无 |
| pm-operation-guide | 独立会话 | Sonnet | 无固定后续 |
| pm-workflow | 可在任意会话中调用 | — | — |
| pm-baseline-mapping | 被自动调用 | Sonnet | — |

**后续引导格式**（流程终止后追加）：
> ⚠️ **建议结束当前会话**
> 📋 下一步（复制到新会话中使用）：
> ```
> {下一步 skill 的调用命令和参数}
> ```