---
name: jira-report-page
description: Use when building an internal Jira ticket evaluation report page (Jira ticket 评价报告页) that shows completeness/compliance review results — covers stack choice, Tailwind v4 + shadcn tokens, the "Audit Dossier" visual direction, and the exact design brief to hand to a coding agent.
---

# Jira Ticket 评价报告页：Agent 工具集与设计方向

## 何时使用

- 要做一个内网报告页，展示 Jira ticket 的「完整性 / 规范性」评价结果
- 需要决定这个页面用什么技术栈、装哪些 skill、怎么把设计风格讲给 agent
- 已经有一版页面，但看起来像后台模板 / SaaS 落地页，需要拉回「审计报告」的气质

## 怎么用

1. 把「一、结论」的表作为技术选型基线，按实际项目调整（团队已有 antd/Mantine 时可替换组件层）
2. 按「三、Skills」装 5 个核心 skill，`frontend-design` 这类营销页取向的只在明确需要时加
3. 把「四、设计方向」的 markdown 块原样放进项目 `docs/design-brief.md`，或在 prompt 里直接粘贴
4. 把「五、AGENTS.md 片段」落到仓库根目录，作为 agent 的长期约束
5. 交付前跑「六、验收清单」，并让 agent 用 Playwright 截图自验

> 场景：内网部署，展示 Jira ticket 的「完整性 / 规范性」评价结果。
> 目标：让 coding agent 一次产出**可信、克制、可读**的业务报告页，而不是又一个 SaaS 模板。

---

## 一、结论：一套就够了

| 层 | 选择 |
|---|---|
| 构建 | Vite 8 + React 19 + TypeScript |
| 样式 | Tailwind CSS 4.3（`@tailwindcss/vite`，CSS-first `@theme`） |
| 组件 | shadcn/ui（`-b base`，组件 vendored 进仓库）+ Base UI |
| 图表 | 分数条用 CSS；趋势/构成再上 Recharts 3，复杂图表用 ECharts 6 |
| 图标 | lucide-react（按需引入，随包打包） |
| 字体 | Univers Next（有授权）/ Inter 正文 + 思源黑体回退 + 更纱等宽证据块 |
| 表格 | 原生 `<table>`；只有需要排序/筛选/虚拟滚动才上 TanStack Table 9 |
| 测试 | Playwright（agent 自验 + 截图基线）+ Vitest（纯函数） |

**为什么不用 Mantine / antd**：它们更快，但默认长相就是「后台模板」，而这份页面的核心是**让人相信评价结论**——需要文档感、证据感，不是组件堆叠。资源不缺时，没有理由为了省事牺牲这一点。

---

## 二、资源不是瓶颈，把预算花在这三处

镜像源和字库齐了，之前那些「离线兜底」的妥协都可以扔掉。真正决定成败的是下面三件事：

### 1. 字体：英文正文 + 中文原文引用，最容易露怯也最容易出质感

| 用途 | 字体 | 注意 |
|---|---|---|
| 正文 | Univers Next for HSBC（有授权）/ Inter VF | 16px / 行高 1.5；引用中文原文时回退思源黑体 VF，行高放宽到 1.7 |
| 数字、英文、单位 | Inter VF | 全局 `font-variant-numeric: tabular-nums`，分数和百分比必须等宽 |
| 大标题（报告名/总分） | Inter 700 / Univers Next Bold | HSBC 全站不用衬线；层级靠字号与字重拉开，不靠字体切换 |
| 原文证据块 | 更纱等宽（Sarasa Mono SC） | 中英混排等宽的最佳解，比 JetBrains Mono 多一套中文字形 |

中文全量字体动辄 5–10MB，用 `cn-font-split` / `fontmin` 按 `unicode-range` 切片，或至少 `woff2` + 首屏切片 `preload`；数字字体单独 preload。内网带宽不是问题，但首屏抖动会直接毁掉「报告」的正式感。

### 2. 组件地基：一次性配好，别让 agent 每次猜

npm 镜像解决的是**包**，不等于 shadcn 的 **registry**（CLI 拉的是 ui.shadcn.com 的 JSON）。两条路选一条，一次配好：

- 内网静态托管 registry，`components.json` 的 `registries` 指向它 —— 之后 agent 可以正常 `shadcn add`；
- 有网机器上 `init` + `add` 全量组件，把 `components/ui/*` 提交进仓库，agent 只改本地文件。

同时把 `ui.shadcn.com/llms.txt`、`base-ui.com/llms.txt`、Tailwind 的 theme/utility 文档落到 `docs/vendor/`，AGENTS.md 里写死「查组件 API 先读 vendor 文档」。

shadcn 组件默认带 8px 圆角和投影。初始化后第一件事是把 `--radius-*` 全部归零、删掉 `shadow-*` 类——否则页面立刻不像 HSBC，这是套用这套视觉语言最容易漏的一步。

### 3. 验证闭环：报告页的 bug 都在「展开之后」

