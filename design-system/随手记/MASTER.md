# 随手记 设计系统

> 移动端 Web 优先 · 极简克制 · 零干扰

## 设计取向依据

「随手记」是一款移动端优先的轻量清单工具，对应「打开即记、记完就走」的核心价值。这决定了它的视觉气质：**极简、平静、克制**——任何多余的装饰（强色块、阴影、动效、装饰插画）都和「3 秒内记完一件事」的目标冲突。整个界面只有三类元素：日期分组、任务卡片、底部悬浮「+」。没有侧栏、没有顶栏菜单、没有品牌色块，让用户注意力完全落在「当下要做什么」。

### 用户画像适配

| 维度 | 来自 prd.md 的要点 | 对应设计决策 |
| --- | --- | --- |
| 用户熟练度 | 「要点 5 下才能输完」——对手指操作敏感、厌恶繁琐 | 大触控区（≥48px）、FAB 单点唤起、输入面板只一个必填项 |
| 使用频率 | 每天打开 5~10 次，单次停留 8~15 秒 | 极简主屏、零层级深度；首屏直接是任务列表，无欢迎页 |
| 注意力状态 | 通勤、午休、睡前——碎片化、被环境干扰 | 无开屏动画、无弹窗打扰；CSS 动效 ≤200ms，输入即保存 |
| 任务严肃度 | 「记一件芝麻小事」——非专业项目管理 | 无标签、无优先级、无截止时刻；只有日期一档 |
| 信息密度 | 一天 ≤15 条，多为短标题 | 列表紧凑但每条卡片之间留白 8~12px；折叠已完成降低视觉负担 |
| 情绪调性 | 厌倦开屏广告和强制注册——需要「被尊重感」 | 全站零广告位、零弹窗；空态文案温和（「记第一件事吧」而非「立即创建」CTA） |
| 核心转化动作 | 「点开就写、勾掉就走」 | 底部悬浮「+」常驻，1 步进入输入态；勾选用整行点击区 + 大圆圈 |

## UI 技术栈与组件策略

- **样式**：Tailwind CSS + CSS variables。`app/globals.css` 用 `@tailwind base; @tailwind components; @tailwind utilities;`；色彩、圆角、阴影走 CSS 变量（`--color-primary`、`--color-surface`、`--radius-card` 等），便于深色模式切换。
- **组件**：默认 Tailwind 组合原子样式。输入面板用 `position: fixed` + `backdrop-blur` 实现轻量弹层，**不**引入 shadcn/Radix；本 MVP 不需要 Dialog / Dropdown / Select 等复杂组件。
- **图表**：无（PRD 不含图表）。
- **禁止**：独立 CSS 文件堆叠、CSS-in-JS、用 emoji 充当功能图标、装饰性阴影、装饰性渐变背景。

## 图标 Iconography

- **图标库（全站唯一）**：`lucide-react`（与 Next.js / React 栈默认一致）。
- **安装包名**：`lucide-react`。
- **尺寸 token**：
  - 导航/分组头 `--icon-nav`：20px
  - 按钮/勾选内 `--icon-inline`：16px
  - 空态插画区 `--icon-empty`：48px
- **线宽 / 风格**：stroke 1.75，统一圆角端点；全站禁止多套线宽混用。
- **无障碍**：纯图标按钮必须 `aria-label`（例如 FAB「+」需要 `aria-label="新建任务"`）。
- **动作映射**（来自 `ui-interaction-planner`）：
  - 新建任务 → `Plus`（FAB）
  - 完成任务 → `Check` / `Circle`（未勾）/ `CheckCircle2`（已勾）
  - 删除 → `Trash2`（折叠区内）
  - 折叠/展开 → `ChevronDown` / `ChevronUp`
  - 关闭输入面板 → `X`
  - 切换日期 → `Calendar`（轻量装饰）

## 媒体与占位图

- **默认策略**：本 MVP 不需要任何图片资源；纯文字 + 图标 + 卡片背景。
- **占位**：空态用 `lucide-react` 大号 `ListTodo`（48px，opacity 0.5）+ 一句文案，**不**插图。
- **Unsplash / 远程图源**：禁用。`IMAGE_PROVIDER=none`，不申请任何外部资源。
- **a11y**：空态图标 `aria-hidden="true"`，文案承载语义。

## 数据与演示内容（设计侧约定）

