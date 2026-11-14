# 10 — Generating Ideas for End Users

How to come up with ideas for widgets, features or tools that the people using Zoho CRM will actually want.

Good ideas mostly come from watching where users struggle, not from brainstorming.

---

## 1. Watch the work

- **Sit with a user for 30 minutes** while they do their normal job. Note every time they switch tabs, copy and paste, use a spreadsheet next to the CRM, or say "I always have to...". Each of these is a possible idea.
- **Look for workarounds.** An Excel sheet kept beside the CRM, notes in a text field, a WhatsApp group for approvals: each one is an unmet need. The Payment Tracker idea ([06-EXAMPLE-PAYMENT-TRACKER.md](06-EXAMPLE-PAYMENT-TRACKER.md)) is this kind: payment data kept outside the CRM.
- **Ask about the last time it hurt,** not about what they want. "Tell me about the last time a deal got stuck" gets you real problems. "What features do you want?" gets you guesses.

---

## 2. Let the data point the way

You can do much of this through the Zoho MCP tools with Claude:

- **Fields that are always empty.** A field nobody fills in may be too hard to fill in where it is, which is a widget opportunity.
- **Text fields full of structured data.** Notes like "paid 500 on 3rd, 200 pending" are asking for a proper module.
- **Records that stall.** Deals that sit in one stage for weeks show where a reminder, checklist or approval flow would help.
- **Repeated manual edits.** Timelines and audit logs show the same field being edited by hand again and again, which could be automated.
- **Common searches and reports.** If everyone runs the same report every Monday, that's a dashboard widget.

---

## 3. Look across clients

As a partner you see many orgs, which is an advantage:

- When several clients ask for the same custom work, that's a reusable widget or product.
- Keep a simple log of each request: who asked, what the problem was, and what you built. After 20 entries, patterns appear.
- Check what clients currently pay other tools for that could live inside the CRM.

---

## 4. Use Claude to widen the list, not to decide

- Give Claude real material (interview notes, field-usage numbers, support tickets) and ask: "List the 10 biggest sources of friction and a widget idea for each."
- Ask Claude to take the role of a specific user, such as a sales rep, a finance user or a manager, and walk through their day in the CRM.
- Ask for the boring ideas too. The most-used tools often save 30 seconds, 50 times a day.

Claude's ideas are only as good as what you give it. Without real data from users, you get generic CRM features.

---

## 5. Filter before you build

Score each idea from 1 to 5 on:

| Question | Why it matters |
|---|---|
| How often does the pain happen? | Daily pain beats yearly pain |
| How many users feel it? | One power user vs the whole sales team |
| How painful is it now? | Lost money or deals vs mild annoyance |
| Can you build a small first version in days? | Fast feedback beats a perfect plan |
| Will they use it without training? | Adoption kills most CRM add-ons |

Build the highest scorers as small versions, put them in front of real users within a week, and watch whether they keep using them. Whether people use it is the only real test of an idea.

---

## Summary

Watch users, read the data, and log requests. Claude helps you expand and sort that material, but the idea has to start from something real.

Once you've picked an idea, check the build against [09-IMPLEMENTATION-PITFALLS.md](09-IMPLEMENTATION-PITFALLS.md).
