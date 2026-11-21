# Running a React Zoho CRM widget on localhost

Run the widget at `http://localhost:3000` with real data from your Zoho CRM org.
No deploy per change and no licence seat needed.

## 1. Create OAuth credentials

- Open https://api-console.zoho.com (use `.in`, `.eu`, `.com.au` etc. if your org is there).
- Click **Add Client** → **Self Client** → **Create**.
- Go to **Generate Code** and enter these scopes:

  ```
  ZohoCRM.modules.ALL,ZohoCRM.settings.modules.READ
  ```

- Copy the code, then swap it for a refresh token straight away (the code expires):

  ```bash
  curl -X POST "https://accounts.zoho.com/oauth/v2/token" \
    -d "grant_type=authorization_code" \
    -d "client_id=<client_id>" \
    -d "client_secret=<client_secret>" \
    -d "code=<code>"
  ```

- Copy `refresh_token` from the response.

## 2. Create `.env`

In the project root:

```
ZOHO_CLIENT_ID=1000.xxxxxxxx
ZOHO_CLIENT_SECRET=xxxxxxxx
ZOHO_REFRESH_TOKEN=1000.xxxxxxxx
ZOHO_DC=com
```

- `ZOHO_DC` must match your org. Check the CRM URL: `crm.zoho.com` → `com`,
  `crm.zoho.in` → `in`, and so on.
- Add `.env` to `.gitignore`.

## 3. Add the code

Copy these from [Code](#code) below:

| File | What it does |
|---|---|
| `vite-zoho-proxy.ts` | Sends `/zoho/*` requests to Zoho with your token |
| `vite.config.ts` | Turns the proxy on |
| `src/vite-env.d.ts` | Type for the `LIVE` flag |
| `src/zoho/live.ts` | Fake `ZOHO` SDK that calls the real API |
| Your entry file | Loads the fake SDK in dev |

## 4. Start it

```bash
npm run dev
```

- The terminal should show `[zoho-proxy] live mode ON`.
- Open http://localhost:3000.
- Restart after changing `.env`.

## 5. Register the widget in CRM (demo5)

- Setup → Developer Space → Widgets → Create New Widget
  - Name `Payment Tracker (dev)`, Type **Related List**, Hosting **External**
  - Base URL `https://127.0.0.1:3000/index.html`

## 6. Test inside CRM before shipping

```bash
zet run
```

- Register the printed URL as the widget URL in Zoho CRM.
- Open the widget in CRM to test the real SDK.

## Gotchas

- **Writes are real.** Use a demo org.
- **`invalid_client` on start:** `ZOHO_DC` is wrong.
- **Empty response (204):** means "no records found", not an error.
- **Bulk create:** max 100 records per call.
- **Record widgets:** pass the record to the fake SDK, e.g.
  `installLiveZoho({ Entity: 'Leads', EntityId: ['<id>'] })`.
- **Workflows don't fire locally.** The fake SDK sends `trigger: []` by
  default, so workflow rules, approvals and blueprints are skipped. Inside CRM
  the real SDK does fire them. Test workflow side effects with `zet run`.
- **Permissions aren't the logged-in user's.** Every live call runs as the
  owner of the refresh token, usually an admin. Inside CRM the widget runs as
  whoever is logged in. Test as a non-admin user with `zet run`.

See [09-IMPLEMENTATION-PITFALLS.md](09-IMPLEMENTATION-PITFALLS.md) for what else
to check before shipping.

## Security

- Never commit `.env`. The refresh token gives full access to the org.
- The secret stays on the dev server; the browser never sees it.
- To revoke, delete the Self Client in the API console.

---

## Code

### `vite-zoho-proxy.ts`

```ts
import type { Plugin } from 'vite';

export function zohoProxy(opts: {
  clientId?: string; clientSecret?: string; refreshToken?: string; dc?: string;
}): Plugin {
  const dc = opts.dc || 'com';
  const apiHost = `https://www.zohoapis.${dc}`;
  const live = Boolean(opts.clientId && opts.clientSecret && opts.refreshToken);

  let token: string | null = null;
  let expiresAt = 0;
  let inFlight: Promise<string> | null = null;

  async function refresh(): Promise<string> {
    const res = await fetch(`https://accounts.zoho.${dc}/oauth/v2/token`, {
      method: 'POST',
      body: new URLSearchParams({
        refresh_token: opts.refreshToken!, client_id: opts.clientId!,
        client_secret: opts.clientSecret!, grant_type: 'refresh_token',
      }),
    });
    const json = await res.json();
    if (!json.access_token) throw new Error(`token refresh failed: ${json.error} (ZOHO_DC=${dc}?)`);
    token = json.access_token;
    expiresAt = Date.now() + (json.expires_in ?? 3600) * 1000;
    return token!;
  }

  function getToken(): Promise<string> {
    if (token && Date.now() < expiresAt - 60_000) return Promise.resolve(token);
    inFlight ??= refresh().finally(() => { inFlight = null; });
    return inFlight;
  }

  return {
    name: 'zoho-proxy',
    resolveId: (id) => (id === 'virtual:zoho-mode' ? '\0virtual:zoho-mode' : null),
    load: (id) => (id === '\0virtual:zoho-mode' ? `export const LIVE = ${live};` : null),

    configureServer(server) {
      if (!live) {
        console.warn('[zoho-proxy] ZOHO_* not set -- live mode is OFF');
        return;
      }
      console.info(`[zoho-proxy] live mode ON -> ${apiHost}`);

      server.middlewares.use(async (req, res, next) => {
        if (!req.url?.startsWith('/zoho/')) return next();
        try {
          const chunks: Buffer[] = [];
          for await (const c of req) chunks.push(c as Buffer);
          const body = Buffer.concat(chunks);

          const upstream = await fetch(apiHost + req.url.slice('/zoho'.length), {
            method: req.method,
            headers: {
              Authorization: `Zoho-oauthtoken ${await getToken()}`,
              'Content-Type': 'application/json',
            },
            body: body.length ? body : undefined,
          });
          res.statusCode = upstream.status;
          const ct = upstream.headers.get('content-type');
          if (ct) res.setHeader('Content-Type', ct);
          res.end(await upstream.text());
        } catch (err) {
          res.statusCode = 502;
          res.end(JSON.stringify({ code: 'PROXY_ERROR', message: String(err) }));
        }
      });
    },
  };
}
```

### `vite.config.ts`

```ts
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';
import { zohoProxy } from './vite-zoho-proxy';

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '');
  return {
    plugins: [
      react(),
      zohoProxy({
        clientId: env.ZOHO_CLIENT_ID,
        clientSecret: env.ZOHO_CLIENT_SECRET,
        refreshToken: env.ZOHO_REFRESH_TOKEN,
        dc: env.ZOHO_DC,
      }),
    ],
    base: './',
    server: { port: 3000, strictPort: true },
  };
});
```

### `src/vite-env.d.ts`

```ts
declare module 'virtual:zoho-mode' {
  export const LIVE: boolean;
}
```

### `src/zoho/live.ts`

```ts
const BASE = '/zoho/crm/v8';
type Json = Record<string, unknown>;

