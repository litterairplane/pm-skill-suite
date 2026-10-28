# Mermaid 语法规范与模板库

## 语法与样式要点

- 节点文字一律用双引号包裹：`A["提示文字"]`
- 布局：业务流程 `graph TD`，泳道 `graph LR` + `subgraph`，时序 `sequenceDiagram`
- 样式定义（每个流程图开头引入）：
  - `classDef startEnd fill:#f0f0f0,stroke:#999,stroke-width:1px,color:#666`
  - `classDef core fill:#fff,stroke:#5b8def,stroke-width:2px,color:#333`
  - `classDef disabled fill:#f5f5f5,stroke:#ccc,color:#bbb`
- 开始/结束用 `(["文字"]):::startEnd`，判断用 `{"文字"}`，核心节点加 `:::core`，非本期加 `:::disabled`
- 连线：正常 `-->`，条件 `-->|条件文字|`（标签≤8字），异步 `-.->`
- 复杂度：单图节点≤15个，并行分支≤4个，超出则拆分

---

经过验证可正确渲染的 Mermaid 模板，生成流程图时优先使用。

## 流程图模板（业务审批/处理流程）

```
graph TD
    classDef startEnd fill:#f0f0f0,stroke:#999,stroke-width:1px,color:#666
    classDef core fill:#fff,stroke:#5b8def,stroke-width:2px,color:#333

    S(["开始"]):::startEnd
    A["步骤A"]
    B{"判断条件"}
    C["正常处理"]:::core
    D["异常处理"]
    E(["结束"]):::startEnd

    S --> A --> B
    B -->|"是"| C --> E
    B -->|"否"| D --> E
```

## 泳道图模板（跨系统/跨角色协作）

```
graph LR
    subgraph 用户端
        A["发起请求"]
    end
    subgraph 服务A
        B["接收处理"]
        C{"校验"}
    end
    subgraph 服务B
        D["业务处理"]
        E["返回结果"]
    end

    A --> B --> C
    C -->|"通过"| D --> E
    C -->|"不通过"| F["返回错误"]
```

## 时序图模板（系统间调用）

```
sequenceDiagram
    participant U as 用户
    participant A as 服务A
    participant B as 服务B

    U->>A: 请求操作
    A->>B: 调用服务
    B-->>A: 返回结果
    A-->>U: 响应
    alt 服务B异常
        B-->>A: 错误响应
        A-->>U: 提示信息
    end
```

## 状态机模板（实体状态流转）

```
stateDiagram-v2
    [*] --> 草稿
    草稿 --> 审核中: 提交审核
    审核中 --> 已通过: 审核通过
    审核中 --> 已驳回: 审核驳回
    已驳回 --> 草稿: 重新编辑
    已通过 --> [*]
```
