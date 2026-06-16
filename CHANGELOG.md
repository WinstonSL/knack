# Changelog

按倒序记录每个版本的变更（最新在最上）。AI 续接时通常只看最近 1-2 版即可。

格式约定：
- **Added** 新增能力
- **Changed** 改动行为
- **Fixed** 修 bug
- **Notes** 设计取舍 / 已知问题

---

## v2.3.4 — 2026-06-16

本版做一组防御性 patch，根据 v2.3.3 上线后的代码 review 收尾遗留的体验毛刺与边界 bug。

**Fixed / Changed**
- **A1 · 大文件 token 保护**：`runGlobalRewriteFlow` 在调用 LLM 前估算 `body.outerHTML` 字节数。
  - 80KB～200KB：弹 confirm 提示"体量偏大，建议先点选再改"，给用户取消机会。
  - 超 200KB：强劝退 confirm，文案明确"很可能撞模型上下文上限"。
  - 用户确认后才进入正常发送流程。
- **A2 · 全局改写后交互失效提示**：DOM 整页替换后用户原页面里的 inline `onclick` / 动画初始化等会丢，toast 文案补一句"页面内交互如失效，可重新打开 HTML"。
- **A3 · 清空选区时同步退多选**：点 `🚫 清空` 若处于多选模式，自动 `setSelectMode('single')`，避免清空后舞台还残留橙色边框。
- **A5 · 多选 0 选中时状态栏修正**：原逻辑 `selected.length === 0` 一律显示"🌐 全局模式"。修正后优先看 `selectMode`：多选 ON + 0 个选中显示「🎯 多选 · 等你选元素」，placeholder 也对应改为引导文案。
- **C4 · AI 调用中防重复**：
  - `applyAIRewrite` 入口加 `state.aiBusy` 拦截，重复点击 toast 提示"按 ESC 可暂停"，不会再启动新调用。
  - `startAI` / `endAI` 同时禁用 / 解禁 `#promptInput` 与顶栏 `➕ ➖ 🚫` 三个选区按钮。
  - 解禁时调 `syncSelectModeButtons()` 让选区按钮 disabled 状态按当前选区情况重新计算（不会误开启）。

**Notes**
- 80KB / 200KB 阈值是经验值（≈25K～40K token / ≈70K～120K token），真实失败上限和模型有关；后续可考虑做成可配置。
- A2 的 toast 提示是临时方案，根除"DOM 重建丢交互"需要更深的改造（保留事件监听、或 morphdom 类增量更新），等真实需求强时再做。
- A5 顺手把单选模式的 status 文案从「🎯 已选 N 个」改成「🎯 单选 · 已选 N 个」，三种模式（🌐 全局 / 🎯 单选 / 🎯 多选）格式对齐。

---

## v2.3.3 — 2026-06-11

本版只解决一个高频痛点：**全局对话模式经常因为 JSON 转义不齐而失败**。

**Fixed**
- 全局对话改写时 LLM 返回的 `{"body":"...","head_styles":"..."}` 经常因为 HTML 里双引号没逐个转义而 JSON.parse 失败，弹出"AI 返回的格式有错"。根因：v2.3.2 全局模式新加的 `body` / `head_styles` 字段没接入 v1.x 时代为局部模式 `html` 字段建好的兜底修复链。
- 修复 3 件套：
  - **JSON 兜底名单扩容**：`repairJsonStrings()` 的 `fieldNames` 加入 `body` / `head_styles`，复用同一套未转义双引号修复逻辑。
  - **全局 system prompt 强化**：JSON ESCAPING 段从 4 行扩成约 12 行，给出 CORRECT 示例 + WRONG 示例，明确禁止 markdown 围栏 / 单引号包裹属性 / 真换行等常见踩坑。
  - **全局模式专属"超级兜底"**：当标准 JSON 解析（含修复链）彻底失败时，**直接对原始返回文本用正则 `/<body\b[^>]*>[\s\S]*?<\/body>/i` 抠出 body**（如允许动 head 时，再正则抓 `<style>` 块），照常完成改写。toast 文案变成"全局改写完成（已用兜底解析）"提示用户。

**Notes**
- 三层兜底（标准 JSON → 修复链 → 正则）按从严到松的顺序尝试。最坏情况只要 LLM 返回里 body 标签完整，就一定能改成功。
- 局部改写（`runRewriteFlow / executeRewrite`）的 `html` 字段早就在兜底名单里，无需改动；这次只补全局模式新字段。
- 没动模型温度、没换 prompt 主体逻辑——纯防御性修复。

