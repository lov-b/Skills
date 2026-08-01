# Personal Skills

本目录用于存放**个人使用的 Cursor Agent Skills**，作为统一管理与备份位置。

每个 skill 是一个独立子目录，内含必需的 `SKILL.md` 文件。需要在本机生效时，复制或链接到 Cursor 全局目录：

```
C:\Users\Bingo\.cursor\skills\<skill-name>\
```

> Cursor 默认不会自动扫描 `D:\Program\Skills`，需手动同步到上述路径，或在项目中使用 `.cursor/skills/`。

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
| **bugfix** | [`bugfix/`](bugfix/) | 结构化 Bug 修复：拆解分析 → 根因定位 → 方案实施 → 环境刷新 → 回测验证 → 修复总结 | `bugfix：` / `bugfix:` 开头；说「解决bug」「修复bug」；或 @ 选中该 skill |
| **manage-skills** | [`manage-skills/`](manage-skills/) | 新增/更新个人 Skills：需求拆解 → diff 预览 → 确认落库 → 可选 Git 提交与全局同步 | 「更新skills」「新增skills」；或 @ 选中该 skill |
| **one-time-ask** | [`one-time-ask/`](one-time-ask/) | 不打断任务：所有决策用 AskQuestion 弹框，同轮继续执行 | `/one-time-ask`；或 @ 选中该 skill |

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

在 `D:\Program\Skills` 中新增或更新 Skill 的统一流程：

1. **需求拆解** — 明确新增/更新、目标 skill、变更内容
2. **定位目标** — 扫描仓库现有 skills，读取待改文件
3. **草案与 diff** — 生成预览，**确认前不落库**
4. **确认落库** — AskQuestion：「对比差异后，是否做当前更新」
5. **写入仓库** — 更新 `SKILL.md` 与 `README.md`（新增时）
6. **后续操作** — 可多选：本地 Git commit / 远程 push / 同步到 `~/.cursor/skills/`
7. **完成提醒** — 提示重启 Cursor 或重新打开项目后生效

### one-time-ask

约束 Agent **交互方式**（可与 bugfix 等叠加）：

1. **禁止中断式追问** — 不结束 turn 等待用户下一条消息
2. **一律 AskQuestion** — 缺信息、选方案、要确认 → 对话内弹框
3. **同轮续跑** — 用户在弹框中选择后，Agent 立即继续执行
4. **上下文切换** — 无关任务插入时记录 checkpoint，处理完可 AskQuestion 是否回到原任务

---

## 同步到 Cursor

**复制（一次性）：**

```powershell
Copy-Item -Path "D:\Program\Skills\bugfix" -Destination "C:\Users\Bingo\.cursor\skills\bugfix" -Recurse -Force
```

**符号链接（本目录为源，改一处即生效）：**

```powershell
New-Item -ItemType SymbolicLink `
  -Path "C:\Users\Bingo\.cursor\skills\bugfix" `
  -Target "D:\Program\Skills\bugfix"
```

同步后重启 Cursor 或新开对话即可使用。

---

## 注意事项

- 不要将 skill 放入 `C:\Users\Bingo\.cursor\skills-cursor\`，该目录为 Cursor 内置 skill，勿手动修改。
- 团队共享的 skill 建议放在各项目的 `.cursor/skills/` 并提交 Git；本目录适合个人跨项目通用 workflow。
- 编辑 skill 后，若已同步到全局目录，需重新复制或确保符号链接指向最新内容。
