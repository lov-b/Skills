---
name: auto-conclusion
description: >-
  对当前开发中的 bug 修复或需求实现做完整性总结，检索相关历史对话，
  整理需求、思路、问题、方案、流程、经验、提测变更与 git commit，并写入本地文档。
  Use when the user says「总结这个对话中解决的bug」「总结刚刚解决的bug」
  「总结刚刚完成的需求」「总结xx需求」, mentions 开发总结/结论归档,
  or explicitly selects this skill.
---

# Auto Conclusion — 开发完整性总结

对用户当前完成的 **bug 修复** 或 **需求开发** 做完整性总结，写入本地 Markdown 文档。
覆盖多轮对话中的相关内容；穿插的无关话题不收容。

## 触发条件

以下任一情况立即启用本 skill：

- 用户说「总结这个对话中解决的bug」「总结刚刚解决的bug」
- 用户说「总结刚刚完成的需求」「总结xx需求」（某一特定需求，不一定是上一轮对话）
- 用户说「开发总结」「结论归档」「auto-conclusion」
- 用户显式选中或 @ 本 skill

触发后解析：
1. **总结类型**：`bugfix` / `feat`（可从措辞推断；不明时用 AskQuestion）
2. **总结对象**：刚刚完成的事项 / 当前对话 / 用户点名的特定需求或 bug
3. **保存地址**（见「保存地址」）

---

## 工作流程

```
Auto Conclusion Progress:
- [ ] Phase 1: 解析意图与范围
- [ ] Phase 2: 历史对话检索与筛选
- [ ] Phase 3: 结构化整理
- [ ] Phase 4: 确认保存地址
- [ ] Phase 5: 写入文档并回显
```

---

### Phase 1: 解析意图与范围

1. 判定类型：
   - 含「bug」「修复」「缺陷」等 → `bugfix`
   - 含「需求」「功能」「feat」「开发」等 → `feat`
   - 同时涉及两者 → 以用户主述为准；可用 AskQuestion 确认
2. 确定主题 slug（`xxx`）：一句话主题的英文或拼音短名，连字符分隔，去特殊字符，≤40 字符
3. 明确检索范围提示词（需求名、模块、报错、接口等）

**输出**：类型、主题 slug、检索关键词。

---

### Phase 2: 历史对话检索与筛选

一次需求/修复常跨多轮对话，须主动检索，不只看上一轮。

1. **检索来源**（按可用性）：
   - 当前对话全文
   - 当前项目的 agent-transcripts（按关键词、模块、报错、需求名搜索）
   - 用户点名的历史对话 / transcript
2. **收容标准**（满足任一则纳入）：
   - 与目标需求/bug 直接相关的分析、方案、改动、验证
   - 开发过程中遇到的相关问题（尝试过的方案、失败原因、最终解法）
   - 相关的代码变更说明、配置调整、测试结果
3. **排除标准**（不收容）：
   - 对话中途插入的无关任务、闲聊、其他需求/bug
   - 与当前主题无因果关系的讨论
4. **时间线整理**：按发生顺序梳理「尝试 → 问题 → 解决」，避免堆砌原文

**输出**：相关对话摘要时间线；已排除的无关段落说明（一两句即可）。

---

### Phase 3: 结构化整理

按下列模板组织内容（缺项写「无」或「未涉及」，禁止编造）：

```markdown
# {feat|bugfix}-{slug}

> 归档时间：{YYYY-MM-DD HH:mm:ss}
> 类型：需求开发 | Bug 修复
> 主题：{一句话主题}

## 1. 需求 / 问题是什么
[要解决什么；背景、目标、验收点]

## 2. 解决思路
[总体思路与关键决策；为何选此方案]

## 3. 解决过程中遇到的问题
[按条列出；含尝试过但未采纳的方案]

## 4. 问题的解决方案
[每个问题对应如何解决；指向关键改动]

## 5. 整体解决流程
[从发现问题/接到需求到完成的阶段顺序]

## 6. 开发流程
[实际开发步骤：改了哪些模块、如何验证、是否环境刷新等]

## 7. 问题点与经验总结
[可复用的经验、避坑点、下次可直接参考的结论]

## 8. 变更说明（提测用）
[给测试的变更清单：功能点、影响范围、回归建议、已知限制]

## 9. Git Commit Message
```
<type>(<scope>): <English description>
<type>(<scope>): <中文描述>
```
```