async function call(path: string, init?: RequestInit) {
  const res = await fetch(BASE + path, {
    ...init,
    headers: { 'Content-Type': 'application/json' },
  });
  if (res.status === 204) return { data: [] };
  const text = await res.text();
  if (!text) return { data: [] };
  const body = JSON.parse(text);
  if (!res.ok) throw new Error(`${body.code ?? res.status}: ${body.message ?? res.statusText}`);
  return body;
}

const api = {
  getRecord: ({ Entity, RecordID }: { Entity: string; RecordID: string }) =>
    call(`/${Entity}/${RecordID}`),

  getAllRecords: ({ Entity, per_page = 200, page = 1 }: { Entity: string; per_page?: number; page?: number }) =>
    call(`/${Entity}?per_page=${per_page}&page=${page}`),

  searchRecord: ({ Entity, Query }: { Entity: string; Type: 'criteria'; Query: string }) =>
    call(`/${Entity}/search?criteria=${encodeURIComponent(Query)}`),

  getRelatedRecords: ({ Entity, RecordID, RelatedList }: { Entity: string; RecordID: string; RelatedList: string }) =>
    call(`/${Entity}/${RecordID}/${RelatedList}`),

  insertRecord: ({ Entity, APIData, Trigger = [] }: { Entity: string; APIData: Json | Json[]; Trigger?: string[] }) =>
    call(`/${Entity}`, {
      method: 'POST',
      body: JSON.stringify({ data: Array.isArray(APIData) ? APIData : [APIData], trigger: Trigger }),
    }),

  updateRecord: ({ Entity, RecordID, APIData, Trigger = [] }: { Entity: string; RecordID: string; APIData: Json; Trigger?: string[] }) =>
    call(`/${Entity}/${RecordID}`, {
      method: 'PUT',
      body: JSON.stringify({ data: [APIData], trigger: Trigger }),
    }),

  deleteRecord: ({ Entity, RecordID }: { Entity: string; RecordID: string }) =>
    call(`/${Entity}/${RecordID}`, { method: 'DELETE' }),
};

export function installLiveZoho(pageLoadData: Json = {}) {
  let onPageLoad: ((data: Json) => void) | null = null;
  (globalThis as any).ZOHO = {
    embeddedApp: {
      on: (_event: 'PageLoad', handler: (data: Json) => void) => { onPageLoad = handler; },
      init: async () => { setTimeout(() => onPageLoad?.(pageLoadData), 0); },
    },
    CRM: { API: api },
  };
  console.info('[live] fake ZOHO SDK installed');
}
```

### Entry file (e.g. `src/main.tsx`)

Put this at the top, before rendering:

```tsx
if (import.meta.env.DEV && window.self === window.top) {
  const { LIVE } = await import('virtual:zoho-mode');
  if (LIVE) {
    const { installLiveZoho } = await import('./zoho/live');
    installLiveZoho();
  }
}
```
