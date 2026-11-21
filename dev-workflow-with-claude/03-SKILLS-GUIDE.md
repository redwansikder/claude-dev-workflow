# Using Skills with CLAUDE.md and PLAN.md

A **skill** is a set of instructions Claude loads **only when it's needed**. `CLAUDE.md` loads in every chat, so skills keep it short and save tokens.

## Each file has its own job

| File | Job | Loaded |
|---|---|---|
| `CLAUDE.md` | Project rules | Every chat |
| `PLAN.md` | Current task and progress | When Claude reads it |
| **Skills** | **How to do one repeated job** | Only when needed |

## How to create a skill

```
project/
├── CLAUDE.md
├── PLAN.md
└── .claude/
    └── skills/
        ├── resume-work/
        │   └── SKILL.md
        └── finish-step/
            └── SKILL.md
```

- `.claude/skills/` in the project: works only in this project, committed to git.
- `C:\Users\<you>\.claude\skills\`: works in **all** your projects.

Each `SKILL.md` has two parts:

```md
---
name: finish-step
description: Use when a PLAN.md step is finished... (says WHEN to use it)
---

# Steps
1. ...
```

- **The top part** (`name`, `description`): Claude reads only this to decide when to use the skill.
- **The body**: the steps. These load only when the skill is used.

Use a skill by typing `/finish-step`, or just say "step 3 is done" and Claude matches the description. Start a new chat after adding a skill.

## 1. Mention your skills in CLAUDE.md

List them only; don't copy the steps:

```md
## Skills
- `resume-work` — use at the start of a chat or after a break
- `finish-step` — use after every PLAN.md step
```

## 2. Use them in PLAN.md steps

```md
## Steps
- [ ] 1. Create `OrderList` component (use `new-component`)
- [ ] 2. Create `GET /api/orders` route (use `api-route`)
- [ ] 3. Connect list to API
```

## 3. Daily routine with skills

```
New chat → /resume-work            (reads PLAN.md, git status, says what's next)
         → "do step 3"             (Claude uses other skills if needed)
         → /finish-step            (lint, build, tick PLAN.md, suggest commit)
         → you: git commit + push
```

After a power cut, it's the same: `/resume-work` checks the half-done changes and suggests keeping or restoring them.

## 4. When to create skills

**New project:**
- **Phase 0 (setup):** none needed except `resume-work` and `finish-step`.
- **Phase 1 and later:** when you've explained the same steps **3 times**, make a skill.

**Existing project:**
- Start with `resume-work` and `finish-step`.
- Add others as patterns repeat (`new-component`, `api-route`, ...).

## 5. Moving rules from CLAUDE.md into a skill

If `CLAUDE.md` gets long, move task-specific rules into a skill.

**Before** (in CLAUDE.md, loaded every chat):
```md
- Components: one per file, props interface on top, export default, add a story file...
```

**After:**
- `CLAUDE.md` keeps one line: `- Creating a component: use the new-component skill`
- The details move to `.claude/skills/new-component/SKILL.md`

The result is a **shorter CLAUDE.md** and **fewer tokens in every chat**.

## Tips

- One skill = one job, with a specific description.
- Don't create skills in advance; make them when a task repeats.
- No secrets in skills, because project skills are committed to git.
- Let Claude write one: `/skill-creator create a project skill called ...`

## Summary

- **CLAUDE.md** = what's true always, plus a list of skills
- **PLAN.md** = what to do now, and can name a skill per step
- **Skills** = how to do a repeated job, loaded only when used

## Samples in this folder

- [.claude/skills/resume-work/SKILL.md](.claude/skills/resume-work/SKILL.md)
- [.claude/skills/finish-step/SKILL.md](.claude/skills/finish-step/SKILL.md)
- [CLAUDE.md](CLAUDE.md) has a "Skills" section
- [PLAN.md](PLAN.md) has "How to work on this plan"
