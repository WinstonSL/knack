# Changelog

按倒序记录每个版本的变更（最新在最上）。AI 续接时通常只看最近 1-2 版即可。

格式约定：
- **Added** 新增能力
- **Changed** 改动行为
- **Fixed** 修 bug
- **Notes** 设计取舍 / 已知问题

---

## 规则变更 — 2026-06-11

**版本命名规则升级为三位数 `vX.Y.Z`**（用户明确指示）。

- 从 v2.3 之后，每次普通迭代仅 `Z+1`：v2.3 → v2.3.1 → v2.3.2 → … → v2.3.9
- `Z` 到 9 后**不进 10**，而是 `Y+1, Z=0`：v2.3.9 → v2.4.0
- 大版本 `X+1` 仅用于破坏性改动，且必须用户明确授权
- "覆盖不升版"成为唯一例外，且必须用户明确授权 + 在 Notes 里注明
- 详见 `AGENTS.md` §1

---

## v2.3 — 2026-06-11

可拉伸 / 可折叠侧栏 + 整套高级感视觉系统重构 + 深色模式（同日内补丁，未升版）。

### Added
- **左右两栏可鼠标拖拽调宽**：栏与栏之间各加一根 6px 的 `.resizer`，鼠标按住能左右拉伸；双击重置默认宽度。
- **左右两栏可一键折叠成 36px 窄条**：每个 `col-head` 加 `.collapse-btn`；折叠后窄条本身（`.collapse-strip`）也是可点击的展开按钮，纵向竖排显示栏标题。
- **限度**：`outlineCol` 180~420px（默认 240），`sideCol` 320~560px（默认 380）。
- **状态持久化**：宽度和折叠态写入 `localStorage[LAYOUT_KEY='knack_layout_v1']`，下次打开自动恢复。
- **快捷键**：`⌘B` 折叠左栏 / `⌘\` 折叠右栏（IDE 通用习惯）；按钮 title 也有提示。
- 关键代码：`restoreLayout` / `bindResizer` / `setCollapsed` / `toggleCollapsed` / `clampWidth`。
- **深色模式**：顶栏右上角加切换按钮，三档循环 `跟随系统 → 浅色 → 深色`，写 `localStorage[knack_theme_v1]`。
  - 通过 `html[data-theme="dark"]` 覆盖 CSS 变量，整套设计 token 自动反色。
  - **iframe 预览舞台保持白色**（`--k-stage-bg`），用户 HTML 大多是浅色稿，反色会破坏所见即所得（业内做法同 Figma / Notion）。
  - `<head>` 内置极小 bootstrap 内联脚本，先于 `<style>` 设置 `data-theme`，避免 FOUC。
  - 跟随系统模式下监听 `prefers-color-scheme` 媒体查询，系统切换会实时跟进。
  - 关键代码：`applyTheme` / `cycleTheme` / `resolvedTheme` / `bindTheme`；常量 `THEME_KEY` / `THEME_ORDER` / `THEME_META`。

### Changed
- **整套视觉系统重构**，从「品牌色到处用」改成「品牌色只做点缀」，整体走高级中性灰路线：
  - 引入完整设计 token：中性色阶（`--k-ink/-2/-3/-4` + `--k-bg/-soft/--k-surface`）、半透明描边（`--k-line` / `--k-line-strong` 用 `rgba(0,0,0,.07/.12)`）、圆角（`--r-sm/-md/-lg`）、阴影三档（`--shadow-1/2/3`）。
  - 顶栏：高度 50→52px；按钮组用 `.group` + `::before` 微妙竖线分隔；按钮去掉 emoji，改纯文字（打开 / 粘贴 / 导出 / 设置）；撤销重做用箭头符号 `↶ ↷`；设置按钮带边框（`.btn.bordered`）。
  - 品牌名「Knack」用 `background-clip:text` 渐变填充，副标小到 10.5px。
  - 空态从「粉橘渐变 + 大表情」改成「径向微光 + 细描边卡片 + uppercase 标签」，更克制有质感。
  - prompt 输入区 focus 时不再用品牌橙发光，改成 ink 黑细描边 + 极淡阴影（更专业）。
  - prompt-tag hover 从暖橙底改成黑底白字（克制）。
  - 澄清面板从暖橙渐变改成「白底 + 左侧 3px 橙色 accent bar」（信息卡片范式）。
  - 修改历史 hover 从「行内淡化 + ↶ 居中浮出」改成「左侧 2px 橙竖条 + 右侧『回滚到这里』标签」，更克制更明确。
  - toast 加上滑入动画 + 更深的阴影；spinner 边框更细、更细腻。
  - 模态用半透明黑 + `backdrop-filter:blur(4px)`；输入框 focus 用 ink 黑而非品牌橙。
  - 自定义滚动条样式（8px，半透明灰）。
  - kbd 标签首次引入，用于快捷键提示。
- 修复 layout 高度算式：`calc(100% - 48px)` → `calc(100% - 52px)`，跟上 topbar 实际高度。

### Fixed
- v2.2 顶栏的手工竖线分隔（`<span style="width:1px;...">`）改成 CSS `.group + .group::before`，更稳更优雅。

### Notes
- v2.2 已搬入 `archive/`。
- 这次没动 LLM 调用协议、JSON 解析三道防线、点选与框选逻辑 —— 那些都是稳定的核心。
- 「品牌色降饱和」策略：橙黄渐变只用在 logo 和 `.btn.brand`（发送给 AI / 选择文件 / 澄清-发送）三个地方，其他地方一律中性色 —— 这才是让产品看起来「高级」的关键。
- **关于「深色模式同日补丁未升版」**：用户明确要求覆盖 v2.3 不再起新版本号（"免得就这么点功能又搞个新版本"）。这是对铁律的一次特例豁免，仅限本次。后续如再加功能仍按 v2.4 起步。
- **同日二次补丁（覆盖 v2.3，未升版）**：用户反馈深色模式下多处白底白字、按钮过小等可用性问题，本次仅做补丁。
  - 引入「配对色 token」：`--k-on-ink`（与 `--k-ink` 背景对比的文字色）、`--k-on-ink-soft/-2`（选中态次级 highlight）、`--k-hover`（主题感知的 hover 叠色）。
  - 引入「浮动工具栏专用 token」：`--k-floating-bg/-text/-hover`（不随主题翻转，永远是高对比工具条）。修复浮动工具栏（锁定/复制/删除那条）在深色下白底白字看不清的 bug。
  - 引入「行内代码 token」：`--k-code-bg/-text` + `.k-code` 类。修复「当前选中」面板里 `<code style="background:#fff">` 在深色下白底白字看不清的 bug。
  - 替换所有 `color:#fff` 配 `var(--k-ink)` 的旧写法，改为 `var(--k-on-ink)`。涉及：`.btn.primary`、`.tree-node.sel`、`.crumb-item.current`、`.prompt-tag:hover`、`.toast`。
  - 替换所有硬编码 `rgba(15,17,21,.05)` hover 叠色，改为 `var(--k-hover)`。涉及：`.btn:hover`、`.tree-node:hover`、`.collapse-strip:hover`。
  - 替换 JS 内联 `color:#bbb`、`color:#777`、`color:#f0a040` 等硬编码颜色，全部走 CSS 变量。
  - **侧栏折叠按钮**：从 22×22 透明小图标升级为 28×28 带描边按钮，hover 反色，更易点更显眼。
  - **拖拽分隔条**：hover 时橙色 indicator 从 36px 拉长到 56px、变粗 2→3px，分隔线也整体染橙，提示性更强。
  - **顶栏撤销/重做**：从纯箭头 `↶ / ↷` 改为 `↶ 撤销 / ↷ 重做` 带文字标签，title 也写清「重做」的含义（撤销后想反悔时用）。
  - **favicon**：浏览器标签页图标从空白图换成内联 SVG 版 Knack logo（data URI，仍是单文件无依赖）。
  - 一同顺手修：浮动工具栏边框透明度提升、toast error/success/info 的边框设为 transparent（避免在深色下出现细黑边）。

