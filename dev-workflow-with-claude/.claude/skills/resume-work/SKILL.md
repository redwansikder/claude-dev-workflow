---
name: resume-work
description: Use at the start of a chat, after a break, power cut, or PC shutdown, or when the user asks "where was I" / "what's next". Checks git and PLAN.md and reports where work stopped. Read-only.
---

# Resume work

Do not edit any files in this skill. Only check and report.

1. Run `git status` and `git diff --stat` to find uncommitted changes.
2. Read `PLAN.md`. Find the first unchecked step and the last progress note.
3. If there are uncommitted changes:
   - Say which files changed and whether they look related to the current step.
   - Run `npm run build` to see if the code still compiles.
   - Recommend one: **keep** (changes look complete and build passes) or **restore** (`git restore .`, changes are half-done or broken). Let the user decide.
4. Report in this format:

```
Last done:   Step N — <progress note>
Next step:   Step M — <step text>
Uncommitted: <none | list of files + keep/restore advice>
Open questions: <from PLAN.md, if any>
```

5. Ask: "Continue with step M?"
