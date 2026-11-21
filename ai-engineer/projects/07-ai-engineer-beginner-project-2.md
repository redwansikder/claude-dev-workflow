# Project 2: JobScout BD, an AI agent that finds jobs that fit you

[Project 1 (DocChat)](06-ai-engineer-beginner-project.md) taught RAG. This one covers what the [roadmap](../03-ai-engineer-roadmap.md) puts in **Phase 3: agents and MCP**, plus **structured output**, which is extracting clean JSON from messy text.

You paste in job posts for React, Next.js, .NET or full-stack roles. The app turns each post into **structured data** (role, stack, salary, experience, location, deadline) and **scores how well it matches your CV**. An **agent** then gives you a weekly shortlist and drafts a cover letter for the jobs you approve.

**What you learn:** structured output (JSON schemas), classification and scoring, tool use, agent loops, MCP servers, human approval steps, evals for extraction, and scheduled jobs.

**Stack**
- Next.js (App Router) + TypeScript + Tailwind
- **Claude API** (`@anthropic-ai/sdk`): `claude-haiku-5-5` for extraction (many posts, needs to be cheap), `claude-sonnet-5-5` for matching and cover letters
- **Zod** for the job schema
- PostgreSQL (Supabase) + Prisma or Drizzle
- **MCP TypeScript SDK** (`@modelcontextprotocol/sdk`) for the MCP server

---

## Finding the phases (working backwards)

**What users must be able to do:**
- add job posts
- see them as clean cards (stack, salary, experience)
- filter by stack and salary
- see a match score with reasons
- get a weekly shortlist
- get a cover letter draft
- track application status
- use the jobs from Claude Desktop or Claude Code (MCP)
- get jobs collected automatically

**The final thing:** *"Show me the jobs I should apply to this week, and why."* That means a **ranked shortlist with reasons**.

```
Ranked shortlist with reasons
   ↑ each job has a match score against my CV
      ↑ each job is structured data (stack, salary, experience…)
         ↑ job posts are in the database
            ↑ job posts are collected
               ↑ project setup
```

**Can it wait or be done by hand?**
- **Automatic job collection: done by hand at first.** You **paste** job text from bdjobs or LinkedIn yourself. Many job sites don't allow scraping in their terms, so a pasted-text or RSS approach is safer anyway.
- **Cover letters, application tracking and MCP: they can wait.** They all build on the shortlist.
- **Extraction accuracy: no, this one must stay, and it's risky.** If salary or stack is extracted wrong, every score after it is wrong. Test it early (Phase 2).

---

## Roadmap

```md
- [ ] Phase 0: Setup
- [ ] Phase 1: Paste a job → structured data
- [ ] Phase 2: Measure extraction quality (evals)
- [ ] Phase 3: Match score against my CV
- [ ] Phase 4: Weekly shortlist agent   ← core works here
- [ ] Phase 5: Cover letters with human approval
- [ ] Phase 6: MCP server
- [ ] Phase 7: Automate, deploy, cost
```

---

### Phase 0: Setup
- [ ] 1. Create a Next.js + TypeScript + Tailwind project and push it to GitHub
- [ ] 2. Claude API key in `.env.local`, plus `.env.example` with no real values
- [ ] 3. Supabase Postgres + Prisma/Drizzle with a `jobs` table (raw text, created date)
- [ ] 4. Collect **30 real job posts** by hand into a `samples/` folder: about 10 frontend, 10 .NET and 10 full-stack
- [ ] 5. Write CLAUDE.md and PLAN.md

**Done when:** the app runs, the database connects, and the 30 sample posts are saved.

### Phase 1: Paste a job → structured data
- [ ] 1. Define a **Zod schema**: `title, company, roleType (frontend|backend|fullstack), stack[], minExp, maxExp, salaryMin, salaryMax, location, remote, deadline`
- [ ] 2. API route that sends the raw text to Claude and asks for JSON **matching the schema** (structured output / tool with an input schema)
- [ ] 3. Validate the reply with Zod. If it fails, retry once with the error message
- [ ] 4. Handle messy cases: "Negotiable" salary → `null`, Bangla text, "2–4 years"
- [ ] 5. A page: paste box → saved job shown as a card

**Done when:** pasting any of the 30 samples gives a valid, saved job card.
**You learn:** structured output, schemas, validation and retry, handling messy real-world text.

### Phase 2: Measure extraction quality (evals) ← the risky link
- [ ] 1. Hand-write the **correct answers** for the 30 samples in `evals/expected.json`
- [ ] 2. A script that extracts all 30 and compares them **field by field** (stack, salary, experience…)
- [ ] 3. A report: accuracy per field, for example `salary: 26/30, stack: 29/30`
- [ ] 4. Fix the worst field (better prompt, examples in the prompt, schema descriptions) and re-run
- [ ] 5. Compare Haiku and Sonnet on accuracy **and** cost, and record the choice in Decisions

