# Project: AccountHub — user accounts web app

Users sign up, manage their profile, and change account settings.

## Stack
- Next.js (App Router) + TypeScript
- Tailwind CSS
- MUI for complex components (tables, dialogs)

## Commands
- Dev server: `npm run dev`
- Build: `npm run build`
- Lint: `npm run lint`

## Folder structure
- `src/app/` — routes and API routes (`src/app/api/`)
- `src/components/` — UI components, one per file
- `src/lib/` — shared helpers
- `docs/plans/` — finished task plans, archived from PLAN.md

## Code style
- Use functional components and hooks
- Components in `src/components/`, one component per file
- Use `async/await`, not `.then()`
- Keep files under ~200 lines; split if bigger

## Skills
- `resume-work` — use at the start of a chat or after a break (checks git + PLAN.md)
- `finish-step` — use after every PLAN.md step (lint, build, tick, commit message)

## How to work
- Start with the `resume-work` skill, then work on the first unchecked step only
- After finishing a step, use the `finish-step` skill
- Ask before deleting files, changing the database, or installing new packages
