# Where an AI agent's knowledge lives

The model brings general knowledge and reasoning. Everything about *your* organisation comes from 6 places, delivered with each request. The examples column shows each one in a CRM setup.

## Where knowledge lives

| # | Knowledge | What's there | Where it lives | Example in a CRM setup | Who maintains it |
|---|---|---|---|---|---|
| 1 | **Instructions (system prompt)** | Rules, role, tone, limits | A database or config, edited in an admin portal | "Max 10% discount. Never promise delivery dates." (PostgreSQL) | Managers |
| 2 | **Business systems** (read live through tools) | Current data: customers, orders, stock, payments | The systems that already hold it | Zoho CRM (deals, contacts, products) and Zoho Books (invoices, payments), through MCP | Teams, through normal daily work |
| 3 | **Documents** (searched when needed) | Policies, manuals, FAQs, price lists, procedures | Original files in drives or wikis, plus a searchable copy | Files in WorkDrive or Drive, with a search copy in PostgreSQL (pgvector) | Each team, by keeping files up to date |
| 4 | **Conversation** | What's been said in this chat | The app's database | PostgreSQL (`messages`) | Automatic |
| 5 | **Memory store** | Saved facts from past chats and preferences | The app's database | "This customer prefers 30-day terms" (PostgreSQL) | Automatic or confirmed by users |
| 6 | **Request context** | Who is asking, from where, and when | Sent by the app with each message | The widget sends the user, role and open record (Deal #123) | Automatic |

**Two more things the agent's system keeps:**

- **Tool knowledge:** what tools exist and how to use them. This lives in the tool names and descriptions in the MCP servers' code, maintained by developers.
- **Action history:** what the agent did and who approved it. This lives in a change log (`audit_log` in PostgreSQL) and is written automatically.

## How it reaches the model

```
Instructions ──────────────┐
Request context ───────────┤
Conversation + memory ─────┼──▶ sent with the request ──▶ Model ──▶ Answer / action
Documents (search result) ─┤
Live data (tool results) ──┘
```

In a CRM setup:

```
                 ┌── Zoho CRM ───── customers, deals, products
                 ├── Zoho Books ─── invoices, payments
   Agent ──MCP──┼── Other apps ─── stock, orders, etc.
     │           └── .NET tools ─── business calculations
     │
     ├── PostgreSQL ── rules, chat memory, change log, document search
     └── Widget ────── who is asking, which record
```

## Two ways knowledge is delivered

- **Always sent:** instructions, context and recent conversation are included every time.
- **Fetched on demand:** documents are searched and live data is pulled by tools **only when the question needs them**. That keeps requests small and data current.

## Key points

- **The model doesn't learn or remember.** Your org's knowledge is delivered on every request and forgotten after the answer.
- **Business data stays where it already is.** The agent reads it live and doesn't copy it.
- **Only store what has no other home:** rules, memory, the change log and a searchable copy of documents.
- **Each piece of knowledge has one owner,** so a wrong answer tells you where to fix it:
  - Wrong price: fix the source system.
  - Wrong policy: fix the document.
  - Wrong behaviour: fix the instructions.

## Simple rule

> **Rules → instructions. Facts that change → live systems via tools. Facts that are written down → documents. Past interactions → memory. Who and where → context.**
