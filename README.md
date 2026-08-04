# Personal Skills

本目录用于存放**个人使用的 Agent Skills**，作为统一管理与备份位置。

每个 skill 是一个独立子目录，内含必需的 `SKILL.md` 文件。仓库可以放在任意本地路径，例如：

```
<skills-repo>/
├── README.md
└── <skill-name>/
    └── SKILL.md
```

其中 `<skills-repo>` 代表你 clone 或保存本仓库的实际路径，不要求固定为某台电脑上的指定目录。需要在本机生效时，将目标 skill 复制或链接到对应工具的全局 skills 目录。

---

## 目录结构

```
Skills/
├── README.md           # 本说明文件
└── <skill-name>/
    └── SKILL.md        # skill 主文件（必需）
```

新增 skill 时，在本目录下创建 `<skill-name>/SKILL.md`，并更新本文档的「已有 Skills」列表。

---

## 已有 Skills

| Skill | 目录 | 用途 | 触发方式 |
|-------|------|------|----------|
| **auto-conclusion** | [`auto-conclusion/`](auto-conclusion/) | 对 bug/需求、分支或 commit 做完整性总结（正文优先、Git 附录；对话合并；文档补充；可选落盘 commit+push） | 「总结…」「补充某份总结文档」；或 @ 选中该 skill |
| **bugfix** | [`bugfix/`](bugfix/) | 结构化 Bug 修复：拆解分析 → 根因定位 → 方案实施 → 环境刷新 → 回测验证 → 修复总结 | `bugfix：` / `bugfix:` 开头；说「解决bug」「修复bug」；或 @ 选中该 skill |
| **manage-skills** | [`manage-skills/`](manage-skills/) | 新增/更新个人 Skills：需求拆解 → diff 预览 → 确认落库 → 可选 Git 提交与全局同步 | 「更新skills」「新增skills」；或 @ 选中该 skill |
| **one-time-ask** | [`one-time-ask/`](one-time-ask/) | 不打断任务：所有决策用 AskQuestion 弹框，同轮继续执行 | `/one-time-ask`；或 @ 选中该 skill |

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

### manage-skills

在本地 Skills 仓库中新增或更新 Skill 的统一流程：

1. **需求拆解** — 明确新增/更新、目标 skill、变更内容
2. **定位目标** — 扫描仓库现有 skills，读取待改文件
3. **草案与 diff** — 生成预览，**确认前不落库**
4. **确认落库** — AskQuestion：「对比差异后，是否做当前更新」
5. **写入仓库** — 更新 `SKILL.md` 与 `README.md`（新增时）
6. **后续操作** — 可多选：本地 Git commit / 远程 push / 同步到全局 skills 目录
7. **完成提醒** — 提示重启 Cursor 或重新打开项目后生效

### one-time-ask

约束 Agent **交互方式**（可与 bugfix 等叠加）：

1. **禁止中断式追问** — 不结束 turn 等待用户下一条消息
2. **一律 AskQuestion** — 缺信息、选方案、要确认 → 对话内弹框
3. **同轮续跑** — 用户在弹框中选择后，Agent 立即继续执行
4. **上下文切换** — 无关任务插入时记录 checkpoint，处理完可 AskQuestion 是否回到原任务

---

## 安装到全局

选择要启用的 skill 后，将 `<skill-name>` 替换为实际目录名，例如 `manage-skills`、`bugfix` 或 `one-time-ask`。

如果当前终端已经位于本仓库根目录，可以直接安装 `manage-skills`：

```bash
mkdir -p "$HOME/.codex/skills"
ln -sfn "$(pwd)/manage-skills" "$HOME/.codex/skills/manage-skills"
```

### Codex

**macOS / Linux：推荐符号链接**

```bash
mkdir -p "$HOME/.codex/skills"
ln -sfn "<skills-repo>/<skill-name>" "$HOME/.codex/skills/<skill-name>"
```

**macOS / Linux：复制（一次性）**

```bash
mkdir -p "$HOME/.codex/skills"
cp -R "<skills-repo>/<skill-name>" "$HOME/.codex/skills/<skill-name>"
```

### Cursor

**macOS / Linux：推荐符号链接**

```bash
mkdir -p "$HOME/.cursor/skills"
ln -sfn "<skills-repo>/<skill-name>" "$HOME/.cursor/skills/<skill-name>"
```

**macOS / Linux：复制（一次性）**

```bash
mkdir -p "$HOME/.cursor/skills"
cp -R "<skills-repo>/<skill-name>" "$HOME/.cursor/skills/<skill-name>"
```

**Windows PowerShell：推荐符号链接**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.cursor\skills"
New-Item -ItemType SymbolicLink `
  -Path "$env:USERPROFILE\.cursor\skills\<skill-name>" `
  -Target "<skills-repo>\<skill-name>"
```

**Windows PowerShell：复制（一次性）**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.cursor\skills"
Copy-Item -Path "<skills-repo>\<skill-name>" -Destination "$env:USERPROFILE\.cursor\skills\<skill-name>" -Recurse -Force
```

同步后重启对应工具，或重新打开项目 / 新开对话即可使用。

---

## 注意事项

- 不要将 skill 放入 Cursor 的 `skills-cursor` 内置目录，该目录由 Cursor 管理，勿手动修改。
- 团队共享的 skill 建议放在各项目的 `.cursor/skills/` 并提交 Git；本目录适合个人跨项目通用 workflow。
- 编辑 skill 后，若已同步到全局目录，需重新复制或确保符号链接指向最新内容。
