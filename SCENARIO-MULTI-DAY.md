# Real-World Scenario: A New Project from Zero, with Interruptions

**You:** a frontend developer in Bangladesh, working remotely for a freelance client.
**Project:** TutorBD, a web app where students find and book local tutors. **Nothing exists yet.**
**First milestone:** a working base app (setup) + tutor profiles. Deadline: in 6 days.
**Tools:** VS Code with the Claude panel, git + GitHub, desktop PC, used laptop.

This story shows two things:
1. How a project grows from nothing: decisions → `CLAUDE.md` + `PLAN.md` → skills.
2. What to do when work stops in the middle: you leave, the power goes, the PC restarts, or the chat ends.

---

## Day 0 (Sunday evening): Decide first, no code

You open a notebook (paper is fine) and answer five questions:

| Question | Your answer |
|---|---|
| What is it? | Students find and book local tutors |
| Who uses it? | Students, tutors (admin later) |
| MVP features | Sign in, tutor profiles, search, booking request |
| Stack | Next.js + TypeScript + Tailwind + MUI, Prisma + PostgreSQL (Neon), NextAuth |
| Not in v1 | Payments, mobile app, chat, reviews |

You're unsure about the database, so you open a new Claude chat:

> I'm building a tutor booking web app with Next.js. Free or cheap PostgreSQL hosting options? Short answer, don't write code.

Claude suggests a few options. **You decide:** Neon (free tier, nothing to install locally).

You close the chat. **Nothing to save from it.** The decision is in your notebook.

---

## Day 1 (Monday): Foundation

### 10:00 — Create the project yourself

```bash
cd C:\Users\Administrator\Documents\GitHub
npx create-next-app@latest tutorbd
# TypeScript: Yes, ESLint: Yes, Tailwind: Yes, src/: Yes, App Router: Yes
cd tutorbd
code .
```

`create-next-app` already runs `git init` and makes the first commit. You create an empty **private repo on GitHub** and connect it:

```bash
git remote add origin https://github.com/<you>/tutorbd.git
git push -u origin main
```

**Why do this yourself?** It takes 2 minutes and costs zero tokens.

### 10:15 — Write the first CLAUDE.md (tiny)

You create `CLAUDE.md` in the project root, with **only your decisions**:

```md
# Project: TutorBD — tutor booking web app

## What it is
Students find and book local tutors. v1 = web only.

## Stack (decided)
- Next.js (App Router), TypeScript strict, `src/` folder
- Tailwind for layout, MUI for forms/dialogs
- Prisma + PostgreSQL (Neon)
- NextAuth (Google login)

## Rules
- Don't add packages not listed above without asking
- No secrets in code; use `.env.local`, list keys in `.env.example`

## How to work
- Read PLAN.md first; do one step at a time
- After each step: run `npm run lint`, tick the step, add a progress note
```

It's 18 lines. That's fine; it will grow.

You also make plan-mode plans stay in the project. Create `.claude/settings.json`:

```json
{
  "plansDirectory": ".claude/plans"
}
```

### 10:30 — Make PLAN.md with Claude (Session 1)

New chat:

> Read CLAUDE.md. Write PLAN.md for Phase 0: project setup. Include Goal, Done when, Out of scope, small steps, and a Roadmap for later phases (tutor profiles, search, booking, admin). Don't write code.

Claude writes:

```md
# Task: Phase 0 — Project setup

## Goal
Empty but working app: runs, lints, has DB + Google login, deployed.

## Done when
- [ ] Home page runs locally
- [ ] Lint + build pass
- [ ] DB connected with a User table
- [ ] Google login works
- [ ] Deployed to Vercel

## Out of scope
- Any real features (tutors, booking)

## Steps
- [ ] 1. Folder structure: `src/features`, `src/lib`, `src/components/ui`
- [ ] 2. Add MUI + basic theme (works with Tailwind)
- [ ] 3. Add Prisma, connect Neon, `User` model, first migration
- [ ] 4. Add NextAuth with Google
- [ ] 5. Base layout: header, footer, responsive
- [ ] 6. `.env.example` + README setup steps
- [ ] 7. Deploy to Vercel

## Decisions
## Open questions
## Progress notes

## Roadmap
- [ ] Phase 0: Setup  ← current
- [ ] Phase 1: Tutor profiles
- [ ] Phase 2: Search & filter
- [ ] Phase 3: Booking requests
- [ ] Phase 4: Admin
```

