# Free API Tiers: Connect, Use, Limits

Part of the $0 setup in [11-free-setup-overview.md](11-free-setup-overview.md). This note has what is the same for every free API provider: the test code (JS and .NET), the rules for base URLs, and a summary of all providers.

For each provider step by step (getting a key, finding the URL in its docs, testing it, limits), see [13-free-api-providers-part-1.md](13-free-api-providers-part-1.md) and [14-free-api-providers-part-2.md](14-free-api-providers-part-2.md).

> **About the limits:** they were looked up in Nov 2026, mostly from third-party sites, and sources often disagree. Where they disagree, a range is given. Model names and limits change often, so check each provider's console before you depend on them.

## 1. One piece of code for every provider

Every provider below has an **OpenAI-compatible** endpoint, so you write the code once. To switch provider, you change three values:

| Value | What it is |
|---|---|
| `LLM_BASE_URL` | The provider's endpoint |
| `LLM_API_KEY` | Your key from that provider |
| `LLM_MODEL` | A model name from that provider |

Store the key in a `.env` file (JS) or user secrets (.NET). **Never commit API keys to Git.**

### Rules for any base URL

Each provider section in notes 13 and 14 has a **"Find the URL in ... docs"** part. It says which page to open, what you'll see there, what to copy, and how to test it. These rules apply to all of them.

**Rule 1: Keep only the base part.** Some docs show the full address, with `/chat/completions` at the end. If so, cut that part off:

```text
https://api.groq.com/openai/v1/chat/completions
└──────── LLM_BASE_URL ───────┘└─ remove this ─┘
```

**Rule 2: The slash at the end depends on the language.**

| In | Write it as | Why |
|---|---|---|
| JavaScript `.env` | `https://api.groq.com/openai/v1` (no `/` at the end) | The code adds `/chat/completions` itself |
| .NET user secrets | `https://api.groq.com/openai/v1/` (with `/` at the end) | .NET drops the last part of the URL if the `/` is missing |

**Rule 3: Test before you write any code.** Each provider section has a `curl` test command. Paste it into a terminal (Git Bash, or the VS Code terminal) and put your key in place of `YOUR_KEY`. The result tells you what's wrong, if anything:

| What you get back | What it means |
|---|---|
| A list of models, or a short reply from the model | **The URL and key are right** |
| `404` or "Not Found" | Wrong URL. Check Rule 1, and look for typos |
| `401` / `403`, or "Invalid API key" / "Unauthorized" | The URL is right, but the key is wrong or is missing a permission. Copy the key again |
| `429` | The URL and key are right, but you've hit a rate limit. Wait and try again |

### Find the current model names

Model names change often. For most providers you can ask for the current list:

```bash
curl -s "$LLM_BASE_URL/models" -H "Authorization: Bearer $LLM_API_KEY"
```

The `id` values in the reply are the names you can use as `LLM_MODEL`. GitHub Models and Cloudflare work differently: their sections in notes 13 and 14 say where to find the names.

### Test call: JavaScript (Node 20+)

```js
// test.mjs  →  run: node --env-file=.env test.mjs
const res = await fetch(`${process.env.LLM_BASE_URL}/chat/completions`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.LLM_API_KEY}`,
  },
  body: JSON.stringify({
    model: process.env.LLM_MODEL,
    messages: [{ role: "user", content: "Explain RAG in 3 lines" }],
  }),
});

if (res.status === 429) {
  console.log("Rate limited. Wait, or switch provider.");
} else {
  const data = await res.json();
  console.log(data.choices[0].message.content);
}
```

Example `.env` file:

```text
LLM_BASE_URL=https://api.groq.com/openai/v1
LLM_API_KEY=gsk_your_key_here
LLM_MODEL=llama-3.1-8b-instant
```

### Test call: .NET (8+)

```bash
dotnet new console -n ApiTest
cd ApiTest
dotnet user-secrets init
dotnet user-secrets set "Llm:BaseUrl" "https://api.groq.com/openai/v1/"
dotnet user-secrets set "Llm:ApiKey"  "gsk_your_key_here"
dotnet user-secrets set "Llm:Model"   "llama-3.1-8b-instant"
dotnet add package Microsoft.Extensions.Configuration.UserSecrets
```

Note the **trailing `/`** on the base URL. .NET needs it to join the URL correctly.

Replace the contents of `Program.cs` with:

```csharp
using System.Net.Http.Headers;
using System.Net.Http.Json;
using System.Text.Json;
using Microsoft.Extensions.Configuration;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

