# Learning AI Engineering for $0 (No Local Model)

You can learn the whole path for free. Use **free cloud APIs** for most of the work, and **free cloud GPUs** when you want to run open models yourself.

> **About the limits below:** they were looked up in Nov 2026, but most come from third-party sites, and sources often disagree. Where they disagree, a range is given. Free limits change often, so check each provider's own console before you depend on a number.

## 1. Free API tiers

| Provider                            | What you get                                               | Free limits (approx.)                                                                                                                                                                                                                                    | Good for                               |
| ----------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **Google Gemini (AI Studio)**       | Strong Flash models, long context, and free embeddings     | Flash only. About 10–15 requests/min and somewhere from about **20 to 1,500 requests/day** (sources disagree a lot, so check your console). Pro models are paid-only or tightly capped. Limits apply per project, and the day resets at midnight Pacific | Main model for all projects            |
| **Groq**                            | Open models (Llama, Qwen and others) on very fast hardware | About **30 requests/min**, about **1,000–14,400 requests/day**, and about 6K–30K tokens/min. The numbers vary by model and apply to your whole organisation, so extra keys don't help                                                                    | Chat, agents, tool calling             |
| **GitHub Models**                   | Several models with just a GitHub account                  | Low-tier models: **15/min, 150/day**. High-tier models: **10/min, 50/day**. About 8K input / 4K output tokens per request, and 5 requests at once. Limits apply per model                                                                                | Trying different models                |
| **OpenRouter**                      | Many `:free` models behind one API                         | **20/min**. **50/day**, or **1,000/day** once you've bought $10 of credit in total                                                                                                                                                                       | Switching models without changing code |
| **Cloudflare Workers AI**           | Hosted open models plus embeddings                         | **10,000 "Neurons" a day**, shared across all models (big models use more). Resets at 00:00 UTC                                                                                                                                                          | Small apps you deploy                  |
| **Mistral, Cerebras, Hugging Face** | Free or trial tiers                                        | Varies; check each console                                                                                                                                                                                                                               | Backups when you hit a limit           |

For every step to get each key, the values to put in your code, and more detail on the limits, see [14-free-api-tiers.md](14-free-api-tiers.md).

## 2. Free cloud GPUs: Google Colab and Kaggle

You can also run open models (Llama, Qwen, Gemma) yourself on a free cloud GPU. This is useful for eval runs that would use up an API's daily quota, and for learning how open models behave.

| Site | Free GPU | Limits (approx.) |
|---|---|---|
| **Google Colab** | T4 (15 GB) | Session ends after about 90 minutes idle or about 12 hours. Hidden usage limits |
| **Kaggle** | T4 | About 30 GPU hours a week. Sessions up to about 12 hours |

The step-by-step guides are in two separate notes:

1. [12-free-gpu-colab-kaggle.md](12-free-gpu-colab-kaggle.md) explains what Colab and Kaggle are and how they relate. It has every step from creating an account to turning on the GPU, plus the limits and how to use both together.
2. [13-run-ollama-on-free-gpu.md](13-run-ollama-on-free-gpu.md) shows how to install Ollama, download a model and get a URL like `https://abc-xyz.trycloudflare.com`. It also has JavaScript and .NET code to call the model.

## 3. Write one function and switch providers later

Colab Ollama, Gemini, Groq and OpenRouter all offer an **OpenAI-compatible** endpoint. When one provider hits its daily limit, you change the config and keep working.

**JavaScript:**

```js
async function chat(prompt) {
  const res = await fetch(`${process.env.LLM_BASE_URL}/chat/completions`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.LLM_API_KEY ?? "none"}`,
    },
    body: JSON.stringify({
      model: process.env.LLM_MODEL,
      messages: [{ role: "user", content: prompt }],
    }),
  });
  if (res.status === 429)
    throw new Error("Rate limited: switch provider or wait");
  const data = await res.json();
  return data.choices[0].message.content;
}
```

**.NET:**

```csharp
async Task<string> ChatAsync(HttpClient http, string model, string prompt)
{
    // http.BaseAddress = LLM_BASE_URL with a trailing "/";
    // set the Authorization header from LLM_API_KEY
    var res = await http.PostAsJsonAsync("chat/completions", new
    {
        model,
        messages = new[] { new { role = "user", content = prompt } }
    });
    if ((int)res.StatusCode == 429)
        throw new HttpRequestException("Rate limited: switch provider or wait");
    res.EnsureSuccessStatusCode();
    var json = await res.Content.ReadFromJsonAsync<JsonElement>();
    return json.GetProperty("choices")[0]
               .GetProperty("message").GetProperty("content").GetString()!;
}
```

| Provider     | `LLM_BASE_URL`                                            |
| ------------ | --------------------------------------------------------- |
| Colab Ollama | `https://abc-xyz.trycloudflare.com/v1`                    |
| Groq         | `https://api.groq.com/openai/v1`                          |
| OpenRouter   | `https://openrouter.ai/api/v1`                            |
| Gemini       | `https://generativelanguage.googleapis.com/v1beta/openai` |

