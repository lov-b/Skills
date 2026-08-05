---
name: manage-skills
description: >-
  在个人 Skills 仓库中新增或更新 Agent Skill：需求拆解、差异对比、确认后落库、可选 Git 提交与全局同步。
  Use when the user says "/manage-skills", "更新skills", "新增skills", "更新 skill",
  "新增 skill", "检查skills", explicitly selects this skill, or asks to create,
  modify, check, pull, or globally sync skills in
  a local or remote personal Skills repository.
---

# Manage Skills — 新增与更新

在任意本地 Skills 仓库中统一管理个人 Skills，并可通过 Git 远程仓库跨设备同步。**确认前不落库**；所有需用户决策的步骤优先用 **AskQuestion** 对话内弹框，不中断当前任务另开追问对话。

## 路径约定

| 用途 | 路径 |
|------|------|
| Skills 仓库（源） | `<skills-repo>`：当前工作区、用户指定路径，或包含 `README.md` 与各 skill 子目录的 Git 根目录 |
| 远程仓库（可选） | `<remote-url>`：通过 `git remote -v` 读取，不写死平台或账号 |
| Codex 全局 Skills | `$CODEX_HOME/skills`；未设置 `$CODEX_HOME` 时使用 `$HOME/.codex/skills` |
| Cursor 全局 Skills（macOS / Linux） | `$HOME/.cursor/skills` |
| Cursor 全局 Skills（Windows） | `%USERPROFILE%\.cursor\skills` |
| Skill 目录结构 | `<skills-repo>/<skill-name>/SKILL.md` |
| 仓库说明 | `<skills-repo>/README.md` |

编写规范优先参考当前环境可用的 skill 创建指南（如 `skill-creator` / `create-skill`），不要依赖某台电脑上的固定内置目录。

## 触发条件

以下任一情况立即启用本 skill：

- 用户消息以 **`/manage-skills`** 开头（推荐写法，见下方「斜杠指令格式」）
- 用户说「更新skills」「新增skills」（含大小写、空格变体）
- 用户说「更新 skill」「新增 skill」「修改 skill」
- 用户说「检查skills」「检查 skills」「同步skills」「同步 skills」
- 用户显式选中或 @ 本 skill

## 检查 Skills — 拉取最新与全局配置

当用户说「检查skills」或等价表达时，进入检查流程，而不是新增/更新 skill 内容流程。

目标：确认当前 `<skills-repo>` 是否为远程最新；若不是最新，先从远程拉取；随后询问是否将最新 skills 配置到全局。

流程：

1. **定位仓库**：按 Phase 2 的 `<skills-repo>` 定位规则确认本地 Skills 仓库。
2. **检查 Git 状态**：
   - 执行 `git status --short`，若存在未提交改动，先报告改动文件。
   - 未提交改动可能与远程拉取冲突时，不执行 pull；用 AskQuestion 或文本 fallback 询问用户处理方式。
3. **检查远程最新状态**：
   - 若存在远程仓库，执行 `git fetch` 更新远程引用。
   - 对比 `HEAD` 与上游分支（如 `@{u}` / `origin/<branch>`）。
   - 本地已最新：报告当前 commit、分支、远程。
   - 本地落后：执行安全拉取，优先使用 `git pull --ff-only`；若无法 fast-forward，停止并报告原因，不自动 merge / rebase。
4. **网络与权限**：
   - `git fetch` / `git pull` 因网络、凭据、权限或沙箱限制失败时，按当前执行环境请求必要授权。
   - 授权仍失败或用户拒绝时，明确报告未完成项，不 silent 跳过。
5. **询问全局配置**：
   - 拉取完成或确认已最新后，必须询问是否同步到全局 skills。
   - 可选目标：Codex 全局、Cursor 全局、两者都同步、跳过同步。
   - 可选范围：全部 skills、仅指定 skills、仅当前已存在于全局的 skills。
   - 可选方式：macOS / Linux 优先符号链接；Windows 可复制或符号链接。
