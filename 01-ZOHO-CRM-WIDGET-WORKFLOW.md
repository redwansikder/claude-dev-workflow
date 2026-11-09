# Building a Zoho CRM widget with Claude Code

Use both guides. They cover two sides of the same job:

- **[02-ZOHO-CRM-MCP-SETUP.md](02-ZOHO-CRM-MCP-SETUP.md) lets Claude see your CRM.**
  Claude can look up module and field API names, create test records and check
  what your widget actually saved.
- **[03-ZOHO-CRM-WIDGET-LOCAL-DEV.md](03-ZOHO-CRM-WIDGET-LOCAL-DEV.md) runs your widget.**
  React on localhost, opened inside the CRM widget frame, doing CRUD through the SDK.

## One-time setup per widget project

1. **Create the project:** Vite + React + TypeScript, then initialize it with
   `zet init` (or reuse an existing widget repo as a template).
2. **Connect Claude to a demo org.** Follow
   [02-ZOHO-CRM-MCP-SETUP.md](02-ZOHO-CRM-MCP-SETUP.md) and put the `.mcp.json` entry
   in the widget project. Use a demo org, never production.
3. **Make localhost load inside CRM:**
   - HTTPS on the Vite dev server, with a fixed port
   - the SDK `<script>` tag in the `<head>` of `index.html`, before your app's script:

     ```html
     <script src="https://live.zwidgets.com/js-sdk/1.2/ZohoEmbededAppSDK.min.js"></script>
     ```

   - the CRM widget URL set to `https://127.0.0.1:<port>`
4. **Add a `CLAUDE.md`** to the project so Claude starts each session with the
   right context:
   - which demo org/user and which MCP server name
   - the modules the widget uses
   - the rules: CRUD only through `ZOHO.CRM.API`, wait for `PageLoad`, use API
     names not display labels

## Each feature

1. **You describe it.** For example: "On a Deal, show its related Quotes and
   let the user mark one as accepted."
2. **Claude checks the CRM over MCP.** It finds the real module and field API
   names and related lists, so the code isn't built on guesses.
3. **Claude writes the code** using SDK calls.
4. **You test it:** `npm run dev`, then open the widget in CRM. Hot reload
   means you just refresh.
5. **Claude verifies over MCP.** It reads the records back to confirm the
   widget saved exactly what it should.
6. **Before shipping:** test as a **non-admin** user, then build and upload
   (`zet pack`).

## Things to agree on up front

- **Demo org only.** Writes from both the widget and Claude are real.
- **The two connections can run as different users.** MCP calls run as the
  demo user who authorized the server. The widget runs as whoever is logged
  into CRM. Keep them in the same org.
- **Not a secrets store:** don't put tokens in the repo, and keep `.mcp.json`
  in `.gitignore`.
