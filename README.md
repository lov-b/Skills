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
| **auto-conclusion** | [`skills/auto-conclusion/`](skills/auto-conclusion/) | 对 bug/需求、分支或 commit 做完整性总结（正文优先、Git 附录；对话合并；文档补充；可选落盘 commit+push） | 「总结…」「补充某份总结文档」；或 @ 选中该 skill |
| **auto-plan** | [`skills/auto-plan/`](skills/auto-plan/) | 任务/需求结构化规划：判定规模（小/中/大），输出需求全貌、阶段拆解、验收标准、执行进度四段式计划 | `/auto-plan`、「制定计划」「任务规划」「需求分析」「拆解任务」；或 @ 选中该 skill |
| **bugfix** | [`skills/bugfix/`](skills/bugfix/) | 结构化 Bug 修复：拆解分析 → 根因定位 → 方案实施 → 环境刷新 → 回测验证 → 修复总结 | `bugfix：` / `bugfix:` 开头；说「解决bug」「修复bug」；或 @ 选中该 skill |
| **manage-skills** | [`skills/manage-skills/`](skills/manage-skills/) | 新增/更新个人 Skills：需求拆解 → diff 预览 → 确认落库 → 可选 Git 提交与全局同步 | 「更新skills」「新增skills」；或 @ 选中该 skill |
| **optimize** | [`skills/optimize/`](skills/optimize/) | 结构化优化流程：现状分析 → 目标定义 → 方案设计 → 实施优化 → 效果验证 → 优化总结 | `optimize：` 开头；说「优化」「提升」「改进」「增强」「重构」；或 @ 选中该 skill |

### auto-conclusion

对 bug 修复、需求开发、指定 Git 分支/commit，或 Git 范围与对话合并做完整性总结；也支持对已有总结文档按需求从当前对话补充。主要能力：

1. **多轮检索** — 收容与目标相关的分析/方案/问题/验证，排除无关穿插
2. **分支 / 提交 Diff** — 单仓或多项目收集分支相对基线或指定 commit 的新增/修改，汇总后串成跨仓逻辑链路
3. **对话×Git 合并** — 分支+对话 / 提交+对话：diff 写清改动，对话补充动机、踩坑与验证
4. **正文优先** — 需求→思路→问题→方案→流程→经验→提测→git comment；分支/提交明细仅文末附录
5. **文档补充** — 从已有 conclusion 提取需求，收容当前对话相关内容并归位写入（默写回原文件）
6. **灵活落盘** — 对话指定路径、安装时配置默认目录、或确认后写入系统下载文件夹
7. **文档仓可选推送** — 安装时探测保存目录（或父目录）git 仓写入用户级配置；落盘确认可选 commit+push，失败回报原因
8. **命名规范** — `feat-{slug}-{timestamp}.md` / `bugfix-{slug}-{timestamp}.md`

本机配置存于 `~/.config/auto-conclusion/config.json`（Windows：`%APPDATA%\auto-conclusion\config.json`），**不要**写进技能源码目录。

### auto-plan

根据用户提出的任务/需求，分析项目上下文并做结构化规划。主要能力：

1. **需求收集** — 读取用户描述，不足时 AskQuestion 补齐关键信息
2. **项目分析** — 扫描项目结构与现有代码，总结与任务相关的现状
3. **规模判定** — 按小/中/大三档分类，给出判定理由
4. **四段式计划** — ①需求全貌（目标、现状、完成标准）②阶段拆解（内容、依赖、工作量）③验收标准（大任务含测试用例）④执行进度
5. **风险识别** — 主动标注技术风险、外部依赖、数据风险、时间风险
6. **进度追踪** — 进度看板实时更新，跨对话续接无需重新检索
7. **确认与调整** — 输出后用 AskQuestion 确认，支持调整或直接开始执行

### bugfix

按 8 个阶段固定流程修复 Bug，避免跳过分析或过早下结论。主要能力：

1. **拆解分析** — 梳理现象、复现、期望、影响范围
2. **问题研判** — 判断类型、优先级与修复范围
3. **根因定位** — 用证据定位，不凭猜测改代码
4. **方案制定** — 选最小正确修复
5. **实施修复** — 最小改动，遵循项目规范
6. **环境刷新** — 按需重建/重启服务，并在对话中告知
7. **回测验证** — 原 Bug 复测 + 回归，有证据才声称已修复
8. **修复总结** — 输出 Bug 内容、原因、方法、现状及 Git commit subject

信息不完整或需人工决策时，优先用 **AskQuestion** 对话内弹框追问，不另开纯追问对话。

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
