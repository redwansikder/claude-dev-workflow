# CLAUDE.md, PLAN.md & Skills Checklist

## CLAUDE.md

### Must have
- Project name and one-line description
- Tech stack (framework, language, styling, UI library)
- Commands: dev, build, lint (and test, if any)
- Folder structure (where things go)
- Code style rules that Claude can't guess
- How to work (read PLAN.md, one step at a time, check before done)
- What to ask before doing (delete files, database changes, new packages)
- Under ~80 lines
- Updated when Claude repeats a mistake (add one line to fix it)

### Recommended
- List of skills and when to use them
- Gotchas / known problems
- Things not to touch (legacy folders, auth, config)
- Environment setup (`.env.example`, required keys — no real values)
- Backend / API info (base URL variable, separate repo)
- Testing rules (what must be tested)
- Links to longer docs (`docs/api.md`) instead of copying them

### Never
- Secrets (API keys, passwords, tokens)
- Long documentation
- Obvious rules ("write clean code")
- Things Claude can see from the code
- Personal preferences that teammates don't share
- Current task details (those go in PLAN.md)
- Full skill steps (list the skill name only; steps go in the skill)

---

## PLAN.md

### Must have
- Task name
- Goal (1–2 sentences)
- Steps as a checklist (`- [ ]`), small and testable
- Progress notes (one line per finished step)

### Recommended
- Branch name
- "Done when" (clear finish rules)
- "Out of scope" (what not to do)
- Decisions made during the task
- Open questions
- Skill names in the steps that need them
- How to work on this plan (`/resume-work` → step → `/finish-step`)
- Roadmap: every task or phase, with finished ones linked to `docs/plans/`
- A `---` line between the task part (moves when done) and the permanent part (Roadmap, how to work)

### Never
- Project-wide rules (those go in CLAUDE.md)
- Old finished tasks (move everything above `---` to `docs/plans/<task>.md` when done)
- Long essays or code dumps
- Secrets

---

## Skills

### Must have
- One skill = one job
- A `description` that says **when** to use it (Claude reads only this to decide)
- Clear numbered steps in the body
- Listed by name in CLAUDE.md

### Never
- Made in advance (wait until you've explained the same steps 3 times)
- Secrets (project skills are committed to git)
