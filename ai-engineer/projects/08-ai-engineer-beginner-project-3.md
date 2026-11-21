# Project 3: KhataSnap, from receipt photo to bookkeeping entry

[Project 1](06-ai-engineer-beginner-project.md) taught **RAG** and [Project 2](07-ai-engineer-beginner-project-2.md) taught **agents and MCP**. Project 3 covers what's still missing for a junior AI engineer: **vision (reading images)**, **connecting to a real business system safely**, and **production skills** like tracing, prompt versions, fallbacks and evals that run automatically. That last group is the [roadmap's](../03-ai-engineer-roadmap.md) Phase 4.

A shop owner or office accountant **takes a photo** of a receipt, bill or supplier invoice. It can be printed or handwritten, in English or Bangla. The app reads it, fills in an expense form (vendor, date, amount, VAT, category), the owner **checks and approves** it, and the expense is recorded in the bookkeeping system, for example **Zoho Books**.

**What you learn:** vision / multimodal input, OCR-quality evals, confidence and review screens, safely writing to a real business API, tracing, prompt versioning, fallback models, feedback loops, and evals that run automatically on GitHub.

**Stack**
- Next.js (App Router) + TypeScript + Tailwind, as a **PWA** so the phone camera works
- **Claude API** with image input: `claude-sonnet-5-5` for reading receipts, `claude-haiku-5-5` for category suggestions
- Zod for the expense schema
- PostgreSQL (Supabase) + Supabase Storage for photos
- **Zoho Books API** (OAuth), always against a **test organization** first
- GitHub Actions for automatic evals

---

## Finding the phases (working backwards)

**What users must be able to do:**
- photograph a receipt
- see the fields filled in
- fix mistakes
- see which fields the AI is unsure about
- pick a category
- get warned about duplicates
- approve
- have it recorded in the books
- see monthly totals
- export to CSV

**The final thing:** *an expense is recorded correctly in the books from a photo, after the owner approves it.*

```
Expense recorded in the books
   ↑ owner approved it
      ↑ correct expense fields (vendor, date, amount, VAT, category)
         ↑ AI read the photo correctly         ← risky: messy Bangla receipts
            ↑ photo uploaded
               ↑ setup
```

**Check each link:**

| Link | Decision |
|---|---|
| Reading the photo | **Must stay, and it's the riskiest part.** Crumpled, handwritten Bangla receipts may fail. **Test it first, before building any UI.** |
| Owner approval | **Must stay.** Money data never goes in without a human check. |
| Recording in Zoho Books | **Done by hand at first:** export a CSV and import it into Zoho yourself. The real API comes later. |
| Duplicates, monthly totals | Can wait. |

---

## Roadmap

```md
- [ ] Phase 0: Setup + collect 40 real receipt photos
- [ ] Phase 1: Risk test: can the AI read receipts? (evals first)
- [ ] Phase 2: Snap → review screen
- [ ] Phase 3: Categories + duplicate check
- [ ] Phase 4: Approve → CSV export   ← core works here (manual import)
- [ ] Phase 5: Real Zoho Books integration
- [ ] Phase 6: Production basics (tracing, prompt versions, fallbacks)
- [ ] Phase 7: Feedback loop + automatic evals in CI
- [ ] Phase 8: Deploy, privacy, cost
```

---

### Phase 0: Setup
- [ ] 1. Next.js + TypeScript + Tailwind project as a PWA, pushed to GitHub
- [ ] 2. Claude API key and Supabase keys in `.env.local`, plus `.env.example`
- [ ] 3. Collect **40 real receipt photos** in `samples/`, with a mix of printed, handwritten, Bangla, crumpled, dark and angled photos
- [ ] 4. Write CLAUDE.md and PLAN.md

**Done when:** the app runs and you have 40 varied receipt photos.

### Phase 1: Risk test, can the AI read receipts? ← evals before UI
- [ ] 1. Hand-write the correct answers for all 40: `vendor, date, total, vat, currency, items[]` in `evals/expected.json`
- [ ] 2. A script that sends each image to Claude and asks for JSON matching a Zod schema
- [ ] 3. Compare field by field and report, for example `total: 37/40, date: 33/40, vendor: 30/40`
- [ ] 4. Ask the model for a **confidence level per field** (high / low), and check whether "low" really matches the wrong answers
- [ ] 5. Try fixes: image resize or rotate, prompt examples, Bangla number hints (৳, ০–৯), then re-run

**Done when:** `total` is right at least 95% of the time, and low-confidence flags catch most mistakes.
**You learn:** vision input, image preprocessing, and **testing the risky part before building anything around it**.

### Phase 2: Snap → review screen
- [ ] 1. Upload page that opens the phone camera, with an image preview
- [ ] 2. Save the photo to Supabase Storage and run extraction
- [ ] 3. **Review form:** filled fields shown next to the photo, all editable
- [ ] 4. **Highlight low-confidence fields** in yellow so the owner checks those first
- [ ] 5. Loading states, plus a friendly error when the photo is too blurry ("Please retake")