---

## v2.2 — 2026-06-11

UI 品牌化 + 可暂停 AI + 修澄清面板重叠 bug。

### Added
- **AI 调用可暂停**：「发送给 AI 改写」旁新增 ⏸ 暂停按钮，调用中按一下即中断当前 fetch（`AbortController`）。`ESC` 也能触发。
- `startAI` / `endAI` / `abortAI` / `isAbortError` 一组 AI 调用生命周期函数，统一控制 loading、按钮 disabled 和 abort signal。
- 空态新增渐变 logo + tagline，整体颜值升级。
- Loading 区新增子提示「点右侧 ⏸ 暂停可中断本次调用」。

### Changed
- **去掉「汇报」字样**：空态主标语改为「把 HTML 拖到这里」，placeholder 也改成更通用的引导文案。
- 引入品牌色变量（`--k-orange` / `--k-orange-light` / `--k-purple` / `--k-yellow` / `--k-grad`），主按钮、prompt focus、hover 态、spinner、history hover、prompt-tag hover 全部统一到品牌橙。
- 顶栏品牌名用渐变文字（`-webkit-background-clip:text`）。
- `.btn.brand` 新主按钮样式（橙紫渐变 + 阴影），用于「发送给 AI」「澄清-发送」「空态-选择文件」。
- 修改历史去掉「（点击回滚）」灰字提示，改用 hover 时浮出 ↶ 图标 + 行内淡化的视觉表达。
- 澄清面板配色升级（暖橙渐变 + 阴影 + hover 微动效），更精致也更品牌化。
- 空态文案补充「所有改动都只在你的本地浏览器，不会上传任何内容」，强化信任。

