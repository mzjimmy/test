# 开发计划

> 仓库根目录 `prd.md` 为产品事实来源；本文件为开发计划与验收清单。
> 任何代码改动必须同步回写本文件，见 `.claude/rules/todo-writeback.md`。
> 已稳定完成的模块按 `.claude/rules/todo-prd-archive.md` 卸货到 `prd/`，并在此处折叠索引。

## 1. 项目框架

- **工具形态**：网站，优先在移动端使用
- **主框架**：Next.js（App Router + TypeScript）
- **框架映射**：网站 → Next.js
- **选择理由**：PRD 的工具形态明确为「移动端 Web」，Next.js 自带 SSR/SSG、移动优先响应式、App Router 文件式路由，能让「主列表页」一文件落地；TypeScript 让 `Task` 数据结构与 IndexedDB 读写契约有静态保障。
- **Agent 启用矩阵**：
  - **必选**：mvp-requirement-analyst、project-architecture-planner、qa-acceptance-planner
  - **条件启用**：ui-interaction-planner（移动端 Web 含主列表页 + 输入面板 + FAB + 折叠区，可见 UI 入口丰富）
  - **未启用**：
    - api-integration-planner（PRD 不涉及远程 API；所有数据走本地 IndexedDB）
    - platform-capability-planner（纯展示型 Web + 无浏览器特殊权限；IndexedDB 属核心浏览器能力，不构成权限申请）

> **执行说明**：本计划阶段的 subagent 拆解在当前 Trae IDE 环境无独立 subagent 工具调用入口，已由主会话按各 agent 文档职责合并产出，规格同步到本 TODO 与 `design-system/随手记/MASTER.md`。
> 后续如需对某条 §2.x 重做深度拆解，可在具备 subagent 工具的环境里重新 spawn 补强；本文件结构无需调整。

## 已完成模块（详见 prd/）

> 首版留空。某 §2.x 功能全部 `[x]` 且经 project-verify 验证后，按 `.claude/rules/todo-prd-archive.md` 写入对应 `prd/<模块>/README.md` 和 `prd/<模块>/technical.md`，并在此处追加索引；§2 中该块折叠为「已归档 → prd/<模块>/README.md」。

（暂无）

## 2. MVP 功能拆解

### 2.1 快速新建任务

- **用户目标**：在 3 秒内创建一条带「标题 + 日期」的任务，保存后立刻出现在对应日期分组。
- **使用入口**：主列表页右下角悬浮「+」按钮（FAB，固定在视口右下角）→ 唤起底部抽屉式输入面板。
- **子功能**：
  - [ ] FAB 唤起输入面板：点击后从底部 200ms 滑入，键盘自动 focus 标题输入框
  - [ ] 标题输入：单一文本框、必填、占位文案「要做什么？」、trim 后非空才允许保存
  - [ ] 日期选择：三个 chip ——「今天（默认选中）」「明天」「选日期」；选「选日期」时弹出原生 `<input type="date">`
  - [ ] 标题为空时「保存」按钮 disabled（视觉降级 + 不可点）
  - [ ] 点击「保存」后：写入 IndexedDB → 关闭面板 → 列表即时新增（无需刷新）
  - [ ] 面板外点击 / 点击「×」/ 按 Esc：关闭面板，已填内容丢弃（不提示二次确认——快速为先）
  - [ ] 标题最大长度 100 字符（超长截断 + 提示「已达上限」）
- **主流程**：
  1. 用户点击 FAB
  2. 输入面板 200ms 滑入，标题输入框 focus
  3. 用户输入文字（必要时切日期）
  4. 点击「保存」按钮
  5. 写入本地 IndexedDB，面板滑出消失
  6. 列表对应日期分组顶部即时出现该任务
- **状态**：
  - 默认态：FAB 始终可见（不滚动隐藏）
  - 空态：标题为空 → 保存按钮 disabled
  - 加载态：无（同步操作；保存过程 <50ms）
  - 成功态：面板关闭 + 列表新增
  - 错误态：IndexedDB 写入失败时面板不关闭，标题上方红色提示「保存失败，请重试」（罕见）
