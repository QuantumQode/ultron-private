# Ultron: personal life-consolidation assistant

## Context
You want one assistant that knows your email, notes, calendar and goals, talks with you through the day, and plans each next day around what matters to you right now. The project folder (`Desktop\Ultron`). Ultron aims to converse with you, hold you accountable, ask questions, and push you to act.


**Verdict: feasible.** Every piece has a stable API. A usable core (phases 1–5) is about 2–3 weekends of work, and the full feature set about 4–6 weekends.

**Decisions so far:** Yahoo mail (read and summarize only) · **Notion** as the one place you write notes (an Inbox page), read Ultron's organised view (a Board page) and see your plan (a Schedule page, instead of Apple Calendar) · Telegram for briefs, planning and conversation · small cloud VPS · Python · plans are approved before anything is written · nightly review + morning brief + weekly review · switchable **focus modes** · a **Reflect** space for talking about your thoughts and life, with suggestions to personalise Ultron · all the suggested extras and safeguards.

### Why Notion and not Apple Notes
Apple Notes has no public API, there's no iCloud Notes web API, and the only reliable automation is AppleScript on an always-on Mac. You're on Windows, so that's ruled out. Notion has a good official API, works on phone, desktop and web, and lets you see and edit everything Ultron knows at any time. One-time migration: export your Apple Notes (or copy them across) into the Notion Inbox page, and Ultron will sort them on its first sync.

## Architecture

```
 Telegram (phone/desktop) ◄──── chat, buttons, briefs ────┐
                                                          │
 ┌──────────────── Ultron service (Python, VPS) ──────────┴─┐
 │ bot/        python-telegram-bot v22 (async)              │
 │ scheduler   PTB JobQueue: polls, nightly, morning, weekly│
 │ brain/      Claude API + tool use; focus-mode aware      │
 │ state       SQLite: sync cursors, email digest, plan log │
 └──────┬───────────────────┬────────────────────┬──────────┘
        │ IMAP (read-only)                       │ Notion API (read/write)
        ▼                                        ▼
   Yahoo Mail                              Notion: three pages
                                           ├─ 📥 Inbox    (you write, messy is fine)
                                           ├─ 🗂 Board    (what to do: Ultron organises;
                                           │               you read and tick things off)
                                           └─ 📅 Schedule (when: today + tomorrow in time
                                                           blocks, rest of the week listed)
```

**How data is split:**
- **Notion is where you write and read.** You jot anything into the Inbox. Ultron sorts it onto the Board, which is the readable view of your tasks, goals, schedule and what Ultron knows about you.
- **SQLite holds the structure behind the Board** that you don't need to see: time estimates, due dates, "times moved" counts for accountability, completion history, sync bookmarks, chat history, processed-email IDs and pending plans. Each Board item is linked to its record.

### Notion layout (Ultron creates the pages on first run, inside a page you share with the integration)

**📥 Inbox** (yours): one page. Write anything, any format, whenever you like; you never need to talk to Ultron while jotting.
- **Hourly** (and right before each brief), Ultron reads what's new, sorts it onto the Board, and moves your original text into a collapsed **✓ Processed** section at the bottom of the Inbox. Your exact words are kept, but the top of the Inbox stays clean.
- Notion's API can't move text, so "move" means copy into ✓ Processed, then remove the original. Anything removed sits in Notion's trash for 30 days.

**🗂 Board** (Ultron keeps it tidy). One page of collapsible sections. A section appears only once it has something in it:

