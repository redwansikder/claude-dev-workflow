# CLAUDE.md & PLAN.md Checklist

## CLAUDE.md

### Must have
- Project name and one-line description
- Tech stack (framework, language, styling, UI library)
- Commands: dev, build, lint (and test, if any)
- Folder structure (where things go)
- Code style rules that Claude can't guess
- How to work (read PLAN.md, one step at a time, check before done)
- What to ask before doing (delete files, database changes, new packages)

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
- Current task details (those go in PLAN.md)

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
- Roadmap / phases (for new projects)

### Never
- Project-wide rules (those go in CLAUDE.md)
- Old finished phases (move them to `docs/plans/` when done)
- Long essays or code dumps
- Secrets