- **数据来源**：`EMPTY`（用户自建，无示例数据）
- **演示策略**：`EMPTY_CTA`（首次打开无任务时显示空态引导「点下方 + 记第一件事」）
- **数据需求**：本地 IndexedDB 单条写入 `tasks` 表。
- **MVP 必做**：标题必填 + 日期默认今天 + 即写即现。
- **暂不实现**：标签、优先级、提醒时刻、重复任务、子任务、附件、图片。
- **待确认点及推荐默认值**：
  - 标题最大长度 → 100 字符（中等手机单行能容纳的长度上限）
  - 面板关闭是否二次确认 → 否（用户打字 ≤3 秒就完事，二次确认反而拖慢）
  - 是否支持换行标题 → 否（textarea 一律不引入；单行输入框即可）
- **验收标准**：
  - [ ] 点击 FAB，输入面板在 300ms 内完全出现
  - [ ] 标题输入框自动 focus，键盘自动展开（移动端）
  - [ ] 标题为空时「保存」按钮不可点击
  - [ ] 标题填写 + 点保存后，面板在 200ms 内关闭
  - [ ] 列表对应日期分组顶部即时出现新任务（无刷新）
  - [ ] 点击面板外区域 / 「×」/ Esc 均可关闭面板
  - [ ] 关闭后再次打开面板，标题输入框为空（不保留草稿）
  - [ ] 标题超过 100 字符时无法继续输入，提示文案出现

### 2.2 按日期分组与倒序列表

- **用户目标**：一眼看清「今天 / 明天 / 更晚」各自还要做什么；已完成不打扰视线。
- **使用入口**：主列表页（产品唯一页面，访问根路径 `/` 即进入）。
- **子功能**：
  - [ ] 启动时从 IndexedDB 读取全部任务，内存中按 `dueDate` 分组
  - [ ] 三个分组：**今天** / **明天** / **更晚**（更晚 = 距今 2 天及以后的所有未完成任务）
  - [ ] 同组内排序：未完成按 `dueDate` 升序、其次 `createdAt` 倒序
  - [ ] 已完成统一沉到列表底部「已完成 (N)」折叠区
  - [ ] 各分组头显示任务计数（如「今天 · 3」）
  - [ ] 没有任务的分组头不显示（避免空标题占空间）
  - [ ] 列表支持触摸滚动；`overscroll-behavior: contain` 防止整页弹性
- **主流程**：
  1. 用户打开 `/`
  2. 应用读取 IndexedDB，按规则分组排序
  3. 渲染三个分组头 + 任务卡片列表
  4. 用户上下滚动浏览
- **状态**：
  - 默认态：见主流程
  - 空态：全部任务都完成时显示「今天的事都做完啦 🎉」（emoji 禁用 → 改为 ✓ 图标 + 文案「今天的事都做完啦」）
  - 加载态：首屏同步读取 <50ms，无需显示骨架屏
  - 成功态：列表稳定渲染
  - 错误态：IndexedDB 读取失败时显示「数据加载失败，请刷新」+ 重新加载按钮
  - 未配置态：N/A
- **数据来源**：`EMPTY`（首次为空 → `EMPTY_CTA`；用户开始记录后 → 本地真实数据）
- **演示策略**：`EMPTY_CTA`（空态）
- **数据需求**：IndexedDB 全量读取 + 内存中分组（任务量预计 ≤200 条，无需虚拟滚动）
- **MVP 必做**：三个分组 + 倒序 + 已完成折叠。
- **暂不实现**：搜索、筛选、按优先级排序、按标签筛选、按完成时间排序、月历视图。
- **待确认点及推荐默认值**：
  - 跨日期的「昨天」任务 → 仍按 `dueDate` 归入对应日期分组（即使已过期），不自动转「今天」；用户可手动编辑修改
  - 「更晚」分组是否再拆分为「本周 / 本月」→ 暂不拆，保持 PRD 极简精神
