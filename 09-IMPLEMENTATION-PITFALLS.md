# 09 — Implementation Pitfalls (Universal + Zoho)

A first draft, whether written by Claude or by a person, usually handles the normal path and misses what happens with real data, real users and real failures. This file lists 24 of those gaps. Each one has a universal solution and a Zoho solution.

**How to use it**

- **Before you build:** paste the [prompt snippet](#prompt-snippet) into `CLAUDE.md` or a skill so the first draft already handles these.
- **After the first draft:** go through the [quick checklist](#quick-checklist), or ask Claude to review the code against this file.

**Scope tags**

- `Universal`: any software that talks to an API or a database.
- `CRM platforms`: Zoho, Salesforce, HubSpot, Dynamics and other platforms where admins can change the data model.
- `Zoho-specific`: mostly a Zoho problem.

> Zoho limits change by API version and edition. Check the numbers in the [appendix](#zoho-limits-appendix) against the current Zoho docs and your org's edition.

---

## The four wrong assumptions

Most of these pitfalls come from a first draft assuming one of these:

1. **Small data.** The demo org has 20 records, but production has 200,000.
2. **One user at a time.** In reality, several people edit the same record.
3. **A stable org configuration.** In reality, admins rename fields, add required fields and turn on workflows.
4. **Every call succeeding.** In reality, calls hit rate limits, tokens expire and networks drop.

---

## Scale and performance

### 1. Queries that don't scale — `Universal`

**Problem:** Code works on demo data and times out or freezes on real volumes, for example by loading every record and filtering in the browser.

**Universal solution:** Filter and sort on the server. Fetch only the fields and rows you need. Test with production-sized data before you ship.

**Zoho solution:** Use COQL (`SELECT field1, field2 FROM Deals WHERE ...`) or `searchRecord` with criteria instead of `getAllRecords` followed by filtering in JavaScript. Request only the fields you need. For exports or reports over large data, use the Bulk Read API instead of a loop over pages. Seed the demo org with a few thousand records before testing.

### 2. No paging — `Universal`

**Problem:** A single fetch returns only the first page, so records are silently missing.

**Universal solution:** Always loop until the API says there are no more results. Never assume one call returns everything.

**Zoho solution:** `getRecords` returns at most 200 per page. Loop over `page` and `per_page: 200` while `info.more_records` is `true`. Past about 2,000 records, recent API versions require `page_token` instead of `page`. COQL results are capped per query (2,000 rows in recent versions), so page with `LIMIT` and `OFFSET`. The Search API also has a total result cap.

```js
async function getAll(entity) {
  let page = 1, all = [], more = true;
  while (more) {
    const res = await ZOHO.CRM.API.getAllRecords({ Entity: entity, page, per_page: 200 });
    all = all.concat(res.data || []);
    more = res.info?.more_records === true;
    page++;
  }
  return all;
}
```

### 3. No batching — `Universal`

**Problem:** Sending all records in one write call fails, or only part of it is accepted.

**Universal solution:** Split writes into chunks that fit the API's per-call limit, and process the chunks in order.

**Zoho solution:** Insert, update and upsert accept at most 100 records per call. Split into chunks of 100 and check each chunk's results (see #14).

### 4. API credits and rate limits — `Universal`

**Problem:** Fetching on every render, keystroke or tab switch uses up the quota, and the whole org's integrations stop.

**Universal solution:** Cache results, debounce user-driven calls, and fetch only when the data actually needs refreshing. Track how many calls each user action makes.

**Zoho solution:** Zoho uses a daily pool of API credits, shared by the whole org and based on edition and number of licenses. Widgets, functions, integrations and MCP calls all draw from it. Cache data in the widget for the session. Prefer one COQL query to many `getRecord` calls. Use Bulk Read for large jobs. Check usage under **Setup → Developer Hub → APIs → API Dashboard**.

---

## Reliability

### 5. No retries or error handling — `Universal`

**Problem:** The code only handles `.then()`. Rate-limit, network or token errors fail silently and the user sees nothing.

**Universal solution:** Catch every error. Retry errors that may pass on a second try (timeouts, 429, 5xx) with exponential backoff and a maximum number of attempts. Show a clear message for the rest.

**Zoho solution:** Wrap SDK calls in `try`/`catch` and inspect the response `code`. Retry `TOO_MANY_REQUESTS` (HTTP 429) with backoff. On `INVALID_TOKEN` or `AUTHENTICATION_FAILURE` in server-side code, refresh the token and retry once (see #20). Don't retry `INVALID_DATA` or `MANDATORY_NOT_FOUND`; fix the data instead.

### 6. Concurrent edits — `Universal`

**Problem:** Two users read the same record, change it and save. The second save silently overwrites the first.

**Universal solution:** Use optimistic locking: keep a version number or timestamp, and refuse the write if it changed since you read it. Use atomic operations for counters and totals.

**Zoho solution:** Zoho has no built-in optimistic lock on the API. Before you save, re-read the record and compare `Modified_Time` with the value you loaded. If it changed, reload and ask the user to confirm. Don't keep running totals in a field that several users update; compute them with a rollup or formula field, or in a Deluge function.

---

## Security

### 7. Permission gaps — `Universal`

**Problem:** A rule like "only the deal owner sees payments" is enforced in the UI but not on every path to the data.

**Universal solution:** Enforce access on the server for every read and write. Hiding a button is not access control.

**Zoho solution:** Widget SDK calls run as the logged-in user, so profile and role permissions already apply to them. Use **profiles, roles, sharing rules and field-level security** for access. For rules beyond that, put the check in a Deluge function the widget calls, not in widget JavaScript. Test with users in different roles (see #19).

### 8. Tokens in the browser — `Universal`

**Problem:** A client secret, refresh token or third-party API key is placed in front-end code, where any user can read it.

**Universal solution:** Never ship secrets to the browser. Keep them on a server and expose only narrow endpoints.

**Zoho solution:** Don't put OAuth secrets or third-party API keys in widget files. For external APIs, use a **Zoho CRM Connection** and call it with `ZOHO.CRM.CONNECTION.invoke`, or run the call inside a Deluge function with `ZOHO.CRM.FUNCTIONS.execute`. Store org settings in **CRM Variables**, not in JavaScript.

### 9. No server-side checks — `Universal`

**Problem:** Validation and business rules run only in the widget. A user can change the JavaScript, or call the API directly, and bypass them.

**Universal solution:** Treat every client as untrusted. Repeat the validation on the server; client-side checks only improve the user experience.

**Zoho solution:** Use **validation rules**, mandatory fields, unique fields and picklists on the module. For complex rules, such as "a payment can't exceed the deal's remaining balance", use a Deluge function or a validation-rule function that runs on the server.

---

## Data correctness

### 10. Rounding with money — `Universal`

**Problem:** JavaScript floats give `0.1 + 0.2 = 0.30000000000000004`, so totals drift by cents.

**Universal solution:** Store and add money in the smallest unit (cents) as integers, or use a decimal library. Round only when you display a value.

**Zoho solution:** Use **Currency** fields, which have fixed decimal places, not Decimal or Single Line fields. Convert to cents for any math in the widget. Where possible, let Zoho compute totals with rollup summary or formula fields.

### 11. Currency in orgs with several currencies — `CRM platforms`

**Problem:** Amounts in USD, EUR and INR are added together as if they were the same currency.

**Universal solution:** Store the currency with every amount. Convert to one reporting currency, using a recorded exchange rate, before you aggregate.

**Zoho solution:** With multiple currencies on, each record has a `Currency` and an `Exchange_Rate`, and Zoho keeps home-currency values for reports. Read the record's currency, show it next to the amount, and convert with the record's exchange rate before you add amounts across records. Use the org's home currency for totals.

### 12. Time zones and dates — `Universal`

**Problem:** The browser, the user profile and the server disagree about what "today" is, so due dates and filters are off by a day.

**Universal solution:** Store timestamps in UTC with an offset (ISO 8601). Keep dates with no time as plain dates. Convert to local time only for display, and decide explicitly which time zone counts as "today".

**Zoho solution:** Date fields take `yyyy-MM-dd`. DateTime fields take ISO 8601 with an offset, for example `2026-11-14T10:00:00+05:30`. The CRM shows times in the user's time zone. Get it with `ZOHO.CRM.CONFIG.getCurrentUser()` rather than using the browser's time zone. In Deluge, use `zoho.currentdate` and `zoho.currenttime`, which follow the org's time zone.

### 13. Duplicates from retries or double-clicks — `Universal`

**Problem:** A double-click or an automatic retry creates two copies of the same payment.

**Universal solution:** Disable the button while a save is running. Use idempotency: a unique key per operation, so a repeated request updates instead of inserting again.

**Zoho solution:** Mark a field such as `Payment_Reference` as **unique**. Use `upsertRecords` with `duplicate_check_fields: ["Payment_Reference"]` instead of `insertRecord`. Disable the save button until the SDK promise settles.

### 14. Partial failure — `Universal`

**Problem:** The code checks that the call succeeded, but inside a batch some records saved and others failed.

**Universal solution:** Check the result of each item. Report the failures, retry or fix them, and never report "all saved" just because the request returned 200.

**Zoho solution:** Write responses return a `data[]` array with one entry per record, each with its own `code` and `status` (`success` or `error`) and an error `message`/`details`. Loop over it, collect the failures, and show the user which records failed and why.

---

## Platform behaviour

### 15. Field and module renames — `CRM platforms`

**Problem:** Code refers to fields by their display label. An admin renames "Amount Paid" to "Paid Amount" and the widget breaks.

**Universal solution:** Refer to stable identifiers, not display text.

**Zoho solution:** Always use **API names** (`Amount_Paid`, `Payments`), never labels. Get them from `getFields` or **Setup → Developer Hub → APIs → API Names**. Renaming a label doesn't change its API name.

### 16. Layouts and required fields — `CRM platforms`

**Problem:** A save works in one layout and fails in another, or an admin adds a required field and every insert fails.

**Universal solution:** Read the schema and required fields at runtime, or at least check them before you release. Report missing required fields clearly instead of failing with a generic error.

**Zoho solution:** Call `getLayouts` and `getFields` to find which fields are mandatory in which layout. Pass `Layout: { id }` when you insert into a module with several layouts. Handle `MANDATORY_NOT_FOUND` by naming the missing field in the error message.

### 17. Workflows firing — `CRM platforms`

**Problem:** A write from your code triggers workflow rules, emails, webhooks or functions you didn't plan for. A bulk import can set off hundreds of them.

**Universal solution:** Know which automations listen to the objects you write to. Decide for each write whether they should run, and test bulk writes in a sandbox first.

**Zoho solution:** List the active rules with **Setup → Automation → Workflow Rules**, or with `getWorkflowRules` over MCP. On API writes, pass `trigger: []` to skip workflows, approvals and blueprints, or pass `trigger: ["workflow"]` to run only workflows. Remember that each function a workflow runs also uses API credits.

### 18. Data centres — `Zoho-specific`

**Problem:** The API URL is hardcoded to `zohoapis.com`, so the code fails for orgs hosted in another region.

**Universal solution:** For any multi-region service, use the region endpoint returned during authentication, not a hardcoded one.

**Zoho solution:** Zoho runs separate data centres (`.com`, `.eu`, `.in`, `.com.au`, `.jp`, `.ca`, `.com.cn` and others). For server-side OAuth, use the `api_domain` from the token response and the matching `accounts.zoho.*` server. Widget SDK calls and Connections handle the region for you, which is another reason to prefer them.

### 19. Profile permissions — `CRM platforms`

**Problem:** The widget works for the admin who built it and fails for a sales rep whose profile can't see the module or a field.

**Universal solution:** Test with users in the least privileged role. Handle "no access" as a normal state, not a crash.

**Zoho solution:** Create a test user with the **Standard** profile, or the client's real sales profile, and test the widget as that user. Check module permissions and field-level security for each field the widget reads or writes. Handle `NO_PERMISSION` responses with a clear message.

---

## Operations

### 20. Token expiry — `Universal`

**Problem:** An access token works in testing and the integration fails an hour later, or overnight.

**Universal solution:** Store the refresh token securely on the server. Refresh the access token before it expires, or when you get a 401, and retry the call once.

**Zoho solution:** Zoho access tokens last about 1 hour. Use the refresh-token flow on the `accounts.zoho.*` server for your org's data centre. Widgets avoid this, because the SDK uses the user's session. Deluge Connections also refresh automatically.

### 21. No logging — `Universal`

**Problem:** Something fails in a client's org and there's no record of what happened.

**Universal solution:** Log every failed external call with its input, response code and a correlation ID. Keep logs where you can reach them without the user's browser.

**Zoho solution:** In Deluge, use `info` statements and read them under **Setup → Developer Hub → Functions → Logs**. For widgets, send errors to a Deluge function that writes to a custom **Integration Log** module, or to an external logger. Look for failed webhook deliveries in the webhook failure reports.

### 22. Deleted or merged records — `Universal`

**Problem:** A lookup points at a record that was deleted or merged, and the widget crashes on a null value.

**Universal solution:** Treat every reference as possibly missing. Null-check lookups, and decide what the UI shows for a missing parent.

**Zoho solution:** Lookup fields can come back `null`, or as an ID whose `getRecord` returns no data. Check before you read `.name` or `.id`. To sync or clean up, use the deleted-records endpoint (`GET /crm/v{n}/{module}/deleted`). Remember that merging contacts or accounts changes which record an ID points to.

### 23. Sandbox and production differences — `Universal`

**Problem:** Record IDs, user IDs, layout IDs or settings are hardcoded from the dev org and don't exist in production.

**Universal solution:** Never hardcode IDs from one environment. Look them up by stable name at runtime, or keep them in configuration for each environment.

**Zoho solution:** Look up layouts, users and picklist values by API name or value at runtime. Keep org settings in **CRM Variables** (`getVariables`), which you set separately in each org. When you move a widget from demo5 to a client org, check the module API names, because custom modules can get different names.

---

## UI states

### 24. Empty, loading and error states — `Universal`

**Problem:** The widget only handles the case where data loads. A deal with no payments shows a blank panel, and a failed call leaves a spinner running forever.

**Universal solution:** Design four states for every view: loading, empty, error and loaded. Every async call ends in one of them.

**Zoho solution:** Start work inside `ZOHO.embeddedApp.on("PageLoad", ...)` and then call `ZOHO.embeddedApp.init()`. Show a loading state right away, an empty state such as "No payments yet. Add the first one.", and an error state with a retry button. Use `ZOHO.CRM.UI.Resize` so each state fits the related-list frame.

---

## Quick checklist

Use this to review a first draft.

**Scale and performance**
- [ ] 1. Filtering happens on the server, and the code was tested with production-sized data
- [ ] 2. Every list fetch loops over pages (`more_records`, `page_token`, COQL `OFFSET`)
- [ ] 3. Writes are split into chunks of 100 or fewer
- [ ] 4. No fetch on every render; results are cached and credit use is known

**Reliability**
- [ ] 5. Every call has error handling, and 429s and timeouts are retried with backoff
- [ ] 6. Concurrent edits are detected (`Modified_Time` check), and totals are not updated by hand

**Security**
- [ ] 7. Access is enforced by profiles, roles or server functions, not only in the UI
- [ ] 8. No secrets in widget code; external APIs go through Connections or functions
- [ ] 9. Business rules are enforced by validation rules or functions on the server

**Data correctness**
- [ ] 10. Money math uses cents or Currency fields, not raw floats
- [ ] 11. Currency is stored and converted before amounts are added
- [ ] 12. Dates use the correct format and the user's or org's time zone
- [ ] 13. Saves are idempotent (unique field + upsert) and the button is disabled while saving
- [ ] 14. Each record's result in a batch response is checked

**Platform behaviour**
- [ ] 15. Only API names are used, never labels
- [ ] 16. Required fields and layouts are handled, with clear messages
- [ ] 17. It's known which workflows fire, and `trigger` is set on purpose
- [ ] 18. No hardcoded data-centre URL
- [ ] 19. Tested as a user with a non-admin profile

**Operations**
- [ ] 20. Server-side tokens refresh automatically
- [ ] 21. Failures are logged somewhere you can read them
- [ ] 22. Lookups are null-checked, and deleted or merged records are handled
- [ ] 23. No hardcoded IDs from the dev org; settings live in CRM Variables

**UI states**
- [ ] 24. Loading, empty, error and loaded states all exist

---

## Zoho limits appendix

Typical values. **Check them against the current Zoho CRM API docs and your org's edition.**

| Limit | Typical value | Related pitfall |
|---|---|---|
| Records per page (`getRecords`) | 200 | #2 |
| Records before `page_token` is needed | about 2,000 (recent API versions) | #2 |
| COQL rows per query | 2,000 (recent API versions); page with `LIMIT`/`OFFSET` | #1, #2 |
| Records per insert, update or upsert | 100 | #3, #14 |
| API credits | Daily pool for the whole org, set by edition and licenses | #4 |
| Concurrent API calls | Limited per org, set by edition | #4, #5 |
| Access token lifetime | About 1 hour | #20 |
| Data centres | `.com`, `.eu`, `.in`, `.com.au`, `.jp`, `.ca`, `.com.cn` and others | #18 |

---

## Prompt snippet

Paste this into `CLAUDE.md` or a skill so first drafts handle these pitfalls by default:

```markdown
## Implementation rules (Zoho CRM)
Assume production has 10k+ records, several users editing at once, admins who change
fields and layouts, and API calls that can fail.

- Page every read (200 per page, `more_records` / `page_token`; COQL with LIMIT/OFFSET).
- Batch every write at 100 or fewer, and check each record's status in the response.
- Cache in the widget; never fetch on every render. API credits are shared by the whole org.
- Wrap every call in try/catch; retry 429/timeouts with backoff; show clear errors.
- Use API names only. Never hardcode record, user or layout IDs; use CRM Variables for settings.
- No secrets in widget code. External APIs go through Connections or Deluge functions.
- Enforce business rules on the server (validation rules or functions), not only in the UI.
- Money: Currency fields or integer cents; respect each record's Currency and Exchange_Rate.
- Dates: yyyy-MM-dd for dates, ISO 8601 with offset for datetimes, in the user's time zone.
- Make saves idempotent (unique field + upsert) and disable the button while saving.
- Set `trigger` on writes on purpose; know which workflows will fire.
- Null-check lookups. Build loading, empty, error and loaded states for every view.
- Before saying it's done, review the code against 09-IMPLEMENTATION-PITFALLS.md.
```
