# Knack

点选 + 自然语言改 HTML。不懂代码也能像改 PPT 一样改 HTML 稿件。

## 当前最新版

- **入口文件**：`Knack-v2.3.4.html`（双击在浏览器打开）
- **历史版本**：见 `archive/`
- **变更日志**：见 `CHANGELOG.md`
- **项目铁律**：见 `AGENTS.md`（AI 改本目录任何文件前**必读**）
- **版本命名**：从 v2.3 起采用三位数 `vX.Y.Z`，详见 `AGENTS.md` 第 1 条。下一次迭代命名为 `Knack-v2.3.5.html`。

## v2.3.4 能力快照

- **点选**：单选 / Shift 多选 / 拖拽框选 / Alt 进入子级 / 同点循环扩大
- **编辑**：双击改文字 + 浮动工具栏（上下移、复制、锁定、复制 HTML、删除）
- **大纲**：左侧大纲树跨结构跳转；可拖拽调宽、可折叠成窄条（v2.3 新增）
- **AI 改写**：二阶段澄清（直接执行 / 给选项卡 / 让你打字补充）+ 错误友好化
- **AI 可暂停**：调用中点「暂停」或按 `ESC` 可中断
- **工作区**：右侧工作区也可拖拽调宽、可折叠；宽度与折叠态写 localStorage（v2.3 新增）
- **快捷键**：`⌘B` 折叠左栏 / `⌘\` 折叠右栏 / `⌘Z` 撤销 / `⇧⌘Z` 重做 / `⌘Enter` 发送 / `ESC` 暂停 AI
- **草稿**：localStorage 自动保存，下次打开提示恢复
- **导出**：整页另存 / 选中 HTML 单独复制
- **UI**：v2.3 引入完整设计系统（中性色阶 + 圆角/阴影 token + 极简按钮 + 高级感留白），品牌橙仅作点缀
- **深色模式**：顶栏右上角切换，三档「跟随系统 / 浅色 / 深色」；iframe 预览舞台保持白色不反色
- **尺寸/比例改写更准**（v2.3.1 新增）：发指令时同时把元素的当前渲染尺寸 + 父容器尺寸 + computed style 一起喂给 LLM；指令含「三分之一 / 一半 / 放大一倍」等比例词时按当前真实尺寸换算，输出 px 值并兼容 flex/grid。
- **删除图标更直觉**（v2.3.1 改）：浮动工具栏的删除按钮从 `✕` 改为 `🗑`，避免被误认作「关闭工具栏」。
- **多选模式**（v2.3.2 新增）：顶栏 `➕ 多选 / ➖ 取消选 / 🚫 清空` 三按钮。多选下点击元素 = 加入选区，不再触发"扩父级"；右栏选区列表每行带 `⬆ / ×` 按钮，可对单个选中项独立升级到父级或移除（也能"全部升级"）。
- **全局对话模式**（v2.3.2 新增）：不选元素直接输入指令，会对整个 HTML 改写。能搞定"卡片排成 3 列"、"互换位置"等结构性重排；指令含「主题色 / 整体风格 / 暗色化」等关键词时允许动 `<head>` 里的 `<style>`，否则保守只改 `<body>`。
- **AI 输入框模式提示**（v2.3.2）：状态栏显示 `🌐 全局模式` 或 `🎯 已选 N 个`；输入框 placeholder 根据选区自动切换。
- **全局模式更稳**（v2.3.3 修）：全局对话返回的 JSON 中 `body` / `head_styles` 字段加入兜底修复名单；prompt 强化转义示例 + 反例；最终保留正则兜底——LLM 真把 JSON 写废了，仍能从原始返回里抠出 `<body>...</body>` 完成改写。
- **防误操作 + 大文件保护**（v2.3.4 修）：
  - 全局模式发送前估算 HTML 体积，超 80KB 弹"是否继续"，超 200KB 强劝退（避免撞模型上下文上限）。
  - 全局改写完成 toast 提示"页面内交互如失效，可重新打开 HTML"。
  - 点 `🚫 清空` 时同步退出多选模式（不再残留橙边）。
  - 多选 ON + 选区为空时状态栏正确显示「🎯 多选 · 等你选元素」，不再误显示全局模式。
  - AI 调用进行中：禁用输入框 + 顶栏选区按钮，再次点发送会被拦下并提示。

## 配置

首次打开点「设置」填：

- API Key（公司内 LLM key 即可，OpenAI 兼容协议）
- Base URL（自动补 `/chat/completions`）
- 额外 Headers（公司网关如需）
- 模型名

## 跨对话续接（重要）

新对话第一句直接用：

> 继续改 Knack。先读 `~/dchat-workplace/Knack/README.md` 和 `AGENTS.md`，按规则做：XXX

AI 会自动加载 `AGENTS.md` 的铁律 + 当前快照 + 必要时翻 `CHANGELOG.md`。**无需依赖对话记忆**。

## 关键代码位置（v2.3）

- `parseJSON` / `repairJsonStrings` / `scanAndEscape`：JSON 三道修复
- `runRewriteFlow` / `executeRewrite` / `showClarify`：AI 二阶段澄清
- `startAI` / `endAI` / `abortAI` / `isAbortError`：AI 调用生命周期与暂停
- `restoreLayout` / `bindResizer` / `setCollapsed` / `toggleCollapsed`：侧栏拖拽与折叠（v2.3）
- `applyTheme` / `cycleTheme` / `resolvedTheme` / `bindTheme`：深色模式（v2.3 patch）
- `getElementAtPoint` / `startMarquee`：点选与框选
- `floatToolbar`：浮动工具栏
- `renderOutline` / `findByPath`：大纲树
- `formatAIError`：错误分类

## 关键常量

- `LOCK_ATTR='data-hm-lock'`
- `EDIT_ATTR='data-hm-editing'`
- `CFG_KEY='html_modifier_cfg_v1'`
- `DRAFT_KEY='html_modifier_draft_v1'`
- `LAYOUT_KEY='knack_layout_v1'`（v2.3 新增，记侧栏宽度+折叠态）
- `THEME_KEY='knack_theme_v1'`（v2.3 patch，记主题：system / light / dark）
- `LAYOUT_LIMITS={outlineCol:{min:180,max:420,def:240}, sideCol:{min:320,max:560,def:380}}`
- CSS 变量：`--k-orange` / `--k-orange-light` / `--k-purple` / `--k-yellow` / `--k-grad`
- 新增中性色阶：`--k-ink/--k-ink-2/--k-ink-3/--k-ink-4` / `--k-bg/--k-bg-soft/--k-surface` / `--k-line/--k-line-strong`
- 新增 token：`--r-sm/--r-md/--r-lg` 圆角 / `--shadow-1/--shadow-2/--shadow-3` 阴影

## 后续候选迭代

- 流式输出
- 主题切换
- 贴图给 AI
- 形态升级方向：申域名 + Landing Page（详见 2026-06-11 产品方向讨论）

## 品牌

- 中文名「巧刻」，对外正式名 **Knack**。
- 主色：橙黄渐变（#FF8A3D → #FF6A00）+ 紫蓝点缀（#7C5CFF）+ 高光黄（#FFD66B）。
- 气质：活泼、好上手、敢动手改。