- **验收标准**：
  - [ ] 首次打开应用，无任务时显示空态引导
  - [ ] 新建一条「今天」任务后，列表「今天」分组顶部立即出现
  - [ ] 新建一条「明天」任务后，列表「明天」分组顶部立即出现
  - [ ] 新建一条 5 天后任务，列表「更晚」分组顶部立即出现
  - [ ] 同组内多条任务时，按创建时间倒序（最新在上）排列
  - [ ] 已完成任务从原分组消失，进入底部「已完成 (N)」
  - [ ] 「已完成 (N)」点击可展开 / 收起
  - [ ] 全部任务都未完成时，底部无「已完成」折叠区
  - [ ] 关闭浏览器再打开，所有任务按规则正确恢复

### 2.3 一键勾选完成

- **用户目标**：点一下勾选；再点一下撤回。
- **使用入口**：主列表页中任意任务卡片的左侧圆形勾选按钮。
- **子功能**：
  - [ ] 点击圆形勾选 → 状态翻转为 `isCompleted: true` + `completedAt: Date.now()`
  - [ ] 已完成项从原日期分组移除，进入「已完成 (N)」折叠区
  - [ ] 视觉变化：标题加删除线 + 卡片 opacity 0.6 + 勾选显示为实心 `CheckCircle2`
  - [ ] 点击「已完成」区域内的任务 → 状态翻转为 `isCompleted: false` + `completedAt: null` → 回到原日期分组
  - [ ] 整个任务卡片都是点击区（不只是小圆圈）—— 移动端友好
  - [ ] 状态翻转 ≤50ms 完成，无 loading 态
- **主流程**：
  1. 用户在主列表点击未完成任务卡片
  2. 卡片 150ms 内变为已完成样式，从当前分组移除
  3. 底部「已完成 (N)」计数 +1；如果折叠区是展开的，新条目立即出现在折叠区底部
  4. 写入 IndexedDB（异步，不阻塞 UI）
- **状态**：
  - 默认态：圆形未勾选环 + 标题无删除线
  - 已完成态：圆形实心 + 标题删除线 + opacity 0.6
  - 错误态：IndexedDB 写入失败时回滚 UI 到原状态（罕见）
  - 其他态：N/A
- **数据来源**：N/A（操作本地数据）
- **演示策略**：N/A
- **数据需求**：IndexedDB 单条 `tasks` 行更新（`isCompleted`、`completedAt`）。
- **MVP 必做**：整行点击可完成 + 再次点击可撤回。
- **暂不实现**：长按显示菜单、滑动完成（移动端手势冲突风险大）、批量勾选、自动过期清理。
- **待确认点及推荐默认值**：
  - 已完成项是否自动从数据库删除 → 否，保留在「已完成」折叠区供用户主动清理
  - 撤回后是否回到原位置 → 是，按 `dueDate` 回到对应分组顶部
- **验收标准**：
  - [ ] 点击未完成任务任意位置，状态变为已完成
  - [ ] 已完成任务从原日期分组消失
  - [ ] 已完成计数 +1
  - [ ] 展开「已完成」折叠区时，新完成项出现在折叠区底部
  - [ ] 点击已完成项，状态恢复为未完成
  - [ ] 撤回后该任务回到原日期分组顶部
  - [ ] 状态翻转的视觉过渡 ≤200ms
  - [ ] 关闭浏览器再打开，状态正确保持

### 2.4 本地持久化与无登录

- **用户目标**：关掉浏览器、隔天再打开、换台手机浏览器，任务都在；不被要求注册。
- **使用入口**：贯穿所有功能（新建、勾选、撤回、删除均依赖此能力）。
- **子功能**：
  - [ ] IndexedDB 数据库 `snj-db`、版本 1、object store `tasks`、keyPath `id`
  - [ ] 启动时全量读取 → 内存列表
  - [ ] 任意写操作后即时写入 IndexedDB（无需手动保存）
  - [ ] 全部操作 100% 在前端完成；零后端调用
  - [ ] 零注册、零登录、零账号体系
  - [ ] 零广告位、零付费墙、零弹窗
- **主流程**：
  1. 用户首次访问 → IndexedDB 初始化 → 空列表
  2. 用户添加任务 → 写入 IndexedDB
  3. 用户关闭浏览器 → 数据保留在 IndexedDB
  4. 用户再次访问 → 自动读取并恢复
