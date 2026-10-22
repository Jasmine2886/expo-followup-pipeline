---
name: expo-followup-pipeline
version: 2.0.0
agent_created: true
description: "Turn a conference, trade show, or business trip into a tracked follow-up pipeline. Collects scattered lead evidence from five sources (trip calendar, AI voice transcripts, event app messages, email, CRM/ledger) → reconciles what actually happened against what was planned → structures it into a prioritized follow-up table → lands it into real tasks + calendar reminders + CRM write-back. Use when the user says: 'just got back from a trip, organize the follow-ups', 'turn what I discussed at the expo into todos', 'expo debrief', 'conference recap', 'follow-up table', 'post-meeting action points', 'turn meeting notes into tasks', 'who should I chase this week'. Also triggers on: 展会复盘、出差归来、跟进表、跟进待办、Action Points、线索回收、会议纪要整理. NOT for: pre-event outreach scheduling, or drafting a single email."
---

# Post-Event Lead Recovery & Follow-Up Pipeline

Turn a trip that "ended in conversation" into **one follow-up table you can actually work from, plus tasks and reminders that will genuinely ping you**.

**Use for:** debriefing 1–7 days after a conference, trade show, or multi-meeting business trip. Especially when travel conditions (laptop closed, bad network) meant you could only capture voice notes on your phone.
**Not for:** pre-event outreach scheduling (different workflow), or writing a single email.

---

## 0. Required variables — the ONLY thing you change when porting

This skill contains **no machine-specific absolute paths**. Everything environment-dependent is collected here; the body references it as `<PLACEHOLDER>`.

| Variable | Meaning | How to fill |
| --- | --- | --- |
| `WORKSPACE_ROOT` | Workspace root (base for all relative paths) | e.g. `./` |
| `OUTPUT_DIR` | Output directory | `<WORKSPACE_ROOT>/output/<project>/` |
| `MEMORY_DIR` | Work-log directory | `<WORKSPACE_ROOT>/.workbuddy/memory/` (create if missing) |
| `LEDGER_DOC` | Your deal ledger / CRM / spreadsheet identifier | Any system: a doc ID, a Notion database, a Google Sheet, an Airtable base. Must expose two fields per contact: *on-site notes* and *post-meeting action points* |
| `CALENDAR_ID` | Target calendar | Default `primary` — write to the **personal default calendar**, never create a dedicated one |
| `MAIL_USER` | Email account | Used for event-app login codes and searching meeting mail |
| `EVENT_APP_URL` | Event app entry point | e.g. `https://app.<event-domain>/event/<slug>` |
| `VOICE_APP_URL` / `VOICE_SESSION_TITLE` | Transcription service + target session | Any AI notetaker or chat-with-transcripts service |
| `TZ_BASE` | Base timezone for all output | **Your home timezone** — set it explicitly, don't assume |
| `TZ_LOCAL` | Timezone where the event happened | e.g. Dubai `UTC+04` = base −4h |
| `TODOOL` | Task + calendar delivery tool | Whatever your stack uses — see §4/§5 for the capability contract |

> **Porting note:** the body below describes *capabilities* plus a reference implementation. When moving to a new platform, keep the steps and decision logic, and swap the `TODOOL` commands for your platform's equivalents. Never hardcode a path to make one machine work.

---

## 1. The six steps

### Step 1 · Collect (five sources, in this priority order)

| Priority | Source | What it gives you | Key move |
| --- | --- | --- | --- |
| ★★★ | **Calendar covering the trip** (incl. events you typed in on site) | **Which meetings actually happened** — usually closer to truth than any pre-trip schedule | Pull the full date range, sort by time. Short manually-created titles ("Nsano meeting", "dinner w/ X") *are* your real meeting list |
| ★★★ | **AI voice transcripts** (phone recordings / notetaker) | What was actually said and promised | See §2. If login fails, fall back to "user pastes the raw text" |
| ★★ | **Event app messages** | What was discussed online, any no-shows or reschedules | See §3. Produces the "in-app conversation log" |
| ★★ | **Email** | Confirmations, cancellations, inbound follow-ups | Search event name + key contact surnames |
| ★ | **Ledger / CRM** (`LEDGER_DOC`) | Pre-event intent and priority (P1/P2/P3) for alignment | Check whether the two fields are empty — empty means on-site never got filled in, which is exactly what this run fixes |

**Act first, ask later.** Sweep all five sources yourself and assemble "who did I actually meet". *Then* ask about the 1–2 things still ambiguous (e.g. "Bitpace and OpenPayd both list booth B50 — same booth or a data error?").

### Step 2 · Structure: one card per contact

Fixed fields per contact. Write `TBD` for anything missing — never invent:

```
Company / Name / Role / Contact
What we discussed (their ask + what we offered)
Next action (verb-first, one action per line)
Due date (use the agreed date; if none, suggest by priority: P1=7d, P2=14d, P3=30d)
Priority P1/P2/P3 (reuse the ledger's existing tiers — don't invent a new scheme)
Status: Met·to advance / Met·waiting on them / Not met·need to reach out / Expired·drop
```

**The status field is where the value is.** `Not met·need to reach out` (expired invitations, unanswered last-minute requests) must sit in **its own section** — folding it into the active pipeline is self-deception.

### Step 3 · Emit the follow-up table (local Markdown)

Write to `<OUTPUT_DIR>/followup-<event>-<date>.md`. Template: `assets/followup-table-template.md`.

Requirements:
- You can see "who to chase today" at a glance — sort by **due date**, not company name
- Every line is individually copy-pasteable (verb-first + concrete deliverable; no "stay in touch" filler)
- Always include a **"not met" list** and a **"targets vs. actual"** section