- **数据来源类型**：`EMPTY`（用户自建内容；本地 IndexedDB 没有任何演示种子）。
- **演示策略类型**：`EMPTY_CTA`。
- **FIXTURE 路径**：无（本 MVP 无示例数据需要）。
- **首屏可演示形态**：打开即空态 —— 居中显示「这里还是空的。点下方 + 记第一件事」+ 悬浮「+」按钮可点。
- **首启体验**：无引导、无介绍页、无弹窗——空态文案本身就是引导。

## 视觉 Token

### 颜色（CSS 变量，明暗模式成对）

| Token | 浅色 | 深色 | 用途 |
| --- | --- | --- | --- |
| `--color-bg` | `#FAFAF9`（stone-50） | `#0C0A09`（stone-950） | 页面底色 |
| `--color-surface` | `#FFFFFF` | `#1C1917`（stone-900） | 卡片背景 |
| `--color-surface-elevated` | `#FFFFFF` | `#292524`（stone-800） | 输入面板 / FAB 阴影层 |
| `--color-text` | `#1C1917`（stone-900） | `#FAFAF9`（stone-50） | 主文本 |
| `--color-text-muted` | `#78716C`（stone-500） | `#A8A29E`（stone-400） | 分组头、辅助文案 |
| `--color-text-faint` | `#A8A29E`（stone-400） | `#78716C`（stone-500） | 占位、禁用 |
| `--color-border` | `#E7E5E4`（stone-200） | `#292524`（stone-800） | 1px 分割线 |
| `--color-primary` | `#0F766E`（teal-700） | `#5EEAD4`（teal-300） | 主操作、FAB、未完成勾选环 |
| `--color-primary-soft` | `#CCFBF1`（teal-100） | `#134E4A`（teal-900） | 主操作 hover/pressed |
| `--color-danger` | `#B91C1C`（red-700） | `#FCA5A5`（red-300） | 删除提示 |

> 选择 teal（青绿）作主色：低饱和、克制的「完成感」色，不刺眼、不像红色那样带警告感、也不像纯灰那样冷淡。比纯黑/纯白更温柔、更适合「随手记」这种偏个人感受的工具。

### 字体

- **字体族**：`ui-sans-serif, system-ui, -apple-system, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif`。
- **字号阶梯**（rem 基准 16）：
  - `text-xs` 0.75rem（12px）—— 元数据、时间戳
  - `text-sm` 0.875rem（14px）—— 任务标题（移动端主文本）
  - `text-base` 1rem（16px）—— 输入框内容
  - `text-lg` 1.125rem（18px）—— 分组头「今天 / 明天」
  - `text-xl` 1.25rem（20px）—— 保留
- **字重**：标题 600（`font-semibold`）、正文 400、按钮 500。
- **行高**：正文 1.55，标题 1.4。
- **数字**：任务计数用等宽数字（`font-variant-numeric: tabular-nums`）。

### 圆角

- `--radius-card`：`1rem`（rounded-2xl）—— 任务卡片
- `--radius-button`：`0.75rem`（rounded-xl）—— 输入面板内按钮
- `--radius-pill`：`9999px`（rounded-full）—— FAB、勾选圆圈
- `--radius-input`：`0.75rem`（rounded-xl）—— 输入框

### 阴影

- `--shadow-fab`：`0 4px 12px rgba(0, 0, 0, 0.08)` —— 底部悬浮按钮
- `--shadow-panel`：`0 -4px 24px rgba(0, 0, 0, 0.06)` —— 输入面板
- `--shadow-card`：`0 1px 2px rgba(0, 0, 0, 0.04)` —— 任务卡片（极轻，几乎只用于层级区分）
- 深色模式阴影减弱为 `rgba(0, 0, 0, 0.4)` 系，避免在深色背景上「过亮」。

### 间距

- 使用 4pt 网格：`4 / 8 / 12 / 16 / 24 / 32 / 48`。
- 卡片间距：12px（`gap-3`）。
- 列表内边距：左右 16px（`px-4`）。
- 分组之间：24px（`space-y-6`）。
- FAB 距底部：safe-area + 24px。

### 动效

- 微交互：150ms（hover/press/checkbox 翻转）。
- 弹层进出：200ms（输入面板 `ease-out` 滑入，`ease-in` 退出）。
- 遵守 `prefers-reduced-motion`：开启时禁用所有 transform/opacity 动效。
- 仅动画 `transform` 和 `opacity`，不动画 `width/height/top/left`。