- **状态**：
  - 默认态：所有数据本地存储
  - 加载态：首屏 <50ms 同步读取
  - 错误态：IndexedDB 不可用（极少见，隐私模式 + 浏览器禁用）→ 显示「您的浏览器不支持本地存储，请更换浏览器访问」
  - 未配置态：N/A（无 Key 概念）
- **数据来源**：`EMPTY`（用户自建）
- **演示策略**：`EMPTY_CTA`
- **数据需求**：IndexedDB 单一 object store `tasks`。
- **MVP 必做**：本地读写 + 零登录。
- **暂不实现**：云端同步、跨设备同步、导出 / 导入、备份提示。
- **待确认点及推荐默认值**：
  - 是否提供「清空所有数据」按钮 → 否（MVP 极简，用户可手动逐条删除）
  - 浏览器隐私模式行为 → 正常支持（IndexedDB 仍可用），关闭隐私窗口时数据随之清空——这是浏览器行为，不需特别提示
- **验收标准**：
  - [ ] 首次打开应用，IndexedDB 中无 `snj-db` 数据库
  - [ ] 添加任务后，IndexedDB `tasks` 表新增一行
  - [ ] 关闭浏览器再打开，所有任务保持
  - [ ] 跨天打开（明天 / 后天）打开，已过期的昨日任务仍在「昨天」对应日期分组（不会自动归到「今天」）
  - [ ] 全局零网络请求（DevTools Network 面板验证）
  - [ ] 全局无任何 `cookie`、`localStorage` 业务字段（IndexedDB 即可）
  - [ ] 全局无广告位、无付费墙、无注册入口

## 3. 界面与交互规格

> 完整视觉规范见 `design-system/随手记/MASTER.md`。本节只列界面入口与可操作项。

- [ ] 主列表页（唯一页面，根路径 `/`）：展示日期分组 + 任务卡片 + 折叠区 + FAB
  - `data-testid="task-list-root"`：最外层容器
  - `data-testid="date-group-today"` / `"date-group-tomorrow"` / `"date-group-later"`：三个分组容器
  - `data-testid="date-group-header"`：分组头（标题 + 计数）
  - `data-testid="task-card"`：单条任务卡片（包含勾选 + 标题 + 删除按钮）
  - `data-testid="task-checkbox"`：圆形勾选按钮
  - `data-testid="task-title"`：任务标题文本
  - `data-testid="completed-section"`：底部折叠区容器
  - `data-testid="completed-toggle"`：折叠区标题行
  - `data-testid="completed-count"`：折叠区计数
  - `data-testid="empty-state"`：空态容器
- [ ] 快速输入面板（底部抽屉，fixed 定位）：标题输入 + 日期 chip + 保存按钮
  - `data-testid="add-panel"`：面板容器
  - `data-testid="add-panel-backdrop"`：背景遮罩
  - `data-testid="add-title-input"`：标题输入框
  - `data-testid="add-date-today"` / `"add-date-tomorrow"` / `"add-date-later"`：三个日期 chip
  - `data-testid="add-save"`：保存按钮
  - `data-testid="add-cancel"`：关闭按钮（X）
- [ ] 删除确认弹层（在「已完成」折叠区触发）：单条「确定删除？」+ 确认 / 取消
  - `data-testid="delete-confirm"`：弹层容器
  - `data-testid="delete-confirm-ok"`：确认按钮
  - `data-testid="delete-confirm-cancel"`：取消按钮
- [ ] 核心组件：`FAB`、`TaskCard`、`DateGroupHeader`、`CompletedSection`、`AddTaskPanel`、`DeleteConfirm`、`EmptyState`
- [ ] 响应式：移动端优先（<640px 占满宽度）；≥640px 居中最大宽度 480px；移动端安全区避让 `env(safe-area-inset-*)`；横屏在输入面板转为全屏居中卡片

## 4. 架构与模块边界