### Fixed
- **澄清面板与修改历史重叠的视觉 bug**：根因是 `.prompt-area` 没有滚动，澄清面板长起来会越过侧栏边界压到「修改历史」面板。修复方式：
  - `.prompt-area` 加 `overflow-y:auto`
  - `.clarify` 加 `max-height:46vh; overflow-y:auto`
  - 双重保险：内部滚动 + 外层兜底

### Notes
- 文件名严格遵循铁律：v2.1 → v2.2，新建文件不覆盖。
- v2.1 已搬入 `archive/`。
- 本次没有动 LLM 调用协议、状态机、JSON 解析三道防线 —— 那些 v2.1 已经稳定。

---

## v2.1 — 2026-06-10

品牌迁移 + JSON 解析修复版。

### Added
- 品牌升级为 **Knack**（原"HTML 修改器"）。顶栏内联 SVG logo（橙黄→紫蓝渐变 + 魔法笔 + 高光星）。
- `assets/knack-logo.svg` 单独素材，无损缩放给宣传用。
- `formatAIError`：把 401 / 403 / 404 / 429 / 5xx / JSON 解析失败分别给人话提示。

### Fixed
- AI 返回 JSON 时常出现未转义双引号导致 `JSON.parse` 失败 → 加三道防线：
  1. 直接 parse
  2. 清掉控制字符再 parse
  3. `repairJsonStrings` + `scanAndEscape` 智能修复 `html` / `label` / `detail` / `question` / `text` / `content` 字段内的未转义双引号
- 强化 system prompt 的 JSON ESCAPING 段，提示模型 SVG / 双引号必须转义。

### Notes
- 文件名首次启用严格命名规则（v2.1 而非覆盖 v2.0）。
- 旧版 `html修改器-v1.html` / `html修改器-v2.html` 已删除，迁入 `Knack/` 目录。

---

## v2.0 — 2026-06-10

点选与 AI 交互的体验大升级。

### Added
- **F1 精准点选**：`getElementAtPoint` + 同点循环扩大选区 + Alt 进入子级。
- **F2 框选修复**：重写 `startMarquee`，按下拖出 5px 才触发，避免误触。
- **a 双击编辑文字**：`startEditing` + contenteditable，双击直接改。
- **c 浮动工具栏**：选中元素后浮出工具栏 —— 上下移、编辑、复制、锁定、复制 HTML、删除。
- **g 左侧大纲树**：`renderOutline` + `findByPath` + `nodePath`，跨结构精准跳转。
- **j 自动保存草稿**：`saveDraft` 写 `localStorage[DRAFT_KEY]`，`checkDraftOnStart` 启动时提示恢复。
- **N1 AI 二阶段澄清**：
  - `runRewriteFlow` 先判定意图
  - 返回 `{action:"execute"|"options"|"ask"}` 三种策略
  - 可枚举 → `showClarify` 出选项卡；不可枚举 → 直接问让你打字回答

### Notes
- 锁定能力（h）和单独导出选中 HTML（k）此版已实现，藏在浮动工具栏里。

---

## v1.0 — 2026-06-10

第一个能跑的最小版本。

### Added
- 单文件 HTML，浏览器全屏跑，无外部依赖。
- 基础点选：单选 / Shift 多选 / 框选（粗糙版）。
- 路径面包屑显示当前选中元素层级。
- 自然语言改写（一阶段，直接发给 AI 执行）。
- 撤销 / 重做 / 导出 / 设置面板（API Key / Base URL / Headers / 模型名）。

### Notes
- 框选有误触 bug，v2.0 修。
- AI 调用没有错误友好化，401 / 网关挂会直接抛原始报错，v2.1 修。
