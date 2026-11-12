# Free API Providers, Part 2: OpenRouter, Cloudflare, Backups

For each provider: how to get a key, where to find the URL in its docs, what to put in your code, and the free limits.

Read [12-free-api-tiers.md](12-free-api-tiers.md) first. It has the test code (JS and .NET) and the **Rules for any base URL** (Rule 1–3) that these steps refer to. Part 1 is [13-free-api-providers-part-1.md](13-free-api-providers-part-1.md).

> Limits were looked up in Nov 2026, mostly from third-party sites. Check each provider's console before you depend on them.

## 1. OpenRouter

**Connect**

1. Go to **openrouter.ai** and sign in. Google or GitHub sign-in works.
2. Open **Keys** (from your profile menu) and click **Create Key**.
3. Copy the key. It starts with `sk-or-...`.

**Use**

| Value | Setting |
|---|---|
| `LLM_BASE_URL` | `https://openrouter.ai/api/v1` |
| `LLM_API_KEY` | Your `sk-or-...` key |
| `LLM_MODEL` | Any model whose name ends in **`:free`**. Find them on **openrouter.ai/models** by searching for "free" |

**Find the URL in OpenRouter's docs**

1. Open OpenRouter's docs. Click **Docs** in the top menu of **openrouter.ai**.
2. Open **Quickstart**. The page address is **openrouter.ai/docs/quickstart**.
3. Find the sample under **Using the OpenAI SDK**. Click the **TypeScript** tab for JS. It looks like this:

   ```js
   const openai = new OpenAI({
     baseURL: "https://openrouter.ai/api/v1",   // ← the URL
     apiKey: "<OPENROUTER_API_KEY>",
   });
   ```

4. Copy the `baseURL` value. Use it as it is for JS, and add a `/` at the end for .NET (Rule 2).
5. Test it:

   ```bash
   curl -s https://openrouter.ai/api/v1/models -H "Authorization: Bearer YOUR_KEY"
   ```

   A long list of models means the URL works. This list works even with a wrong key, so also send one chat message with the JS test in [12-free-api-tiers.md](12-free-api-tiers.md) to check the key.

**Model names:** go to **openrouter.ai/models** and type **free** in the search box. Click a model, then open its **API** tab. The sample there shows the exact name, like `meta-llama/llama-3.3-70b-instruct:free`. Make sure it ends in **`:free`**, or you'll be charged.

**Limits (free)**

- **20 requests a minute.**
- **50 requests a day** in total across all `:free` models.
- **1,000 requests a day** once you've bought **$10 of credit** in total, ever. The higher limit stays even after you spend the credit.
- The day resets at **00:00 UTC**.

**How much is that in practice**

- Without credit: **50 messages a day**, across all free models together. That's enough for a short coding session, but not for evals.
- After a one-time **$10** top-up: **1,000 a day**. This is the cheapest way to get a large daily free quota from any provider in these notes.

**Check what you have left**

There are two things to check: your **free requests** and your **credit balance**.

1. **Ask the API.** This works with any key:

   ```bash
   curl -s https://openrouter.ai/api/v1/key -H "Authorization: Bearer YOUR_KEY"
   ```

   The reply includes:
   - `limit_remaining`: credit left on this key, or `null` if the key has no spending cap
   - `usage`: what this key has spent so far
   - free-model requests per day (`limit`, `remaining`, `used`), if your account shows them. The docs aren't consistent about this field.
2. **Look in the website.**
   - **openrouter.ai/activity** lists every request you've made, with model, tokens and cost. Free models show `$0`.
   - **openrouter.ai/settings/credits** shows your credit balance.

**Other limitations**

- Free models come and go, and some are slow or busy at peak times. Keep a second `:free` model as a backup.
- Some free models are run by providers that may log your prompts. Check the model's page, and send only test data.
- If you drop the `:free` from a model name, you're charged from your credits.

**Best for:** switching between many models with one key.

## 2. Cloudflare Workers AI

**Connect**

1. Go to **dash.cloudflare.com** and sign up for a free account.
2. Find your **Account ID**. It's shown on the dashboard home page, or under **Workers & Pages** in the right-hand panel.
3. Go to **My Profile → API Tokens → Create Token**. Use the **Workers AI** template, then **Create Token**.
4. Copy the token.

**Use**

| Value | Setting |
|---|---|
| `LLM_BASE_URL` | `https://api.cloudflare.com/client/v4/accounts/YOUR_ACCOUNT_ID/ai/v1` |
| `LLM_API_KEY` | Your API token |
| `LLM_MODEL` | A name starting with `@cf/`, like `@cf/meta/llama-3.1-8b-instruct`. See the Workers AI models page |

**Find the URL in Cloudflare's docs**

Cloudflare's URL contains **your own Account ID**, so you need two things: the URL pattern from the docs, and your ID.

*Part 1: get the URL pattern*

1. Open the Workers AI docs at **developers.cloudflare.com/workers-ai**.
2. In the left menu, open **Configuration → OpenAI compatible API endpoints**. You can also search the docs for **OpenAI compatible**.
3. The page shows the address in a form like this:

   ```text
   https://api.cloudflare.com/client/v4/accounts/{account_id}/ai/v1/chat/completions
   ```

4. Cut off `/chat/completions` (Rule 1). That leaves:

   ```text
   https://api.cloudflare.com/client/v4/accounts/{account_id}/ai/v1
   ```

*Part 2: get your Account ID*