## 组件气质与关键界面状态

### 任务卡片（TaskCard）

- **默认态**：白色卡片（深色模式 stone-900），左 24px 圆形勾选 + 标题，1px 浅边框（无明显阴影），圆角 2xl。
- **hover/press（移动端等价于按下）**：背景轻微变为 `--color-surface-elevated`，勾选环外圈显示。
- **已完成**：标题加删除线、文字色降为 `--color-text-muted`、卡片半透明（opacity 0.6），仍可点击撤回。
- **长按/左滑**（仅在「已完成」区域）：出现删除图标 + 红色背景微闪。

### 日期分组头（DateGroupHeader）

- 标题：「今天」「明天」「更晚」（无年份、月份信息——按需简化）。
- 副标题：日期数字（轻量、不抢眼）。
- 紧下方：1px 分割线，颜色 `--color-border`。

### 快速输入面板（AddTaskPanel）

- 底部抽屉式（不是居中弹窗——更符合「随手记」的手势直觉）。
- 自动 focus 标题输入框，光标自动展开键盘。
- 「保存」按钮在键盘弹出时仍可见（fixed 位置在键盘上方）。
- 三个日期 chip：「今天 / 明天 / 选日期」——chip 选中态用 `--color-primary-soft` 背景 + `--color-primary` 文字。

### FAB（FloatingAddButton）

- 右下角悬浮（`fixed bottom-6 right-6`，考虑 safe-area-inset-bottom）。
- 圆形 56×56，`--color-primary` 背景，白色 `Plus` 图标。
- 轻阴影 `--shadow-fab`。
- 长按不需反馈（无需菜单）。

### 折叠区（CompletedSection）

- 底部「已完成 (N)」行，点击展开/收起。
- Chevron 图标旋转 180° 表达方向。
- 展开后：每条已完成任务 + 右侧删除按钮。

### 空态（EmptyState）

- 居中（垂直居中 + 上移 1/3），上方 48px `ListTodo` 图标（opacity 0.4），下方一行温和文案：
  - 「这里还是空的。」主文（`text-lg`，`text-muted`）
  - 「点下方 + 记第一件事」副文（`text-sm`，`text-faint`）

### 关键界面状态汇总

| 界面 | 默认 | 空 | 加载 | 成功 | 错误 | 未配置 |
| --- | --- | --- | --- | --- | --- | --- |
| 主列表 | 有任务时按日期分组 | 居中 ListTodo + 引导文案 | 首屏不显示（IndexedDB 同步读取极快） | 新增后即时出现在对应分组 | 极少（IndexedDB 失败时显示「数据加载失败，请刷新」重试按钮） | N/A（无 Key 概念） |
| 输入面板 | 半屏抽屉 + 自动 focus | 标题为空时「保存」按钮 disabled | 不显示（同步操作） | 保存后面板下滑消失、新任务即时出现在列表 | 极少 | N/A |

## 响应式或窗口尺寸

- **断点**：移动端优先（`default` = `< 640px`）。`sm: 640px` / `md: 768px` / `lg: 1024px`。
- **移动端（<640px）**：主视图占满宽度；卡片左右各 16px 内边距；FAB 距右下 24px + safe-area。
- **桌面端（≥640px）**：内容居中最大宽度 480px（仍以移动端体验为优先，避免双列浪费空间）；FAB 仍在右下，但内容居中。
- **viewport**：`width=device-width, initial-scale=1, maximum-scale=1`（禁止缩放——单手操作友好）。
- **横屏**：任务列表保持纵向单列滚动；输入面板在横屏转为全屏居中卡片（保留 8px 内边距）。
- **安全区**：`env(safe-area-inset-top)` / `env(safe-area-inset-bottom)` 用作顶/底 padding。

## 与 PRD 的一致性自检

- ✅ 「打开即记，记完就走」 → FAB 常驻、1 步进入输入态、无欢迎页
- ✅ 「不登录、不收费、无广告」 → 全站零弹窗、零广告位
- ✅ 「按日期分组与倒序列表」 → 三个分组头 + 同组内按创建时间倒序
- ✅ 「一键勾选完成」 → 整行可点击 + 大圆圈
- ✅ 「本地持久化、无登录」 → IndexedDB + 无任何鉴权逻辑