| Section | What's in it |
|---|---|
| 🚀 **AI career** (a roadmap) | Always open, above ✅ To-do. A status box (goal, current phase and its checkpoint, steps done, this week's hours against plan, what's behind), then each stage under way as a heading with its steps as tick boxes ("Week 1 — Set up · by Sun 1 Nov"). Later phases are folded headings with their steps inside. See § 16. |
| ✅ **To-do** | Checkboxes **grouped under their project or person first** (e.g. "Copperline app sales", "George (tennis)"), then loose tasks by life area (Health, Wealth, Career, Relationships, Learning, Admin, Fun). Due dates inline ("☐ Book dentist · Tue"). Tick them here. |
| 🔁 **Habits** | Habits with streaks, ticked off in the evening review |
| 🎯 **Goals** | Active goals by horizon (year/quarter/month/week), with progress |
| 💡 **Ideas** | Business ideas and someday/maybe plans |
| 🧭 **Principles & notes** | Your mantras and reminders (in your own words), then frameworks and things you've learned (e.g. NSFAG). Ultron may quote one in a brief now and then. |
| 📌 **My schedule** | Work days and hours, workouts, sleep, immovable commitments (from onboarding, § 0) |
| 🧠 **About me & focus** | Facts Ultron has learned, the current focus mode, coaching style. Edit or delete a line to correct it or make Ultron forget. |
| 📓 **Log & journal** | A folded heading per week (Mon–Sun), newest on top: the to-dos done that week (ticked, with the day), those dropped (struck through), then the week's day reviews (summary, done/partial/skipped, your reflection), Reflect session summaries and progress notes |

**Your edits on the Board win.** Before rewriting a section, Ultron reads what's there now: ticked boxes, deleted lines, your own additions. It applies those first, so it never undoes something you changed.

**How the Board behaves:**
- **✅ To-do is always open.** The other sections are collapsible headings, closed until you click.
- **Ticked to-dos stay ticked for the rest of the day,** so you can see what you got done. The evening review counts them in the day's 📓 Log entry. On the first sync of the next day they move into their week in 📓 Log, where they stay as a record. Unticking one there reopens it; deleting one there only hides it from the Log (decided 25 Oct). Old email to-dos cleared on 20–21 Oct are hidden.
- **Lines you type directly on the Board are sorted like Inbox notes, but stay where you typed them** (decided 25 Oct): a typed tick box becomes the to-do itself, and a plain line is replaced by a to-do in the same spot. If a line seems to belong under another heading, it stays put with a question ("Under X instead?"). Lines indented under a to-do become its notes. Lines typed in 🚀 roadmap, ☑️ Today's habits or 📓 Log are still filed into their place.
- **Ultron updates the Board in place** (it adds, edits or removes only the changed lines) rather than rebuilding it, so expanded sections stay as you left them.

**📅 Schedule** (your plan, instead of Apple Calendar). The Board says *what* to do; the Schedule says *when*:

| Section | What's in it |
|---|---|
| **Today: Tue 20 Oct** (always open) | Time blocks as checkboxes, e.g. "☐ 09:00–11:00 Copperline: call tradies". Fixed commitments from 📌 My schedule are shown as plain lines ("🔒 13:00–17:00 Work"), so you see the whole day. |
| **Tomorrow: Wed 21 Oct** (always open) | The approved plan for tomorrow, timed the same way |
| **Rest of the week** | For each remaining day, what's planned, without times, so it stays flexible and isn't over-planned |

- **It's two-way, like the Board:** ticking a block ticks the matching Board to-do. Deleting or rewording a block is noticed and the plan adapts.
- **Each evening the Schedule rolls forward:** today's results go to 📓 Log, tomorrow becomes today, and the newly approved plan fills in.
- **No phone alerts** come from Notion itself. Optional Telegram nudges (§ 12) can ping you when a block starts.
- **Apple Calendar is untouched.** Fixed commitments come from onboarding (📌 My schedule) and your notes. If appointments that only live in Apple Calendar turn out to matter, a **read-only** iCloud connection can be added later.

## Features

### 0. First-run onboarding: your current life schedule
The first time Ultron runs, it starts a short conversation in Telegram to learn the things it must plan around:
- **Work:** days and hours, shift patterns, commute
- **Workouts:** current days and times
- **Sleep:** usual wake and sleep times
- **Immovable commitments:** classes, school runs, regular appointments, family time

It asks one topic at a time, with follow-ups like "Are Saturdays always off?", then shows a summary for you to confirm. The result is written to the **📌 My schedule** section in Notion, where you can edit it any time. The planner never schedules over these.

