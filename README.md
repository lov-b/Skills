# Personal Skills & Rules

本目录用于存放**个人使用的 Agent Skills 和 Rules**，作为统一管理与备份位置。

## 目录结构

```
Skills/
├── README.md
├── skills/                  # Agent Skills（任务型技能）
│   └── <skill-name>/
│       └── SKILL.md
└── rules/                   # Rules（全局规则/约束）
    └── <rule-name>/
        └── RULE.md
```

- **Skills**：定义具体任务的执行流程（如 bug 修复、需求规划、开发总结等）
- **Rules**：定义跨场景的全局约束（如交互规范、编码原则等）

需要在本机生效时，将目标 skill 复制或链接到对应工具的全局 skills 目录；rules 配置为 User Rule。

---

## 已有 Skills

| Skill | 目录 | 用途 | 触发方式 |
|-------|------|------|----------|
| **auto-conclusion** | [`skills/auto-conclusion/`](skills/auto-conclusion/) | 对 bug/需求、分支或 commit 做完整性总结（正文优先、Git 附录；回答内容覆盖；业务疑问分类与知识点归档；文档补充；可选落盘 commit+push） | 「总结…」「归档未记录的业务疑问」「补充某份总结文档」；或 @ 选中该 skill |
| **auto-know** | [`skills/auto-know/`](skills/auto-know/) | 从网站或 AI 对话检索、校验并整理技术面试八股；支持语义去重、归位合并、纠错、索引维护、可选 Git；以及基于知识库的面试出题、评分与练习归档（评分后自动 commit+push） | 「整理八股」「收录八股」「检索八股」「根据对话整理八股」「维护八股知识库」「出一道八股」「八股面试」「八股练习」；或 @ 选中该 skill |
| **auto-cal** | [`skills/auto-cal/`](skills/auto-cal/) | 创建并维护按日期组织的算法题录，支持 Hot 100 选题、答案检查、官方参考答案、结论和知识沉淀 | `/auto-cal`、`创建算法学习文档`、`新增题库`、`检查 ans`、`补充参考答案`；或 @ 选中该 skill |
| **auto-plan** | [`skills/auto-plan/`](skills/auto-plan/) | 任务/需求结构化规划：判定规模（小/中/大），输出需求全貌、阶段拆解、验收标准、执行进度四段式计划 | `/auto-plan`、「制定计划」「任务规划」「需求分析」「拆解任务」；或 @ 选中该 skill |
| **bugfix** | [`skills/bugfix/`](skills/bugfix/) | 结构化 Bug 修复：拆解分析 → 根因定位 → 方案实施 → 环境刷新 → 回测验证 → 修复总结与归档，可选提交文档仓 | `bugfix：` / `bugfix:` 开头；说「解决bug」「修复bug」；或 @ 选中该 skill |
| **manage-skills** | [`skills/manage-skills/`](skills/manage-skills/) | 新增/更新个人 Skills：需求拆解 → diff 预览 → 确认落库 → 可选 Git 提交与全局同步 | 「更新skills」「新增skills」；或 @ 选中该 skill |
| **optimize** | [`skills/optimize/`](skills/optimize/) | 结构化优化流程：现状分析 → 目标定义 → 方案设计 → 实施优化 → 效果验证 → 优化总结 | `optimize：` 开头；说「优化」「提升」「改进」「增强」「重构」；或 @ 选中该 skill |
| **anti-aigc** | [`skills/anti-aigc/`](skills/anti-aigc/) | 对中文正式文档执行反 AIGC 检测优化（含优化后 AIGC 检测评分）：简写展开、指代词自然化、条件句补充、破折号替换、正式用词替换、冒号精简、并列结构打散等 9 个维度 | 「反AIGC优化」「AIGC检测优化」「降低AI检测率」「anti-aigc」「/anti-aigc」「去AI味」；或 @ 选中该 skill |

### auto-know

用于持续维护技术面试八股知识库，而不是把检索结果机械追加到文档末尾。主要能力：