给 Playwright 配视觉基线（`toHaveScreenshot`），固定三个视口 + 两个状态（默认 / 展开明细）。Agent 改完样式自己跑一遍对比，比任何 code review 都有效。这一步在资源充足时没有理由省。

---

## 三、Skills：装这 5 个，另 3 个按需

### 核心（建议全装）

| Skill | 为什么是它 |
|---|---|
| [`interface-design`](https://skills.sh/dammyjay93/interface-design/interface-design) | 专门面向 **dashboard / admin / 数据界面** 的设计 craft 指南。比 `frontend-design` 更贴这个场景（后者是营销页取向，hero、排版个性那套用在这里会跑偏） |
| [`shadcn`](https://skills.sh/shadcn-ui/ui/shadcn) | 读 `components.json` 拿项目上下文（base 是 radix 还是 base、图标库、已装组件），按正确 API 生成，避免手搓 Dialog/Popover 的焦点与滚动锁 |
| [`webapp-testing`](https://skills.sh/anthropics/skills/webapp-testing) | Playwright 驱动，让 agent 自己开页面、截图、点开折叠项验证。报告页最容易错的是「展开后布局崩」，只有真跑一遍才发现 |
| [`web-design-guidelines`](https://skills.sh/vercel-labs/agent-skills/web-design-guidelines) | review 阶段用：按 Web Interface Guidelines 检查 a11y / 语义 / 表单细节 |
| [`accessibility`](https://skills.sh/addyosmani/web-quality-skills/accessibility) | 内网工具常被键盘用户重度使用（审核岗），焦点顺序、对比度、aria 必须过一遍 |

### 按需

| Skill | 什么时候加 |
|---|---|
| [`impeccable`](https://skills.sh/pbakaus/impeccable/impeccable) | 页面已经能跑，需要一轮 critique / polish 时 |
| [`tanstack-table`](https://skills.sh/tanstack-skills/tanstack-skills/tanstack-table) | 工单列表要做排序、多列筛选、虚拟滚动 |
| [`frontend-design`](https://skills.sh/anthropics/skills/frontend-design) | 只在想要「排版更有主张」时加，并且必须明确告诉它：**这是密集型内部报告，不是 landing page，忽略 hero / 入场动效相关建议** |

安装（有网机器执行，随后提交 `.agents/skills/`）：

```bash
npx skills add dammyjay93/interface-design --skill interface-design -a universal
npx skills add shadcn-ui/ui -a universal
npx skills add anthropics/skills --skill webapp-testing -a universal
npx skills add vercel-labs/agent-skills --skill web-design-guidelines -a universal
npx skills add addyosmani/web-quality-skills --skill accessibility -a universal
```

> 不要装 `frontend-design` + `impeccable` + `ui-ux-pro-max` + 各种 taste 系一起上：四个「审美类」skill 会互相打架，最后产出反而更套路。

---

## 四、设计方向：给 Agent 的 Brief（可直接粘）

视觉基调对齐 HSBC 数字端：**白底黑字、发丝灰线、方角、无阴影，红色只给动作和阻断项。**
报告结构（三栏、证据引用、打印样式）保持不变，只换视觉语言。**不复刻汇丰六边形 Logo。**

把下面这段原样放进 prompt 或 `docs/design-brief.md`：

```markdown
# 设计方向：HSBC 风格审计报告（HSBC-aligned Audit Report）

这是一个面向业务方的 Jira ticket 评价报告页，内网使用，**界面标签与报告正文均为英文**（ticket 原文可能是中文）。
气质参考 HSBC 数字端：精确、克制、纸面感，靠排版和发丝线建立秩序，不靠卡片、圆角、阴影。

## 一句话
白底、黑字、细灰线、方角；红色只留给动作和阻断。

## 页面结构（自上而下）
1. 顶部工具条 56px：面包屑、ticket key、筛选、导出、打印
2. 报告头：ticket 标题、结论徽标（Pass / Pass with conditions / Fail）、总分（大号 tabular-nums）、评审时间
3. 评分总览：各维度横向条形 + 权重 + 得分，一眼看出短板在哪
4. 关键问题：按「阻断 / 重要 / 建议」三级分组，每条必须带**原文证据**
5. 维度明细：表格，可展开看判定依据与命中规则
6. 原始 ticket 字段：默认折叠（标题、描述、验收标准、附件链接）
7. 页脚：评分规则版本、模型/规则来源、生成时间、免责说明

## 布局
- 三栏：左侧 240px 工单列表（可收起）｜中间正文最大 880px 阅读宽度｜右侧 260px 元信息（sticky）
- 内容区最大 1280px，12 列栅格，对齐到 8px
- 正文左对齐，不要居中排版；英文行长 60–75 字符，引用中文原文时 36–45 字
- 区块间距 56px，小节间距 32px；分节用 1px 横线 + 留白，不要用卡片包裹

## 颜色（HSBC token）
- 画布纯白 #FFFFFF；分区与表头 #F3F3F3；分隔线 #D7D8D6；浅线 #EDEDED
- 主文字 #000000；次级 #404040 / #5C5C5C；弱化 #767676
- 品牌红 #DB0011：主按钮、链接、焦点环、阻断级问题；hover #AF000D，active #83000A
- 通过 #008580（HSBC 自己的 agree 青绿）；有条件通过 #8A6D3B（赭黄，只用于文字和小图标）
- 分数高低靠条形长度 + 数字，不靠颜色；不要红黄绿三色齐上
- 禁止：渐变、彩色图标底、多色图表、旧链接蓝 #376798

## 字体
- 正文：Univers Next for HSBC（有授权时）/ Inter（替代），16px / 行高 1.5
- 中文回退：思源黑体 VF；引用中文 ticket 原文时行高放宽到 1.7
- 数字：tabular-nums，分数、百分比、日期右对齐
- 字号阶：12 / 14 / 16 / 20 / 24 / 32 / 40 / 48；字重只用 400 / 500 / 700
- 不用衬线体，不用全大写标签，不用宽字距 eyebrow

## 形状与组件
- 圆角 0（标签最多 2px）；1px 实线边框；**全站不用 box-shadow**，浮层用 1px 边框 + 遮罩
- 主按钮：红底白字、1px 红边、方角、高 40px；次按钮：黑底白字，或白底黑字 + 1px 黑边
- 链接：红色，hover 加下划线并转黑（HSBC 自己的做法）
- 焦点：2px solid #DB0011，offset 2px，任何情况下不移除
- 表格：表头 #F3F3F3，行间 1px 线，无斑马纹，数字右对齐
- 图标：线性 24px、1.5px 描边、单色；用 lucide 近似，不要圆头粗描边

## 动效
- 只做状态反馈：150ms ease-out（展开/折叠、筛选更新）
- 不做入场编排、不做卡片 hover 位移、不做数字滚动
- 尊重 prefers-reduced-motion

## 证据与文案（英文输出）
- 每个扣分项必须引用 ticket 原文片段：左侧 2px 竖线 + 等宽小字 + 字段名标签
- 缺失字段写 "Not provided"，不要编造，也不要用 "N/A" 含糊过去
- 结论用可执行动词："Add acceptance criteria and re-review."
- 语气平实、主动语态（HSBC tone of voice）；标签用 Title Case，正文句子大小写
- 禁止出现：感叹号、emoji、AI-powered、insights、one-click、leverage、✨

## 打印
- @media print：隐藏左侧列表与工具条，正文单栏，A4 分页，区块不跨页断裂，保留页脚规则说明

## 硬性禁止清单
1. 没有 hero 区、没有大标语、没有渐变背景
2. 没有「彩色圆底图标 + 大数字 + 小标签」的 KPI 卡阵列
3. 不用圆角卡片、不用 shadow、不用毛玻璃
4. 不用衬线体、不用紫/靛蓝主色、图表色不超过 2 种
5. 不复刻 HSBC 六边形 Logo，不模仿其版式商标
6. 不写假数据占位，不写 lorem ipsum
7. 不在每个 section 上加 fade-in-up
```

---

## 五、AGENTS.md 片段

```markdown
## 项目
内网 Jira ticket 评价报告页。Vite 8 + React 19 + TS + Tailwind 4.3 + shadcn(base)。

## 命令
- pnpm dev / build / lint / test
- 提交前必须：pnpm build && pnpm lint && pnpm test

## 约束
- 依赖走内部 npm 镜像；不引 CDN、远程字体、运行时外部 API
- 字体自托管（public/fonts，woff2 + 切片），数字字体单独 preload
- 样式只用 Tailwind 类 + `@theme` token；禁止新增 CSS-in-JS 依赖
- 组件查 API 先读 docs/vendor/（shadcn、base-ui、tailwind），不要凭记忆
- 设计方向见 docs/design-brief.md，冲突时以 brief 为准
- 只允许这些库：react, tailwind, shadcn(base), lucide-react, recharts（按需）, date-fns
- 新增任何依赖前先说明理由
```

---

## 六、验收清单（交给 agent 自检）

- [ ] 1280 / 1440 / 1920 三档宽度下无横向滚动，右侧栏不遮挡正文
- [ ] 键盘 Tab 能走完「工单列表 → 维度表 → 展开明细」，焦点可见
- [ ] 对比度：正文 ≥ 4.5:1，次要文字 ≥ 4.5:1，语义色配文字不单靠颜色
- [ ] 超长 ticket 标题（60+ 字）两行截断不撑破布局
- [ ] 全部维度 0 分 / 全部满分两种极端数据下版面不塌
- [ ] 打印预览：单栏、无侧栏、页脚规则说明在
- [ ] 空状态：无 ticket、无问题项、规则未命中，各有明确文案
- [ ] 首屏无布局抖动（字体加载用 `font-display: swap` + 尺寸兜底）
