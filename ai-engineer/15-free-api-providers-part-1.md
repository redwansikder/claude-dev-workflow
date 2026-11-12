# Free API Providers, Part 1: Gemini, Groq, GitHub Models

For each provider: how to get a key, where to find the URL in its docs, what to put in your code, and the free limits.

Read [14-free-api-tiers.md](14-free-api-tiers.md) first. It has the test code (JS and .NET) and the **Rules for any base URL** (Rule 1–3) that these steps refer to. Part 2 is [16-free-api-providers-part-2.md](16-free-api-providers-part-2.md).

> Limits were looked up in Nov 2026, mostly from third-party sites. Check each provider's console before you depend on them.

## 1. Google Gemini (AI Studio)

**Connect**

1. Go to **aistudio.google.com** and sign in with your Google account.
2. Click **Get API key**, then **Create API key**. If it asks, let it create a Google Cloud project for you.
3. Copy the key. It starts with `AIza...`.

No card is needed for the free tier.

**Use**

| Value | Setting |
|---|---|
| `LLM_BASE_URL` | `https://generativelanguage.googleapis.com/v1beta/openai` |
| `LLM_API_KEY` | Your `AIza...` key |
| `LLM_MODEL` | A Flash model, such as `gemini-2.5-flash`. Check the list for newer ones |

**Find the URL in Gemini's docs**

1. Open Gemini's docs at **ai.google.dev/gemini-api/docs**.
2. In the left menu, find **OpenAI compatibility**, or use the search box at the top. The page address is **ai.google.dev/gemini-api/docs/openai**.
3. On that page, find the first code sample. Click the **JavaScript** tab if you prefer JS. It looks like this:

   ```python
   client = OpenAI(
       api_key="GEMINI_API_KEY",
       base_url="https://generativelanguage.googleapis.com/v1beta/openai/"   # ← the URL
   )
   ```

4. Copy the `base_url` value. Remove the `/` at the end for JS, or keep it for .NET (Rule 2).
5. Test it:

   ```bash
   curl -s https://generativelanguage.googleapis.com/v1beta/openai/models -H "Authorization: Bearer YOUR_KEY"
   ```

   A list of models means it works. The `id` values (like `models/gemini-2.5-flash`) are the model names. You can leave out the `models/` part when you use them.

**Don't confuse it with:** Gemini's other docs pages, which use Google's own API format at addresses like `.../v1beta/models/gemini-...:generateContent`. That format doesn't work with the code in this note. Always copy the URL from the **OpenAI compatibility** page.

Embeddings for RAG are also free. Use the same base URL with `/embeddings` and a Gemini embedding model from the list.

**Limits (free)**

- **Flash models only.** Pro models are paid-only or tightly capped.
- About **10–15 requests a minute**.
- Somewhere from about **20 to 1,500 requests a day**. Sources disagree a lot, so check **AI Studio → Usage / Rate limits** for your real number.
- Limits apply per **project**, not per key, so extra keys in the same project don't help.
- The daily count resets at **midnight Pacific time**.

**How much is that in practice**

- With a DocChat-style message (about 3,000 tokens in, 500 out), the daily cap is the limit you'll hit first. That's anywhere from **a few dozen to about 1,500 messages a day**, depending on what your project's limit is.
- The per-minute limit allows about one message every 4–6 seconds, which is fine for chatting and slow for a big eval run.

**Check what you have left**

- Gemini doesn't give a simple "X requests left" number. It shows **your limits and how close you are to them**.
- Sign in at **aistudio.google.com** and open the **Rate limit** / **Usage** dashboard from the left menu. It shows each model's limits for your project and your usage against them.
- When you run out, you get `429 RESOURCE_EXHAUSTED`. Read the `quota_metric` field in the error details: it says whether you hit the **per-minute** limit (wait a minute) or the **daily** limit (wait until midnight Pacific).

**Other limitations**

