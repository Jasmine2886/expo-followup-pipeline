# expo-followup-pipeline

**[English](#english) | [中文](#中文)**

Turn a conference, trade show, or business trip into a **tracked follow-up pipeline** — instead of a pile of conversations that quietly evaporate.

Works with **WorkBuddy, Claude Code, Codex CLI, Gemini CLI, Cursor, and GitHub Copilot**.

---

## English

### The problem

You get back from a trip and your leads are scattered across five places: the event app's chat history, voice memos on your phone, your inbox, your calendar, and your CRM.
Consolidating it by hand takes half a day, and you still miss things.

This skill hardcodes the **collect → reconcile → structure → deliver** chain so it runs the same way every time, and lands the result somewhere that will actually remind you.

### Six steps

1. **Collect** — pull from five sources in priority order (trip calendar > voice transcripts > event app > email > CRM)
2. **Structure** — one card per contact, with an explicit status per contact
3. **Emit** — a Markdown follow-up table sorted by *due date*, not by company name
4. **Land tasks** — create real tasks with real due dates
5. **Land reminders** — calendar reminders (never meetings, never with attendees)
6. **Write back** — fill the on-site notes and action-points fields in your CRM, then log the run

> Step 1 deliberately starts with the **trip calendar**, not the pre-trip plan. Meetings that actually happened — including the short events you typed into your phone on site — beat any schedule made in advance.

### The part that earns its keep

The status field. `Not met · need to reach out` (expired invites, unanswered last-minute requests) goes in **its own section**, so you don't quietly count missed contacts as active pipeline.

### Triggers

- "Just got back from the trip, help me organize the follow-ups"
- "Turn what I discussed at the conference into todos"
- "Expo debrief" / "conference recap" / "follow-up table"
- "Turn these meeting notes into tasks"
- "Who should I chase this week?"

### Install

```bash
git clone https://github.com/Jasmine2886/expo-followup-pipeline.git
```

Then copy the folder into your agent's skills directory:

| Agent | Personal (all projects) | Project-scoped |
| --- | --- | --- |
| **WorkBuddy** | `~/.workbuddy/skills/` | `.workbuddy/skills/` |
| **Claude Code** | `~/.claude/skills/` | `.claude/skills/` |
| **Codex CLI** | `~/.codex/skills/` | `.codex/skills/` |
| **Gemini CLI** | `~/.gemini/skills/` | `.gemini/skills/` |
| **Cursor** | — (project only) | `.cursor/skills/` |
| **GitHub Copilot** | `~/.config/github-copilot/skills/` | `.github/skills/` |

```bash
# Example: Claude Code, personal
cp -R expo-followup-pipeline ~/.claude/skills/

# Example: Codex CLI, personal
cp -R expo-followup-pipeline ~/.codex/skills/

# Example: Cursor (project-scoped only)
cp -R expo-followup-pipeline .cursor/skills/
```

Restart your agent so it picks up the new metadata.

### Configure

Section 0 of `SKILL.md` holds a single **required-variables table** (`WORKSPACE_ROOT`, `LEDGER_DOC`, `MAIL_USER`, `EVENT_APP_URL`, `TZ_BASE`, `TODOOL`, …).
The body references everything as `<PLACEHOLDER>` and contains **no machine-specific absolute paths**.

Fill that one table for your setup — swap `TODOOL` for whatever your stack uses for tasks and calendar. The steps and decision logic stay untouched.

### Cross-agent notes

- Frontmatter uses only `name` + `description`, the two fields every agent parses, so it loads everywhere
- Body is plain Markdown — numbered steps, tables, code blocks. No agent-specific syntax
- The skill states its own constraints in prose (no auto-sending mail, no adding attendees) rather than relying on `allowed-tools`, since tool-permission fields are agent-specific
- Codex users: it triggers most reliably with explicit phrasing — "follow-up table" works better than a vague "help me recap"

### Layout

```
.
├── SKILL.md                          # Main workflow and decision logic
├── assets/
│   └── followup-table-template.md    # Follow-up table template
└── references/
    ├── source-collection.md          # How to pull from each of the five sources
    └── pitfalls.md                   # Browser, login, captcha, timezone traps
```

---

## 中文

展会 / 差旅 / 行业会议归来后的**线索回收与跟进落地**。适用于 **WorkBuddy、Claude Code、Codex CLI、Gemini CLI、Cursor、GitHub Copilot**。

把一场"聊完就散"的行程，收敛成一张按截止日排序的跟进表，并真正落进任务工具 + 日历提醒 + CRM 回填。

### 六步主流程

1. **素材回收** — 五个来源按优先级（出差日历 > 语音纪要 > 展会 App > 邮箱 > CRM）
2. **结构化** — 每人一张卡，状态必须明确
3. **产出** — Markdown 跟进表，按**截止日**排序
4. **落地任务** — 建真实待办，带截止日
5. **落地提醒** — 日历提醒（零参会人，绝不建会议）
6. **回填 CRM** — 填「现场跟进情况」「会后 Action Points」，并写工作日志

### 最值钱的一点

状态字段。`未见到 · 需补联系`（过期邀请、未回的临时约见）**单独成段**，不混进有效商机里自欺欺人。

### 触发场景

「出差回来了整理一下跟进」「把展会上聊的整理成待办」「展会复盘」「follow-up table」「把会议纪要做成任务」「这周该催谁」

### 安装

```bash
git clone https://github.com/Jasmine2886/expo-followup-pipeline.git
```

复制到对应 agent 的 skills 目录：

| Agent | 用户级 | 项目级 |
| --- | --- | --- |
| **WorkBuddy** | `~/.workbuddy/skills/` | `.workbuddy/skills/` |
| **Claude Code** | `~/.claude/skills/` | `.claude/skills/` |
| **Codex CLI** | `~/.codex/skills/` | `.codex/skills/` |
| **Gemini CLI** | `~/.gemini/skills/` | `.gemini/skills/` |
| **Cursor** | —（仅项目级） | `.cursor/skills/` |
| **GitHub Copilot** | `~/.config/github-copilot/skills/` | `.github/skills/` |

复制后重启 agent 让它加载新 skill。

### 配置

`SKILL.md` 第 0 节是**唯一需要改的变量表**（`WORKSPACE_ROOT` / `LEDGER_DOC` / `MAIL_USER` / `EVENT_APP_URL` / `TZ_BASE` / `TODOOL` 等）。
正文全部用 `<占位符>`，不含任何本机绝对路径。把 `TODOOL` 换成你平台的任务/日历能力即可，流程与判定逻辑不用动。

### 跨 agent 说明

- frontmatter 只保留 `name` + `description`（所有 agent 都解析的字段）
- 正文是纯 Markdown，无平台专属语法
- 红线（不自动发邮件、不加参会人）写在正文里，不依赖 `allowed-tools` 这类平台专属字段
- Codex 用户：描述写得越具体越容易触发，「follow-up table」比「帮我复盘」好使

### 目录结构

```
.
├── SKILL.md                          # 主流程与判定逻辑
├── assets/
│   └── followup-table-template.md    # 跟进表模板
└── references/
    ├── source-collection.md          # 五个素材源的采集细节
    └── pitfalls.md                   # 浏览器、登录、验证码、时区坑位
```
