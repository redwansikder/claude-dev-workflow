# Making a Good CLAUDE.md

"Production ready" doesn't mean long. It means **short, accurate, and specific to your project**. Claude reads `CLAUDE.md` in every chat, so each line should earn its place.

## Step 1: Let Claude write the first draft

In a real project, open the Claude panel and type:

```
/init
```

Claude reads your codebase and writes a `CLAUDE.md` with the stack, folders and commands it finds.

## Step 2: Edit it down

- **Remove** anything obvious ("write clean code", "use React") and anything Claude can see from the code itself.
- **Keep** what Claude **can't guess**: your rules, warnings, and the "why" behind decisions.
- **Aim for 30–80 lines.**

## Step 3: Improve it over time

Every time Claude makes the same mistake twice, add one line to fix it. That's how a good `CLAUDE.md` grows: from real problems, not guesses.

## Template

```md
# Project: ShopBD — e-commerce frontend

## Stack
- Next.js 15 (App Router), TypeScript (strict), Tailwind, MUI v6
- State: Zustand. Data fetching: TanStack Query
- Backend: separate Express API (not in this repo), base URL in `NEXT_PUBLIC_API_URL`

## Commands
- `npm run dev` — dev server (port 3000)
- `npm run lint` — must pass before a step is done
- `npm run typecheck` — must pass before a step is done
- `npm test` — Vitest

## Folder structure
- `src/app/` — routes only, keep pages thin
- `src/components/ui/` — reusable, no business logic
- `src/features/<name>/` — feature code (components, hooks, api)
- `src/lib/` — shared helpers

## Rules
- Use TanStack Query for all API calls; never fetch inside useEffect
- Prices are in BDT, stored as integers (poisha). Use `formatPrice()` from `src/lib/money.ts`
- All user-facing text goes through `t()` (Bangla + English)
- Don't add new npm packages without asking
- Don't edit `src/components/ui/legacy/` — scheduled for removal

## Gotchas
- `.env.local` is required; copy from `.env.example`
- MUI and Tailwind conflict on buttons — use MUI `Button` only

## Workflow
- Read `PLAN.md` first; do one step at a time
- After each step: tick it, add a one-line progress note, run lint + typecheck
- Ask before deleting files or changing anything in `src/lib/auth/`
```

Notice that each rule is **specific** ("use `formatPrice()` from `src/lib/money.ts`"), not general ("format prices properly").

## Never put these in CLAUDE.md

- **Secrets**: API keys, passwords, tokens. This file is committed to git.
- Long documentation. Instead, point to it: "API docs: see `docs/api.md`".
- Personal preferences that teammates don't share.

## Checklist

- [ ] Under ~80 lines
- [ ] Has the exact commands
- [ ] Rules are specific, with file paths
- [ ] No secrets
- [ ] Updated when Claude repeats a mistake