You read it. Fine. You commit:

```bash
git add .
git commit -m "docs: CLAUDE.md, PLAN.md, settings"
git push
```

### 10:45 — Step 1

Same chat:

> Do step 1.

Claude creates the folders with small `README` notes inside each. Lint passes. Claude ticks the step and adds a note.

```bash
git add . && git commit -m "step 1: folder structure" && git push
```

### 11:30 — ⏸ Interruption #1: you have to leave (planned)

You're halfway through **step 2** (MUI theme) when your family calls you for an errand. You'll be gone for 2–3 hours.

**Don't just walk away.** Take 1 minute:

> I have to leave. Stop here. In PLAN.md, add a progress note: what's done in step 2 so far and what's left. Don't tick the step.

Claude adds:

```md
- 2026-11-03: Step 2 IN PROGRESS. MUI installed, theme file at src/lib/theme.ts.
  Left: wrap layout with ThemeProvider, fix Tailwind preflight conflict.
```

Then you save a **WIP commit** (work in progress):

```bash
git add .
git commit -m "WIP step 2: MUI theme (not finished)"
git push
```

Now it doesn't matter what happens while you're out: power cut, PC off, chat lost. **The half-done work and the note are on GitHub.**

### 3:00 PM — Back (Session 2)

You're back. The PC was shut down by load-shedding while you were out. You don't care. New chat:

> Read PLAN.md and continue step 2 from the progress note.

Claude reads the note, finishes the ThemeProvider and fixes the conflict. Lint and build pass, and Claude ticks step 2.

```bash
git add . && git commit -m "step 2: MUI theme" && git push
```

You could combine the WIP commit later, but it's fine to keep it.

### 3:45 PM — ⚡ Interruption #2: power cut during `npm install`

> Do step 3.

Claude runs `npm install prisma @prisma/client`... **the power goes off.**

### 5:00 PM — Power is back (Session 3)

```bash
git status
```

```
modified:   package.json
modified:   package-lock.json
```

`npm install` was killed halfway. Is that dangerous? **No. `npm install` is safe to run again.** But `node_modules` might be half-written, so do a clean reinstall:

```bash
git restore package.json package-lock.json
rm -r node_modules        # in PowerShell: Remove-Item -Recurse -Force node_modules
npm install
```

New chat:

> Read PLAN.md. Step 3 was interrupted during npm install; I restored package.json and reinstalled. Start step 3 again.

Claude installs Prisma, creates `prisma/schema.prisma` with a `User` model, and asks:

> I need `DATABASE_URL` in `.env.local`. Please paste your Neon connection string into `.env.local` yourself. Don't paste it in chat.

**Good:** the "No secrets" rule in `CLAUDE.md` worked. You put the URL in `.env.local` yourself.

### 5:30 PM — ⚡ Interruption #3: power cut during the database migration

Claude runs:

```bash
npx prisma migrate dev --name init
```

...**the power goes off again.** This one is **risky**: a migration changes the real database. Running it blindly again might fail or leave the database half-changed.

### 7:00 PM — Power is back (Session 4)

New chat. First, **check the database state** before doing anything:

> Step 3 was interrupted during `prisma migrate dev`. Don't run the migration again yet. Check what happened first.

Claude runs:

```bash
git status
npx prisma migrate status
```

```
Following migration have not yet been applied:
20261103_init
```

Claude explains: the migration file was created, but it wasn't applied to the database, so it's safe to apply it now.

> OK, apply it.

```bash
npx prisma migrate dev
```

Success. Claude ticks step 3.

```bash
git add . && git commit -m "step 3: Prisma + Neon + User model" && git push
```

**Lesson:** for **database, file moves, emails or payments**, always **check the state first** after an interruption. Never just rerun.

---

