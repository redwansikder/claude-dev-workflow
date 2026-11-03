# Claude Dev Workflow

A short guide to working with Claude Code on real projects, using three things:

- **`CLAUDE.md`**: project rules Claude reads in every chat
- **`PLAN.md`**: the current task, split into small steps
- **Skills**: reusable instructions Claude loads only when needed

With this setup, a power cut, a restart or a full chat costs you minutes instead of hours, because the state of your work lives in files and not in the chat.

## Read in this order

| # | Chapter | What you learn |
|---|---------|----------------|
| 1 | [Writing a good CLAUDE.md](01-CLAUDE-MD-GUIDE.md) | What goes in CLAUDE.md, and what never does |
| 2 | [Starting a new project](02-NEW-PROJECT-GUIDE.md) | Decide first, then write CLAUDE.md and PLAN.md before any code |
| 3 | [Using skills](03-SKILLS-GUIDE.md) | When to make a skill and how to use it from PLAN.md |
| 4 | [A real project, day by day](04-SCENARIO-MULTI-DAY.md) | Chapters 1–3 in a new project, built over 5 days with 7 interruptions |
| 5 | [Checklist](05-CHECKLIST.md) | A quick review page to keep open while you work |

## Sample files

These are working examples you can copy into your own project.

| File | What it shows |
|------|---------------|
| [CLAUDE.md](CLAUDE.md) | Project rules for a Next.js + TypeScript app |
| [PLAN.md](PLAN.md) | A task in progress, with a Roadmap |
| [docs/plans/login.md](docs/plans/login.md) | A finished task, archived out of PLAN.md |
| [.claude/skills/resume-work/SKILL.md](.claude/skills/resume-work/SKILL.md) | Skill: check git and PLAN.md, report where work stopped |
| [.claude/skills/finish-step/SKILL.md](.claude/skills/finish-step/SKILL.md) | Skill: lint, build, tick the step, suggest a commit message |
| [.claude/settings.json](.claude/settings.json) | Keeps plan-mode plans inside the project (`.claude/plans/`) |

## The daily routine

```
/resume-work  →  "Do step N"  →  check it yourself  →  /finish-step  →  commit + push
```

When a task is done, move it from `PLAN.md` to `docs/plans/`, tick it in the Roadmap with a link, and write the next task.

## Use it in your project

1. Copy `.claude/` into your project root.
2. Write your own `CLAUDE.md` (chapter 1) and `PLAN.md` (chapter 2).
3. Start a chat with `/resume-work`.

## License

[MIT](LICENSE)
