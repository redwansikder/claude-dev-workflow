# Save the "finding phases" example as a markdown file

## Context
The last chat answer explained how to find a project's phases: list what users must
be able to do, then work backwards from the core feature (doctor appointment app example).
Save it in the repo so it's part of the learning material.

## Changes
1. **Create `extras/finding-phases.md`** with the last message's content:
   - Title: `# Finding Your Phases`
   - One intro line linking it to chapter 2 (`../02-NEW-PROJECT-GUIDE.md`)
   - Steps 1–5 of the doctor appointment example, the final Roadmap, and "What to remember"
   - Text kept as in the chat answer; only the headings adjusted to fit a standalone file
2. **Add an "Extras" section to `README.md`** after "Sample files", with one row
   linking `extras/finding-phases.md`, so readers can find it.

`extras/` is a new folder for side topics, kept apart from the numbered chapters
and from `docs/plans/` (which holds archived task plans).

## Verification
- Open `extras/finding-phases.md` in the VS Code markdown preview: the headings, code blocks and table render correctly
- The README link and the link back to chapter 2 both open the right file
