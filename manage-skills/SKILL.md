---
name: manage-skills
description: >-
  在个人 Skills 仓库中新增或更新 Agent Skill：需求拆解、差异对比、确认后落库、可选 Git 提交与全局同步。
  Use when the user says "/manage-skills", "更新skills", "新增skills", "更新 skill",
  "新增 skill", explicitly selects this skill, or asks to create/modify skills in
  D:\Program\Skills.
---

# Manage Skills — 新增与更新

在 `D:\Program\Skills` 统一管理个人 Skills。**确认前不落库**；所有需用户决策的步骤优先用 **AskQuestion** 对话内弹框，不中断当前任务另开追问对话。

## 路径常量

| 用途 | 路径 |
|------|------|
| Skills 仓库（源） | `D:\Program\Skills` |
| Cursor 全局 Skills | `C:\Users\Bingo\.cursor\skills\` |
| Skill 目录结构 | `D:\Program\Skills\<skill-name>\SKILL.md` |
| 仓库说明 | `D:\Program\Skills\README.md` |

编写规范可参考内置 **create-skill** skill（`~/.cursor/skills-cursor/create-skill`）。

## 触发条件

以下任一情况立即启用本 skill：

- 用户消息以 **`/manage-skills`** 开头（推荐写法，见下方「斜杠指令格式」）
- 用户说「更新skills」「新增skills」（含大小写、空格变体）
- 用户说「更新 skill」「新增 skill」「修改 skill」
- 用户显式选中或 @ 本 skill

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

在 `D:\Program\Skills` 中查找：

1. 列出 `<skill-name>/` 子目录及现有 `SKILL.md` 的 `name`、`description`
2. 读取 `README.md` 中「已有 Skills」表格
3. 判断：
   - **更新**：目标目录已存在 → 读取完整现有文件
   - **新增**：目标目录不存在 → 按 create-skill 规范起草新 skill
   - **名称冲突 / 歧义**：用 **AskQuestion** 让用户选定目标

**输出**：目标路径、操作类型（新增/更新）。

---

### Phase 3: 生成草案并对比差异

1. 基于 Phase 1 需求与 Phase 2 现有内容，生成完整的新版 `SKILL.md` 草案（及需要的附属文件）
2. **禁止在此阶段写入磁盘**
3. 在对话中展示 **更新前 vs 更新后** 的差异：
   - 更新：用 unified diff 或分段「删除/新增」对照
   - 新增：展示将创建的文件路径与完整内容预览
   - 若同时改 `README.md`，一并展示 diff
4. 简要说明变更要点（3～5 条）

**输出**：diff / 预览 + 变更摘要。

---

### Phase 4: 确认落库与后续操作（唯一 AskQuestion）

展示 diff 后，**只弹一次** **AskQuestion**（`allow_multiple: true`），选项如下：

| 选项 | 行为 |
|------|------|
| **全选** | 等同于选中下方三项（本地 commit + 远程 push + 全局同步） |
| 提交到本地 Git 仓库 | 落库 + `git add` + `git commit` |
| 提交到远程 Git 仓库 | 落库 + commit + `git push` |
| 同步到 Cursor 全局目录 | 落库 + 复制到 `C:\Users\Bingo\.cursor\skills\` |
| **取消** | **不落库，流程结束** |

**AskQuestion 文案要求**（`prompt` 须包含）：

1. 简要说明本次将落库的 skill 与变更要点
2. 列出各选项将触发的操作（写盘 / Git commit / push / 全局同步）
3. **授权说明**：「确认后将同一轮执行所选操作；若出现工具授权弹窗，**允许一次**即可完成全部」

授权说明须在 AskQuestion 中**一次性告知**，**禁止**在用户选完后再单独发「接下来请集中授权」等过渡语。

规则：

- 选「**全选**」→ 展开为三项可执行项（本地 commit、远程 push、全局同步；不含取消）
- 「全选」与「取消」互斥；Agent 解析选项时若含全选，自动展开为上述三项
- 选「**取消**」或**未选任何项** → 不写入磁盘，输出「已取消，未落库」后结束
- 选其他项（可多选，不含取消）→ **同一轮立即**进入 Phase 5 落库 + Phase 6 执行，不等待下一轮对话
- **禁止**在此之后再次弹框确认落库或后续操作

**输出**：用户选择项列表（或「取消」；含全选时列出展开后的项）。

---

### Phase 5: 写入仓库

用户 Phase 4 选择非取消项后执行：

1. 写入 `D:\Program\Skills\<skill-name>\SKILL.md`（及 `reference.md`、`scripts/` 等附属文件）
2. **新增 skill** 时，同步更新 `D:\Program\Skills\README.md`：
   - 在「已有 Skills」表格增加一行
   - 可选：增加简短说明小节
3. 写入完成后说明已落库的文件列表

**输出**：已写入路径列表。

**权限与执行**（与 Phase 4 配合，减少二次点击）：

1. **授权前置**：权限说明已并入 Phase 4 AskQuestion，此处**不再重复**「接下来请集中授权」
2. **同一轮执行**：AskQuestion 返回后**立即** Write + Shell，中间**不插入**过渡段落
3. **合并 Shell**：Phase 6 的 git / push / Copy-Item 合并为**单次 Shell 调用**
4. **权限被拒**：明确报告未完成项及原因，不 silent 跳过

---

### Phase 6: 执行 Git / 全局同步

按 Phase 4 用户选择逐项执行（不再 AskQuestion）：

| 选项 | 动作 |
|------|------|
| 提交到本地 Git 仓库 | 在 `D:\Program\Skills` 执行 `git add` + `git commit`（遵循仓库既有 commit 风格） |
| 提交到远程 Git 仓库 | 本地 commit 成功后执行 `git push`（需用户选中；若未选本地 commit 则先 commit 再 push） |
| 同步到 Cursor 全局目录 | `Copy-Item -Recurse -Force` 将 `<skill-name>` 复制到 `C:\Users\Bingo\.cursor\skills\<skill-name>\` |

- 用户未选中的项 **不执行**
- Git 操作仅在 `D:\Program\Skills` 为 git 仓库时进行；若无 `.git`，告知用户并跳过 Git 选项
- `git push` 失败时报告原因，不 force push
- **远程不可联通**：push 前可先 `git ls-remote` 探测，或直接尝试 push 并捕获网络/连接类错误
  - 若远程不可达：**跳过 push**，**不回滚**已成功的本地 commit
  - 在对话与 Phase 7「Git 记录」中**明确通知**用户（含失败原因，如 timeout / could not resolve host）
  - 本地 commit 与用户已选的其它操作（如全局同步）仍视为成功
- 同步全局目录时覆盖同名 skill

**Commit Message 规范**（用户选中 Git 提交时 **必须** 遵守）：

1. **Subject**（第一行）：conventional commits 格式，如 `feat(manage-skills): ...` / `fix(bugfix): ...`
2. **Body**（空一行后）：2～4 句说明**改了什么、为什么改**，覆盖本次所有落库文件
3. 在对话中**展示完整 commit message**（subject + body），再执行 commit
4. push 成功后，在对话中告知推送的 **commit hash、分支、远程仓库**

**输出**：每项操作的结果（成功 / 跳过 / 失败原因）。

---

### Phase 7: 完成提醒

在对话末尾 **必须** 输出：

```markdown
## Skills 更新完成