var http = new HttpClient { BaseAddress = new Uri(config["Llm:BaseUrl"]!) };
http.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", config["Llm:ApiKey"]);

var res = await http.PostAsJsonAsync("chat/completions", new
{
    model = config["Llm:Model"],
    messages = new[] { new { role = "user", content = "Explain RAG in 3 lines" } }
});

if ((int)res.StatusCode == 429)
{
    Console.WriteLine("Rate limited. Wait, or switch provider.");
    return;
}
res.EnsureSuccessStatusCode();

var json = await res.Content.ReadFromJsonAsync<JsonElement>();
Console.WriteLine(json.GetProperty("choices")[0]
    .GetProperty("message").GetProperty("content").GetString());
```

Then run `dotnet run`.

### Free tiers count requests, not money

On a free tier you don't have a money **balance**. Instead, the provider counts your **requests** or **tokens** per minute and per day, and blocks you with `429` when you go over. Nothing is charged.

There are two exceptions where real credit is involved:
- **OpenRouter**, once you top up. Free `:free` models still cost $0.
- **Hugging Face**, which gives a small amount of free credit each month.

Each provider section in notes 13 and 14 has **How much is that in practice** and **Check what you have left**.

### Read the rate-limit headers in code

Some providers (Groq for sure, others sometimes) report what you have left in the **response headers** of every reply. Print any header with `ratelimit` or `retry-after` in its name to see what your provider sends.

**JavaScript:** add this after the `fetch` in the test call:

```js
for (const [name, value] of res.headers) {
  if (name.includes("ratelimit") || name === "retry-after") {
    console.log(`${name}: ${value}`);
  }
}
```

**.NET:** add this after `PostAsJsonAsync` in the test call:

```csharp
foreach (var header in res.Headers)
{
    if (header.Key.Contains("ratelimit", StringComparison.OrdinalIgnoreCase)
        || header.Key.Equals("retry-after", StringComparison.OrdinalIgnoreCase))
    {
        Console.WriteLine($"{header.Key}: {string.Join(",", header.Value)}");
    }
}
```

For example, Groq prints `x-ratelimit-remaining-requests` (requests left today) and `x-ratelimit-remaining-tokens` (tokens left this minute).

## 2. Each provider, step by step

Getting a key, finding the URL in each provider's docs, testing it, and the limits are in two notes:

| Note | Providers |
|---|---|
| [13-free-api-providers-part-1.md](13-free-api-providers-part-1.md) | Google Gemini, Groq, GitHub Models |
| [14-free-api-providers-part-2.md](14-free-api-providers-part-2.md) | OpenRouter, Cloudflare Workers AI, and backups (Mistral, Cerebras, Hugging Face) |

## 3. Summary

**Gemini**
- **Key starts with:** `AIza...`
- **Daily free amount:** about 20–1,500 requests (check your console)
- **Main limitation:** Flash models only. Google may use your prompts
- **How to handle it:** Flash is good enough for learning RAG, tools and agents, so use it for everything. Send only test or public data. When you hit the daily cap, switch to Groq
- **Check what's left:** the AI Studio **Rate limit / Usage** dashboard
- **In a real long chat:** with 20 messages a day, less than one long chat. With 1,500, you can chat all day
- **Best for:** main model and embeddings

**Groq**
- **Key starts with:** `gsk_...`
- **Daily free amount:** about 1,000–14,400 requests
- **Main limitation:** low tokens per minute (1–2 RAG messages a minute)
- **How to handle it:** keep prompts short. Send 2–3 retrieved chunks instead of 10, and only the last few chat messages. Use the small fast model (`llama-3.1-8b-instant`). On a 429, wait the `retry-after` seconds and try again
- **Check what's left:** the `x-ratelimit-remaining-*` headers on every reply
- **In a real long chat:** short chats only. After about 5–15 messages, the history is too big for the per-minute token limit
- **Best for:** backup model, fast agents

**GitHub Models**
- **Key starts with:** `github_pat_...`
- **Daily free amount:** 50–150 requests per model
- **Main limitation:** only about 8K tokens of input
- **How to handle it:** use it for short tests and for comparing models, not for long chats or big RAG prompts. Trim the history to the last few messages. When one model hits its daily cap, switch to another model of the same tier
- **Check what's left:** no counter, so keep your own count. On a 429, `retry-after` says how long to wait
- **In a real long chat:** about 5–10 messages per chat before the history passes 8K. About 150 messages a day, or 1–2 hours at a normal pace
- **Best for:** trying many models

**OpenRouter**
- **Key starts with:** `sk-or-...`
- **Daily free amount:** 50 requests, or 1,000 after a one-time $10 top-up
- **Main limitation:** free models come and go
- **How to handle it:** keep two or three `:free` model names in your config, and fall back to the next one when a call fails. If you can, top up $10 once to get 1,000 requests a day
- **Check what's left:** `GET /api/v1/key`, and **openrouter.ai/activity**
- **In a real long chat:** with 50 a day, about one long chat (1–2 hours), then you're done for the day. With 1,000, you can chat all day
- **Best for:** one key, many models

**Cloudflare Workers AI**
- **Key:** an API token
- **Daily free amount:** 10,000 Neurons
- **Main limitation:** mostly small models
- **How to handle it:** use it for simple jobs like short answers, classification and embeddings, not hard reasoning. Pick small models so the Neurons last, and send the harder questions to Gemini or Groq
- **Check what's left:** the dashboard, under **AI → Workers AI**
- **In a real long chat:** a few short chats. Long histories use up the Neurons fast
- **Best for:** small apps you deploy on Cloudflare

**What "a real long chat" means here**

The "In a real long chat" estimates above assume a conversation like a long session with an AI assistant: about **30–50 messages** over **2–3 hours**, with long answers.

- **Every message re-sends the whole conversation so far.** Message 1 might be 1,000 tokens, but by message 30 each request carries about 30,000–50,000 tokens of history.
- **Request limits** (Gemini, GitHub, OpenRouter) count **messages**. A long history doesn't use them up faster.
- **Token limits** (Groq's tokens per minute, GitHub's 8K input, Cloudflare's Neurons) are hit by the **growing history**. A chat that works at message 3 can fail at message 15.
- **At a normal human pace** of about one message every 1–2 minutes, a 5-hour day is about **150–300 messages**. Only Gemini (if your limit is high) and OpenRouter after $10 cover that comfortably.

**How to make free limits last longer in your own chat app**

- Send only the **last few messages**, not the whole history.
- Or **summarise older messages** into one short message and send that instead.
- Start a **new chat** when the topic changes.

These figures are estimates for planning, not measurements. Your real numbers depend on the model, your project's limits, and how long your messages are.

**When you hit a limit:** you get HTTP **429**. Either wait (per-minute limits clear in a minute, daily ones at the reset time) or switch `LLM_BASE_URL`, `LLM_API_KEY` and `LLM_MODEL` to the next provider.

> Steps and code not yet tested end to end. Console menus move around, so if a button isn't where a step says, look for the same words (**API Keys**, **Create Key**, **Tokens**).

## Sources

- [Gemini free tier guide](https://pecollective-com.pages.dev/tools/gemini-free-tier-guide/)
- [Gemini free tier limits 2026](https://yingtu.ai/en/blog/google-gemini-api-free-tier-limits-2026)
- [Groq free tier (benchlm)](https://benchlm.ai/free-tier/groq)
- [Groq free plan (costbench)](https://costbench.com/software/llm-api-providers/groq/free-plan/)
- [GitHub Models docs: prototyping with AI models](https://docs.github.com/en/github-models/use-github-models/prototyping-with-ai-models)
- [OpenRouter pricing](https://openrouter.ai/pricing)
- [OpenRouter rate limits (TrueFoundry)](https://www.truefoundry.com/es/blog/openrouter-rate-limits-your-free-tier-depends-on-what-youve-already-paid)
- [Cloudflare Workers AI pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/)
- [Free LLM APIs compared (OpenRouter blog)](https://openrouter.ai/blog/tutorials/free-llm-apis-compared/)
- [Groq docs: rate limits and headers](https://console.groq.com/docs/rate-limits)
- [Gemini docs: rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)
- [OpenRouter docs: limits](https://openrouter.ai/docs/api-reference/limits)
- [OpenRouter docs: get current API key](https://openrouter.ai/docs/api/api-reference/api-keys/get-current-key)
- [GitHub Models rate limits and 429 headers (DEV Community)](https://dev.to/devactivity/mastering-github-models-api-rate-limits-quotas-and-software-engineering-quality-4fk1)
