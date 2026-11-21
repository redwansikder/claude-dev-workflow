# Building an MCP server and an agent for your own service

There are two separate things to build. **Build the MCP server first.**

| | What it is | Who uses it |
|---|---|---|
| **MCP server** | A set of tools that lets an AI use your service | *Any* AI: Claude, ChatGPT, Zoho's agent, your clients' agents, your own agent |
| **Agent** | Your own AI assistant that uses those tools to do work for your users | Your users, in your app, widget, WhatsApp, etc. |

Your own agent is just one more client of your MCP server. You build the tools once and every AI can use them.

---

## Part A: Build the MCP server

### Step 1: List what users actually do

- Write down the top 10–20 tasks people do with your service, for example:
  - "Create an invoice"
  - "Check order status"
  - "Get a customer's balance"
- Each task becomes one **tool**.
- Start with the most common tasks, not every API endpoint.

**Output:** a list of tools, each with a name and one sentence saying what it does.

### Step 2: Design tools around tasks, not API endpoints

| ✅ Good | ❌ Bad |
|---|---|
| `get_customer_summary(customer_id)`, which makes 3 API calls inside and returns one clean result | `GET /customers`, `GET /orders` and `GET /payments` exposed as 3 separate tools |

**More good and bad examples:**

**Naming**

| ✅ Good | ❌ Bad |
|---|---|
| `create_invoice`, `get_order_status`, `cancel_order` | `invoice_handler`, `process`, `doAction`, `api_v2_post` |
| One tool per clear job: `send_payment_reminder` | One tool that does many jobs: `manage_customer(action="create" \| "update" \| "delete" \| "email")` |

**Descriptions**

| ✅ Good | ❌ Bad |
|---|---|
| "Get the status and delivery date of one order. Use when the user asks where an order is or when it will arrive." | "Gets order." |
| "Creates a draft invoice. Does not send it. Use `send_invoice` to send." | No mention of what happens next, so the AI assumes the invoice was sent |
| "Search customers by name, email or phone. Returns up to 10 matches." | "Search API." |

**Inputs**

| ✅ Good | ❌ Bad |
|---|---|
| `find_customer(email="rahim@abc.com")`, using values the user actually knows | `get_customer(internal_id="8a7f3c2e-91b4…")`, which the user never sees |
| `status` limited to `"open" \| "paid" \| "overdue"` | `status` as free text, so the AI guesses "unpaid", "pending", "due"… |
| `date="2026-11-15"` in one fixed format (YYYY-MM-DD) | Any date format accepted: "15/11", "next Friday", "Nov 15" |
| 2–4 inputs, with optional ones clearly marked | 15 inputs, most of them unused |
| `amount=1500, currency="BDT"` | `amount="1,500 Tk"` |

**Outputs**

| ✅ Good | ❌ Bad |
|---|---|
| `{ status: "Shipped", expected: "2026-11-15", items: 3 }` | The full 200-field raw API response |
| `"Paid"` as plain words | `"status_code": 3`, which the AI has to guess the meaning of |
| `{ customer: "ABC Ltd", owner: "Rahim" }` with names included | `{ customer_id: 9921, owner_id: 4410 }`, so the AI needs more calls to get names |
| "Showing 10 of 240 results. Use `page=2` for more." | A silently cut list, so the AI thinks there are only 10 |

**Errors**

| ✅ Good | ❌ Bad |
|---|---|
| "No customer found with email rahim@abc.com. Try searching by name or phone." | `Error 404` |
| "Discount 25% is above the 10% limit. Ask a manager to approve." | `Validation failed` |
| "You don't have permission to delete invoices." | A crash or an empty result |

**Read and write**

| ✅ Good | ❌ Bad |
|---|---|
| Separate tools: `get_invoice` (read) and `update_invoice` (write) | One `invoice(action=…)` tool that both reads and changes data |
| `preview_bulk_update` shows what will change, then `apply_bulk_update` does it | A bulk update that runs immediately with no preview |
| `delete_order` turned off by default, or needing confirmation | Delete available to every AI and every user |
| Repeating `create_payment` with the same `request_id` doesn't create a duplicate | Calling it twice by mistake records the payment twice |

**Size and number of tools**

| ✅ Good | ❌ Bad |
|---|---|
| 10–20 well-chosen tools for the main tasks | 150 tools, one for every API endpoint, so the AI gets confused |
| Group related tools with a shared prefix: `order_get`, `order_create`, `order_cancel` | Similar tools with unrelated names: `fetchOrder`, `new_order_v2`, `OrderCancel` |

For each tool:

