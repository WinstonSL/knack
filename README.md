# Knack

点选 + 自然语言改 HTML。不懂代码也能像改 PPT 一样改 HTML 稿件。

## 当前最新版

- **入口文件**：`Knack-v2.3.html`（双击在浏览器打开）
- **历史版本**：见 `archive/`
- **变更日志**：见 `CHANGELOG.md`
- **项目铁律**：见 `AGENTS.md`（AI 改本目录任何文件前**必读**）
- **版本命名**：从 v2.3 起采用三位数 `vX.Y.Z`，详见 `AGENTS.md` 第 1 条。下一次迭代命名为 `Knack-v2.3.1.html`。

## v2.3 能力快照

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

- 中文工作名「巧刻」，对外正式名 **Knack**。
- 主色：橙黄渐变（#FF8A3D → #FF6A00）+ 紫蓝点缀（#7C5CFF）+ 高光黄（#FFD66B）。
- 气质：活泼、好上手、敢动手改。
