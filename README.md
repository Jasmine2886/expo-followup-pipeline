# expo-followup-pipeline

展会 / 差旅归来后的**线索回收与跟进落地** WorkBuddy Skill。

把一场"聊完就散"的展会或差旅，收敛成一张能自己盯的跟进表，并真正落进飞书任务 + 日历提醒 + 台账回填。

## 它解决什么

出差回来，线索散在五个地方：展会 App 聊天记录、手机语音纪要、邮箱、日历、作战台账。
人工整理一遍要半天，还容易漏。这个 skill 把回收 → 结构化 → 落地这条链路固化下来。

## 六步主流程

1. **素材回收** — 五个来源按优先级拉取（出差期间日历 > 展会 App > 语音纪要 > 邮箱 > 台账）
2. **交叉核对** — 用真实发生的会面校准客户自报信息，口径冲突时以事实为准
3. **结构化** — 输出统一字段的跟进待办表
4. **落地飞书** — 建任务、挂日历提醒
5. **台账回填** — 写回「现场跟进情况」「会后 Action Points」两栏
6. **复盘留痕** — 工作日志记录本次产出

## 触发场景

- 「出差回来了整理一下跟进」
- 「把展会上聊的整理成待办」
- 「展会复盘」「follow-up table」
- 「把会议纪要做成任务」

## 安装

放到 WorkBuddy 的 skills 目录即可：

```bash
# 用户级（所有项目可用）
cp -R . ~/.workbuddy/skills/expo-followup-pipeline/

# 或项目级（团队共享）
cp -R . <项目根>/.workbuddy/skills/expo-followup-pipeline/
```

## 迁移到新平台

`SKILL.md` 第 0 节有一张**必填变量表**（`WORKSPACE_ROOT` / `LEDGER_DOC` / `MAIL_USER` / `TODOOL` 等），
正文一律用 `<占位符>` 引用，不含任何本机绝对路径。

换平台时只改这张表，把 `TODOOL` 替换成目标平台的对应能力即可，步骤与判定逻辑保持不变。

## 目录结构

```
.
├── SKILL.md                          # 主流程与判定逻辑
├── assets/
│   └── followup-table-template.md    # 跟进待办表模板
└── references/
    ├── source-collection.md          # 五个素材源的采集方法
    └── pitfalls.md                   # 踩坑记录
```