- On the free tier, Google may use your prompts to improve its products. Send only test data.
- No Pro models, so the free tier is weaker on hard reasoning.
- Google has cut its free limits several times. Don't build anything that depends on today's numbers.

**Best for:** your main model and embeddings.

## 2. Groq

**Connect**

1. Go to **console.groq.com** and sign up. Google or GitHub sign-in works.
2. Open **API Keys** and click **Create API Key**.
3. Copy the key. It starts with `gsk_...`, and it's shown only once.

No card is needed.

**Use**

| Value | Setting |
|---|---|
| `LLM_BASE_URL` | `https://api.groq.com/openai/v1` |
| `LLM_API_KEY` | Your `gsk_...` key |
| `LLM_MODEL` | For example `llama-3.1-8b-instant` (fast) or `llama-3.3-70b-versatile` (smarter). Check the list |

**Find the URL in Groq's docs**

1. Open Groq's docs at **console.groq.com/docs**. You can also click **Docs** in the top menu of the Groq console.
2. In the left menu, find **OpenAI Compatibility**. The page address is **console.groq.com/docs/openai**.
3. On that page you'll see a code sample like this:

   ```python
   client = openai.OpenAI(
       base_url="https://api.groq.com/openai/v1",   # ← the URL
       api_key=os.environ.get("GROQ_API_KEY")
   )
   ```

4. Copy the `base_url` value. Use it as it is for JS, and add a `/` at the end for .NET (Rule 2).
5. Test it:

   ```bash
   curl -s https://api.groq.com/openai/v1/models -H "Authorization: Bearer YOUR_KEY"
   ```

   A list of models means it works. The `id` values (like `llama-3.1-8b-instant`) are the model names.

**Model names** are also listed on the **Models** page in the left menu of the docs. It shows each model's context size and limits.

**Limits (free)**

- About **30 requests a minute**.
- About **1,000–14,400 requests a day**, depending on the model. Smaller models get more.
- About **6,000–30,000 tokens a minute**.
- Limits apply to your whole **organisation**, so extra keys don't help.
- Your exact numbers are in **console.groq.com → Settings → Limits**.

**How much is that in practice**

- **Per day:** 1,000 or more messages, which is more than you'll use while learning.
- **Per minute:** the **tokens-per-minute** limit is the problem. With a DocChat-style message (about 3,500 tokens in total), 6,000 tokens a minute means only about **1–2 messages a minute**. At 30,000 it's about 8. Short chat messages without RAG context get far more.

**Check what you have left**

Groq tells you on **every reply**, in the response headers:

| Header | What it tells you |
|---|---|
| `x-ratelimit-remaining-requests` | Requests left **today** |
| `x-ratelimit-remaining-tokens` | Tokens left **this minute** |
| `x-ratelimit-reset-requests` | Time until the daily count resets, like `2m59.56s` |
| `x-ratelimit-reset-tokens` | Time until the minute's tokens reset, like `7.66s` |
| `retry-after` | Only on a `429`: seconds to wait |

See them with `curl -i` (the `-i` prints the headers):

```bash
curl -s -i https://api.groq.com/openai/v1/chat/completions \
  -H "Authorization: Bearer YOUR_KEY" -H "Content-Type: application/json" \
  -d '{"model":"llama-3.1-8b-instant","messages":[{"role":"user","content":"hi"}]}' \
  | grep -i "ratelimit\|retry-after"
```

To read the headers in your own code, see "Read the rate-limit headers in code" in [14-free-api-tiers.md](14-free-api-tiers.md).

**Other limitations**

- Open models only (Llama, Qwen and others). No GPT or Gemini.
- The token-per-minute limit is easy to hit with long RAG prompts, even when your request count is low.
- Context size and limits differ per model. Check the **Models** page.

**Best for:** backup model with high daily limits, and very fast responses for agents and tool calling.

## 3. GitHub Models

**Connect**