- **Name:** a clear verb and noun, such as `create_invoice` or `get_order_status`.
- **Description:** what it does *and when to use it*. The AI chooses tools from this text, so it matters most.
- **Inputs:** few and simple, with examples. Use plain values ("email", "order number") rather than internal IDs where possible.
- **Output:** short and clean. Return only what the AI needs, not the full raw API response.
- **Type:** mark each tool as **read** (safe) or **write** (changes data).

**Example tool definition:**

```text
Name:        get_order_status
Description: Get the current status, delivery date and items of one order.
             Use when the user asks where an order is or when it will arrive.
Inputs:      order_number (text, e.g. "SO-10452")
Output:      status, expected_delivery, items[name, qty], last_update
Type:        read
```

### Step 3: Decide how users sign in

| Option | How it works | Use when |
|---|---|---|
| **Remote server with OAuth (recommended)** | Hosted at a URL. Users sign in with their own account, so they only see their own data. | Real customers and production use |
| **Local server with an API key** | Runs on the user's machine | Developers and internal testing only |

**Rule:** the AI should never have more access than the person using it.

### Step 4: Build it

- Use an official MCP SDK: **C#, TypeScript or Python**.
- The MCP server calls **your existing API**. Keep business logic in one place, not copied into the MCP server.
- Use the **HTTP transport**, so it runs as a hosted URL, for example `https://mcp.yourservice.com`.

### Step 5: Add safety

- Every call respects the signed-in user's permissions.
- Validate inputs and set rate limits.
- Destructive tools (delete, bulk update) are **off by default** or need confirmation.
- Log every call: who, which tool, inputs, result, errors.

### Step 6: Test with real AI clients

- Connect it to Claude (desktop app, Claude Code, claude.ai connectors) and to other MCP clients.
- Ask **20–30 real questions** your users would ask.
- If the AI picks the wrong tool or sends wrong inputs, **improve the tool names and descriptions first**. That fixes most problems.
- Keep these questions as a test set and re-run them after every change.

### Step 7: Publish

- Host it at a stable URL.
- Write a setup page covering how to connect, the list of tools, and which permissions are needed.
- List it in MCP and connector directories (Claude, Zoho and others) so people can find it.

---

## Part B: Build your own agent

### Step 8: Pick one clear job

- For example: "Support assistant for our customers" or "Assistant inside the CRM record page".
- **Start narrow.** A "do everything" assistant is hard to make reliable.

### Step 9: Build the agent loop

The backend (.NET or Node) repeats this until it has an answer:

```
1. Receive the user's message
2. Send it to Claude with instructions and the allowed tools
3. Claude asks for a tool → run it through MCP
4. Send the tool result back to Claude
5. Repeat until Claude gives the final answer
```

- **Tools:** your own MCP server, plus others (CRM, accounting, etc.).
- **Instructions (system prompt):** your business rules, tone, and what the agent must never do.
- **Knowledge:** see [09-where-agent-knowledge-lives.md](09-where-agent-knowledge-lives.md).

### Step 10: Put it where users already are

- A widget inside the CRM, a chat on your website or app, WhatsApp, Slack or email.
- Keep API keys on your server, never in the browser or app.

### Step 11: Add control

- **Approval before any write action.** Show what will change, then the user clicks Approve.
- Keep a change log and an undo option.
- Set usage limits per customer, to control cost.

### Step 12: Measure and improve

- Re-run the test set after every change to prompts, tools or the model.
- Watch for:
  - Wrong tool choices
  - Failed tool calls
  - Rejected approvals (the agent is suggesting wrong actions)
  - Thumbs-down feedback from users

### Step 13: Launch and charge

- Pilot with 1–3 customers, fix what you learn, then expand.
- **Pricing ideas:**
  - The **MCP server is free** with your service, which makes customers less likely to switch away.
  - The **agent is a paid add-on**, charged per user or by usage.

---

## Order and time

| Order | What | Steps | Time (rough) |
|---|---|---|---|
| 1 | Tool list and design | 1–3 | 1 week |
| 2 | MCP server built and tested | 4–6 | 2–3 weeks |
| 3 | MCP server published | 7 | 1 week |
| 4 | Agent pilot | 8–11 | 3–4 weeks |
| 5 | Improve and launch | 12–13 | Ongoing |

## Checklist before going live

- [ ] Tools are task-based, with clear names and descriptions
- [ ] Each tool is marked read or write
- [ ] Users sign in with their own account (OAuth)
- [ ] Destructive tools are off or need confirmation
- [ ] Every call is logged
- [ ] The test set passes with real AI clients
- [ ] Write actions in the agent need approval
- [ ] Usage limits are set per customer

## Summary

> **Expose your service as clean, safe, task-based MCP tools → test them with real AIs → publish → then build a focused agent on top for your own users.**
