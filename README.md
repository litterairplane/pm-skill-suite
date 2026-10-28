# pm-skill-suite

> 一套用于产品经理的 Claude Code Skill 工作流体系——从模糊想法到可交付开发的需求文档，11 个协作 Skill 覆盖需求完整生命周期。

---

## 解决什么问题

产品经理用 AI 写需求文档时的常见痛点：

| 痛点 | 具体表现 |
|---|---|
| AI 容易自行脑补 | 直接让 AI 写 PRD，产出充满未经确认的假设 |
| 写需求靠经验 | 每次从零开始，容易遗漏异常场景、权限控制、状态流转 |
| 需求脱离系统现状 | AI 不了解现有系统，写出来的需求与现有功能冲突或重复造轮子 |
| 迭代改动易出错 | 修改时关联的流程图、接口定义、交互说明容易漏改 |
| 评审靠人力 | 需求评审依赖资深同事逐条检查，效率低且覆盖面有限 |
| 交互设计后置 | 先写完需求再做原型，发现交互问题时已付出大量修改成本 |

---

## Skill 清单

### PM 层（10 个 Skill）

| # | Skill | 定位 | 说明 | 推荐模型 |
|---|---|---|---|---|
| 1 | `pm-req-analysis` | 需求分析 | 从模糊想法到结构化分析报告：探索上下文 → 识别模糊点 → 输出候选方案与待确认问题清单 | Opus |
| 2 | `pm-baseline-gen` | Baseline 生成 | 方案收敛 → 功能架构设计 → 生成 Baseline 文档 | Opus |
| 3 | `pm-prd-draft` | 需求文档生成 | 基于 Baseline 标准化写作，逐章确认后输出需求文档 | Opus |
| 4 | `pm-prd-review` | PRD 评审 | Baseline 覆盖性校验 + 结构性评审 + 位置映射，三步闭环 | Sonnet |
| 5 | `pm-prd-update` | 需求迭代更新 | 修改建议 → Baseline 更新 → 需求定稿，三阶段闭环 | Opus |
| 6 | `pm-proto-gen` | 原型生成 | 双输入模式：Baseline 驱动探索性原型 / PRD 驱动精确原型 | Sonnet |
| 7 | `pm-proto-review` | 交互评审 | 交互评估 → 评审意见 → 更新 Baseline（交互层）+ 原型 | Sonnet |
| 8 | `pm-baseline-mapping` | 位置映射 | 将 Baseline 确认点映射到需求文档的具体位置（组件型） | Sonnet |
| 9 | `pm-operation-guide` | 操作指引生成 | 面向一线操作人员的"照着做"文档 | Sonnet |
| 10 | `pm-workflow` | 工作流导航 | 扫描需求目录产物，展示进度总览，引导下一步 Skill 调用 | 任意 |

### 约定层（1 个 Skill）

| Skill | 说明 |
|---|---|
| `pm-conventions` | PM Skill 体系的共享规范——PRD 章节结构、文档格式、变更管理流程、知识库路径约定等 |

---

## 工作流

### 完整流程（涉及前端页面）

```
pm-req-analysis
    → pm-baseline-gen
        → pm-proto-gen（探索性原型）
            → pm-proto-review
                → pm-prd-draft
                    → pm-prd-review
                        ↕
                    pm-prd-update
                        ↓
                pm-operation-guide
```

### 精简流程（不涉及前端页面）

```
pm-req-analysis → pm-baseline-gen → pm-prd-draft → pm-prd-review
```

---

## 核心设计理念

### 分析、收敛、写作三阶段分离

让 AI 直接写 PRD 的最大问题是：AI 同时承担了"帮你想需求"和"帮你写文档"两个角色——一边想一边写，容易把"我猜测"包装成"已确认需求"。

本体系将三种思维模式拆到三个独立 Skill：

| 阶段 | Skill | 目标 |
|---|---|---|
| 发散 | `pm-req-analysis` | 打开问题空间，输出候选方案和待确认问题 |
| 收敛 | `pm-baseline-gen` | 选定方案、补全细节、设计功能架构、输出 Baseline |
| 写作 | `pm-prd-draft` | 基于已确认的 Baseline 忠实还原为标准化文档 |

### Baseline 驱动

Baseline 文档是整套流程的核心，每条确认结论都有唯一编号（C-XX）。核心规则：

- 每次需求变更，先更新 Baseline，再更新 PRD
- 需求评审时，自动校验 Baseline 与 PRD 的双向一致性
- Baseline 中的功能架构是 PRD 的结构蓝图

### 基于代码库的知识库

业务文档描述的是"产品希望系统做什么"，代码才是"系统实际在做什么"。基于代码生成的知识库能准确反映系统现状，避免 AI 发明系统不存在的能力。

### 成本优化设计

两个关键设计：

**Skill 拆分**：每个 Skill 结束后开新会话执行下一个，打断对话历史的平方效应，净省 30-50% token。

**知识上下文文件**：`pm-baseline-gen` 输出一份精简的知识上下文文件（从完整知识库中筛选当前需求相关子集），下游 Skill 优先读取该文件而非完整知识库，知识库部分节省约 75% token。

---

## 使用方式

### 安装

将各 Skill 目录放到你的 Claude Code Skill 目录下：

```bash
cp -r pm-*  ~/.claude/skills/
```

### 触发关键词

| 你想做什么 | 说 | 触发的 Skill |
|---|---|---|
| 从零写需求 | "帮我写个需求" / "新需求" | `pm-req-analysis` |
| 生成 Baseline | "生成 Baseline" | `pm-baseline-gen` |
| 写 PRD | "写 PRD" / "需求定稿" | `pm-prd-draft` |
| 修改需求 | "改需求" / "更新需求" | `pm-prd-update` |
| 评审需求 | "检查需求" / "需求评审" | `pm-prd-review` |
| 生成原型 | "生成原型" / "做原型" | `pm-proto-gen` |
| 评审原型 | "交互评审" / "评审原型" | `pm-proto-review` |
| 写操作指引 | "写操作指引" | `pm-operation-guide` |
| 查看需求进度 | "需求进度" / "需求状态" | `pm-workflow` |

---

## 输出目录结构

```
Developer/my-project/产品需求/{需求名}/
├── {需求简称}-需求分析-v{N}.md        # pm-req-analysis 产出
├── {需求简称}-Baseline.md             # pm-baseline-gen 产出（原地更新）
├── {需求简称}-知识上下文.md           # pm-baseline-gen 同步产出
├── 变更记录/
│   └── baseline-v{N}.md
├── 需求文档/
│   ├── {需求简称}-需求文档-v1.md
│   └── {需求简称}-需求文档-v2.md
├── 修改建议/
├── 评审报告/
├── 原型/
│   └── {需求名}.html
├── 操作指引/
└── sessions/                          # 过程文件（确认记录等）
```

---

## 版本信息

| Skill | 版本 |
|---|---|
| pm-req-analysis | v1.3 |
| pm-baseline-gen | v1.2 |
| pm-prd-draft | v12.2 |
| pm-prd-review | v3.1 |
| pm-prd-update | v4.1 |
| pm-proto-gen | v6.2 |
| pm-proto-review | v1.1 |
| pm-baseline-mapping | v2.0 |
| pm-operation-guide | v4.1 |
| pm-workflow | v1.1 |
| pm-conventions | v2.4 |

---

## License

MIT