**Done when:** you can snap a receipt on your phone and see a filled, editable form in under 10 seconds.
**You learn:** building UI around uncertain AI output. Your frontend skills are an advantage here.

### Phase 3: Categories + duplicate check
- [ ] 1. A fixed category list (Rent, Utilities, Transport, Office supplies, Food…), later loaded from Zoho's chart of accounts
- [ ] 2. Haiku suggests a category, and the owner can change it
- [ ] 3. **Remember corrections:** if "Pathao" was changed to Transport once, use it automatically next time, **in code with no AI**
- [ ] 4. **Duplicate check:** same vendor + same amount + date within 3 days → warning
- [ ] 5. Eval: category accuracy on the 40 samples

**Done when:** most categories are right without editing, and the same receipt uploaded twice shows a warning.
**You learn:** combining simple stored rules with AI, so AI isn't used where a lookup is enough.

### Phase 4: Approve → CSV export ← the final thing, by hand
- [ ] 1. "Approve" button: the expense moves from *draft* to *approved*
- [ ] 2. Approved expenses are locked, and editing needs an explicit "unlock"
- [ ] 3. Export approved expenses as **CSV in Zoho Books' import format**
- [ ] 4. Import the CSV into a **Zoho test organization** by hand and check that it looks right
- [ ] 5. A monthly view: total by category

**Done when:** a real week of receipts goes from photos to the Zoho test org through CSV, with nothing typed by hand.
**You learn:** finishing the core value with a manual step first, and automating it later.

### Phase 5: Real Zoho Books integration
- [ ] 1. Zoho **OAuth** connection, with tokens stored server-side and refreshed automatically
- [ ] 2. "Record in Zoho" creates the expense through the API **only for approved items**
- [ ] 3. **Idempotency:** save the Zoho expense ID, so a retry or double-click never creates two entries
- [ ] 4. Attach the receipt photo to the Zoho expense
- [ ] 5. Handle failures: Zoho down, token expired, or validation error, by showing the error and keeping it as "approved, not yet recorded"

**Done when:** clicking "Record" twice still makes exactly one Zoho expense, with the photo attached.
**You learn:** safe writes to real business systems: OAuth, idempotency, failure states. Most AI demos skip this, and real companies care about it most.

### Phase 6: Production basics
- [ ] 1. **Tracing:** log every AI call with prompt version, model, input size, tokens, latency, cost and result
- [ ] 2. **Prompt versioning:** keep prompts in files (`prompts/extract-v3.md`) and store the version used with each expense
- [ ] 3. **Timeouts and retries** with backoff, and a **fallback model** if the main one fails or times out
- [ ] 4. **Limits:** maximum image size, and a maximum number of calls per user per day
- [ ] 5. An admin page: calls per day, error rate, average latency, cost per receipt

**Done when:** for any expense you can see exactly which prompt and model produced it, and what it cost.
**You learn:** observability and reliability, the "production" part of AI engineering.

### Phase 7: Feedback loop + automatic evals in CI
- [ ] 1. Every time the owner **edits a field** before approving, save the photo, the AI's answer and the corrected answer
- [ ] 2. A script that adds corrected receipts to the eval set (with permission, and with personal details removed)
- [ ] 3. **GitHub Action:** when a file in `prompts/` changes, run the evals automatically
- [ ] 4. **Fail the pull request** if `total` accuracy drops below 95%
- [ ] 5. Show the eval score history in the README or admin page

**Done when:** a bad prompt change is blocked by CI before it reaches users.
**You learn:** regression testing for AI, the habit that separates engineers from prompt tinkerers.

### Phase 8: Deploy, privacy, cost
- [ ] 1. Deploy to Vercel, with Supabase in production mode
- [ ] 2. **Each shop sees only its own data** (row-level security in Supabase)
- [ ] 3. Delete photos after Zoho has them, or after 90 days
- [ ] 4. Cost cap per shop per month, with a warning at 80%
- [ ] 5. A README covering architecture, eval scores, cost per receipt, and the trade-offs you made

**Done when:** two test shops use it without seeing each other's data, and the monthly cost is known and capped.

---

## Projects 1, 2 and 3 together

| | Project 1: DocChat | Project 2: JobScout BD | Project 3: KhataSnap |
|---|---|---|---|
| Main skill | RAG | Structured output + agents + MCP | Vision + real integration + production |
| Input | Text documents | Pasted text | **Photos** |
| AI writes to | Chat answer | Your own database | **A real business system** (Zoho Books) |
| Evals | LLM-as-judge | Exact match | Exact match + **automatic in CI** |
| Safety focus | No made-up answers | No fake skills in letters | Approval, idempotency, data isolation |
| Production skills | Rate limit, caching | Cron, batching | **Tracing, prompt versions, fallbacks, feedback loop** |

## After this project you can say

- "I built a vision pipeline that reads Bangla and English receipts with 95%+ accuracy on totals."
- "It writes to Zoho Books safely, with human approval and idempotent API calls."
- "Every AI call is traced with prompt version, cost and latency."
- "Prompt changes are tested automatically in CI, and a drop in accuracy blocks the merge."
