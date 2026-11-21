# AI Engineer Role

The usual name is **AI Engineer**, sometimes written **LLM Engineer** or **Applied AI Engineer**.

## Where it fits

| Role | Builds |
|---|---|
| Frontend | The UI in the browser |
| Backend | APIs, databases, business logic |
| Full stack | Both |
| **AI Engineer** | Features that use large language models (LLMs): RAG, agents, MCP servers, tool calling, prompts, evals |
| ML Engineer / Data Scientist | Training and fine-tuning the models themselves |

The main difference: an **ML Engineer** builds or trains models. An **AI Engineer** builds products on top of existing models like Claude or GPT, through their APIs.

## What an AI Engineer works on
- **RAG (retrieval-augmented generation):** splitting documents into chunks, embeddings, vector databases (pgvector, Pinecone, Qdrant), search and reranking.
- **Agents:** tool-use loops, planning, memory, multi-agent setups, frameworks like the Claude Agent SDK or LangGraph.
- **MCP (Model Context Protocol):** writing MCP servers that expose tools and data to models, and wiring up MCP clients.
- **Prompt and context engineering:** system prompts, structured output, caching.
- **Evals and reliability:** test sets, using an LLM to grade outputs, tracking quality, cost and speed.
- **LLMOps:** deployment, monitoring, guardrails, managing cost.

## Other job titles you'll see
- AI Engineer / Applied AI Engineer
- LLM Engineer / GenAI Engineer
- AI Application Developer
- Agent Engineer (newer, more specialized)
- AI Platform Engineer (builds the shared infrastructure other teams use)

## How people get into it
Most AI Engineers start as **backend or full-stack developers**. The work is mostly APIs, data pipelines and system design, with LLMs added on top. With a Next.js and TypeScript background, the usual next steps would be:
1. Call the Claude API from an API route in Next.js (`src/app/api/`).
2. Build a simple RAG feature, for example "chat with my docs".
3. Write a small MCP server.
4. Build an agent that uses tools.
