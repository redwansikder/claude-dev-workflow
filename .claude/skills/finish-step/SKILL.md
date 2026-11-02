---
name: finish-step
description: Use when a PLAN.md step is finished, or the user says "step N is done" / "finish this step". Runs lint and build, ticks the step in PLAN.md, adds a progress note, and suggests a commit message.
---

# Finish a PLAN.md step

1. Run `npm run lint`, then `npm run build`.
   - If either fails, fix the problem first, then run both again.
   - Do not tick the step until both pass.
2. In `PLAN.md`, change the current step from `- [ ]` to `- [x]`.
3. Under "Progress notes", add one line:
   `- YYYY-MM-DD: Step N done. <one sentence on what changed>`
4. If anything is undecided, add it under "Open questions".
5. Run `git status` and show the changed files.
6. Suggest a commit message: `step N: <short summary>`.
7. Do not commit or push unless the user says so.