1. **网站检索** — 支持指定 JavaGuide 等站点，也能用官方文档、规范和源码校验技术结论
2. **对话提炼** — 同时扫描用户问题和 Agent 回答，提取原理、追问、误区、版本与适用边界
3. **语义去重** — 写入前检索已有条目，逐题决定新增、合并、修订、冲突保留、迁移或跳过
4. **层级化回答** — 网站模式保留原文栏目树和页面顺序，再还原父问题、子问题、前置依赖和递进关系；答案按简答、原理展开、递进追问、易错点与来源组织
5. **可追溯维护** — 使用稳定 ID、专题索引、适用版本与 Git 历史追踪
6. **安全落库** — 先展示覆盖矩阵和补丁预览，再写入目标知识库并校验索引与重复项
7. **可选 Git 操作** — 知识库正文整理落盘后单独询问 commit+push、仅 commit 或仅落盘，只暂存本轮文件
8. **面试出题模式** — 从已保存知识库抽题（排除最近出过的），面试口吻提问；用户作答后压缩口述要点评分并指出差距；写入 `practice/know-record.md` 后自动 commit+push；作答后可将某题标为「重点」，重点题冷却默认 7 天（普通题默认 14 天）

本机配置存于 `~/.config/auto-know/config.json`（Windows：`%APPDATA%\auto-know\config.json`），不要写进技能源码目录。可选 `quizExcludeDays`（默认 14）、`quizFocusExcludeDays`（默认 7）控制普通题与重点题冷却天数。

### auto-conclusion

对 bug 修复、需求开发、指定 Git 分支/commit，或 Git 范围与对话合并做完整性总结；也支持对已有总结文档按需求从当前对话补充。主要能力：

1. **多轮检索** — 收容与目标相关的分析/方案/问题/验证，排除无关穿插
2. **回答内容覆盖** — 同时拆解用户问题与 Agent 回答中的派生问题、取舍、风险和边界，并用覆盖矩阵逐项确认新增、合并或排除
3. **分支 / 提交 Diff** — 单仓或多项目收集分支相对基线或指定 commit 的新增/修改，汇总后串成跨仓逻辑链路
4. **对话×Git 合并** — 分支+对话 / 提交+对话：diff 写清改动，对话补充动机、踩坑与验证
5. **正文优先** — 需求→思路→问题→方案→流程→经验→提测→git comment；分支/提交明细仅文末附录
6. **文档补充** — 从已有 conclusion 提取需求，收容当前对话相关内容并归位写入（默认写回原文件）
7. **业务疑问与知识点归档** — 先展开、去重并分类；业务问题和知识标题都优先保留用户原始疑问，正文以独立可读为准，按需使用段落、分点、表格或代码块；补充旧文档时保护原文与编号，无匹配才新建
8. **灵活落盘** — 对话指定路径、安装时配置默认目录；均未指定时解析系统下载文件夹并直接落盘
9. **文档仓可选推送** — 安装时探测保存目录（或父目录）git 仓写入用户级配置；落盘后检测 Git 仓库并确认是否 commit+push，失败回报原因
10. **命名规范** — `feat-{slug}-{timestamp}.md` / `bugfix-{slug}-{timestamp}.md`

本机配置存于 `~/.config/auto-conclusion/config.json`（Windows：`%APPDATA%\auto-conclusion\config.json`），**不要**写进技能源码目录。

### auto-cal

面向 LeetCode 等算法学习场景，维护按真实日期组织、可跳转的长期刷题记录：

1. 创建包含题目总览、日期章节和算法知识点章节的学习文档。
2. 默认从 LeetCode Hot 100 随机选择 3 道 Medium 和 1 道 Hard，并兼顾考点差异与最近 20 天去重。
3. 新增题目时只写完整题干和难度，不泄露答案、解法提示或题型 / 考点。
4. 答案检查只评判现有 `ans` 是否可行，不查询标准答案。
5. 参考答案模式先补充题型 / 考点，再写入官方思路、实现与复杂度，并在 `conclusion` 中对比用户答案。
6. 将可复用的算法规律和注意点沉淀到文档底部。

---

### auto-plan

根据用户提出的任务/需求，分析项目上下文并做结构化规划。主要能力：

