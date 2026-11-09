# The owner's instructions (private)

Loaded by the public CLAUDE.md (`@private/OWNER.md`). This repo, `QuantumQode/ultron-private`, is private and lives in the public repo's git-ignored `private/` folder. It holds:

* `TODO.md`: the owner's running list (rules below).
* `PLAN.md`: the full design, with the owner's own context.
* `roadmaps/`: roadmap source files. The server loads them from `~/ultron/data/roadmaps/` (mounted into the container), so copy a changed file there before `ultron.load_roadmap`.

Public repo: `QuantumQode/ultron` (all code). The old private repo with the full history up to 2026-11-09 is `QuantumQode/ultron-archive`.

## Working with the owner

* When starting a new session, or picking up a session after time away, check `private/TODO.md` and ask the owner to confirm any decisions that need to be made.
* Commits to the public repo use `Pouria Ghasemi <QuantumQode@users.noreply.github.com>`, never the owner's personal email.
* Apple Calendar is never written to; read-only access may be added later. A future iPhone app (Expo + TestFlight) will be a second front end on the same core.

## Keep TODO.md up to date (every prompt, every action)

`private/TODO.md` (in the private repo `QuantumQode/ultron-private`, cloned into `private/`) is the owner's running list of things they need to decide or do. It is committed to the private repo so the owner can see it on other devices. Never copy it, or anything else from `private/`, into the public repo.

* Start of every session: read `TODO.md` so open items stay in view. If it's missing, recreate it with the same sections.
* After every prompt or action, before finishing your reply, update it:
   * Add anything new the owner has to decide or do: open questions you asked, manual steps (dashboards, accounts, keys, content, legal), PRs waiting for review or merge, and placeholders you left in the code.
   * When something gets done or decided, tick it off and move it to Done with the date (`- [x] YYYY-MM-DD: …`).
   * Rewrite items that have changed. Don't duplicate them.
   * Bump the `_Last updated:` date.
* Keep items short and actionable. Bold the lead, and include exact values, file paths or links the owner will need.
* Don't list Claude's step-by-step implementation details.
* Commit TODO.md changes to the private repo (`git -C private …`) alongside the public work they relate to. Never put secrets (passwords, tokens, keys) in it.
* The owner edits TODO.md and PLAN.md (both in `private/`) directly too. Keep their edits: never overwrite or revert them, and include them in the next commit. Re-read the file before editing it.
* Mention in your reply when TODO.md changed (one line is enough), so the owner knows to check it.

## My preferences

_Filled in from the owner's answers (2026-10-21). The owner edits this freely; keep their edits. The live versions of these facts sit on the Board (📌 My schedule, 🧠 About me & focus), and Ultron reads those._

* **Coaching tone:** direct, blunt and analytical. No padding or cheerleading; name the trade-off, use numbers. Straight but never harsh on grief, stress or anything painful.
* **Daily rhythm:**
  * Sleep 22:00–07:00 (changed 25 Oct 2026; the Sleep entry is the only place the wake time lives). On early days Ultron wakes them 30 minutes before the first commitment: 06:00 on work days, 06:30 before a 07:00 booking such as padel. The last 30 minutes (21:30–22:00) are wind-down: journaling or meditating, including a 15-minute meditation. Never planned over.
  * Work Thu and Sun 07:00–18:00 with a 30-minute commute each way (blocked 06:30–18:30).
  * Morning routine: water, wash face and brush teeth, 10 minutes of mindfulness, then gratitude journaling (10 things). It starts on waking (35 min) and is squeezed short on early days.
  * Skips breakfast (just a coffee); eats lunch and dinner. Meals aren't scheduled.
* **Training:** workouts Mon, Tue, Fri and Sat mornings, and Thu after work (around 18:30); Sunday is a rest day. Prefers to train as early as possible, then do deep work. Likes a short morning jog to raise cortisol, with weights later; after a jog the weights session is shorter. Padel or tennis counts as that day's cardio, but the gym still happens later the same day (at least 90 minutes after).
* **Planner rules:**
  * Nothing goes on the 📅 Schedule without approval (Approve / Edit / Skip).
  * Fill at most 70% of free time (50% in Recovery), with 10 minutes between blocks. Claude estimates durations while planning, using the track record of past blocks.
  * Fewer, bigger blocks: at most 8 a day, at most 3 under 25 minutes, deep work in 60–90 minute blocks, small admin batched into one block.
  * At 21:00 Ultron suggests what moves the needle most, then asks for tomorrow's priorities.
  * Focus is Balanced by default. Custom focus modes are written in plain text on the Board.
* **Until 3 Nov 2026:** a fortnight focused on mindfulness and reducing stress. Keep the normal plan but add stress relief; the "12 hours of work a day" habit is paused.
* **Accountability:** commitments, follow-ups on misses, pattern call-outs and a midday check-in are all wanted (PLAN.md § 15).
* **Working with Claude:** keep Notion simple (three pages). Be careful with Claude API spend: ask before live tests and say the cost.