## Day 2 (Tuesday): Skills appear, and a restart

### 9:00 AM — You notice a pattern

Yesterday you typed these instructions **many times**:
- "Read PLAN.md and continue..."
- "Check git status, what was interrupted?"
- "Run lint and build, tick the step, write a note, suggest a commit message"

**Repeated 3+ times = make a skill.** New chat:

> Create two project skills in .claude/skills/:
> 1. `resume-work`: read-only. Check git status and diff, read PLAN.md, report last done step, next step, uncommitted changes with keep/restore advice. For risky steps (migrations), check the state first.
> 2. `finish-step`: run lint and build, tick the step, add a progress note, show git status, suggest a commit message. Never commit.

Claude creates both. You add one section to `CLAUDE.md`:

```md
## Skills
- `resume-work` — start of every chat or after any interruption
- `finish-step` — end of every PLAN.md step
```

And change "How to work" to use them. Commit:

```bash
git add . && git commit -m "add resume-work and finish-step skills" && git push
```

**Start a new chat** so the skills and the new `CLAUDE.md` load.

### 9:30 AM — Step 4 (Session 5)

> /resume-work

```
Last done:   Step 3 — Prisma + Neon + User model
Next step:   Step 4 — Add NextAuth with Google
Uncommitted: none
Continue with step 4?
```

> Yes.

Claude sets up NextAuth. It needs `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`, so it adds the names to `.env.example` and asks you to put the real values in `.env.local`.

You test the login in the browser. It works.

> /finish-step

```bash
git commit -am "step 4: NextAuth Google login" && git push
```

### 11:00 AM — ☎ Interruption #4: client call in the middle of a step

> Do step 5.

Claude has built the header and is working on the footer when the client calls. The call takes **1.5 hours**.

**The PC stays on, and the chat is still open.** When you come back:

> Continue step 5.

That's it. If nothing was interrupted, **you can just continue in the same chat.** You only need `/resume-work` when the chat was lost or you're unsure.

Step 5 done → `/finish-step` → commit → push.

### 4:00 PM — 💬 Interruption #5: the chat gets too long

You've been in the same chat for steps 5 and 6. Claude starts **forgetting earlier details** and repeating questions, because the conversation is very long.

You don't push on. Instead:

> /finish-step

Commit, then:

> /clear

> /resume-work

You get a fresh, short chat with the same project state. **Long chats cost more tokens and get worse answers.**

Step 6 done → `/finish-step` → commit → push.

### 11:00 PM — 🔄 Interruption #6: Windows restarts overnight

You start step 7 (deploy) late at night, then go to sleep **without committing**. Claude had added a `vercel.json` and edited the README.

Overnight, **Windows Update restarts the PC.**

---

## Day 3 (Wednesday): Recover, then finish Phase 0

### 9:00 AM — Morning after the restart (Session 6)

Is the work from last night still there? Saved files survive a restart. **Only unsaved editor tabs and the chat are lost.** Turn on VS Code **File → Auto Save** so files are always saved.

New chat:

> /resume-work

```
Last done:   Step 6 — .env.example + README
Next step:   Step 7 — Deploy to Vercel
Uncommitted: vercel.json (new), README.md (modified)
             npm run build: PASSED
Advice:      keep — changes look complete for the start of step 7
Continue with step 7?
```

You look at the diff yourself. It looks right. **Keep it**, and commit right away:

```bash
git add . && git commit -m "WIP step 7: vercel config" && git push
```

> Yes, continue step 7.

You connect the GitHub repo to Vercel and add the environment variables **in the Vercel dashboard** (not in code). The deploy works.

> /finish-step

```bash
git commit -am "step 7: deployed to Vercel" && git push
```

### 11:00 AM — Phase 0 done: update the files

**1. Check "Done when" yourself.** Open the Vercel URL and test login. All five boxes are done.