1. Sign in to **github.com**.
2. Go to **Settings → Developer settings → Personal access tokens → Fine-grained tokens**, then **Generate new token**.
3. Give it a name and an expiry date. Under **Permissions**, set **Models** to **Read-only**.
4. Click **Generate token** and copy it. It starts with `github_pat_...`.

You can also try models in the browser at **github.com/marketplace/models** before writing code.

**Use**

| Value | Setting |
|---|---|
| `LLM_BASE_URL` | `https://models.github.ai/inference` |
| `LLM_API_KEY` | Your `github_pat_...` token |
| `LLM_MODEL` | A name like `openai/gpt-4.1-mini` or `meta/Llama-3.3-70B-Instruct`. Copy the exact name from the model's page in the marketplace |

**Find the URL in GitHub Models**

GitHub shows the URL next to each model, not on a separate docs page.

1. Go to **github.com/marketplace/models**, signed in to GitHub.
2. Click any model, for example **OpenAI GPT-4.1-mini** or **Meta Llama 3.3 70B Instruct**.
3. Click **Use this model** (top right of the model page).
4. In the panel that opens, choose the language **JavaScript** and the SDK **OpenAI SDK**. If you don't see that option, choose **REST**.
5. The sample code shows two lines you need:

   ```js
   const endpoint = "https://models.github.ai/inference";   // ← the URL
   const model = "openai/gpt-4.1-mini";                     // ← the model name
   ```

   With **REST** you'll see the full address instead: `https://models.github.ai/inference/chat/completions`. Cut off `/chat/completions` (Rule 1).
6. Test it. GitHub Models has no `/models` list at this URL, so send a short chat message instead:

   ```bash
   curl -s https://models.github.ai/inference/chat/completions \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"model":"openai/gpt-4.1-mini","messages":[{"role":"user","content":"hi"}]}'
   ```

   A reply containing `"choices"` means it works. A `401` or `403` means your token is missing the **Models: Read-only** permission.

**Model names:** each model's page shows its exact name in the **Use this model** sample. Copy it exactly, including the part before the `/` (like `openai/` or `meta/`).

**Don't confuse it with:** older guides that use `https://models.inference.ai.azure.com`. That's GitHub's old endpoint. Use the one from the current sample.

Older guides show `https://models.inference.ai.azure.com`. That's GitHub's old endpoint, so use the one from the current sample instead.

**Limits (free)**

| Model tier | Per minute | Per day | Per request |
|---|---|---|---|
| Low tier | 15 | 150 | About 8K tokens in, 4K out |
| High tier | 10 | 50 | About 8K tokens in, 4K out |

- At most **5 requests at the same time**.
- Limits apply **per model**, so using a few different models gives you more in total.
- Each model's page shows whether it's low or high tier.

**How much is that in practice**

- **150 messages a day per low-tier model**, or **50 per high-tier model**. Because the limits are per model, using three low-tier models gives you about 450 a day in total.
- Each message can carry only about **8,000 tokens of input**, which is roughly 6 pages of text, including your RAG context.

**Check what you have left**

- GitHub doesn't show a "requests left" counter for GitHub Models. Keep your own count, for example by logging each call.
- When you run out, you get `429`. The reply headers tell you more. Look for them with `curl -i`:
  - `retry-after`: how many **seconds** to wait. It can be many hours if you used up the daily limit.
  - `x-ratelimit-type`: which limit you hit. For example, `UserByModelByDay` means the daily limit for that model.
- The quickest fix is to switch to another model of the same tier, which has its own count.

**Other limitations**

- The 8K input limit is small for RAG. Keep the retrieved text short.
- At most 5 requests at the same time, so parallel eval runs are slow.
- It's meant for **prototyping**, not for a live app.

**Best for:** trying many different models, including well-known commercial ones, for free.


> Steps and code not yet tested end to end. Console menus move around, so if a button isn't where a step says, look for the same words (**API Keys**, **Create Key**, **Tokens**, **OpenAI compatibility**).
