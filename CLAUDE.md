# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

一个个人记账 Web 应用（"记账小助手"），依据产品需求文档实现。整个应用是**一个自包含的单文件**：`产品prd/index.html` —— HTML、CSS、JavaScript 全部内联，无构建步骤、无依赖、无网络请求。

- `产品prd/记账小助手-PRD.md` —— 作为唯一事实来源的 PRD（中文）。功能编号（F1-1、F2-3……）、分类清单、页面结构、验收标准都来自此文档。新增或修改功能前请先查阅。
- `产品prd/index.html` —— 已实现的应用（V1.0 MVP：快速记账、账单流水、分类、报表）。

## 运行 / 测试

直接用浏览器打开 `产品prd/index.html`（双击，或 `file://`）。无需服务器、安装或构建。应用是**移动端优先**的 —— 请使用浏览器的设备模拟视图 / 窄窗口（设计宽度约 460px）。首次加载会注入演示数据，以免报表为空。

项目没有测试框架。若要快速验证纯逻辑（如计算器引擎），用单行 `node -e '...'` 命令跑代码片段。

> **Shell 注意事项：** 本环境的 Bash 有初始化 bug，多行命令和 heredoc 会失败（向不存在的路径写 cwd 文件 → 退出码 127）。单行 `node -e` 命令可用。文件操作请优先使用 Read/Write/Edit/Glob/Grep 工具，而非 shell。

## 架构

所有代码都在 `产品prd/index.html`，自上而下组织为：CSS 设计令牌 → 标记骨架 → 单个 `<script>`。关键结构要点：

- **状态**是单一的 `db` 对象 `{ records:[{id,type,amount,cat,note,date}], budget }`，通过 `load()` / `save()` 持久化到 `localStorage`，键名为 `jz_helper_v1`。所有数据都在本地 —— 无账号、无同步（这是 PRD 中刻意的隐私定位）。
- **渲染是手制的** —— 无框架。`render()` 根据模块级变量 `activeView`（`detail` | `report` | `mine`）分发到 `renderTop()` + `renderDetail/renderReport/renderMine`，这些函数重建 `innerHTML` 并重新绑定事件。任何状态变更后，先调 `save()` 再调 `render()`。
- **`cursor`**（一个 `Date`）表示当前查看的月份；`recsOfMonth()` 按 `monthKey(cursor)` 过滤 `db.records`。月份导航会修改 `cursor` 并重新渲染。
- **快速记账抽屉**是标志性 UI：底部抽屉配自定义数字键盘。`entry` 草稿对象驱动它；`handleKey()` + `currentValue()` 实现了一个小型 `+`/`-` 算式引擎（如 `15+8`）。`openSheet(id)` 同时承担编辑（传记录 id）与新建（传 null）。
- **图表是手写的 SVG/CSS**（环形图 = 用 `stroke-dasharray` 叠加的 `<circle>`；趋势图 = flexbox 柱状）。系列颜色取自固定的 `SERIES` 数组 —— 一个经过色盲安全（CVD）验证的分类调色板。保持顺序固定，且绝不让文字仅靠颜色传达含义（图例始终显示金额 + 百分比）。
- **`showModal()`** 是唯一可复用的对话框（确认 / 文本输入 / 列表），用于预算、删除确认、分类下钻、清空数据。

## 约定

- 分类在 `CATS` 常量中统一定义（支出 + 收入，每项为 `[名称, emoji]`）。PRD 固定了这些清单 —— 需与之一致。
- 金额：存储原始数字；仅在显示时通过 `fmt()` 格式化。凡是金额需按列对齐处，使用 `tabular-nums`（`.tnum` / `--tnum`）。
- 颜色与字体来自文件顶部的 CSS 自定义属性（`--brand` 绿色 = 收入/正向，`--expense` 橙色 = 支出）。新 UI 应从这些令牌派生，而非硬编码十六进制色值。
- 尊重 `prefers-reduced-motion`（已全局处理）；新增交互时保持键盘/触摸的对等支持。