Ultron re-checks the schedule roughly monthly, or when your notes suggest a change ("new job starts Monday"). `/schedule` re-runs onboarding on demand.

### 1. Chat (Telegram)
Telegram is Ultron's voice, not a notes inbox: notes go in Notion only. Ultron answers only your chat ID. Free-form messages go to Claude, which has these tools: `search_notes`, `list_tasks`, `update_task`, `remember_fact`, `search_email`, `get_calendar(range)`, `get_goals`, `set_focus`, `replan_today`, `propose_plan`. Commands: `/tasks`, `/today`, `/week`, `/inbox`, `/plan`, `/replan`, `/review`, `/brief`, `/weekly`, `/focus`, `/goals`, `/habits`, `/reflect`, `/nudges`, `/tz`, `/status`, `/usage`, `/forget`, `/undo-change`, `/help`.

### 2. Inbox → Board: sorting your notes
- **Hourly**, and right before each brief, Ultron checks whether the Inbox changed. The check is free; Claude is called only when there's something new.
- New jottings go to Claude, which sorts each one into tasks (with any due date, estimate and area it can infer), goals, ideas, schedule changes, facts about you, and feelings or reflections for the log.
- Ultron updates the Board and its internal records, then moves the originals into ✓ Processed.
- **When something is unclear** (e.g. "dentist tues??"), Ultron puts its best guess on the Board marked "(?)" and asks about it in the next brief, rather than guessing silently.
- **Sorting reads the whole Inbox in order,** because sections are made by label lines ("GOALS:", "George Tennis:") rather than by indentation. Duplicates are merged.
- **Done and stale items:** lines marked DONE are recorded as completed (for the accountability history) but kept off the to-do list. Items with past or unanchored dates ("Wednesday 14th Oct", "today") go on the Board marked (?) and are asked about in the next brief.
- **Nothing is lost:** any line that sorting didn't account for stays in the Inbox (it isn't moved to ✓ Processed) and is mentioned in the next brief.
- Note text is treated as untrusted data in prompts. A note can't make Ultron do anything beyond sorting it.

### 3. Email triage
`imap-tools` against Yahoo with an **app password**, opened read-only (`EXAMINE`, `BODY.PEEK`, so mail stays unread), read during the **hourly sync** (up to 100 new emails per hour; the first run reads the last 30 days). Ultron has no ability to send, move or delete mail.
- **Triage (Haiku, batches of 25):** each email is **important** (money due, a deadline/appointment, or a real person writing), **FYI** (receipts, deliveries, security alerts), **promo** (only counted) or **ignore**. It gets a one-line summary and a "waiting for your reply" flag. Summaries never contain codes, passwords or card numbers; likely phishing is flagged and never becomes a to-do.
- **To-dos:** bills, deadlines, appointments and tasks go into the Board's To-do list as e.g. "Pay council tax · £120.00 · 📧 Council · due Sun 01 Nov". Duplicates are merged with simple, free matching: same thread or subject, same sender and amount, or near-identical wording. To-dos from emails more than a week old get "(?) Still needed?" instead of a stale date. An email saying something is done only asks "Tick it?", never ticks it.
- **Telling you:** one Telegram ping per sync for new important mail (not during quiet hours, nothing older than a day), `/inbox` for the last 24 hours, and chat can search the summaries.
- **Sent folder:** headers only (never bodies, never sent to Claude), to clear "waiting for your reply" once you've answered.
- **Privacy:** blocklisted senders are spotted from headers and never downloaded. Only summaries are stored, for 90 days.

### 4. Focus modes ("personality")
- Built-in modes: **Balanced** (default), **Health**, **Wealth**, **Career**, **Relationships**, **Learning**, **Recovery** (light days, rest first). Each mode defines:
  - **area weights** (e.g. Health: health 0.5 · career 0.25 · other 0.25), which the planner uses to rank tasks,
  - **non-negotiables** (Health → workout block, sleep at a fixed time, meal prep; Wealth → a daily deep-work block on income work and a weekly finance review),
  - **tone/coaching style** (encouraging, drill-sergeant, analytical), which goes into the system prompt.