6. **执行全局同步**：
   - 只同步用户确认的目标、范围和方式。
   - 不写入 Cursor / Codex 管理的内置 skills 目录。
   - 完成后提示重启对应工具或新开会话。

Codex 兼容：

- 弹框/选择工具可用时，用弹框询问全局配置。
- 弹框不可用时，用文本选项询问；用户回复前，不执行全局同步。
- 如果用户原话已经明确说「检查skills并同步到 Codex / Cursor / 全局」，可在仓库检查和必要 pull 后，直接同步到明示目标。

## 斜杠指令格式

消息以 `/manage-skills` 开头时：

1. **只启用本 skill**（manage-skills 流程），**不要**加载或执行消息中提到的其他 skill
2. 后续出现的 **目标 skill 标记** 均表示「要新增/更新的 skill 名称」，**不是** 执行该 skill：
   - `【bugfix】`、`【manage-skills】` 等中文方括号包裹的名称
   - `/bugfix`、`/manage-skills` 等斜杠形式（**不含**消息开头用于触发本 skill 的 `/manage-skills`）
3. 方括号或斜杠标记**之后**的正文为**变更需求**，进入 Phase 1 拆解

**示例**：

```
/manage-skills 【bugfix】commit subject 要中英文双语
```

→ 更新 `bugfix` skill，**不**运行 bugfix 修复流程

```
/manage-skills /manage-skills 斜杠指令解析规则
```

→ 更新 `manage-skills` skill 自身

---

## 工作流程

```
Manage Skills Progress:
- [ ] Phase 1: 需求拆解
- [ ] Phase 2: 定位目标 Skill
- [ ] Phase 3: 生成草案并对比差异
- [ ] Phase 4: 确认落库与后续操作（唯一 AskQuestion）
- [ ] Phase 5: 写入仓库
- [ ] Phase 6: 执行 Git / 全局同步
- [ ] Phase 7: 完成提醒
```

---

### Phase 1: 需求拆解

将用户意图拆成可执行项：

1. **操作类型**：新增 skill / 更新已有 skill / 不确定（需判断）
2. **目标 skill**：名称（kebab-case）、用途、触发条件
3. **变更内容**：要增删改的具体章节、流程、输出格式
4. **用户原文**：若用户提供固定措辞，标记为 **verbatim**，写入时原样保留

若消息以 `/manage-skills` 开头，从正文中解析**目标 skill 名称**（`【xxx】` 或 `/xxx`）与**变更需求**，**勿**将内嵌的 skill 名称当作执行指令。

信息不足时，用 **AskQuestion** 弹框补齐（例如：新增还是更新？目标 skill 名称？）。

**输出**：结构化需求摘要。

---

### Phase 2: 定位目标 Skill

在 `<skills-repo>` 中查找。仓库定位规则：

1. 用户显式给出路径时，以该路径为准
2. 未给出路径时，优先使用当前工作区
3. 当前工作区不是 Skills 仓库时，查找最近的 Git 根目录，并确认其中存在 `README.md` 与 `<skill-name>/SKILL.md` 这类结构
4. 仍无法定位时，用 **AskQuestion** 让用户选择或填写本地仓库路径

定位仓库后：

1. 列出 `<skill-name>/` 子目录及现有 `SKILL.md` 的 `name`、`description`
2. 读取 `README.md` 中「已有 Skills」表格
3. 可选读取 `git remote -v`，用于展示远程仓库信息；不要把远程地址写入 skill 内容
4. 判断：
   - **更新**：目标目录已存在 → 读取完整现有文件
   - **新增**：目标目录不存在 → 按 create-skill 规范起草新 skill
   - **名称冲突 / 歧义**：用 **AskQuestion** 让用户选定目标

**输出**：目标路径、操作类型（新增/更新）。

---

### Phase 3: 生成草案并对比差异