1. **需求收集** — 读取用户描述，不足时 AskQuestion 补齐关键信息
2. **项目分析** — 扫描项目结构与现有代码，总结与任务相关的现状
3. **规模判定** — 按小/中/大三档分类，给出判定理由
4. **四段式计划** — ①需求全貌（目标、现状、完成标准）②阶段拆解（内容、依赖、工作量）③验收标准（大任务含测试用例）④执行进度
5. **风险识别** — 主动标注技术风险、外部依赖、数据风险、时间风险
6. **进度追踪** — 进度看板实时更新，跨对话续接无需重新检索
7. **确认与调整** — 输出后用 AskQuestion 确认，支持调整或直接开始执行
8. **灵活落盘** — 支持用户指定目录、用户级默认目录与 Downloads 回退；肯定执行且未明确“不保存”时默认先保存计划
9. **文档仓可选推送** — 落盘后优先读取配置仓，否则从目标目录向上探测 Git 仓库，再单独确认 commit+push

本机配置存于 `~/.config/auto-plan/config.json`（Windows：`%APPDATA%/auto-plan/config.json`），**不要**写进技能源码目录。

### bugfix

按 Phase 0～9 的完整流程修复 Bug，避免跳过分析或过早下结论。主要能力：

1. **拆解分析** — 梳理现象、复现、期望、影响范围
2. **问题研判** — 判断类型、优先级与修复范围
3. **根因定位** — 用证据定位，不凭猜测改代码
4. **方案制定** — 选最小正确修复
5. **实施修复** — 最小改动，遵循项目规范
6. **环境刷新** — 按需重建/重启服务，并在对话中告知
7. **回测验证** — 原 Bug 复测 + 回归，有证据才声称已修复
8. **修复总结** — 输出 Bug 内容、原因、方法、现状及 Git commit subject
9. **总结归档** — 按用户配置、项目目录或 Downloads 自动落盘；探测文档仓后单独确认归档 commit+push

信息不完整或需人工决策时，优先用 **AskQuestion** 对话内弹框追问，不另开纯追问对话。

本机配置存于 `~/.config/bugfix/config.json`（Windows：`%APPDATA%/bugfix/config.json`），**不要**写进技能源码目录。

### optimize

按固定流程推进已有功能的优化，面向性能/质量/架构/安全/体验提升。主要能力：

1. **上下文收集** — 查阅历史对话和相关文档，阅读待优化模块代码
2. **现状分析** — 记录基线指标（延迟、准确率、覆盖率等），明确当前不足
3. **目标定义** — 设定量化验收标准和回归底线
4. **方案设计** — 列举并对比 1-3 个方案，推荐最优方案
5. **确认关口** — 展示 diff 预览并获用户确认后再实施
6. **实施优化** — 按方案改动，允许重构和新增（区别于 bugfix 的最小修复）
7. **效果验证** — 对比优化前后指标，运行回归测试
8. **优化总结** — 输出效果对比表、已知限制及 Git commit subject
9. **总结归档** — 归档至 `docs/optimize/`，含「如何避免类似退化」

信息不完整或需人工决策时，优先用 **AskQuestion** 对话内弹框确认。

### anti-aigc

对中文正式文档（如软著操作手册、项目文档）执行系统性的反 AIGC 检测优化。主要能力：

1. **简写展开** — 将 AI 偏好的文言简写（如需、即可、均可、无需、尚未）展开为人类自然表达
2. **指代词自然化** — 把「该」「其」替换为「这个」「它的」等日常指代
3. **条件句连贯性补充** — 在条件从句后的主句中补充"会"字使表达连贯
4. **破折号替换** — 减少 AI 常见的解释性破折号，改为逗号或连接词
5. **正式用词替换** — 部分替换「例如→比如」「若→如果」「以+动词→从而+动词」等
6. **减少冒号** — 将不必要的冒号引导改为逗号直接衔接
7. **并列结构打散** — 变换高度规律的并列枚举句式
8. **引号统一** — 将「」改为""
9. **口语化与正式的平衡** — 避免过度口语或过度 AI 化表达
10. **AIGC 检测评估** — 优化完成后逐段落评分（高/中/低风险），展示检测结果表格

核心原则：内容不变只改表达，部分替换（60%~80%）而非全量，保留适度不一致以模拟真人写作。优化后自动进行检测评估，高风险段落可继续迭代优化。

