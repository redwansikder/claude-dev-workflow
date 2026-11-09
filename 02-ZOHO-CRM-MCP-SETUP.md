# Connecting a Zoho CRM demo user to Claude Code

Each demo user needs its own Zoho MCP server. A server always acts as the
account that authorized it, so one server can only see and change one org.

## 1. Sign in as the demo user

- Open https://mcp.zoho.com in a **private window** (or sign out of any other
  Zoho account first).
- Sign in as the demo user, e.g. `user1@demo5.instawebworks.com.au`.

If you skip this, the server can end up connected to the previous user's org.

## 2. Create the server

- Click **Create MCP server** → **Pre-configured servers** → **Lead Management System**.
- Name it after the user, e.g. `Lead-Management-demo5`.
- Under **Tools**, the preset already has `getRecords`, `searchRecords`,
  `createRecords`, `updateRecords` and `convertInventory`.
- Click **Add More Tools** → **Zoho CRM** and add the rest, so Claude can create
  custom modules and do full CRUD (see [Tools and what they allow](#tools-and-what-they-allow)):
  - `createModules`
  - `createFields`
  - `getModules`
  - `getFields`
  - `deleteRecords`

## 3. Lock the connection to this demo user

- Go to **Connection** → **Change** → **Authorization via Connection** →
  **Rotate & Switch**.
- Under **Authorized Tools**, connect **Native Connection** and sign in as the
  demo user.
- Check the row shows:
  - **Authorized By**: the demo user's email
  - **Status**: Connected

## 4. Copy the server URL

- Go to **Connect** and copy the server URL.
- Copy it **after** step 3 — switching authorization generates a new URL and
  the old one stops working.

## 5. Add it to `.mcp.json`

Add an entry with a unique name per demo user:

```json
{
  "mcpServers": {
    "zoho-crm-demo4": {
      "type": "http",
      "url": "https://lead-management-XXXX.zohomcp.com/mcp/<key>/message"
    },
    "zoho-crm-demo5": {
      "type": "http",
      "url": "https://lead-management-YYYY.zohomcp.com/mcp/<key>/message"
    }
  }
}
```

## 6. Reload Claude Code

- VS Code: **Ctrl+Shift+P** → **Developer: Reload Window**.
- Approve the new MCP server if asked.

## 7. Use it

- Say which demo user each task is for, e.g. "demo user 5, Leads, record …".
- Claude reads the record first to confirm it is in the right org, makes the
  change, then reads it back.

## Tools and what they allow

| Task | Tool |
|---|---|
| Create a custom module | `createModules` |
| Add fields to a module | `createFields` |
| List modules / fields and their API names | `getModules`, `getFields` |
| **C**reate records | `createRecords` |
| **R**ead records | `getRecords`, `searchRecords` |
| **U**pdate records | `updateRecords` |
| **D**elete records | `deleteRecords` |
| Convert Quote → Sales Order → Invoice | `convertInventory` |

Without `getModules` / `getFields`, Claude has to guess API names (e.g.
`record_data`) and ask you when a guess is wrong.

Notes:

- Creating modules and fields needs admin rights for that demo user.
- Custom module limits depend on the org's Zoho CRM edition.
- Deleted records go to the Recycle Bin and can be restored from there.
- Every call runs as the demo user, so that user's profile permissions apply.

## Adding tools to an existing server

The server URL does not change when you add tools, so `.mcp.json` stays the same.

1. Sign in to https://mcp.zoho.com as that demo user.
2. Pick the server in the top-left dropdown (e.g. **Lead-Management**) → **Tools**.
3. **Add More Tools** → **Zoho CRM** → tick the tools you need → save.
4. Check **Connection** still shows **Connected** as that demo user; if the new
   tools ask for extra permissions, reauthorize **Native Connection**.
5. Reload Claude Code (**Ctrl+Shift+P** → **Developer: Reload Window**).

## Security

- The URL contains an access key. Anyone who has it can act as that demo user
  without logging in.
- Keep `.mcp.json` private and never commit it (add it to `.gitignore`).
- To revoke access, delete the server or rotate its key in the Zoho MCP console.