1. 基于 Phase 1 需求与 Phase 2 现有内容，生成完整的新版 `SKILL.md` 草案（及需要的附属文件）
2. **禁止在此阶段写入磁盘**
3. **在对话中展示具体改动**（Phase 4 之前必填，不可省略）：
   - 更新：用 unified diff 或分段「删除/新增」对照
   - 新增：展示将创建的文件路径与完整内容预览
   - 若同时改 `README.md`，一并展示 diff
   - **禁止**仅用「会改 xxx」等概括代替具体 diff
4. 简要说明变更要点（3～5 条）
5. **AskQuestion 降级条款（强制）**：若草案/目标 skill 使用了 AskQuestion（含「优先 AskQuestion」「弹框确认」等），必须在该 skill 的注意事项或专用小节写入与下方「AskQuestion 不可用时」等价的强制规则；缺失则不得进入 Phase 4。

**硬性要求**：未在对话中展示具体改动前，**不得**弹出 Phase 4 AskQuestion。

**输出**：diff / 预览 + 变更摘要。

---

### Phase 4: 确认落库与后续操作（唯一 AskQuestion）

**先展示 Phase 3 具体改动，再**弹 **AskQuestion**；用户须基于可见 diff 做确认，**禁止**无 diff 直接询问是否落库。

展示 diff 后，**只弹一次** **AskQuestion**（`allow_multiple: true`），选项如下：

| 选项 | 行为 |
|------|------|
| **全选** | 等同于选中下方三项（本地 commit + 远程 push + 全局同步） |
| 提交到本地 Git 仓库 | 落库 + `git add` + `git commit` |
| 提交到远程 Git 仓库 | 落库 + commit + `git push` |
| 同步到全局 Skills 目录 | 落库 + 复制或链接到当前工具对应的全局 skills 目录（Codex / Cursor，按当前平台解析） |
| **取消** | **不落库，流程结束** |

**AskQuestion 文案要求**（`prompt` 须包含）：

1. 简要说明本次将落库的 skill 与变更要点
2. 列出各选项将触发的操作（写盘 / Git commit / push / 全局同步）
3. **授权说明**：「确认后将同一轮执行所选操作；若出现工具授权弹窗，**允许一次**即可完成全部」

授权说明须在 AskQuestion 中**一次性告知**，**禁止**在用户选完后再单独发「接下来请集中授权」等过渡语。

规则：

- **先 diff 后确认**：Phase 3 具体改动未在对话中展示完整，不得 AskQuestion
- 选「**全选**」→ 展开为三项可执行项（本地 commit、远程 push、全局同步；不含取消）
- 「全选」与「取消」互斥；Agent 解析选项时若含全选，自动展开为上述三项
- 选「**取消**」或**未选任何项** → 不写入磁盘，输出「已取消，未落库」后结束
- 选其他项（可多选，不含取消）→ **同一轮立即**进入 Phase 5 落库 + Phase 6 执行，不等待下一轮对话
- **禁止**在此之后再次弹框确认落库或后续操作

#### Codex 兼容与明示授权

Codex 环境中 AskQuestion / `request_user_input` 可能不可用，或不支持 `allow_multiple`。此时按以下规则降级，保证流程仍可用：

1. 若存在可用的弹框/选择工具：优先使用弹框，并按 Phase 4 选项执行。
2. 若弹框工具不可用：**必须先提示**「当前模型无法呼出 AskQuestion，需要纯文本确认。」，再在对话中展示文本选项，让用户用编号或明确措辞确认；在用户回复前，不执行未获授权的写盘、commit、push、全局同步。
3. 若用户原话已经明确授权某些动作，可跳过弹框/文本确认，只执行被明示授权的动作：
   - 「落库 / 写入 / 保存 / 按 diff 更新 / 确认更新」→ 仅允许写入文件
   - 「提交 / commit / 本地提交」→ 允许写入文件 + 本地 Git commit
   - 「推送 / push / 提交到远程」→ 允许写入文件 + 必要的本地 Git commit + `git push`
   - 「同步 / 全局同步 / 安装到全局 / 同步到 Codex / 同步到 Cursor」→ 允许写入文件 + 对应全局目录同步
   - 「全选 / 全部执行」→ 允许写入文件 + 本地 Git commit + 远程 push + 全局同步
