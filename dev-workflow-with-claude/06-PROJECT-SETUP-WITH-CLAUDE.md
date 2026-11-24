# From Empty Folder to First "Go Build It"

How to set up a project so Claude understands it from day one, and keeps understanding it for weeks.

The example is the **BE site**: a website for a service business, built from a client's copy PDF. It has a set of service pages, and **Fauna** is the first one built.

> **How this differs from [chapter 2](02-NEW-PROJECT-GUIDE.md):** chapter 2 keeps everything in one `PLAN.md`. A content-heavy website has a long sitemap and a long checklist, so here the work is split into three files: `Plan.md` (what and why), `Steps.md` (the ordered checklist) and `PROGRESS.md` (the daily log). Use whichever fits your project.

---

## Before you touch anything

The biggest mistake people make with Claude is opening a chat and typing "build me a website". You get *something*, but it's Claude's guess at what you wanted.

Your job isn't to write the code. **Your job is to set up the folder so Claude can't misunderstand you.** That takes an hour or two, and it saves you weeks.

---

## Step 1: Create the folder and start git

```bash
mkdir be-site
cd be-site
git init
```

Do this first, before any files. Git is your undo button, and you want it from the very beginning.

---

## Step 2: Put the source material in

Whatever the client gave you goes in first, untouched:

```
be-site/
└─ source/
   ├─ BE Website Copy FINAL v8.pdf
   ├─ logo.svg            (if you have it)
   └─ old-site-crawl.csv  (later, from Screaming Frog)
```

**Rule: never edit these.** They're the truth. Everything else is built from them.

---

## Step 3: Write Plan.md (with Claude's help)

This is the first time you open Claude. You're **not building anything yet**, just thinking together:

> Read source/BE Website Copy FINAL v8.pdf fully. Don't write any code.
> Draft a Plan.md with: project summary, tech stack, sitemap with URLs,
> phases, decisions still needed, missing content, and risks.

Then **you** read it carefully and fix it. This is where you catch Claude's wrong guesses, like a URL it made up or a page it missed. Plan.md is *yours*: Claude drafts it, you own it.

---

## Step 4: Write Steps.md

Same approach:

> Based on Plan.md, write Steps.md: an ordered checklist from setup to launch,
> grouped in stages. Add a "Checklist for every page" at the bottom.

Read it again and ask yourself: *"Is this the order I'd actually do it in?"* Move things around until the answer is yes.

---

## Step 5: Write CLAUDE.md

This is the rulebook. Write it **yourself**, or have Claude draft it and then edit it hard. Keep it short; it should fit on one screen. It covers:

- what the project is and the tech stack
- what to read at the start of each session
- how to do one task (plan → build → verify → commit)
- what Claude must **never** decide alone
- what "done" means

It must also say what your two daily phrases mean, or Claude won't know what to do with them:

```md
## Session phrases
- "Start session": read Plan.md, Steps.md and PROGRESS.md and check git status.
  Then report where we are, what's blocked and what's next. Don't change anything.
- "Wrap up": add today's entry to PROGRESS.md (done, next, blocked),
  tick finished items in Steps.md, and list anything not yet committed.
```

If you prefer skills, the `resume-work` skill from [chapter 3](03-SKILLS-GUIDE.md) does the same job as "Start session".

See [CLAUDE.md](CLAUDE.md) in this folder for a full example. Once it works, reuse it as your template for future projects.

---

## Step 6: Create PROGRESS.md and .gitignore

PROGRESS.md starts almost empty:

```md
# Progress log
## 2026-11-24
- Done: Plan, Steps, CLAUDE.md written
- Next: Phase 0 – questions, then content inventory
```

`.gitignore` keeps `node_modules`, `.next` and `.env` files out of git.

---

## Step 7: Your first commit

```bash
git add .
git commit -m "Project setup: plan, steps, rules"
```

`git add .` is fine here: it's your own first commit and you know every file in the folder. Now if anything goes wrong later, you can always get back to this clean starting point.

---

## Step 8: Quick sanity check

Open a **brand new** Claude session and type:

> Start session.

If Claude answers with where you are, what's blocked and what's next, without you explaining anything, your setup works. If it gets something wrong, **fix the file, not the chat.**

---

## Step 9: The kickoff instruction

One bit of honest advice: **don't say "finish the whole site".** Even with perfect files, one giant instruction leads to one giant mess that's hard to review.

Instead, give Claude the whole goal **but with checkpoints**:

> Follow CLAUDE.md, Plan.md and Steps.md to build this site.
>
> Work one Steps.md item at a time. For each item: plan, wait for my OK,
> build, verify, commit, update Steps.md and PROGRESS.md.
>
> Start with Phase 0: ask me any questions the PDF and Plan.md don't answer.
> Then build the content inventory from the PDF.
>
> Stop and ask me before anything in the "Stop and ask" list.

Claude knows the full goal, but you stay in control at every step.

---

## Every day after that

You only need three short messages.

**Morning:**

> Start session.

**During the day**, after you OK each plan:

> Approved. Go.

or, once you trust a pattern (like the 8 service pages after Fauna):

> Do the next 3 service pages the same way as Fauna. Commit each one separately.

**Evening:**

> Wrap up.

That's it. The files hold everything else.

---

## How the trust builds over time

- **Week 1:** you review every plan closely. You'll find things to fix in CLAUDE.md, and that's normal.
- **Week 2:** the rules are tight, so you approve faster and give bigger batches.
- **Week 3+:** you mostly review results, not plans.

That long-term working relationship doesn't come from Claude remembering you. **It comes from your files getting better every time something goes wrong.** Every fix you make to CLAUDE.md is a lesson Claude never needs to learn again.

---

## The whole path at a glance

```
1. mkdir + git init
2. source/  ← client PDF, untouched
3. Plan.md      ← Claude drafts, you own
4. Steps.md     ← Claude drafts, you reorder
5. CLAUDE.md    ← you write the rules
6. PROGRESS.md + .gitignore
7. First commit
8. "Start session." → check it understands
9. Kickoff prompt with checkpoints
10. Daily: Start session → approve tasks → Wrap up
```
