# 页面类型模板

本文件定义原型支持的页面类型及其结构模板。

> **适用场景**：本模板适用于多文件模式和首次生成参考；单文件模式下 CSS 内联到 `<style>` 标签中，页面结构模板同样适用。

## HTML 基础模板

所有页面必须使用以下头部声明：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{页面标题} - {项目名称}</title>
  <link rel="stylesheet" href="styles.css">
</head>
```

---

## 列表页 (table)

对齐项目实际结构，是 B 端系统最常见的页面类型。

### 结构模板

```
div.layout-container
  aside.sidebar                    ← 侧边栏
  div.main-area
    div.topbar                     ← 顶栏
    div.x-container
      el-card.box-card.no-padding  ← 搜索区
        [header] "🔍 查询"（纯 HTML 环境用 emoji 或文字替代 svg-icon）
        [body]   el-form(label-position="right" label-width="150px")
                   el-row(:span="24")
                     el-col(:xs="24" :sm="8") > el-form-item > el-input/el-select
        [footer] el-button(type="success" 新增) + el-button(type="primary" 查询) + ...
      el-card                      ← 表格区
        el-table(size="mini" stripe border height="100%")
          el-table-column(align="center" 居中)
          el-table-column(操作列: el-button--text)
        el-pagination(background :page-sizes="[10,20,30,50]" layout="total, sizes, prev, pager, next, jumper")
      el-dialog(:close-on-click-modal="false")  ← 弹窗内嵌
        el-form > el-form-item
        [footer] el-button(取消) + el-button(type="primary" 确认保存)
```

### 关键规则

- 搜索栏使用 `el-card` + `el-form` 组合，不是独立的搜索组件
- 表格填充 5-8 行基于 PRD 数据模型的示例数据
- 空状态使用 `el-table` 的 `empty-text="暂无数据"` 属性
- 操作列使用 `el-button--text` 样式（查看、编辑、删除等）
- 操作列按钮超过 3 个时，前 2 个直接展示，其余收入"更多"下拉菜单（`el-dropdown`）

### 列宽分配策略

| 列类型 | 宽度策略 | CSS class |
|--------|---------|-----------|
| 序号/勾选列 | 固定 60px | `.col-index` |
| 状态列 | 固定 100px | `.col-status` |
| 日期列 | 固定 160px | `.col-date` |
| 操作列 | 固定 120-180px（按按钮数调整） | `.col-action` |
| 内容列 | 自适应（`min-width` 控制最小宽度） | 默认 |

**超过 8 列时**：固定首列（序号）和尾列（操作），中间列开启横向滚动。使用 `table-layout: fixed` + `overflow-x: auto` 的外层容器实现。

### 空状态文案规范

原型中只实现默认空态（`暂无数据`），以下文案供交互说明引用：

| 空态类型 | 文案模板 | 使用场景 |
|---------|---------|---------|
| 初始空态 | "暂无数据，请点击「新增」创建{业务对象}" | 列表从未有过数据 |
| 搜索空态 | "未找到匹配结果，请调整搜索条件后重试" | 有筛选条件但无匹配 |
| 异常空态 | "数据加载失败，请刷新页面重试" | 接口异常或网络错误 |

---

## 详情页 (detail)

展示单条数据的完整信息，通常从列表页跳转进入。

### 结构模板

```
div.layout-container
  aside.sidebar
  div.main-area
    div.topbar
    div.x-container
      el-breadcrumb                ← 面包屑导航（可点击跳转）
        <a href="list.html">列表页</a>
        <span class="separator">/</span>
        <span class="current">详情</span>
      el-card                      ← 基础信息卡片
        [header] "基本信息" + 操作按钮（编辑、删除、状态变更等）
        [body] 信息展示（key-value 或描述列表）
      el-tabs                      ← 关联信息 Tab（如有多维度数据）
        el-tabs__header
          el-tabs__item 标签1
          el-tabs__item 标签2
        el-tabs__content
          el-tab-pane 内容区
