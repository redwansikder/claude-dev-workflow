# AI Engineer Roadmap

Your frontend background helps more than you might think. Much AI Engineer work is product work: streaming chat UIs, tool-call interfaces, latency and user experience. Few AI people do that part well. Here is a path of about **6–9 months**, part time, built on what you already know.

## Phase 0: Get stronger at backend (3–4 weeks)
Your backend knowledge is "some". Make it solid first, because most AI work happens on the server.
- **Postgres:** schema design, indexes, transactions. Use Prisma or Drizzle.
- **API design:** auth, rate limiting, error handling, background jobs (queues).
- **Basic Python:** enough to read examples and write scripts. Most AI papers, tutorials and eval tools use Python. Stay TypeScript-first for now.
- **Docker** and deploying one service, on Vercel plus a separate host like Railway or Fly.

## Phase 1: LLM basics (3–4 weeks)
- How LLMs work at a working level: tokens, context window, temperature, why they hallucinate. You don't need to train models.
- **Call the APIs directly** (Anthropic, OpenAI) before using frameworks: messages, system prompts, streaming, structured JSON output, prompt caching.
- **Tool use / function calling.** This is the base of agents.
- **Vercel AI SDK.** It fits your Next.js skills well.

**Project 1:** a streaming chat app in Next.js that can call 2–3 tools, such as weather, a calculator and a database lookup. Show the tool calls in the UI.

## Phase 2: RAG (4–6 weeks)
- Embeddings and vector search. Start with **pgvector** in the Postgres you already use.
- Splitting documents into chunks, metadata filters, hybrid search (keyword plus vector), reranking.
- Citations in answers, and handling "I don't know" cases.
- **Evals:** build a test set of questions with expected answers and measure retrieval quality. Most tutorials skip this, and it's what separates juniors from hireable engineers.

**Project 2:** "Chat with your docs." Upload PDFs or markdown, get answers with citations, and include an eval script that reports accuracy before and after each improvement.

## Phase 3: Agents and MCP (4–6 weeks)
- Agent loops: plan, act, observe. Retries, stop conditions, human approval steps.
- Agent frameworks: **Claude Agent SDK**, LangGraph, or the OpenAI Agents SDK. Learn one well.
- **MCP:** write an MCP server in TypeScript (`@modelcontextprotocol/sdk`) that exposes tools and resources, then connect it to Claude Desktop or Claude Code.
- Memory, long tasks, and splitting work across multiple agents, plus when *not* to use an agent.

**Project 3:** an MCP server for a real system. For example, wrap a CRM or bookkeeping API you use at work, or GitHub. Then build an agent that uses it for a multi-step task.

## Phase 4: Production skills (ongoing, 4+ weeks)
This is what gets you hired instead of just "knowing AI":
- **Observability:** tracing tools like Langfuse, LangSmith or Helicone.
- **Cost and latency:** caching, choosing cheaper models for easy steps, streaming.
- **Safety:** prompt injection, guardrails, permission limits for agent tools.
- **Eval pipelines in CI:** run them on every prompt change, like tests.

**Project 4 (capstone):** take one earlier project to production quality. That means a deployed app with auth, tracing, evals, a cost dashboard, and a README that explains the tradeoffs you made.

## Making the switch
1. **Start at your current job.** Offer to add one AI feature, such as search, summarization or a support assistant. "Shipped an AI feature to production" counts for more than any course.
2. **Target the title "AI Engineer" or "Full-stack AI Engineer".** Many of these roles want strong frontend skills. Avoid "ML Engineer" roles for now, since those need math and model training.
3. **Write about it in public.** Short posts like "What I learned building RAG evals" get noticed.
4. **Portfolio:** 3–4 deployed projects on GitHub, each with evals and a clear README.

## What to skip for now
- Training models from scratch, deep math, and PyTorch. That's ML Engineer work.
- Collecting courses. Build something every week instead.
- Leaning hard on LangChain early. Learn the raw APIs first so you understand what frameworks hide.

## Resources
- The Anthropic and OpenAI docs and cookbooks (best first source)
- Chip Huyen, *AI Engineering* (book)
- Hamel Husain's blog on evals
- The modelcontextprotocol.io docs
- The Vercel AI SDK docs and templates