### Step 4 · Land tasks

```
# Preferred: create one list + batch-create into it (tasks stay grouped)
<TASK_CLI> tasklist-create --name "<Event> follow-up" --data '<JSON array>'

# Or one at a time
<TASK_CLI> task-create --summary "<verb-first sentence>" --description "<context + action + figures>" --due "YYYY-MM-DD"
```

- Title the **action**, not the company ("Send Onlypay acquiring proposal to Leonardo", not "Checkout.com")
- `--due` comes from Step 2; where no date was agreed, use the priority-based suggestion and mark it `suggested`
- After creating, **re-read** the task list to verify — don't trust an `ok` response

### Step 5 · Land calendar reminders (not meetings)

```
<CAL_CLI> calendar-create --summary "🔔 Follow up: <Company> · <action>" --start "..." --end "..." --description @<file>
```

- Reminder events have **zero attendees** — hard rule, never invite anyone, including the counterparty
- Write only to the user's own default calendar
- After batch creation, **always** verify with an agenda read-back

### Step 6 · Write back to the ledger + wrap up

- Fill the *on-site notes* and *post-meeting action points* fields using a **targeted string replace, never a whole-block replace**
- After writing, read back and confirm you only changed the intended cells
- Append a work log to `<MEMORY_DIR>/YYYY-MM-DD.md`: what ran, what got confirmed, what's still open

---

## 2. Transcription service: login and read (the step most likely to stall)

Follow the order, don't skip:

1. Open `VOICE_APP_URL`, check whether you're already logged in (does the URL sit on a login page?)
2. **Avoid third-party one-tap SSO** — it can bind to **a different account than yours** (this cost one run 40 minutes). Prefer phone-code or username/password
3. Phone login usually raises an **image-selection captcha** ("click the smallest blue cone"): screenshot → interpret → compute coordinates → submit. Wrong answers rotate the image; cap at 3 attempts
4. Getting the code: ask the user for their phone number → trigger send → enter the 6-digit code
5. Once in, locate the session by `VOICE_SESSION_TITLE` and read it in full

**Three hard rules:**
- The automation browser uses an **isolated profile** — the user being logged into their own Chrome means nothing here. Login must happen inside this session
- Sandboxes often **cannot screenshot** (no screen recording permission), so you can't confirm the window is visible → **ask the user directly**, don't guess
- **The fallback always beats grinding on login.** Let the user paste the raw transcript and the run continues. After 3 failed login attempts, switch

---

## 3. Event app message scraping (Swapcard-style platforms)

1. Open `EVENT_APP_URL` → a **Client Challenge image captcha** (blocks headless too): screenshot → read → fill → submit
2. Accept the cookie banner → Log in → **one-time email code** from `MAIL_USER`
3. Don't try to extract browser cookies directly (keychain reads are usually denied — wasted time)
4. Once in Messages, grab all conversation permalinks in one pass — more reliable than clicking through a list (element refs shift on interaction):

```js
JSON.stringify(Array.from(document.querySelectorAll('a[href]'))
  .filter(a => /messages\/channel/.test(a.href))
  .map(a => [a.textContent.trim().slice(0,25), a.getAttribute('href')]))
```

5. Open each permalink and read the body: `document.querySelector('main').innerText`
   - The leading block is the conversation list; **content starts at the first "Month D, YYYY" line** and ends at `Scroll to bottom`
6. Timestamps render in the **browser's local timezone** → convert to `TZ_BASE` and note the conversion in the output
7. Append the raw chat + a summary into the **notes field of the matching calendar event** (`calendar-update --description @file`), under a heading like "💬 In-app conversation log"
8. Conversations with no matching calendar event are new leads — list them separately, don't drop them

---

## 4. Quality red lines

1. **Separate claimed from verified.** A contact's self-reported figures ("we process $240B a year") are not established fact. Label the source in your output
2. **Never take external actions for the user.** Don't send mail automatically, don't add attendees to events, confirm before touching shared documents
3. **Always carry a timezone.** Output in `TZ_BASE`; where local time matters, state the conversion (e.g. "Dubai = home −4h")
4. **Don't fabricate.** Transcripts have errors and gaps — mark them `TBD` rather than writing a polished fiction
5. **Expired means expired.** Unaccepted invites and messages that went unanswered belong in `Not met·need to reach out`, not in the active pipeline

---

## 5. Gotcha quick-reference

Full detail in `references/pitfalls.md` (browser / captcha / coordinates / timezone) and `references/source-collection.md` (per-source scraping). The ones that bite most often:

- After batch-creating calendar events, **always verify** — unsupported params can fail the whole call while still returning something that looks like success
- Notification/reminder params usually belong to **update**, not **create**
- Deleting events typically requires an explicit confirm flag; without it the call returns ok and deletes nothing
- Whole-block document replacement changes block IDs → reusing an old ID always fails. Use targeted string replace for text edits
- Browser coordinates: when screenshot size equals viewport CSS pixels (dpr=1), it's **1:1** with `getBoundingClientRect()`. Confirm before clicking; don't click based on how the scaled-down preview looks
- Never judge success from `| head`-truncated output — write to disk, then parse

---

## 6. Deliverables

Every run should produce:

1. `<OUTPUT_DIR>/followup-<event>-<date>.md` (the main working table)
2. A task list (grouped, with due dates)
3. Calendar reminder events (milestones for key accounts)
4. The two ledger fields filled + verified
5. One work-log entry

## Bundled files

- `references/source-collection.md` — per-source scraping detail and merge rules
- `references/pitfalls.md` — browser, login, captcha, coordinate, and timezone traps
- `assets/followup-table-template.md` — copy-paste follow-up table template
