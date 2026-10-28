# AGENT_MODE 自动编排协议

> [适用范围] 本文件仅在 pm-agent（AGENT_MODE）场景下加载。交互模式下无需读取。

## 触发条件

当用户输入以 `[AGENT_MODE]` 开头时，skill 进入自动编排模式。此模式由外部编排脚本（pm-agent）驱动，每个 skill 在独立的 `claude -p` 会话中执行。

## 确认点分级

每个 PM skill 在其 SKILL.md 中定义各确认点的分级。三个级别：

| 级别 | 标记 | AGENT_MODE 下的行为 |
|------|------|-------------------|
| 🟢 auto | 低风险，可自动决策 | 直接使用默认策略执行，不输出确认请求 |
| 🟡 summary | 中风险，可先做后审 | 正常执行，将产出和确认内容写入 `sessions/agent-review-items.json`，不暂停等待 |
| 🔴 blocking | 高风险，影响后续全部产出 | 正常执行分析，AI 做出最佳判断继续执行，但将判断依据和替代选项写入 review items |

## agent-review-items.json 格式

```json
{
  "skill": "skill-name",
  "timestamp": "ISO8601",
  "review_items": [
    {
      "id": "confirmation_point_id",
      "level": "summary|blocking",
      "description": "确认点描述",
      "ai_decision": "AI 的判断和理由",
      "alternatives": ["备选方案1", "备选方案2"],
      "content": "需要审阅的具体内容（截取关键部分）"
    }
  ],
  "generated_files": ["相对于需求目录的文件路径"]
}
```

## AGENT_MODE 下的行为变更

1. **正常生成所有产物文件**——文件生成规则（版本管理、命名、目录结构）不变
2. **跳过会话切换强引导**——不输出 §11.4 的会话切换引导（由编排脚本管理流程衔接）
3. **文件读取铁律不变**——仍必须从文件路径重新读取
4. **知识库读取规则不变**
5. **在响应末尾追加 `[AGENT_COMPLETE]` 标记**——编排脚本用此判断执行是否成功

## AGENT_MODE 检测代码（各 skill 统一引用）

在 skill 执行的最初步骤，检查用户输入是否以 `[AGENT_MODE]` 开头：
- 是 → 按确认点分级处理各确认点，最终写入 `sessions/agent-review-items.json`
- 否 → 完全按原有交互流程执行（向后兼容，不影响手动使用）