1. Sign in at **dash.cloudflare.com**.
2. Look at the browser's address bar. It shows something like `dash.cloudflare.com/1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d/home`.
3. The long string after `dash.cloudflare.com/` is your **Account ID**. You can also find it on the account's home page, under **Account ID**, with a copy button.

*Part 3: put them together and test*

1. Replace `{account_id}` with your ID:

   ```text
   https://api.cloudflare.com/client/v4/accounts/1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d/ai/v1
   ```

2. Test it with a short chat message:

   ```bash
   curl -s https://api.cloudflare.com/client/v4/accounts/YOUR_ACCOUNT_ID/ai/v1/chat/completions \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"model":"@cf/meta/llama-3.1-8b-instruct","messages":[{"role":"user","content":"hi"}]}'
   ```

   A reply containing `"choices"` means it works. An auth error means the token doesn't have Workers AI permission. Make a new one from the **Workers AI** template.

**Model names:** in the same docs, open **Models** in the left menu. Each model page shows its name, starting with `@cf/` (like `@cf/meta/llama-3.1-8b-instruct`). Pick models marked **Text Generation**. Use the full name, including `@cf/`.

Embedding models are also available, so it can cover RAG as well.

**Limits (free)**

- **10,000 "Neurons" a day.** A Neuron is Cloudflare's unit of compute.
- The pool is **shared across all models**, and big models use it up faster. Each model's cost in Neurons is on the Workers AI pricing page.
- The pool resets at **00:00 UTC**.

**How much is that in practice**

- It depends on the model. Neurons are counted per token, and each model costs a different number of Neurons per token. The **Workers AI pricing page** lists each model's rate.
- To work out your daily number: (Neurons per message for your model) × (messages) must stay under 10,000. Small 1–8B models give **far more** messages than big ones.

**Check what you have left**

- Sign in at **dash.cloudflare.com** and open **AI → Workers AI**. The dashboard shows your usage over time. Compare today's total with the 10,000 daily allowance.
- Cloudflare is moving from Neurons to a different billing unit, so the dashboard may show a different unit by the time you read this.
- On the free plan, when the daily Neurons run out, requests fail until 00:00 UTC. You're never charged unless you upgrade to Workers Paid.

**Other limitations**

- 10,000 Neurons is a lot of small-model calls but only a modest number of big-model calls. Use small models here.
- Mostly smaller open models, so quality is lower than Gemini Flash or big Groq models.

**Best for:** small apps you deploy on Cloudflare.

## 3. Backup providers

These also have free or trial tiers and OpenAI-compatible endpoints. I didn't check their current limits, so look at each console before relying on them.

| Provider | Connect | `LLM_BASE_URL` | Example `LLM_MODEL` |
|---|---|---|---|
| **Mistral** | **console.mistral.ai**: choose the free **Experiment** plan (needs phone verification), then **API Keys** | `https://api.mistral.ai/v1` | `mistral-small-latest` |
| **Cerebras** | **cloud.cerebras.ai**: sign up, then **API Keys** | `https://api.cerebras.ai/v1` | `llama3.1-8b` |
| **Hugging Face** | **huggingface.co**: **Settings → Access Tokens**, create a token with permission to call Inference Providers | `https://router.huggingface.co/v1` | Copy from the model's page |

Hugging Face gives only a small amount of free credit each month, so treat it as a last resort.

**Limits and checking what's left (not verified)**

| Provider | Free amount | Where to check |
|---|---|---|
| **Mistral** | Free **Experiment** plan, with low rate limits | **console.mistral.ai**: look for **Limits** or **Usage** |
| **Cerebras** | Free tier with daily request and per-minute token limits | **cloud.cerebras.ai**: look for **Limits** or **Usage**. Replies may also carry `x-ratelimit-*` headers (check with `curl -i`) |
| **Hugging Face** | A small amount of free **credit** each month | **huggingface.co/settings/billing** shows the credit used this month |

**Find the URL in Mistral's docs**

1. Open Mistral's docs at **docs.mistral.ai**.
2. Click **API** in the top menu. The page address is **docs.mistral.ai/api**.
3. Open **Chat → Chat Completion**. The page shows `POST /v1/chat/completions`, and the server is `https://api.mistral.ai`.
4. Put them together and cut off `/chat/completions` (Rule 1): **`https://api.mistral.ai/v1`**.
5. Test it:

   ```bash
   curl -s https://api.mistral.ai/v1/models -H "Authorization: Bearer YOUR_KEY"
   ```

**Find the URL in Cerebras's docs**

1. Open Cerebras's docs at **inference-docs.cerebras.ai**.
2. In the left menu, find **OpenAI Compatibility**, or search the docs for **OpenAI**.
3. The sample sets `base_url="https://api.cerebras.ai/v1"`. Copy it.
4. Test it:

   ```bash
   curl -s https://api.cerebras.ai/v1/models -H "Authorization: Bearer YOUR_KEY"
   ```

**Find the URL in Hugging Face's docs**

1. Open **huggingface.co/docs/inference-providers**.
2. Find the **Chat Completion** page, or a sample that uses the **OpenAI** client.
3. The sample sets `base_url="https://router.huggingface.co/v1"`. Copy it.
4. Test it:

   ```bash
   curl -s https://router.huggingface.co/v1/models -H "Authorization: Bearer YOUR_TOKEN"
   ```

For model names, use the `id` values from each test's reply.


> Steps and code not yet tested end to end. Console menus move around, so if a button isn't where a step says, look for the same words (**API Keys**, **Create Key**, **Tokens**, **OpenAI compatible**).