**Done when:** every field is at least 90% accurate, and you know which model you use and why.
**You learn:** evals with **exact answers**. This is different from Project 1's LLM-as-judge, and the two are complementary.

### Phase 3: Match score against my CV
- [ ] 1. Store your CV and preferences: stack, years, minimum salary, remote or Dhaka
- [ ] 2. **Rules first:** hard filters in code (salary too low, needs 8+ years) with no AI
- [ ] 3. **Then AI:** Claude gives a 0–100 score with 3 short reasons (strong matches, gaps) as JSON
- [ ] 4. Show the score and reasons on each card, and sort by score
- [ ] 5. Check 10 scores by hand: do you agree? Adjust the prompt if not

**Done when:** the top 5 jobs by score are the ones you'd actually pick yourself.
**You learn:** mixing plain code rules with LLM judgment, and when *not* to use AI.

### Phase 4: Weekly shortlist agent ← the final thing
- [ ] 1. Write tools as plain functions: `search_jobs(filters)`, `get_job(id)`, `get_my_profile()`
- [ ] 2. An **agent loop**: Claude gets the goal ("pick the best 5 jobs this week") and calls tools until it's done
- [ ] 3. **Stop conditions:** a maximum of 10 tool calls and a time limit
- [ ] 4. **Log every step** (tool called, input, result) and show it in the UI as a timeline
- [ ] 5. Output: a ranked shortlist with reasons, saved to the database

**Done when:** one click produces a sensible shortlist, and you can see every step the agent took.
**You learn:** tool use, agent loops, stop conditions, making an agent's work visible.

### Phase 5: Cover letters with human approval
- [ ] 1. A "Draft cover letter" button on shortlisted jobs
- [ ] 2. Claude writes one using the job data and your CV, short and specific, with no invented experience
- [ ] 3. **Human in the loop:** draft → you edit → you mark it "approved". Nothing is ever sent automatically
- [ ] 4. Application status per job: saved → applied → interview → rejected or offer
- [ ] 5. Check: does the letter claim any skill that isn't in your CV? Add a check that flags it

**Done when:** you've used it for real applications, and no letter claims a fake skill.
**You learn:** approval steps and catching hallucinations in generated text.

### Phase 6: MCP server
- [ ] 1. Build an **MCP server** with the MCP TypeScript SDK that exposes the same tools: `search_jobs`, `get_job`, `get_my_profile`, `update_status`
- [ ] 2. Connect it to **Claude Desktop** or **Claude Code**
- [ ] 3. Try it: *"Which .NET jobs this week pay over 80k and are remote?"* from Claude Desktop
- [ ] 4. Make `update_status` ask for confirmation before changing anything
- [ ] 5. Write a README section on how to install and connect it

**Done when:** you can manage your job search by talking to Claude Desktop.
**You learn:** MCP, the skill most in demand for AI engineers right now.

### Phase 7: Automate, deploy, cost
- [ ] 1. Add jobs from **RSS feeds or company career pages** that allow it, instead of pasting only
- [ ] 2. **Remove duplicates** (the same job on two sites) using company + title + similarity
- [ ] 3. A **scheduled job** (Vercel Cron) that runs extraction and the weekly agent every Friday
- [ ] 4. **Batch** extraction and use **prompt caching** to cut cost, and log cost per week
- [ ] 5. Deploy, and add a dashboard: jobs per week by role type (frontend / .NET / full-stack)

**Done when:** the shortlist arrives every Friday without you doing anything, and weekly cost is known.
**Bonus:** that dashboard gives you real month-by-month job counts for Bangladesh, so next November you can compare against this year.

---

## Project 1 vs Project 2

| | Project 1: DocChat | Project 2: JobScout BD |
|---|---|---|
| Main skill | RAG (search + answer) | Structured output + agents |
| Evals | LLM-as-judge (open answers) | Exact match (field by field) |
| AI decides | What to answer | Which tools to call, and when |
| New tech | Embeddings, pgvector | Zod schemas, agent loop, MCP server |
| Human role | Asks questions | Approves before anything happens |

## After this project you can say

- "I built an agent with tool use, stop conditions and a visible step log."
- "I built an MCP server and used it from Claude Desktop."
- "My extraction pipeline is over 90% accurate per field, measured with an eval set."
- "I chose Haiku over Sonnet for extraction based on measured accuracy and cost."