```

### 关键规则

- **面包屑必须可点击跳转**：使用 `<a href="xxx.html">` 实现，非纯文字展示
- 基础信息使用 2-3 列布局展示 key-value 对
- 操作按钮放在卡片 header 的右侧
- Tab 内可嵌套表格或其他卡片

---

## 表单页 (form)

用于新建或编辑数据，可作为独立页面或弹窗内容。

### 结构模板

```
div.layout-container
  aside.sidebar
  div.main-area
    div.topbar
    div.x-container
      el-breadcrumb                ← 面包屑导航（可点击跳转）
        <a href="list.html">列表页</a>
        <span class="separator">/</span>
        <span class="current">新建/编辑</span>
      el-card                      ← 主表单卡片（字段多时用多个 el-card 分组）
        el-form(label-position="right" label-width="根据最长标签调整")
          el-row + el-col(:sm="8") 栅格布局排列字段
        [footer]
          el-button(取消) + el-button(type="primary" 确认保存)
```

### 关键规则

- **面包屑必须可点击跳转**
- 字段类型、必填标记、校验提示与 PRD 一致
- 使用 `el-row` + `el-col` 栅格布局，默认 3 列（`:sm="8"`）
- 字段较多时用多个 `el-card` 分组展示，每个卡片有标题
- 底部按钮（取消 + 确认保存）固定在表单底部

---

## 弹窗 (modal)

不单独生成文件，嵌入在触发它的页面中。

### 结构模板

```
div.el-dialog-overlay              ← 遮罩层
  div.el-dialog(width="600px")
    div.el-dialog__header
      span 弹窗标题
      button.el-dialog__close ×
    div.el-dialog__body
      el-form > el-form-item      ← 表单内容
    div.el-dialog__footer
      el-button(取消) + el-button(type="primary" 确认保存)
```

### 关键规则

- 默认宽度 600px，`:close-on-click-modal="false"`
- 点击按钮弹出，点击关闭/遮罩收起
- **单页弹窗数量建议控制在 4 个以内**；超过 4 个时评估是否可复用同一弹窗或拆分为独立页面
- 弹窗开关交互用纯 JS 实现（切换 `.visible` class）

---

## 概览页 (dashboard)

展示关键指标和趋势，用于运营后台或管理驾驶舱。

### 结构模板

```
div.layout-container
  aside.sidebar
  div.main-area
    div.topbar
    div.x-container
      div.dashboard-grid           ← 指标卡片区（4 列栅格）
        div.stat-card              ← 单个指标卡片（重复 N 个）
          div.stat-card__title  指标名称
          div.stat-card__value 数值
          div.stat-card__footer 同比/环比描述
      el-row                       ← 图表区（2 列布局）
        el-col-12
          el-card
            [header] "趋势图"
            [body] div.chart-placeholder "📊 {图表名称} — 详见 PRD 数据指标定义"
        el-col-12
          el-card
            [header] "分布图"
            [body] div.chart-placeholder "📊 {图表名称}"
      el-card                      ← 明细表格区（可选）
        el-table + el-pagination
```

### 关键规则

- 指标卡片使用 `.dashboard-grid`（4 列）布局，每个卡片展示：指标名、数值、变化趋势
- 图表区使用 `.chart-placeholder` 占位，标注图表名称和数据来源，不需要实现真实图表（默认高度由 styles.css 控制）
- 指标数值使用基于 PRD 的示例数据（如"1,234"、"98.5%"），不用占位符
- 明细表格复用列表页的 `el-table` 样式

---

## 变更标记规范

存量改造或原型迭代时，使用以下标记方式标识变更点。

### 徽标位置规则

| 变更类型 | 标记方式 | CSS class | 放置位置 |
|---------|---------|-----------|---------|
| **新增** | `NEW` 红色徽标 | `.new-badge` | **放在 `<label>` 标签内**，紧跟字段名称冒号后 |
| **修改** | `UPDATED` 橙色徽标 | `.updated-badge` | **放在 `<label>` 标签内**，紧跟字段名称冒号后 |
| **删除** | 删除线 | `.deleted-mark` | 添加到被删除元素的 class 上 |

### 正确示例

```html
<!-- ✅ 正确：徽标在 label 内 -->
<div class="el-form-item">
  <label class="el-form-item__label">IVR队列：<span class="new-badge">NEW</span></label>
  <div class="el-form-item__content">
    <select class="el-select">...</select>
  </div>
</div>

<!-- ✅ 正确：修改标记 -->
<div class="el-form-item">
  <label class="el-form-item__label">工单来源：<span class="updated-badge">UPDATED</span></label>
  <div class="el-form-item__content">...</div>