In .NET, store the base URL, key and model in `appsettings.json` or user secrets.

## 4. Free storage and hosting

| Service                            | Free limits (approx.)                                                                                                                                                           |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Supabase** (Postgres + pgvector) | **500 MB** database, 1 GB file storage, 5 GB bandwidth, 2 projects. The project **pauses after 7 days of no activity**; your data is kept and you restore it from the dashboard |
| **Vercel Hobby**                   | Personal, **non-commercial** use only. About 100 GB transfer, 1M edge requests a month, and 100 deploys a day                                                                   |
| **Render Free**                    | The app **sleeps after 15 minutes idle** and takes about **1 minute** to wake. 750 instance hours a month. The disk is wiped on every restart                                   |
| **GitHub**                         | Free public and private repos for your code and CV                                                                                                                              |

## The catches

- **Rate limits:** free APIs return "429 too many requests". Handle it with retry and backoff, which is a real AI-engineering skill anyway.
- **Privacy:** some free tiers may use your prompts for training. Use only test data, never anything private.
- **Change:** free tiers change or disappear. Check the current limits before you depend on one.
- **Anthropic and Claude:** there's no permanent free API tier. The free Claude.ai chat is fine for learning and asking questions, but your code can't call it.

## Suggested setup for $0

1. Use **Gemini's free tier** as your main model and embeddings. Keep **Groq** as a backup; it has the highest daily limits.
2. Use **GitHub Models** and **OpenRouter** to try other models. Their daily limits are small (50–150 requests).
3. Use **Colab or Kaggle with Ollama** ([12](12-free-gpu-colab-kaggle.md), [13](13-run-ollama-on-free-gpu.md)) for eval runs that would use up an API's daily quota.
4. Call every model through the **one `chat()` / `ChatAsync()` function** above, so switching provider is a config change.
5. Build projects 06–08 this way. Deploy on Vercel with a Supabase database, and remember the 7-day pause.
6. Later, when you can, add **$5** of Claude API credit. Run your eval set across providers and write up the comparison in your README.

The whole path costs nothing except your PC and internet, same as normal dev. The extra work is handling rate limits and restarting Colab sessions.

> Code not yet tested end to end.

## Sources

- [Gemini free tier guide](https://pecollective-com.pages.dev/tools/gemini-free-tier-guide/)
- [Gemini pricing changes 2026](https://agentdeals.dev/gemini-api-pricing-changes)
- [Gemini free tier limits 2026](https://yingtu.ai/en/blog/google-gemini-api-free-tier-limits-2026)
- [Free LLM APIs compared (OpenRouter blog)](https://openrouter.ai/blog/tutorials/free-llm-apis-compared/)
- [Groq free tier (benchlm)](https://benchlm.ai/free-tier/groq)
- [Groq free plan (costbench)](https://costbench.com/software/llm-api-providers/groq/free-plan/)
- [OpenRouter pricing](https://openrouter.ai/pricing)
- [OpenRouter rate limits (TrueFoundry)](https://www.truefoundry.com/es/blog/openrouter-rate-limits-your-free-tier-depends-on-what-youve-already-paid)
- [GitHub Models docs: prototyping with AI models](https://docs.github.com/en/github-models/use-github-models/prototyping-with-ai-models)
- [Cloudflare Workers AI pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/)
- [Supabase free tier limits 2026](https://automationatlas.io/answers/supabase-free-tier-limits-2026/)
- [Vercel limits](https://vercel.com/docs/platform/limits)
- [Vercel free tier limits 2026](https://deploywise.dev/blog/vercel-free-tier-limits-2026)
- [Render: Deploy for Free](https://render.com/docs/free)
