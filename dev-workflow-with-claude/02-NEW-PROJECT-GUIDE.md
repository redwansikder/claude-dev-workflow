# Starting a New Project with CLAUDE.md and PLAN.md

For a new project, the difference is that **there's no code yet**. So `/init` has nothing to read, and Claude can't guess your choices. **You make the decisions first**, and the files record them.

## Step 1: Decide the basics yourself (15 minutes, no AI needed)

Answer these on paper:

1. **What is it?** One sentence: "A booking app for local tutors in Chattogram."
2. **Who uses it?** Students, tutors, admin.
3. **The first version (MVP)**: 3–5 features only.
4. **Stack**: pick what you know, e.g. Next.js + TypeScript + Tailwind/MUI.
5. **What's not in v1**: payments, mobile app, etc.

You can ask Claude to help you think: *"I want to build X. Help me pick the MVP features and the stack. Don't write code."* But **the final decision is yours.**

## Step 2: Write a short CLAUDE.md before any code

It will be short at first, because you only know your decisions so far:

```md
# Project: TutorBD — tutor booking app

## What it is
Students find and book local tutors. v1 = web only.

## Stack (decided)
- Next.js 15 (App Router), TypeScript strict
- Tailwind for layout, MUI for forms/dialogs
- Backend: Next.js API routes + Prisma + PostgreSQL
- Auth: NextAuth (Google + email)

## Conventions
- Feature folders: `src/features/<name>/`
- Routes in `src/app/` stay thin
- Use async/await, functional components

## Rules
- Don't add packages not listed above without asking
- No secrets in code; use `.env.local` (add keys to `.env.example`)

## Workflow
- Read `PLAN.md` first; one step at a time
- After each step: tick it, add a progress note, run lint
```

It **grows as you build**. After setup you'll add commands and the folder structure. After the first features you'll add the gotchas.

## Step 3: Make PLAN.md the "project setup" plan

For a new project, the first plan is **the foundation, not a feature**:

```md
# Task: Project setup (Phase 0)

## Goal
Empty but working app: runs, lints, deploys, has auth and DB connected.

## Done when
- [ ] `npm run dev` shows a home page
- [ ] Lint + typecheck pass
- [ ] Login with Google works
- [ ] DB connected, one test table
- [ ] Deployed to Vercel (preview URL works)

## Out of scope
- Any real features (tutors, booking) — next phases

## Steps
- [ ] 1. Create Next.js app with TypeScript, Tailwind, ESLint
- [ ] 2. Set up folder structure (`src/features`, `src/lib`, `src/components/ui`)
- [ ] 3. Add MUI and a basic theme
- [ ] 4. Add Prisma + PostgreSQL, one `User` model
- [ ] 5. Add NextAuth with Google login
- [ ] 6. Basic layout: header, footer, responsive
- [ ] 7. `.env.example`, README with setup steps
- [ ] 8. Push to GitHub, deploy to Vercel

## Decisions
## Open questions
## Progress notes
```

## Step 4: Keep a roadmap at the bottom of PLAN.md

For a new project you need the **big picture** as well as the current task:

```md
## Roadmap (big picture)
- [ ] Phase 0: Setup  ← current
- [ ] Phase 1: Tutor profiles (create, list, view)
- [ ] Phase 2: Search & filter
- [ ] Phase 3: Booking requests
- [ ] Phase 4: Admin panel
```

When Phase 0 is done, move the task part of `PLAN.md` (everything above the Roadmap) to `docs/plans/phase-0-setup.md`, write the Phase 1 steps in its place, and tick Phase 0 in the roadmap with a link to the archived file. `PLAN.md` stays short, and every phase is kept.

## Step 5: Start building

```bash
mkdir tutorbd && cd tutorbd
git init
# add CLAUDE.md and PLAN.md
git add . && git commit -m "project docs"
```

Then, in a new chat:

> Read CLAUDE.md and PLAN.md. Do step 1 only.

From here it's the same routine: **check, commit, push, next step.**

## Step 6: Update CLAUDE.md after setup

When Phase 0 is done, ask:

> Update CLAUDE.md with the real commands and folder structure. Keep it short.

Or run `/init` now that there's code, and merge in anything useful. From then on, `CLAUDE.md` works like the one for an existing project.

## How it compares

| | New project | Existing project |
|---|---|---|
| Who writes CLAUDE.md first | **You** (your decisions) | `/init` (reads the code) |
| First PLAN.md | Setup / foundation | A feature |
| Roadmap | Needed (big picture) | Usually not |
| CLAUDE.md size | Starts tiny, grows | Starts full, gets trimmed |

**Key point:** decide first, write it down, then let Claude build. If you let Claude make all the early choices, you end up with a project you don't fully understand.