**Git Commit 规范**：
- conventional commits：`feat` 或 `fix`（与类型一致）
- `scope` 为主要模块
- **中英双语各一行**，可直接用于 `git commit`
- 每条 50 字以内

**输出**：完整 Markdown 正文（先在对话中展示，再写入文件）。

---

### Phase 4: 确认保存地址

按优先级解析保存目录：

| 优先级 | 来源 | 行为 |
|--------|------|------|
| 1 | 用户在本轮对话中明确给出路径 | 使用该路径；可在对话中复述确认 |
| 2 | skill 安装配置中的默认路径（见「安装时配置」） | AskQuestion 确认是否用该配置路径 |
| 3 | 均未指定 | 使用系统「下载」文件夹为默认目录，**必须**用 AskQuestion 请用户确认 |

**默认下载目录（跨平台，禁止写死绝对路径）**：

| 系统 | 解析方式 |
|------|----------|
| macOS / Linux | `$HOME/Downloads`（若目录不存在，回退 `$HOME`） |
| Windows | `%USERPROFILE%\Downloads` 或 `$env:USERPROFILE\Downloads`（若不存在，回退 `%USERPROFILE%`） |

解析命令示例：

```bash
# macOS / Linux
DEFAULT_DIR="${HOME}/Downloads"
[ -d "$DEFAULT_DIR" ] || DEFAULT_DIR="$HOME"

# Windows PowerShell
$DEFAULT_DIR = Join-Path $env:USERPROFILE "Downloads"
if (-not (Test-Path $DEFAULT_DIR)) { $DEFAULT_DIR = $env:USERPROFILE }
```

**AskQuestion 选项**（未指定路径或需确认默认路径时）：

| 选项 | 行为 |
|------|------|
| 确认保存到默认/配置路径 | 写入已解析路径 |
| 自定义路径 | 请用户提供目录后写入 |
| 取消保存 | 仅在对话中展示总结，不落盘 |

Codex / 无弹框时：用文本编号选项；用户回复前不写入文件。

---

### Phase 5: 写入文档并回显

1. **文件名**（verbatim 规则）：
   - 需求：`feat-{slug}-{timestamp}.md`
   - Bug：`bugfix-{slug}-{timestamp}.md`
   - `timestamp`：`YYYYMMDD-HHmmss`（本地时区）
2. 目录不存在则创建
3. 写入完整 Markdown
4. 对话中告知**完整绝对路径**，并再次给出 Git Commit Message 便于复制

**输出**：文件路径 + 是否写入成功。

---

## 安装时配置（默认保存地址）

**安装或首次同步到全局 skills 时**，Agent 须用 AskQuestion 询问默认文档保存目录：

| 选项 | 行为 |
|------|------|
| 使用系统下载文件夹 | 将默认目录记为跨平台 Downloads 解析结果 |
| 自定义目录 | 用户提供路径并写入配置 |
| 跳过（每次再问） | 不写配置；每次 Phase 4 走确认流程 |

配置写入本 skill 目录下的 `config.json`：

```json
{
  "defaultSaveDir": "<user-chosen-or-downloads-path>",
  "configuredAt": "<ISO-8601>"
}
```

规则：
- 有 `config.json` 且 `defaultSaveDir` 有效 → Phase 4 优先提示该路径
- 无配置或路径失效 → 回退 Downloads，并 AskQuestion 确认
- **不要**把机器特定绝对路径写进 `SKILL.md` 正文；只写入 `config.json`

---

## 注意事项

- **只收容相关内容**：多轮、跨对话检索；无关穿插一律排除
- **禁止编造**：对话中未出现的方案、问题、变更不得写入
- **先展示后落盘**：Phase 3 正文须在对话中可见，再经 Phase 4 确认后写入
- **AskQuestion 优先**：路径确认、类型歧义用对话内弹框，不另开纯追问
- **命名 verbatim**：`feat-xxx-timestamp` / `bugfix-xxx-timestamp`（实现为 `feat-{slug}-{timestamp}.md`）
- **跨平台路径**：用 `$HOME` / `%USERPROFILE%` 解析 Downloads，禁止写死 `/Users/...` 或 `C:\Users\...`
- **中英 commit**：提测与 commit 章节必填，便于直接使用
