# expo-followup-pipeline

**[English](#english) | [中文](#中文)**

---

## English

A WorkBuddy skill that turns a trade show or business trip into a **tracked follow-up pipeline** — instead of a pile of conversations that quietly evaporate.

### The problem

After a trip, your leads are scattered across five places: the event app's chat history, voice memos on your phone, your inbox, your calendar, and the deal ledger.
Consolidating all of it by hand takes half a day, and you still miss things.

This skill hardcodes the **collect → structure → deliver** chain so it runs the same way every time, and lands the result somewhere that will actually remind you.

### Six steps

1. **Collect** — pull from five sources in priority order (trip calendar > event app > voice notes > email > ledger)
2. **Cross-check** — reconcile what contacts *claimed* against what *actually happened*; when the two conflict, the facts win
3. **Structure** — emit a follow-up table with a consistent schema
4. **Deliver to Lark** — create tasks, attach calendar reminders
5. **Write back** — fill in the *on-site notes* and *post-meeting action points* columns in the ledger
6. **Log** — record what was produced in the work log

> Step 1 deliberately starts with the **trip calendar**, not the pre-trip schedule. Meetings that actually happened — including the short events you typed into your phone on site ("Nsano meeting", "Mr. Luo, XX") — are closer to the truth than any plan made beforehand.

### Triggers

- "Just got back from the trip, help me organize the follow-ups"
- "Turn what I talked about at the expo into todos"
- "Expo debrief" / "follow-up table"
- "Turn these meeting notes into tasks"

### Install

Drop it into your WorkBuddy skills directory:

```bash
# User level (available across all projects)
cp -R . ~/.workbuddy/skills/expo-followup-pipeline/

# Or project level (shared with your team)
cp -R . <project-root>/.workbuddy/skills/expo-followup-pipeline/
```

### Porting to another platform

Section 0 of `SKILL.md` holds a single **required-variables table** (`WORKSPACE_ROOT`, `LEDGER_DOC`, `MAIL_USER`, `TODOOL`, …).
The body of the skill references everything as `<placeholders>` and contains **no machine-specific absolute paths**.

Moving to a different stack means editing that one table — swap `TODOOL` for whatever your platform uses for tasks and calendar. The steps and decision logic stay untouched.

### Layout

```
.
├── SKILL.md                          # Main workflow and decision logic
├── assets/
│   └── followup-table-template.md    # Follow-up table template
└── references/
    ├── source-collection.md          # How to pull from each of the five sources
    └── pitfalls.md                   # Known gotchas
```

---

## 中文

展会 / 差旅归来后的**线索回收与跟进落地** WorkBuddy Skill。

把一场"聊完就散"的展会或差旅，收敛成一张能自己盯的跟进表，并真正落进飞书任务 + 日历提醒 + 台账回填。

### 它解决什么

出差回来，线索散在五个地方：展会 App 聊天记录、手机语音纪要、邮箱、日历、作战台账。
人工整理一遍要半天，还容易漏。这个 skill 把回收 → 结构化 → 落地这条链路固化下来。

### 六步主流程

1. **素材回收** — 五个来源按优先级拉取（出差期间日历 > 展会 App > 语音纪要 > 邮箱 > 台账）
2. **交叉核对** — 用真实发生的会面校准客户自报信息，口径冲突时以事实为准
3. **结构化** — 输出统一字段的跟进待办表
4. **落地飞书** — 建任务、挂日历提醒
5. **台账回填** — 写回「现场跟进情况」「会后 Action Points」两栏
6. **复盘留痕** — 工作日志记录本次产出

> 第 1 步刻意从**出差期间日历**开始，而不是展前排期。真实发生的会面——包括你在现场用手机手动补记的短标题事件（"Nsano meeting""XX 罗总"）——比任何事先排的计划都更接近事实。

### 触发场景

- 「出差回来了整理一下跟进」
- 「把展会上聊的整理成待办」
- 「展会复盘」「follow-up table」
- 「把会议纪要做成任务」

### 安装

放到 WorkBuddy 的 skills 目录即可：

```bash
# 用户级（所有项目可用）
cp -R . ~/.workbuddy/skills/expo-followup-pipeline/

# 或项目级（团队共享）
cp -R . <项目根>/.workbuddy/skills/expo-followup-pipeline/
```

### 迁移到新平台

`SKILL.md` 第 0 节有一张**必填变量表**（`WORKSPACE_ROOT` / `LEDGER_DOC` / `MAIL_USER` / `TODOOL` 等），
正文一律用 `<占位符>` 引用，不含任何本机绝对路径。

换平台时只改这张表，把 `TODOOL` 替换成目标平台的对应能力即可，步骤与判定逻辑保持不变。

### 目录结构

```
.
├── SKILL.md                          # 主流程与判定逻辑
├── assets/
│   └── followup-table-template.md    # 跟进待办表模板
└── references/
    ├── source-collection.md          # 五个素材源的采集方法
    └── pitfalls.md                   # 踩坑记录
```