- Switch with `/focus health`, `/focus wealth 2w` (time-boxed, then back to the default), or blends like `/focus health 60 wealth 40`. Custom modes are defined in plain text in the Board's 🧠 About me & focus section, so you can make your own without touching code.
- The active mode shows in every brief, and the 📓 Log records it, so the weekly review can compare modes.

### 5. Evening review + plan (21:30, configurable)
A single message in four parts:
1. **Today:** what was planned vs done. Ultron asks you to tick off today's blocks with ✅ / ◐ / ❌ buttons, and also counts Tasks marked Done in Notion, notes added and key emails handled.
2. **Week ahead:** the rest of the week from 📅 Schedule, upcoming deadlines, and how much free time is left this week compared with what the week's goals need.
3. **Goals pulse:** progress on active weekly and quarterly goals, plus any goal that's been neglected for X days, weighted by focus mode.
4. **Suggestions for tomorrow** + "What are your priorities for tomorrow? Anything to add?"

Then: you reply → Claude builds a time-blocked plan for **tomorrow and the day after** around your busy slots (it fills at most ~70% of each day's free time, keeps buffers, and puts high-energy tasks first) → **[Approve] [Edit] [Skip]**. Approve writes the plan to **📅 Schedule** (updating existing blocks in place, so replanning never duplicates) and writes the day's 📓 Log entry. If you don't reply, Ultron sends a ready draft at the nudge time (21:45) built from tonight's suggestions instead of a reminder; it still needs Approve, a later reply redrafts it, and "skip" discards it. A question in reply ("what day is tomorrow?") is answered in chat and doesn't count as priorities, and a reply only counts until the planned day starts. If there's no plan in the morning, the brief offers [📝 Plan today].

### 6. Morning brief (07:30, configurable)
Today's plan from 📅 Schedule, the top 3 priorities, important or action emails that came in overnight, anything overdue, a one-line goal reminder in the active focus mode's tone, and a **[Replan]** button.

### 7. Reflect: talk to Ultron about your thoughts and life
- A dedicated **Reflect** space, separate from quick commands. It runs as a **Telegram private group with Topics**: the 💬 Chat topic is for everyday requests and tasks, 🧠 Reflect is for thoughts and life, and 📋 Briefs holds the evening/morning/weekly messages. This keeps the reflective conversation in one uncluttered thread. (Fallback: `/reflect` … `/done` in the normal chat.)
- In Reflect, Ultron acts as a thoughtful coach rather than a task bot. It listens, asks follow-up questions, notices patterns (e.g. "you've mentioned feeling drained after late work three times this week"), and draws on your goals, the 📓 Log and past reflections.
- **Personalisation suggestions:** when a conversation reveals something actionable, Ultron proposes a concrete change as a card with **[Apply] [Tweak] [Dismiss]**. For example: a new or adjusted focus mode, changed area weights, a new habit, a new goal, moving the review time, a profile fact ("prefers workouts in the morning"), or a change to the coaching tone. Nothing changes until you hit Apply. Applied changes are written to the Board's 🧠 About me & focus section and logged so they can be undone (`/undo-change`).
- Each Reflect session is summarized as an entry in the Board's 📓 **Log & journal** section (date, mood 1–5, themes, key insights, suggestions made/applied). Full transcripts stay only in local SQLite, never in Notion. The weekly review includes a "themes from your reflections" section.
- Ultron can start a reflection too: once a week, or after a rough day (many ❌ in the evening review), it gently asks whether you want to talk it through. You can turn this off.

### 8. Weekly review (Sunday 19:00, configurable)
Wins, misses, completion rate and time spent per life area, goal progress, habit streaks, themes from Reflect, money-out/due-soon items. It suggests a focus mode for next week and sets the week's objectives in Goals (Horizon = Week). This is the source of the "whole week + overall goals" context used by the nightly planner.

### 9. Replan the rest of the day
"Running late" / "meeting overran" / `/replan` → Ultron reads what's done and what's left, then reshuffles the remaining blocks into the free time left. You approve, and it updates the same blocks on 📅 Schedule.

### 10. Voice notes
Send a Telegram voice message → it's transcribed (a speech-to-text API; the provider is set in config) → routed as a note, chat or Reflect message depending on which topic it was sent in.

### 11. Habit tracking
The Board's 🔁 **Habits** section (name, target per week, streak, last done; details kept internally). Habits are ticked in the evening review with one tap each. Focus modes can require habits (Health → gym 3×/week), and the planner reserves time for them.
- **☑️ Today's habits** *(decided 2026-10-21, built with Phase 5)*: an always-open checklist on the Board that clears each morning and records ticks for streaks. Limits ("2,000 calories/day", "≤1h social media") are ticked at the end of the day as "kept to it", so they get streaks like any habit.

### 12. Block-start nudges
A ping 5 minutes before each planned block with [Start] [Snooze 15] [Skip]. Off by default, toggled with `/nudges on`, and **quiet hours** (default 22:30–07:00) apply to every message except ones you trigger yourself.

### 13. Bill and deadline radar
Email triage tags bills, renewals, payments due and deadlines with amount and date. Results go into Tasks (Area = Wealth/Admin) and a weekly "money out / due soon" list in the weekly review. When Wealth mode is active, they're also in the evening review.

### 14. Personal memory
Ultron picks up stable facts from chat and Reflect ("gym Mon/Wed/Fri", "no meetings before 10") and **proposes** them for 🧠 About me & focus, using the same Apply/Dismiss card. If you tell it directly ("remember that I don't work Fridays"), it saves straight away with an [Undo] button. Everything Ultron "knows" about you is visible and editable on the Board; there's no hidden memory.

### 15. Accountability: Ultron pushes you to act
Ultron behaves like a coach who holds you to what you said, not just a planner. *All of these were confirmed by the owner on 2026-10-21, including the midday check-in.*
- **Commitments:** when you approve a plan or say "I'll do X", Ultron records it as a commitment, and the evening review checks each one by name.
- **Follow-ups on misses:** a skipped or ❌ block gets a short "what got in the way?" question. Your answer goes in the 📓 Log, and the task is rescheduled, shrunk or dropped (your choice), so it never silently disappears.
- **Pattern call-outs:** the same task slipping 3+ times, or a goal untouched for a week, gets named directly ("You've moved 'call the accountant' four times. Want to do it first thing tomorrow, or drop it?").
- **Midday check-in** (optional): a quick "how's the day going?" at a set time, with a [Replan] button if things are off track.
- **Asks before telling:** for vague priorities, Ultron asks clarifying questions ("What would make tomorrow a win?") before planning.
- **Intensity follows focus mode and tone:** gentle in Recovery, firm in drill-sergeant. You can adjust it through Reflect suggestions.
- **Never nags in quiet hours,** and you can snooze accountability for a day (`/easy`).

### 16. Roadmaps: a long plan, worked on every day
The owner's AI career roadmap (5 phases, ~48 steps, from Mon 26 Oct 2026) lives in Ultron, so it gets time every day and nothing slips silently. *Agreed with the owner on 2026-10-24.*
- **Where it comes from:** a JSON file made from the owner's roadmap (`data/roadmaps/ai-career.json`, not in git), saved with `python -m ultron.load_roadmap`. Running it again keeps ticks and the owner's wording, and only adds what's new.
- **🗂 Board:** its own always-open section above ✅ To-do (see the Board table). Owner edits work as everywhere: tick, reword or delete a step, or type a line under a stage heading (or inside a folded phase) to add a step there. Deleting a whole folded phase by accident doesn't drop its steps: it's drawn again.
- **📅 Schedule:** every planned day needs **110 min** on it, or **45 min** on a short day (under 6 h free, e.g. Thu/Sun work days), about 10.5 h a week. Code checks each draft: a short plan gets one retry, and if it's still short the draft says so. Blocks are titled "AI career: <step>", on the next steps in order. The owner's priorities can still ask for less on a given day.
- **Sessions vs steps:** a step can take several sessions (a course, a project). Ticking a block on the 📅 Schedule logs the session. The step stays open until it's ticked on the Board, or tapped ✅ at the evening review, where step blocks get ✅ / ⏩ Progress / ❌, plus "Finished any of these?" for sessions ticked earlier.
- **Repeating parts** are habits in the project, paused until their phase starts: the daily learning log (from 26 Oct), LinkedIn messages Tue/Sat and a weekly comment (from 23 Nov), and a weekly applications slot on Sat (from 25 Jan 2027).
- **Accountability:** a stage past its date shows ⚠️ on the Board and is called out by its next step (first thing tomorrow / drop / keep). The Sunday review reports hours against plan, steps finished and the checkpoint, and warns when a phase is over 4 weeks late ("cut scope rather than stop", the roadmap's own rule).

## Built-in safeguards (all implemented)
- **Completion tracking:** tick-off buttons in the evening review, synced with the Board's ✅ checkboxes and the 📓 Log. The evening summary is built from real completion data, not from what was planned.
- **Over-planning guard:** the planner fills at most 70% of free time (configurable), keeps 10-minute buffers between blocks, puts high-energy tasks first, and protects sleep, meal times and 📌 My schedule commitments.
- **Privacy:**
  - A sender/domain **blocklist** in config (e.g. bank, medical). Those emails are never fetched or sent to the API.
  - Only summaries are stored, never full email bodies.
  - Reflect transcripts stay local.
  - `/forget <topic|date>` deletes matching local state and 📓 journal entries.
- **Prompt-injection hardening:**
  - Email and note content is wrapped as untrusted data in prompts.
  - The agent has no outbound tools (no mail sending, no web access).
  - Any task extracted from an email is labeled with its source.
  - Schedule changes still require your approval, so injected content can never act on its own.
- **Time zones and DST:** every datetime is tz-aware (`Europe/London` by default). `/tz` changes it for travel. Tests cover DST transitions.
- **Reliability:**
  - Jobs retry with backoff, and if they still fail you get a Telegram message saying what failed.
  - Each job's last success time is tracked; `/status` shows the health of every integration.
  - Missed jobs (e.g. after a VPS reboot) are caught up on startup.
- **Backups:** nightly SQLite backup + a JSON export of the Inbox and Board pages, with 14 days kept on the VPS. Copying them off the server is optional.
- **Cost guard:** API token use is counted per call. There's a monthly cap in config, with a warning at 80%; at 100% Ultron switches to essential jobs only (briefs and planning) until the month resets. `/usage` shows the spend.

## Project layout
```
Ultron/
  CLAUDE.md                 # project instructions for Claude (you extend this)
  README.md                 # setup steps
  pyproject.toml            # python-telegram-bot[job-queue], anthropic, notion-client,
                            # imap-tools, pydantic-settings (caldav only for an optional read-only iCloud later)
  .env.example              # TELEGRAM_TOKEN, TELEGRAM_CHAT_ID, ANTHROPIC_API_KEY, NOTION_TOKEN,
                            # NOTION_ROOT_PAGE_ID, YAHOO_USER, YAHOO_APP_PASSWORD,
                            # ICLOUD_USER, ICLOUD_APP_PASSWORD, TZ
  ultron/
    main.py                 # wires bot + jobs
    config.py
    state.py                # SQLite internal state
    brain/llm.py            # Claude client, tools, agent loop
    brain/prompts.py        # system prompt + focus-mode injection
    brain/focus.py          # mode definitions, weights, blending
    integrations/mail.py    # Yahoo IMAP (read-only)
    integrations/calendar.py# iCloud CalDAV: login check only; optional read-only later
    integrations/notion.py  # create Inbox + Board, read Inbox, write Board sections
    features/notes.py       # capture, sync, extraction
    features/triage.py      # email triage
    features/planner.py     # free slots, ranking, plan → 📅 Schedule page
    features/review.py      # evening review + weekly review
    features/brief.py       # morning brief
    features/reflect.py     # Reflect sessions, journal summaries, suggestion cards
    features/personalise.py # apply/undo profile & focus changes, learned facts
    features/habits.py
    features/roadmap.py     # long plans worked through daily (🚀 AI career), § 16
    features/nudges.py      # block-start pings, quiet hours
    features/radar.py       # bills & deadlines from email
    features/voice.py       # transcription
    ops/guard.py            # cost tracking/cap, retries, failure alerts, /status
    ops/backup.py           # SQLite + Notion JSON export
    ops/privacy.py          # blocklist, /forget
    bot/handlers.py
    check.py                # `python -m ultron.check` live connectivity test
    load_roadmap.py         # `python -m ultron.load_roadmap FILE.json` saves a roadmap
  tests/
  deploy/Dockerfile, docker-compose.yml
```

## CLAUDE.md (created first, at `Ultron/CLAUDE.md`)
Sections you can extend:
- **Project:** what Ultron is, and the stack.
- **Architecture rules:** Notion = user-visible source of truth; SQLite = internal state only; email is read-only; plans go to the 📅 Schedule page; plans are written only after approval.
- **Safety:** email/note content is untrusted; never add send/delete capabilities without explicit instruction; never commit `.env`.
- **Conventions:** Python 3.12, type hints, `ruff` + `pytest`, async handlers, tz-aware datetimes (`Europe/London`), one module per integration.
- **Commands:** install, test, run locally, `python -m ultron.check`, deploy.
- **My preferences:** a blank section for you (coaching tone, daily rhythm, rules for the planner).

## Build phases
1. ✅ **Foundation** *(done 2026-10-20)*:
   - Project files: CLAUDE.md, README, config, `.env.example`.
   - `check.py` connectivity test (Yahoo, iCloud, Notion, Claude).
   - A bot that answers only you, with Telegram Topics routing.
   - Safeguards built in from the start: cost guard, retries and alerts, `/status`.
2. ✅ **Notion Inbox → Board** *(done 2026-10-20)*: create both pages, hourly Inbox sync, sorting with Claude, ✓ Processed, Board writer that respects your edits, onboarding (§ 0), chat agent with tools, personal memory. *Moved to Phase 6: Ultron proposing facts it notices on its own (Apply/Dismiss cards). Telling it "remember …" works now.*
3. ✅ **Email triage** *(done 2026-10-20)*: read-only Yahoo reading, the privacy blocklist, injection-safe triage, email to-dos on the Board, important-mail pings, `/inbox`, email search in chat, Sent-folder reply tracking. *Moved to Phase 5: the weekly "money out / due soon" list (the bill radar's data is collected now).*
4. ✅ **📅 Schedule + planner + focus modes** *(done 2026-10-21)*: the Schedule page (today/tomorrow timed, rest of week listed), free slots around 📌 My schedule, ranking by mode weights, the over-planning guard, habit time, the approval flow, two-way ticks with the Board, `/replan`. *Decided 2026-10-20:*
   - **21:00 every night**, Ultron suggests what would move the needle furthest for the current focus and priorities, then asks for your priorities. Your reply leads to a plan. `/plan` also works any time. The full evening review comes in Phase 5.
   - The planner uses **Sonnet** and estimates task durations itself while planning (you can correct them).
   - **Meals aren't scheduled** for now; the owner skips breakfast and eats lunch and dinner.
   - **Custom focus modes** are written in plain text on the Board for now; Phase 6 adds Apply/Tweak cards.
   - *Also built:* progress updates via the Inbox ("spoke to 2 tradies, meeting Fri" ticks, moves or reopens the item, keeps the note, adds follow-ups). *Not yet:* habit time beyond workouts and routines the planner places itself; the ☑️ Today's habits checklist comes with Phase 5.
5. ✅ **Rituals** *(done 2026-10-21)*: evening review (✅/◐/❌, habit ticks, call-outs, Tomorrow/Shrink/Drop with "what got in the way?", reflection, then planning), the ☑️ Today's habits checklist with streaks, the 07:00 morning brief, the Sunday weekly review (objectives and focus suggestion), 📓 Log entries per day and week, commitments (plan blocks and "I'll do X" in chat), the 13:00 check-in and `/easy`.
   - **Then deploy** *(decided 2026-10-21)*: move Ultron to an always-on server straight after Phase 5 (the Phase 8 deploy work, done early), because the rituals have to run at fixed times every day.
6. **Reflect + personalisation:** Reflect topic, 📓 journal summaries, Apply/Tweak/Dismiss suggestion cards, `/undo-change`, check-ins that Ultron starts.
   - **Decided 2026-10-21:**
     - Reflect runs in the DM with `/reflect` … `/done` (and in the 🧠 Reflect topic if the group is set up). A session also ends after 30 minutes of silence.
     - Suggestion cards may change anything: profile facts, habits, goals, focus mode (switch or custom), ritual times and coaching tone.
     - Ultron offers to talk after a rough day (half or more of the day's commitments missed or partly done), and once a week after the Sunday review; never more than once a day, never on an `/easy` day, and it can be switched off.
     - Applied changes are listed by `/undo-change` and also noted in that day's 📓 Log entry.
     - Journal summaries use Sonnet.
     - `/forget <topic or date>` is built in this phase (moved from Phase 8).
7. **Voice notes + nudges + quiet hours.**
8. **Deploy:** VPS, Docker with `restart: unless-stopped`, nightly backups, missed-job catch-up, `/forget`.
9. *(Later)* **Personal iPhone app:** React Native + Expo (cloud builds work from Windows), installed via TestFlight, as a second front end on the same core. It adds native push notifications, direct calendar access on the phone, and dictation. App Store release only after living with Ultron for a while.

**App-ready core:** all logic lives in `ultron/` features/ops/integrations and returns plain data. Only `ultron/bot/` knows about Telegram. Accountability (§15) is built into the phases where its pieces belong: commitments and follow-ups in phase 5, pattern call-outs in phases 5–6, the midday check-in in phase 7.

## What you'll need to set up (I can't do these for you)
1. A Telegram bot via @BotFather.
2. A Notion integration token (notion.so/my-integrations), plus an empty "Ultron" page shared with the integration.
3. A Yahoo app password.
4. *(Optional, later)* An Apple app-specific password, only if Ultron should read your Apple Calendar.
5. An Anthropic API key.
6. A VPS (only needed at phase 6).

**Running cost:** VPS about $5/mo + Claude API about $5–20/mo. Telegram and Notion (free plan) cost nothing.

## Verification
- `pytest`: focus-mode weighting/blending, free-slot calculation, plan → Schedule blocks (in-place updates, DST), note → task extraction parsing, planner/review state machines (services mocked).
- `python -m ultron.check`: live access to all four services.
- Manual end-to-end check on this PC:
  1. Jot a few messy lines in the Inbox → after the hourly sync (or `/sync`) they're sorted onto the Board and moved to ✓ Processed.
  2. `/focus health` → `/plan` → the plan includes a workout block → Approve → the blocks appear on 📅 Schedule; tick one and the Board to-do ticks too.
  3. Trigger the evening review and morning brief on demand with `/review` and `/brief`, and check that they contain today's summary, the week ahead and goals.
  4. In the Reflect topic, talk about something (e.g. "I keep skipping the gym after work") → a suggestion card appears → Apply → 🧠 About me & focus updates → `/undo-change` reverts it → a 📓 journal entry is created.
  5. Send a voice note, try `/replan`, check that a blocklisted sender is skipped, and set a low cost cap to see the 80% warning.
- After deploying, leave it running for 2 nights and confirm that the 21:30 review and 07:30 brief arrive and survive a restart.
