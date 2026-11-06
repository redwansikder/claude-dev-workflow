# Project: DocChat, ask questions about your documents

A beginner AI engineering project. It uses Next.js and TypeScript, so all the new learning is on the AI side. The phases were chosen by working backwards: start from the one thing users need, and keep asking "what must exist before this works?"

A chat app where users upload documents, such as a company HR policy, a university admission guide or product manuals. They ask questions in English or Bangla and get answers **with the source shown**.

This kind of system is called RAG (Retrieval-Augmented Generation), and it's the most common AI engineering job in companies. It teaches LLM APIs, embeddings, vector search, prompt design and evaluation.

**Stack**
- Next.js (App Router) + TypeScript + Tailwind
- **Claude API** (`@anthropic-ai/sdk`): `claude-haiku-5-5` while developing (cheap and fast), `claude-sonnet-5-5` for final answers
- **Embeddings:** Voyage AI, or a free local model through Transformers.js
- **Vector database:** PostgreSQL + `pgvector` (Supabase free tier)

---

## Finding the phases (working backwards)

**What users must be able to do:** ask a question, get an answer, see which document and page it came from, upload their own files, see chat history, get answers in Bangla.

**The final thing:** a user asks a question and gets a **correct answer with its source**.

```
Correct answer with source
   ↑ LLM answers using the right pieces of text
      ↑ search finds the right pieces (retrieval)
         ↑ documents split into pieces and stored as embeddings
            ↑ LLM API working in the app
               ↑ project setup
```

**Can it wait or be done by hand?**
- **Uploading your own files: yes, it can wait.** Load 3–5 fixed sample PDFs with a script first.
- **Login and chat history: they can wait.**
- **Checking answer quality: no, this one must stay.** An AI app with no tests gives wrong answers silently. This is the risky part, and it's what separates an AI *engineer* from someone who only calls an API.

---

## Roadmap

```md
- [ ] Phase 0: Setup
- [ ] Phase 1: Talk to the LLM
- [ ] Phase 2: Turn documents into searchable pieces
- [ ] Phase 3: Answer with sources   ← core works here
- [ ] Phase 4: Measure quality (evals)
- [ ] Phase 5: Users upload their own documents
- [ ] Phase 6: Make it smarter (tool use)
- [ ] Phase 7: Deploy, cost and safety
```

---

### Phase 0: Setup
- [ ] 1. Create a Next.js + TypeScript + Tailwind project and push it to GitHub
- [ ] 2. Get a Claude API key and put it in `.env.local`. Add `.env.example` with no real values
- [ ] 3. Create a Supabase project and enable the `pgvector` extension
- [ ] 4. Write CLAUDE.md (project rules) and PLAN.md (current phase + Roadmap)

**Done when:** the dev server runs and the keys load without being committed to git.

### Phase 1: Talk to the LLM
- [ ] 1. API route `POST /api/chat` that sends one message to Claude and returns the reply
- [ ] 2. A simple chat UI: message list and input box
- [ ] 3. **Streaming:** show the answer word by word
- [ ] 4. Add a **system prompt** ("You are a helpful assistant for…") and try changing it
- [ ] 5. Log the **token usage** of each call and work out the cost per message

**Done when:** you can chat, answers stream, and you know what one message costs.
**You learn:** messages API, system prompts, streaming, tokens and cost.

### Phase 2: Turn documents into searchable pieces
- [ ] 1. A script that reads 3–5 sample PDFs and extracts the text
- [ ] 2. **Chunking:** split the text into pieces of about 500 words with a little overlap, keeping the filename and page number with each piece
- [ ] 3. Create an **embedding** for each chunk and store chunk + embedding + source in `pgvector`
- [ ] 4. A test script: type a question, embed it, and print the 5 closest chunks

**Done when:** for 10 test questions, the right chunk shows up in the top 5.
**You learn:** embeddings, vector similarity, chunk size trade-offs.

### Phase 3: Answer with sources ← the final thing
- [ ] 1. Connect it up: question → find top chunks → put them in the prompt → Claude answers
- [ ] 2. Prompt rules: "Answer only from the given text. If it isn't there, say you don't know."
- [ ] 3. Show the sources under each answer (file name + page)
- [ ] 4. Test questions the documents **don't** answer, to check it says "I don't know" instead of making something up
- [ ] 5. Test Bangla questions on English documents

**Done when:** answers are correct, show sources, and the app doesn't invent answers.
**You learn:** RAG, grounding, reducing hallucinations.

### Phase 4: Measure quality (evals)
- [ ] 1. Write 30 test questions with expected answers in `evals/questions.json`
- [ ] 2. A script that runs every question and saves the answers
- [ ] 3. **LLM-as-judge:** Claude grades each answer as correct, partly correct or wrong
- [ ] 4. Get a score (for example 22/30), then change one thing (chunk size, top-k, prompt) and re-run
- [ ] 5. Write down which change helped, in the plan's Decisions section

**Done when:** you have a score and have improved it at least once with a measured change.
**You learn:** evals. This is the skill most beginners skip, and the one interviewers ask about most.

### Phase 5: Users upload their own documents
- [ ] 1. An upload page for PDFs (size limit, file type check)
- [ ] 2. Process the upload in the background (extract → chunk → embed) with a "processing…" status
- [ ] 3. Let users pick which documents to search
- [ ] 4. Delete a document and its chunks

**Done when:** a new user uploads a PDF and gets answers from it within a minute.

### Phase 6: Make it smarter (tool use)
- [ ] 1. Turn search into a **tool** that Claude decides when to call, instead of always searching
- [ ] 2. Let Claude search more than once for questions with several parts
- [ ] 3. Add a second tool, for example `calculate` or `get_today_date`
- [ ] 4. Re-run the Phase 4 evals and compare the score

**Done when:** multi-part questions work better and the eval score didn't drop.
**You learn:** tool use and basic agents.

### Phase 7: Deploy, cost and safety
- [ ] 1. Deploy to Vercel
- [ ] 2. Rate limit each user, for example 20 questions a day
- [ ] 3. **Prompt caching** for the system prompt, to cut cost
- [ ] 4. Basic safety: reject prompt-injection text in uploads ("ignore previous instructions…")
- [ ] 5. A small dashboard: questions per day, cost per day, average eval score

**Done when:** it's live, cost is capped, and you can share the link in your CV.

---

## After this project you can say

- "I built a RAG app with Next.js, Claude and pgvector."
- "I measured answer quality with a 30-question eval set and improved it from X to Y."
- "I used tool use, streaming and prompt caching, and kept cost under ৳Z a month."

That third line comes from the work in Phases 4 and 7, which most beginner portfolios don't have.