4. 授权按动作粒度解析：没有明确允许 **commit** 时，不得执行 `git commit`；没有明确允许 **push** 时，不得执行 `git push`；没有明确允许 **全局同步** 时，不得写入全局 skills 目录。
5. 仅说「更新 / 修改」默认只代表允许生成方案与请求确认；若已展示 diff 且用户明确要求继续落库或确认更新，才允许写入文件，不得自动 commit、push 或全局同步。

**输出**：用户选择项列表（或「取消」；含全选时列出展开后的项）。

---

### Phase 5: 写入仓库

用户 Phase 4 选择非取消项，或原话已按「Codex 兼容与明示授权」明确授权写入后执行：

1. 写入 `<skills-repo>/<skill-name>/SKILL.md`（及 `reference.md`、`scripts/` 等附属文件）
2. **新增 skill** 时，同步更新 `<skills-repo>/README.md`：
   - 在「已有 Skills」表格增加一行
   - 可选：增加简短说明小节
3. 写入完成后说明已落库的文件列表

**输出**：已写入路径列表。

**权限与执行**（与 Phase 4 配合，减少二次点击）：

1. **授权前置**：权限说明已并入 Phase 4 AskQuestion，此处**不再重复**「接下来请集中授权」
2. **同一轮执行**：AskQuestion 返回后**立即** Write + Shell，中间**不插入**过渡段落
3. **合并 Shell**：Phase 6 的 git / push / 全局同步命令尽量合并为**单次 Shell 调用**
4. **权限被拒**：明确报告未完成项及原因，不 silent 跳过

---

### Phase 6: 执行 Git / 全局同步

按 Phase 4 用户选择逐项执行（不再 AskQuestion）：

| 选项 | 动作 |
|------|------|
| 提交到本地 Git 仓库 | 在 `<skills-repo>` 执行 `git add` + `git commit`（遵循仓库既有 commit 风格） |
| 提交到远程 Git 仓库 | 本地 commit 成功后执行 `git push`（需用户选中；若未选本地 commit 则先 commit 再 push） |
| 同步到全局 Skills 目录 | 将 `<skill-name>` 同步到当前工具对应的全局 skills 目录；macOS / Linux 优先使用 `ln -sfn`，Windows PowerShell 使用 `Copy-Item -Recurse -Force` 或 `New-Item -ItemType SymbolicLink` |

- 用户未选中的项 **不执行**
- Git 操作仅在 `<skills-repo>` 为 git 仓库时进行；若无 `.git`，告知用户并跳过 Git 选项
- 同步到全局目录前，先根据当前运行环境和用户意图选择目标：
  - Codex：`$CODEX_HOME/skills`，未设置时 `$HOME/.codex/skills`
  - Cursor macOS / Linux：`$HOME/.cursor/skills`
  - Cursor Windows：`%USERPROFILE%\.cursor\skills`
- 若无法判断同步到 Codex 还是 Cursor，用 **AskQuestion** 选择；不要写入 `skills-cursor` 这类内置目录
- `git push` 失败时报告原因，不 force push
- `git push` 因网络、凭据、权限或沙箱限制失败时，按当前执行环境请求必要授权；授权仍失败或用户拒绝时，明确报告 push 未完成，不 silent 跳过
- **远程不可联通**：push 前可先 `git ls-remote` 探测，或直接尝试 push 并捕获网络/连接类错误
  - 若远程不可达：**跳过 push**，**不回滚**已成功的本地 commit
  - 在对话与 Phase 7「Git 记录」中**明确通知**用户（含失败原因，如 timeout / could not resolve host）
  - 本地 commit 与用户已选的其它操作（如全局同步）仍视为成功
- 同步全局目录时覆盖同名 skill

**Commit Message 规范**（用户选中 Git 提交时 **必须** 遵守）：

