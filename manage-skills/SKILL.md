---
name: manage-skills
description: >-
  在个人 Skills 仓库中新增或更新 Agent Skill：需求拆解、差异对比、确认后落库、可选 Git 提交与全局同步。
  Use when the user says "更新skills", "新增skills", "更新 skill", "新增 skill",
  explicitly selects this skill, or asks to create/modify skills in D:\Program\Skills.
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

- 用户说「更新skills」「新增skills」（含大小写、空格变体）
- 用户说「更新 skill」「新增 skill」「修改 skill」
- 用户显式选中或 @ 本 skill

---

## 工作流程

```
Manage Skills Progress:
- [ ] Phase 1: 需求拆解
- [ ] Phase 2: 定位目标 Skill
- [ ] Phase 3: 生成草案并对比差异
- [ ] Phase 4: 用户确认是否落库
- [ ] Phase 5: 写入仓库
- [ ] Phase 6: 后续操作（Git / 全局同步）
- [ ] Phase 7: 完成提醒
```

---

### Phase 1: 需求拆解

将用户意图拆成可执行项：

1. **操作类型**：新增 skill / 更新已有 skill / 不确定（需判断）
2. **目标 skill**：名称（kebab-case）、用途、触发条件
3. **变更内容**：要增删改的具体章节、流程、输出格式
4. **用户原文**：若用户提供固定措辞，标记为 **verbatim**，写入时原样保留

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

### Phase 4: 用户确认是否落库

用 **AskQuestion** 弹框，选项示例：

- **是，按预览内容落库**
- **否，我再改改需求**（回到 Phase 1）
- **取消，不保存**

仅当用户选择「是」时进入 Phase 5。选「否」则根据反馈修订草案，回到 Phase 3 重新展示 diff。

---

### Phase 5: 写入仓库

用户确认后执行：

1. 写入 `D:\Program\Skills\<skill-name>\SKILL.md`（及 `reference.md`、`scripts/` 等附属文件）
2. **新增 skill** 时，同步更新 `D:\Program\Skills\README.md`：
   - 在「已有 Skills」表格增加一行
   - 可选：增加简短说明小节
3. 写入完成后说明已落库的文件列表

**输出**：已写入路径列表。

---

### Phase 6: 后续操作（Git / 全局同步）

落库成功后，用 **AskQuestion**（`allow_multiple: true`）一次性提供三项，用户可多选：

| 选项 | 动作 |
|------|------|
| 提交到本地 Git 仓库 | 在 `D:\Program\Skills` 执行 `git add` + `git commit`（遵循仓库既有 commit 风格） |
| 提交到远程 Git 仓库 | 本地 commit 成功后执行 `git push`（需用户选中且本地已 commit） |
| 同步到 Cursor 全局目录 | `Copy-Item -Recurse -Force` 将 `<skill-name>` 复制到 `C:\Users\Bingo\.cursor\skills\<skill-name>\` |

规则：

- 用户未选中的项 **不执行**
- Git 操作仅在 `D:\Program\Skills` 为 git 仓库时进行；若无 `.git`，告知用户并跳过 Git 选项
- `git push` 失败时报告原因，不 force push
- 同步全局目录时覆盖同名 skill

**输出**：每项操作的结果（成功 / 跳过 / 失败原因）。

---

### Phase 7: 完成提醒

在对话末尾 **必须** 输出：

```markdown
## Skills 更新完成

- **操作**：[新增 / 更新] `<skill-name>`
- **落库路径**：`D:\Program\Skills\<skill-name>\`
- **已执行**：[本地 commit / 远程 push / 全局同步 — 列出实际执行的项，或「无」]

> **提醒**：Skills 已更新。请 **重启 Cursor 或重新打开项目**，新的 skill 才会生效。
```

---

## 注意事项

- **先 diff 后落库**：Phase 4 确认前不得修改 `D:\Program\Skills` 下任何文件
- **最小改动**：更新时只改与需求相关的部分，不顺手重写无关章节
- **verbatim 优先**：用户指定的触发词、流程措辞原样写入，不擅自改写
- **不碰内置目录**：禁止写入 `C:\Users\Bingo\.cursor\skills-cursor\`
- **README 同步**：新增 skill 必须更新 README；更新 skill 若触发方式/用途变化，同步改 README 表格
- **AskQuestion 优先**：确认落库、后续 Git/同步均用对话内弹框，不另开纯追问对话