- [ ] 按主框架初始化或检查项目结构
  - [ ] Next.js 15+ App Router 项目脚手架：`npx create-next-app@latest . --typescript --tailwind --app --eslint --src-dir=false --import-alias="@/*"`
  - [ ] 关闭 `npm create` 自带的演示内容，仅保留 `app/page.tsx`、`app/layout.tsx`、`app/globals.css`
  - [ ] 在 `app/layout.tsx` 设置 `viewport: { width: 'device-width', initialScale: 1, maximumScale: 1 }` 与 `themeColor`（明 / 暗模式）
- [ ] 建立基础布局、全局样式和核心入口
  - [ ] `app/globals.css`：导入 Tailwind + CSS 变量（明暗模式）；设置 `html, body { background: var(--color-bg); color: var(--color-text); }`；设置 `body { overscroll-behavior: contain; }`
  - [ ] 安装 `lucide-react`（图标库）
  - [ ] 安装 `idb`（IndexedDB 类型安全封装，比原生 API 友好）
  - [ ] 目录约定：
    - `app/page.tsx` —— 主列表页（客户端组件，因为需要读写 IndexedDB 与本地状态）
    - `app/layout.tsx` —— 根布局（HTML + viewport + 主题注入）
    - `app/globals.css` —— 全局样式
    - `components/` —— 全部 UI 组件
    - `lib/db.ts` —— IndexedDB 封装（开 / 读 / 增 / 改 / 删）
    - `lib/types.ts` —— TypeScript 类型
    - `lib/date.ts` —— 日期工具（今天 / 明天 / 更晚的归类）
    - `lib/grouping.ts` —— 任务分组与排序纯函数
    - `lib/uuid.ts` —— `crypto.randomUUID()` 的薄封装
- [ ] 按模块实现：快速新建面板（§2.1）/ 日期分组列表（§2.2）/ 勾选完成（§2.3）/ 本地持久化（§2.4）
- [ ] 数据对象：
  ```ts
  // lib/types.ts
  export type Task = {
    id: string;              // crypto.randomUUID()
    title: string;           // 1-100 字符
    dueDate: string;         // 'YYYY-MM-DD'，本地日期非 ISO
    isCompleted: boolean;
    createdAt: number;       // Date.now()
    completedAt: number | null;
  };
  ```
- [ ] 状态流：用户操作（点击 FAB / 勾选 / 删除）→ 组件本地 state → 调 `lib/db.ts` 写入 IndexedDB → 触发 `useEffect` 重新拉取 → 列表重渲染
- [ ] 数据与演示汇总：
  - 全部 §2 功能数据来源 = `EMPTY`，演示策略 = `EMPTY_CTA`，无 FIXTURE
  - 禁止在 JSX 内散落 mock 数据；本 MVP 不需要 fixture
- [ ] 按工具形态配置密钥保存方式：**不适用**（无 API Key、无 OAuth）
- [ ] **不**为「架构完整」新增后端接口 / Route Handler / Server Action
- [ ] 性能预算：首屏 TTI < 1s（IndexedDB 同步读取 + 极小 JS 包）；列表 200 条内无虚拟滚动

## 5. 设计系统

- [x] 读取 prd.md 与 ui-ux-pro-max，并在缺少 UI 设计风格时基于 PRD（含用户画像、场景故事）和界面规格推导设计取向
- [x] 创建或更新 `design-system/<产品名>/MASTER.md`，且含必备章节：设计取向依据（含用户画像适配表）、UI 技术栈与组件策略、图标 Iconography、媒体与占位图、数据与演示内容约定、视觉 Token、关键界面状态
- [x] MASTER 已写明图标库（React 栈默认 lucide-react）、尺寸 token、禁止 emoji 功能图标与多库混用
- [x] MASTER 已写明占位图策略（默认纯文字 + 图标，无 Unsplash / 远程图源）
- [ ] 将设计 token 与 UI 栈落地到所选框架（Tailwind 配置 + CSS 变量 + 组件库）
- [ ] 覆盖关键界面的默认态、空态、加载态、成功态、错误态和未配置态

## 6. API 与集成

- [ ] **当前 MVP 不需要远程 API**；所有数据走本地 IndexedDB
- [ ] 不需要 API Key / Token / OAuth / 第三方服务
- [ ] 不需要 `【API 接入方案（待确认）】` 章节（保持 prd.md 现状）
- [ ] 未来若需云端同步 → 单独 MVP 重新走 02-project-prepare