1. **Subject**（第一行）：`<type>(<scope>):` 保持英文（conventional commits），冒号后的描述用**中文**，如 `feat(manage-skills): 新增文档保存功能` / `fix(bugfix): 修复提交阶段缺少总结的问题`
2. **Body**（空一行后）：用**中文**写 2～4 句，说明**改了什么、为什么改**，覆盖本次所有落库文件
3. 在对话中**展示完整 commit message**（subject + body），再执行 commit
4. push 成功后，在对话中告知推送的 **commit hash、分支、远程仓库**

**输出**：每项操作的结果（成功 / 跳过 / 失败原因）。

---

### Phase 7: 完成提醒

在对话末尾 **必须** 输出：

```markdown
## Skills 更新完成

- **操作**：[新增 / 更新] `<skill-name>`
- **落库路径**：`<skills-repo>/<skill-name>/`
- **已执行**：[本地 commit / 远程 push / 全局同步 — 列出实际执行的项，或「无」]

### 改动总结

[必填：3～5 条 bullet，说明本次具体改了什么，便于用户快速回顾]

- 变更点 1：...
- 变更点 2：...

### Git 记录（如有提交）

- **Commit**：`<hash>` — `<subject>`
- **Body 摘要**：...
- **Push**：`<remote>/<branch>`（或「未推送」）

> **提醒**：Skills 已更新。请 **重启 Cursor 或重新打开项目**，新的 skill 才会生效。
```

---

## AskQuestion 不可用时（强制提示）

凡本 skill 流程需要 AskQuestion，若 Agent 无法呼起该工具（未挂载或调用失败）：

1. **必须在对话中提示用户**：当前模型无法呼出 AskQuestion，需要纯文本确认。
2. 随后用编号/选项文本列出待确认项，等待用户回复后再继续；**禁止**静默跳过确认。

### 起草/更新其它 skill 时的强制约束

新增或更新 skill 时，若内容涉及 AskQuestion，**必须**写入与上节等价的降级提示规则（可用同一模板）。检查 skills 拉取后同步全局时，不因此条款自动改写未涉及本次变更的 skill。

---

## 注意事项

- **斜杠指令优先**：`/manage-skills` 后续出现的 `【skill】` / `/skill` 仅为**更新目标**，禁止当作 skill 执行指令
- **先 diff 后落库**：Phase 4 确认前不得修改 `<skills-repo>` 下任何文件；**确认前须在对话中展示具体改动**
- **确认前必展示改动**：AskQuestion 前对话中须有可审查的具体 diff/预览，禁止仅口头描述
- **只问一次**：落库与 Git/同步合并在 Phase 4 唯一 AskQuestion，禁止二次确认
- **Codex 降级**：弹框不可用时须先提示「当前模型无法呼出 AskQuestion，需要纯文本确认」，再用文本确认；用户未回复确认前，不执行未获授权的写盘、commit、push、全局同步
- **授权分级**：写盘、commit、push、全局同步是四类独立授权；只执行用户通过弹框、文本确认或原话明示允许的动作
- **全选选项**：`allow_multiple: true` 的 AskQuestion 须提供「全选」；选全选 = 选除「取消」外全部可执行项
- **权限一次申请**：授权说明并入 Phase 4 AskQuestion；确认后同一轮执行，禁止二次提醒授权
- **最小改动**：更新时只改与需求相关的部分，不顺手重写无关章节
- **verbatim 优先**：用户指定的触发词、流程措辞原样写入，不擅自改写
- **不碰内置目录**：禁止写入 Cursor / Codex 管理的内置 skills 目录；只同步到用户级全局 skills 目录
- **README 同步**：新增 skill 必须更新 README；更新 skill 若触发方式/用途变化，同步改 README 表格
- **AskQuestion 优先**：用户决策用对话内弹框，不另开纯追问对话
- **AskQuestion 降级必写**：使用 AskQuestion 的 skill（含本 skill）必须含「无法呼起时提示用户改用纯文本确认」条款
- **改动总结必填**：Phase 7 必须输出改动总结，不可仅说「已更新」
- **Commit 有内容**：Git 提交禁止空 message 或仅写「update」；subject 的 `<type>(<scope>):` 保持英文，描述用中文；body 用中文；均需有意义
