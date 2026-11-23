# TODO

_Last updated: 2026-11-23_

## Decide
- [ ] **📥 Inbox cuts long notes short (investigating 23 Nov):** tell Claude what you actually typed for "Urgent:" (22 Nov ~09:52) and "Start work on mojttaba website work, also find…" (23 Nov ~20:16), and which device/app you typed on. Then Claude finds the root cause.
- [ ] **RAG / semantic recall: decide the design and when to build it** (prompt drafted 9 Nov). Semantic search over the activity log, email summaries, journal and Reflect summaries, and old Board/Log notes, through one read-only `recall` tool. Proposed: `sqlite-vec` inside the existing SQLite database, a local embedding model (no data leaves the VPS), and hybrid keyword (SQLite FTS5) + vector search. Measured on 30–50 labelled questions (recall@5, MRR) for the CV. The index must follow the existing deletion rules: Reflect transcripts (90 days, blanked in the log) and email summaries (90 days) lose their embeddings too. Decide: (1) the embedding model: local (small, ~100 MB, so it fits once friends each get a container in Phase 5, or one shared model for all) or hosted (Voyage AI, Anthropic's recommended partner: a new outbound service that would see your data, needs your OK); (2) what gets indexed; (3) timing: Phase 1 and the repo split are done, so either now on its own `rag` branch, or after the friends phases (3–6). **Claude's full design and 8-step plan: reply of 19 Nov** (recommends a local `bge-small-en-v1.5` via `fastembed`, keyword search first, then vectors, measuring the gain). Also decide whether chat history and full Reflect transcripts get indexed, or only summaries.
- [ ] **Continue Phase 4 (onboarding) later:** steps 1–4 (`/start` offers setup to a new user, onboarding "rules" become facts, a coaching-style question, the `/` command menu) are parked on branch `phase4-onboarding` (commit `521b3f3`, server only, not pushed; 623 tests pass). Not yet tested live or deployed. Next: a real first-run test with a second bot: make a bot in BotFather and an empty "Ultron Test" Notion page shared with your integration (under $0.10 of Claude calls). Then merge into `main`.
- [ ] **Clear the reply box on every bot start?** Say yes and Claude adds it, so an old stuck "Replying to…" can never come back after a deploy.
- [ ] **Pick which "smarter Ultron" ideas to build next** (list in Claude's reply, 22 Oct). Claude suggests starting with: learn real task durations, a morning energy tap, and next steps for each goal.
- [ ] **Speech-to-text provider for voice notes:** decide at Phase 7.
- [ ] **Answer the email questions on your 🗂 Board:** to-dos from emails more than a week old show "(?) From an email on 20 Sep. Still needed?". Tick the done ones, delete the ones you don't need, or delete just the "(?) …" part to keep one.
- [ ] **Look over your 🗂 Board** (https://www.notion.so/3ffa7060f22081acb1b0e1ad835f032c) and say what's filed wrong, including 🚀 AI career (reword, delete or add steps there: your edits win).

## Do (manual setup)
- [ ] **Re-clone on your PC** (your old copy points at the renamed archive): in a new folder run `git clone https://github.com/QuantumQode/ultron.git`, then inside it `git clone https://github.com/QuantumQode/ultron-private.git private`, copy your `.env` across, and reinstall (`python -m venv .venv` then `.\.venv\Scripts\python.exe -m pip install -e ".[dev]"`). TODO.md is now `private/TODO.md`.
- [ ] **First friend on Gmail = the real Gmail test:** when they set up, run the email check (`ultron.check`). If Gmail refuses an app password, tell Claude.
- [ ] **Back up your SSH private key** `C:\Users\PouriaReset\.ssh\id_ed25519` (password manager or USB). Never share it; only the `.pub` file is safe to share.
- [ ] **Check your way back into the server:** in the OVH Control Panel open the VPS → **KVM**, and sign in as `ubuntu` with the emailed password (or set a new one with `sudo passwd ubuntu` over SSH). SSH no longer accepts passwords, so this console is your fallback.
- [ ] **Workout mornings card (5 Nov, "Tue, Fri, Sat"):** if you never tapped ✅ Apply, the 📌 Workout entry is unchanged. Check 📌 My schedule on the Board and ask Ultron to fix it if needed.
- [ ] **How long does your daily video habit take?** Tell Claude or write it on the Board, so the planner times it right.
- [ ] **Want different AI career time or habit days?** Now 110 min (45 min on days with under 6 h free), LinkedIn messages Tue/Sat, a weekly comment Wed, and applications Sat from 25 Jan. Tell Claude: they're set in `data/roadmaps/ai-career.json` on the server.
- [ ] **Move Apple Notes to Notion:** paste your existing notes into the 📥 Inbox whenever you like. Ultron sorts them onto the Board within the hour.
- [ ] **Telegram group (optional, any time):** create a private group with Topics (💬 Chat, 🧠 Reflect, 📋 Briefs), add @UltronOrg_bot and **make it an admin**. Then send `/status` inside each topic and put the chat id → `TELEGRAM_GROUP_ID` and each topic id → `TELEGRAM_TOPIC_CHAT` / `_REFLECT` / `_BRIEFS` in `.env` **on the server** (`nano ~/ultron/.env`, then `docker restart ultron`).
- [ ] **RAG eval set (later): check the labelled questions** once Claude builds them from a backup copy. Correct any wrong expected answers. This set stays in this private repo because it holds your data; a synthetic one goes in the public repo.

## Try and give feedback (whenever)
Tell Claude anything that felt off (tone, length, too pushy, wrong filing, unclear buttons).
- [ ] **Chat:** a look-up, a tick ("I changed the bedsheets"), an add ("remind me to X on Monday": should show "· on Mon"), "remember …" plus [Undo], a `!` message (Sonnet), something personal.
- [ ] **Memory:** ask "what did I change on the Board yesterday?" (~$0.01). It should answer from the log, not guess. You can also read it yourself: `docker exec ultron python -m ultron.log` (`--day`, `--kind`, `--grep`, `--full`).
- [ ] **Reflect:** `/reflect`, a few messages, `/done` (~$0.20). Check the "📓 Journaled" line and the 📓 Log entry.
- [ ] **Daily rhythm:** 21:00 review and planning, 07:00 brief, 13:00 check-in, Sunday 19:00 weekly review. Too many messages?
- [ ] **Two-day planner:** `/plan` or "plan tomorrow" (~$0.04–0.08 a draft).
- [ ] **📥 Inbox updates:** write progress on an existing item ("GP booked"); within the hour it's ticked or moved and you're told in Telegram.
- [ ] **Typing on the Board:** a tick box between two to-dos, an indented line (becomes a grey note), a line under a heading, then `/sync`. Your words and spot should stay.
- [ ] **📓 Log & journal:** one folded heading per week; untick a line to reopen it, delete one to hide it.
- [ ] **⚠️ lines in chat:** if Ultron says "No card was actually made", ask again; tell Claude if one shows up when it shouldn't.

## Waiting on review
- [ ] **One-off appointments from chat: built, ready to deploy** (commit `3648308`, pushed to GitHub 19 Nov; 633 tests pass). Ask in Telegram for a one-off without a time and Ultron asks for the start and end, then adds it to the 📅 Schedule. Next: deploy (`git pull && docker compose -f deploy/docker-compose.yml up -d --build`), then one live check ("dentist Monday", then answer the time; 2 calls, ~$0.02).

## Placeholders in code
- [ ] **Claude prices are hard-coded** in `ultron/ops/guard.py` (`PRICES`, checked 2026-10-19). Update them if Anthropic changes prices or you switch models; until then, unknown models are costed at the highest rate.

## Done
_Older entries (before 1 Nov) are in [DONE-ARCHIVE.md](DONE-ARCHIVE.md)._
- [x] 2026-11-18: **TODO tidied:** out-of-date checks removed, "try" items merged, Done before 1 Nov moved to `DONE-ARCHIVE.md`. Old one-off backups deleted (17 `pre-*` files in `~/ultron/backups/` and `~/ultron-neutralise`); nightly backups untouched.
- [x] 2026-11-09: **Phase 3 deployed** (12:22): started cleanly, email reads as Yahoo with your unchanged `.env`, CI green. Backup from just before: `~/ultron/backups/pre-email-providers-2026-11-09.db`.
- [x] 2026-11-09: **Public README reviewed:** you're happy with it.
- [x] 2026-11-09: **Phase 3 built: email from any provider.** Gmail, Yahoo, iCloud or any IMAP server with an app password; the provider is guessed from the address. Outlook/Hotmail is refused with a clear reason (decided: OAuth later, only if a friend needs it). 623 tests pass.
- [x] 2026-11-09: **Phase 1 + repo switch deployed** (12:06). Started cleanly; the "Planning settings" line was added to 🧠 About me (shows on the Board after the next sync, or `/sync`). Backup from just before: `~/ultron/backups/pre-neutralise-deploy-2026-11-09.db`.
- [x] 2026-11-09: **Repos split:** public https://github.com/QuantumQode/ultron (code, MIT, CI passing), private `ultron-private` (TODO, PLAN, your preferences, roadmaps, cloned into `~/ultron/private`), old history in private `ultron-archive`. The server now pulls from the public repo.
- [x] 2026-11-09: **Phase 2 decisions:** public repo `ultron`, old one renamed `ultron-archive`, personal files in private `ultron-private`; MIT licence; public commits as "Pouria Ghasemi" with the GitHub noreply email; PLAN.md stays private only. Public tree built and checked: gitleaks clean, none of your `.env` secrets or personal names in it, 602 tests pass.
- [x] 2026-11-09: **Phase 1 (neutralise) built:** no personal rules, tone or names left in the code or tests. Your rules now live in your profile: facts (Coaching style expanded, new Planning style, sport fact expanded) and planning settings (70%, 10 min gaps, 8 blocks, 3 under 25 min, wake 30 min early, 30 min wind-down "journal or meditate", routine 35 min, gym 90 min after sport). A before/after snapshot of every prompt showed the same numbers and info reach Claude. 602 tests pass.
- [x] 2026-11-09: **Phase 1 started:** a snapshot of every prompt Claude receives was taken from last night's backup, so the refactor can be checked line by line. Friends get a guide that uses the terminal a bit (they're capable, give them agency).
- [x] 2026-11-09: **Packaging decisions:** your rules become profile settings (shown in 🧠 About me); public repo holds the code with a private repo for your personal files; you handle how friends send keys; no privacy note.
- [x] 2026-11-09: **Packaging direction decided:** strip all personal info into your profile; email provider chosen at setup; friends hosted on your VPS with their own Anthropic keys; friends get every update; friends are non-technical; separate public repo (for CV) from private dev material.
- [x] 2026-11-08: **The server can push to GitHub now:** the deploy key was re-added with write access, and the 43 commits that were only on the server are on GitHub. You no longer need to pull from the server on your PC and push.
- [x] 2026-11-05: **Activity log built and deployed:** every message, button tap, item change (yours on the Board or Ultron's), Notion write, plan block, card, email read (summary only), job and Claude call is kept forever on the server, append-only. Ultron reads it in chat as its memory (`read_log`). Secrets are scrubbed; a deleted 🧠 Reflect session's words are blanked in the log too, matching Reflect's 90-day rule.
- [x] 2026-11-05: **Fake "card sent" / "Applied" fixed (deployed):** chat can't claim a card or an Apply that didn't happen; a typed "apply" points you to the card's buttons (buttons only, your choice). Your Workout change was never saved on 4 Nov, so a real card was sent.