---

## v2.3.2 — 2026-06-11

本版聚焦两个高频痛点：选区表达力不够（没法可控多选）+ 局部改写做不了结构性重排。

**Added**
- **多选模式**：顶栏新增 `➕ 多选` 开关 + `➖ 取消选` 钩子 + `🚫 清空` 一键清。
  - 多选 ON 时点击元素 = 加入选区；点击已选元素无效（避免和"扩父级"歧义）。
  - 舞台外圈高亮橙色边框做模式提示；状态栏显示 `🎯 多选 · 已选 N 个`。
- **右栏选区列表升级**：选了 >1 个或处于多选模式时，右栏改用结构化列表呈现，每行带：
  - `⬆` 把这一项单独升级到父级（其他不动）
  - `×` 把这一项从选区移除
  - 列表底部还有 `⬆ 全部升级`（去重处理，避免父级和子项同时存在）
- **全局对话模式**：不选任何元素直接输入指令，AI 收到整个 `body.outerHTML` + `<head>` 内的 `<style>` 做参考，输出新的 body（必要时连同新的 head styles 一起返回替换）。
  - 用于结构性重排：「卡片排成 3 列」「把第二行第一张挪到第一行末尾」「整体加深背景对比度」等。
  - 默认只动 body；指令含 `主题色 / 整体风格 / 换色 / 暗色化 / theme / dark mode / palette / 配色` 等关键词时升级为允许重写 head `<style>`（D3 智能判断）。
- **AI 输入框模式联动**：placeholder 自动切换为「全局描述…」/「告诉 Knack 怎么改…」；顶栏状态栏显示当前模式。
- **Esc 行为分级**：先退出"取消选"待定态 → 再退多选模式（保留选区）→ 再清空选区。

**Changed**
- 浮动工具栏多选模式下隐藏（多选不需要"上下移/复制/锁定"等单元素操作）。
- AI 局部改写完成后自动退出多选模式，回到单选状态。
- 切多选模式时**保留**已有选区（B1）。

**Fixed**
- 结构性重排（跨元素挪位、改 grid 列数）做不到 —— 由"全局对话模式"承接。

**Notes**
- 多选模式下保留 Shift+Click 加减选（高级用户的快捷小道，与按钮共存）。
- 全局模式没做摘要压缩，超大 HTML 可能撞模型上下文上限；先观察实际使用，必要时再加压缩。
- 全局模式改完后默认清空选区（整页都换了，原选区已无意义）。

---

## v2.3.1 — 2026-06-11

本版聚焦两个真实使用反馈：尺寸类指令算不准、删除按钮图标被误认。

**Fixed**
- 改写时改尺寸/比例算不对（如「卡片改成原来的三分之一」）。根因：之前只把元素的 outerHTML 喂给 LLM，模型不知道当前渲染尺寸，无从换算比例。
  - 现在会一起发送：元素 `getBoundingClientRect()` 实际宽高、`getComputedStyle()` 的 width/height/font-size/display/box-sizing/margin/padding，以及父容器的 tag、宽高、display、flex-direction、grid-template-columns。
  - 在 system prompt 里加了 SIZE & PROPORTION RULES：要求按 current 快照换算（1/3 → round(current/3)、一半 → round(current/2)、翻倍 → round(current*2)），默认输出 px；父容器是 flex 时同时给 `flex:0 0 Npx + width:Npx`，避免被 flex 拉回原宽。
  - 副作用：所有跟尺寸/字号/位置相关的指令都会更准，不止「比例词」一类。

**Changed**
- 浮动工具栏的删除按钮图标 `✕` → `🗑`。原因：`✕` 在大多数用户认知里是「关掉工具栏」，容易误读。`🗑` 直接表达「删除」语义。
- README「当前最新版」「能力快照」段同步到 v2.3.1。

**Notes**
- 比例换算只在 system prompt 层引导 LLM；强约束（前端预先算好再喂）留给后续版本，必要时再做。
- 删除按钮的视觉颜色仍用 `.danger` class（红色调），保持危险动作的视觉语义。

---

## 品牌变更 — 2026-06-11

**Knack 中文名定为「巧刻」**（用户明确指示）。

- 替换 README.md 中的旧中文名「巧改」为「巧刻」
- HTML 主程序未出现中文名，无需改动
- 后续对外宣传统一使用：英文 Knack / 中文 巧刻

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