</div>
```

### 错误示例

```html
<!-- ❌ 错误：徽标在 content 内，宽度有限时会被挤到下一行 -->
<div class="el-form-item">
  <label class="el-form-item__label">IVR队列：</label>
  <div class="el-form-item__content">
    <select class="el-select">...</select>
    <span class="new-badge">NEW</span>   <!-- 容易独占一行 -->
  </div>
</div>

<!-- ❌ 错误：使用区域底色高亮（不直观，干扰阅读） -->
<div class="modified-highlight">
  <div class="el-form-item">...</div>
</div>
```

### 表格列/按钮的标记

对于表格列或按钮的新增/修改，徽标放在列标题或按钮文字后：

```html
<!-- 表格列标题 -->
<th>处理结果 <span class="new-badge">NEW</span></th>

<!-- 按钮 -->
<button class="el-button">批量分配 <span class="new-badge">NEW</span></button>
```

---

## 通用交互规则

### 页面间导航

- 侧边栏菜单使用 `<a href="xxx.html">` 链接到各页面，当前页添加 `.active` class
- 详情页和表单页的面包屑必须使用 `<a>` 标签实现可点击跳转
- 列表页不需要面包屑（框架自动渲染，原型中省略）

### 共享脚本

当原型超过 3 个页面时，将通用交互函数提取到共享的 `scripts.js`：

```javascript
// 弹窗开关
function openDialog(id) {
  document.getElementById(id).classList.add('visible');
}
function closeDialog(id) {
  document.getElementById(id).classList.remove('visible');
}

// Tab 切换
function switchTab(tabGroup, index) {
  const header = document.querySelector(`[data-tab-group="${tabGroup}"]`);
  if (!header) return;
  header.querySelectorAll('.el-tabs__item').forEach((item, i) => {
    item.classList.toggle('active', i === index);
  });
  const content = header.nextElementSibling;
  content.querySelectorAll('.el-tab-pane').forEach((pane, i) => {
    pane.classList.toggle('active', i === index);
  });
}

// 下拉菜单（操作列"更多"）
function toggleDropdown(el) {
  const menu = el.nextElementSibling;
  menu.classList.toggle('visible');
}
document.addEventListener('click', function(e) {
  if (!e.target.closest('.el-dropdown')) {
    document.querySelectorAll('.el-dropdown-menu.visible').forEach(m => m.classList.remove('visible'));
  }
});
```

各页面通过 `<script src="scripts.js"></script>` 引用。

### 交互组件状态展示规范

静态原型无法靠用户点击来展示交互流程。对于复杂交互组件（树形选择器、级联选择器、多选标签框等），必须在原型中**至少展示一个实例的完整交互状态**，让读者无需操作即可理解组件的使用方式。

#### 必须展示的状态

| 交互动作 | 静态原型中的体现方式 |
|---------|-------------------|
| **选中** | 输入框内显示已选项的 tag（含具体数据），下拉面板中对应 checkbox 为勾选态 |
| **删除** | tag 上显示 × 关闭按钮 |
| **搜索** | 下拉面板顶部的搜索框内填入示例关键字 |

#### 执行规则

1. **同类组件首次出现时**：至少有一个实例保持「下拉面板展开 + 已选中若干项 + 搜索框有内容」的状态，完整体现交互链路
2. **同类组件后续出现时**：可仅展示「已选中」的收起态（输入框内有 tag + × 按钮），无需重复展开面板
3. **所有实例均不可为纯空状态**：即使是示意"未填写"的场景，也应至少有一个同类组件展示已填写态作为对照
4. **单页展开数量限制**：单页最多展开 2 个不同类型的复杂组件交互状态，其余组件展示收起态即可

#### 示例

```html
<!-- ✅ 正确：首个树形选择器展开面板，展示完整交互 -->
<div class="tree-select-box" style="border-color:var(--color-primary);">
  <span class="tag">已退费 <span class="close">&times;</span></span>
  <span class="tag">退费确认 <span class="close">&times;</span></span>
  <div class="tree-dropdown visible">
    <input class="tree-search" value="退费">
    <div class="tree-columns">...</div>
  </div>
</div>

<!-- ✅ 正确：后续同类组件可仅展示收起态 -->
<div class="tree-select-box">
  <span class="tag">400-888-0001 <span class="close">&times;</span></span>
  <span class="tag">VIP客服队列 <span class="close">&times;</span></span>
</div>

<!-- ❌ 错误：所有实例都是空状态 placeholder -->
<div class="tree-select-box">
  <span class="placeholder">请选择...</span>
</div>
```
