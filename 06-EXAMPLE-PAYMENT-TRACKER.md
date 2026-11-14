# Scenario: a "Payment Tracker" widget on Deals

A client wants a panel on every Deal record showing what's been paid so far,
with a button to add a payment. You build it with Claude, running on localhost
inside a demo org.

## Day 1: setup (about 30 minutes, once per project)

**1. Create the project**

```bash
mkdir payment-tracker && cd payment-tracker
npm create vite@latest . -- --template react-ts
npm install
npm install -D @vitejs/plugin-basic-ssl
```

**2. Connect Claude to the demo org.** Follow
[02-ZOHO-CRM-MCP-SETUP.md](02-ZOHO-CRM-MCP-SETUP.md) signed in as
`user1@demo5...`, then add the server to `payment-tracker/.mcp.json` as
`zoho-crm-demo5`. Reload VS Code.

**3. Ask Claude to prepare the project**

> "This is a Zoho CRM related-list widget on Deals, developed on localhost
> inside CRM. Set up Vite with HTTPS on port 3000 with strictPort, add the Zoho
> SDK script to index.html, and write a CLAUDE.md. Demo org is demo5 via the
> zoho-crm-demo5 MCP server."

Claude produces:

- `vite.config.ts` with `basicSsl()` and `server: { port: 3000, strictPort: true }`,
  plus `base: './'`
- the SDK `<script>` in `index.html`
- a `CLAUDE.md` with the rules: CRUD only through `ZOHO.CRM.API`, wait for
  `PageLoad`, demo5 only, check API names over MCP before using them

**4. Trust the certificate.** Run `npm run dev`, open
`https://127.0.0.1:3000/index.html` and accept the warning.

**5. Register the widget in CRM (demo5)**

- Setup → Developer Space → Widgets → Create New Widget
  - Name `Payment Tracker (dev)`, Type **Related List**, Hosting **External**
  - Base URL `https://127.0.0.1:3000/index.html`
- Open any Deal → **Add Related List → Widgets → Payment Tracker (dev)**.

The panel shows Vite's starter page. The connection works.

## Day 1: build the feature

**6. Data model.** You tell Claude:

> "Payments need: amount, date, method (Cash/Bank/Card), linked to a Deal.
> Check if anything like that exists in demo5; if not, create it."

Claude calls `getModules` over MCP and finds no payments module. It asks
before making changes, then creates a custom module `Payments` with `Amount`,
`Payment_Date`, `Method` (picklist) and a `Deal` lookup. It then reads the
module back with `getFields` and reports the real API names, such as
`Payment_Date` and `Deal`.

**7. Test data**

> "Create 3 test payments on the deal 'Acme Website Redesign'."

Claude finds the Deal by searching, creates 3 Payments records over MCP, then
reads them back.

**8. The widget**

> "Build the widget: show payments for the current Deal, total paid vs deal
> Amount, and an Add Payment form."

Claude writes:

- `ZOHO.embeddedApp.on('PageLoad', …)`, which reads `EntityId`, the Deal's ID
- `getRelatedRecords` to list that Deal's payments
- `getRecord` on the Deal to get its `Amount`
- `insertRecord` on `Payments` for the form

**9. Try it in CRM.** Refresh the Deal. The 3 test payments show, along with
"Paid 4,500 / 10,000". You add a 2,000 Bank payment through the form.

**10. Claude checks the write**

> "Check what the widget just saved."

Claude reads the newest Payments record over MCP and confirms `Amount: 2000`,
`Method: Bank`, linked to the right Deal. If a field were empty or wrong, it
would compare the code's payload with the field definitions and fix it.

## Day 2: a bug

You report: *"Total shows NaN on some deals."*

Claude checks over MCP and finds Deals where `Amount` is empty, which comes
back as `null`. It fixes the code, then creates a test Deal with no Amount so
you can confirm the fix in CRM.

## Before shipping

1. **Test as a normal user.** Sign in to demo5 as a non-admin. If that profile
   can't see Payments, the widget shows an error. You fix the profile
   permissions or add an empty state in the widget.
2. **Build and package:** `npm run build`, then `zet pack`.
3. **Production widget:** create a new widget `Payment Tracker`, Hosting
   **Zoho**, upload the zip. Keep the `(dev)` one for further work.
4. **Clean up:** ask Claude to delete the test Payments and the test Deal in
   demo5.

## Who does what

| Step | You | Claude |
|---|---|---|
| Project and config | Run commands | Writes the Vite config, script tag and CLAUDE.md |
| CRM widget registration | **You (manual; there's no API for it)** | — |
| Modules and fields | Approve | Creates them over MCP and reports API names |
| Widget code | Describe the feature | Writes the SDK calls |
| Testing in CRM | Click around | — |
| Checking saved data | Ask | Reads it back over MCP |
| Release | Upload the zip | Builds and cleans up test data |

Menu names in step 5 are approximate, because Zoho changes them between versions.

## Next: harden it

This walkthrough is a working first version. Before using it with real client
data, check it against
[09-IMPLEMENTATION-PITFALLS.md](09-IMPLEMENTATION-PITFALLS.md). In particular:

- **Paging (#2):** `getRelatedRecords` returns at most 200 payments per page.
- **Money and currency (#10, #11):** the "Paid 4,500 / 10,000" total is
  calculated with floats and ignores the Deal's currency.
- **Duplicates (#13):** a double-click on Add Payment saves the payment twice.
- **Concurrent edits and errors (#5, #6):** two users adding payments at once,
  and failed saves with no message.