- **操作**：[新增 / 更新] `<skill-name>`
- **落库路径**：`D:\Program\Skills\<skill-name>\`
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

## 注意事项

- **斜杠指令优先**：`/manage-skills` 后续出现的 `【skill】` / `/skill` 仅为**更新目标**，禁止当作 skill 执行指令
- **先 diff 后落库**：Phase 4 确认前不得修改 `D:\Program\Skills` 下任何文件
- **只问一次**：落库与 Git/同步合并在 Phase 4 唯一 AskQuestion，禁止二次确认
- **全选选项**：`allow_multiple: true` 的 AskQuestion 须提供「全选」；选全选 = 选除「取消」外全部可执行项
- **权限一次申请**：授权说明并入 Phase 4 AskQuestion；确认后同一轮执行，禁止二次提醒授权
- **最小改动**：更新时只改与需求相关的部分，不顺手重写无关章节
- **verbatim 优先**：用户指定的触发词、流程措辞原样写入，不擅自改写
- **不碰内置目录**：禁止写入 `C:\Users\Bingo\.cursor\skills-cursor\`
- **README 同步**：新增 skill 必须更新 README；更新 skill 若触发方式/用途变化，同步改 README 表格
- **AskQuestion 优先**：用户决策用对话内弹框，不另开纯追问对话
- **改动总结必填**：Phase 7 必须输出改动总结，不可仅说「已更新」
- **Commit 有内容**：Git 提交禁止空 message 或仅写「update」；subject 与 body 均需有意义