**2. Update CLAUDE.md with real facts** (it's been 2 days of decisions):

> Update CLAUDE.md: add real commands, folder structure, and gotchas we found (Tailwind preflight vs MUI, migrations need a state check). Keep it under 60 lines.

**3. Replace PLAN.md with Phase 1:**

> Phase 0 is done. Tick it in the Roadmap. Replace the task part of PLAN.md with Phase 1: tutor profiles (create, edit, list, view). Keep the Roadmap. Don't write code.

You review the new plan and commit:

```bash
git add . && git commit -m "docs: Phase 0 done, plan Phase 1" && git push
```

`/clear`. **A new phase gets a fresh chat.**

---

## Day 4 (Thursday): Home power is out all day

### 10:00 AM — Café, on the laptop (Session 7)

Load-shedding is scheduled all day at home. You take your laptop to a café.

**First time on the laptop for this project:**

```bash
git clone https://github.com/<you>/tutorbd.git
cd tutorbd
npm install
```

`.env.local` is **not in git** (correctly), so copy it from your PC beforehand by USB or a password manager. **Never send it by email or chat.**

New chat:

> /resume-work

```
Last done:   Phase 0 complete
Next step:   Phase 1, Step 1 — Tutor model in Prisma
Uncommitted: none
Continue with step 1?
```

**Different PC, and the same project, rules, skills and plan.** Nothing to explain again.

Steps 1–3 done today, each ending with `/finish-step` → commit → push.

### 4:30 PM — 🔋 Interruption #7: laptop battery at 10%

You're in the middle of step 4, and there's no charger socket nearby.

> Battery dying. Stop. Write a progress note for step 4 in PLAN.md.

```bash
git add . && git commit -m "WIP step 4: tutor edit form" && git push
```

The laptop dies 5 minutes later. **Nothing lost.**

---

## Day 5 (Friday): Back home

### 10:00 AM — Desktop again (Session 8)

```bash
git pull
```

> /resume-work

It finds the WIP note for step 4 and continues from there.

Steps 4–6 done. Phase 1 "Done when" checked yourself. Then a **review in a fresh chat**:

> /clear
> Review all Phase 1 changes compared to the commit "docs: Phase 0 done". Look for bugs only.

It finds one bug (an empty tutor bio crashes the profile page). You fix it and commit.

You send the client the Vercel link. **The milestone is done a day early.**

---

## Every interruption and what saved you

| # | What happened | What you did | What saved you |
|---|---|---|---|
| 1 | You had to leave mid-step | Progress note + WIP commit + push | Note in `PLAN.md` + GitHub |
| 2 | Power cut during `npm install` | Restore + clean reinstall | `npm install` is safe to rerun |
| 3 | Power cut during DB migration | **Checked state first**, then applied | `prisma migrate status` |
| 4 | Client call mid-step (PC on) | Just continued in the same chat | Nothing needed |
| 5 | Chat got too long | `/finish-step` → `/clear` → `/resume-work` | State in files, not the chat |
| 6 | Windows restarted overnight | Checked the diff, kept it, committed | Auto Save + `/resume-work` |
| 7 | Laptop battery died | Progress note + WIP commit + push | GitHub |

## How the project files grew

| | Day 1 | Day 2 | Day 3 | Day 5 |
|---|---|---|---|---|
| `CLAUDE.md` | 18 lines (decisions) | + Skills section | ~55 lines (commands, structure, gotchas) | + 1–2 new rules |
| `PLAN.md` | Phase 0 + Roadmap | Phase 0 steps ticking | Replaced with Phase 1 | Phase 1 done |
| Skills | none | `resume-work`, `finish-step` | same | same |
| `.claude/settings.json` | `plansDirectory` | same | same | same |
| Chats | 4 | 2 | 1 | 1 |

**8 chats over 5 days, 7 interruptions, and you never needed an old chat.**

## The rules to remember

**Before you leave (planned stop):**
```
"Stop. Write a progress note in PLAN.md."  →  WIP commit  →  push
```

**After an unplanned stop (power, restart, crash):**
```
New chat → /resume-work → check the diff yourself → keep or restore → continue
```

**After a risky step was interrupted (database, file moves, emails, payments):**
```
Check the state first → never just rerun
```

**Normal routine:**
```
/resume-work → "Do step N" → check it yourself → /finish-step → commit + push
```