### manage-skills

在本地 Skills 仓库中新增或更新 Skill 的统一流程：

1. **需求拆解** — 明确新增/更新、目标 skill、变更内容
2. **定位目标** — 扫描仓库现有 skills，读取待改文件
3. **草案与 diff** — 生成预览，**确认前不落库**
4. **确认落库** — AskQuestion：「对比差异后，是否做当前更新」
5. **写入仓库** — 更新 `SKILL.md` 与 `README.md`（新增时）
6. **后续操作** — 可多选：本地 Git commit / 远程 push / 同步到全局 skills 目录
7. **完成提醒** — 提示重启 Cursor 或重新打开项目后生效

---

## 已有 Rules

| Rule | 目录 | 用途 | 生效范围 |
|------|------|------|----------|
| **excellence** | [`rules/excellence/`](rules/excellence/) | 精益求精要求：AskQuestion 强制反馈循环、同轮续跑、上下文切换 checkpoint | User Rule，全局所有项目 |

### excellence（精益求精）

跨场景的全局交互约束，合并了原 `one-time-ask` skill 的独有能力。主要规则：

1. **AskQuestion 强制** — 每次回复末尾必须调用 AskQuestion，选项含继续/调整/无操作
2. **同轮续跑** — AskQuestion 返回后立即继续 workflow，不插入过渡语
3. **上下文切换** — 任务中途插入无关请求时记录 checkpoint，处理完后可回到原任务
4. **禁止空洞选项** — 选项必须具体可执行
5. **违规识别** — 输出文字后直接结束（无 AskQuestion）视为违规

配置方式：将 `RULE.md` 内容添加到 Cursor Settings → Rules → User Rules。

---

## 安装到全局

选择要启用的 skill 后，将 `<skill-name>` 替换为实际目录名，例如 `manage-skills`、`bugfix`。

如果当前终端已经位于本仓库根目录，可以直接安装 `manage-skills`：

```bash
mkdir -p "$HOME/.codex/skills"
ln -sfn "$(pwd)/skills/manage-skills" "$HOME/.codex/skills/manage-skills"
```

### Codex

**macOS / Linux：推荐符号链接**

```bash
mkdir -p "$HOME/.codex/skills"
ln -sfn "<skills-repo>/skills/<skill-name>" "$HOME/.codex/skills/<skill-name>"
```

**macOS / Linux：复制（一次性）**

```bash
mkdir -p "$HOME/.codex/skills"
cp -R "<skills-repo>/skills/<skill-name>" "$HOME/.codex/skills/<skill-name>"
```

### Cursor

**macOS / Linux：推荐符号链接**

```bash
mkdir -p "$HOME/.cursor/skills"
ln -sfn "<skills-repo>/skills/<skill-name>" "$HOME/.cursor/skills/<skill-name>"
```

**macOS / Linux：复制（一次性）**

```bash
mkdir -p "$HOME/.cursor/skills"
cp -R "<skills-repo>/skills/<skill-name>" "$HOME/.cursor/skills/<skill-name>"
```

**Windows PowerShell：推荐符号链接**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.cursor\skills"
New-Item -ItemType SymbolicLink `
  -Path "$env:USERPROFILE\.cursor\skills\<skill-name>" `
  -Target "<skills-repo>\skills\<skill-name>"
```

**Windows PowerShell：复制（一次性）**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.cursor\skills"
Copy-Item -Path "<skills-repo>\skills\<skill-name>" -Destination "$env:USERPROFILE\.cursor\skills\<skill-name>" -Recurse -Force
```

同步后重启对应工具，或重新打开项目 / 新开对话即可使用。

---

## 注意事项

- 不要将 skill 放入 Cursor 的 `skills-cursor` 内置目录，该目录由 Cursor 管理，勿手动修改。
- 团队共享的 skill 建议放在各项目的 `.cursor/skills/` 并提交 Git；本目录适合个人跨项目通用 workflow。
- 编辑 skill 后，若已同步到全局目录，需重新复制或确保符号链接指向最新内容。
- Rules 需手动配置到 Cursor User Rules，不支持符号链接自动生效。