## 7. 平台能力与权限

- [ ] **当前 MVP 无需特殊浏览器权限**（IndexedDB 属于核心浏览器能力，非 permission API）
- [ ] 无后台脚本、无 service worker、无 notification、无 clipboard、无 file system access
- [ ] viewport 元信息：`width=device-width, initial-scale=1, maximum-scale=1`（单手移动端体验）
- [ ] 主题模式：跟随系统 `prefers-color-scheme`，同时提供手动切换入口（位于主列表页右上角小图标；如本期不做可在 §10 列入后续）
- [ ] PWA：MVP 不做 manifest / service worker；如需离线安装，放到 §10
- [ ] 平台专项失败降级：IndexedDB 不可用时显示「您的浏览器不支持本地存储」整页提示

## 8. 验证

- [ ] 开发完成后运行 `project-verify` Skill，按 §2 验收标准逐条执行并回写打勾
- [ ] 运行 `npm run lint` / `npm run test` / `npm run typecheck`（由 project-verify 执行并记入验证记录）；`build` 不作为默认验证项
- [ ] Web UI 的 §2 验收步骤与 `TESTING_CHECKLIST.md` 使用稳定 `data-testid` 定位（命名见 `.claude/skills/project-verify/references/selector-and-testid.md`）；由 `03-project-develop` / `project-iterate` 在实现时预埋，本阶段不写业务代码
- [ ] 非 UI 逻辑（IndexedDB 封装、日期归类、分组排序纯函数）使用 Vitest 单元测试验收
- [ ] 生成或更新 `TESTING_CHECKLIST.md` 与 `test-results/` 证据
- [ ] 验证空态、加载态、错误态和未配置态
- [ ] 按平台验证：Next.js 页面访问（移动端 / 桌面端双断点）
- [ ] 列出失败 / 跳过项与下一步建议

## 9. 开发完成后更新 PRD

- [ ] 调用 `update-prd` Skill
- [ ] 基于实际代码创建或更新 `prd/` 文档体系
- [ ] 对照原始 `prd.md` 总目标与 `prd/` 已实现文档，把未完成差距写入本 TODO
- [ ] 将根目录 `prd.md` 归档为 `prd.original.md`（保留原始总目标；Agent 后续不读取，仅供人类回顾）
- [ ] 生成或更新同内容的 `CLAUDE.md` 和 `AGENTS.md`，提示后续基于 `TODO.md` 继续任务

## 10. 未完成目标 / 后续功能

> 来自原始 prd.md 但尚未在当前 MVP 中完成的功能。开发完成后由 `update-prd` 核对并补全。

- [ ] 跨设备云端同步（需用户注册 + 后端服务，违反「无登录」MVP 边界）
- [ ] 数据导出（JSON / TXT）
- [ ] 提醒通知（基于 Web Push API，需用户授权；不违反「无登录」但增加复杂度）
- [ ] 重复任务 / 习惯打卡
- [ ] 子任务拆解
- [ ] 主题色自定义（当前仅 teal 一套）
- [ ] PWA 离线安装（manifest + service worker）
- [ ] 撤销操作（最近删除的恢复）
- [ ] 桌面端键盘快捷键（`N` 新建、`/` 聚焦搜索等）
- [ ] 桌面浏览器扩展版本

## 开发进度

> 每轮 `project-iterate` 或开发 session 结束前追加一条；规范见 `.claude/rules/todo-writeback.md`。

### 2026-06-06
- 阶段：02-project-prepare 计划生成完成
- 本轮：把 `prd.md` 拆成可开发规格，落 `design-system/随手记/MASTER.md` 与本 `TODO.md`
- 下轮建议：进入 03-project-develop 初始化 Next.js 脚手架，开始按 §2.1 → §2.4 顺序实现

## TODO 卸货记录

> 每次将已完成块从 §2 归档到 `prd/` 时追加；规范见 `.claude/rules/todo-prd-archive.md`。

（暂无）
