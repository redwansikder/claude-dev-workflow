# Learning AI Engineering for $0 (No Local Model)

You can learn the whole path for free. Use **free cloud APIs** for most of the work, and **free cloud GPUs** when you want to run open models yourself.

> **About the limits below:** they were looked up in Nov 2026, but most come from third-party sites, and sources often disagree. Where they disagree, a range is given. Free limits change often, so check each provider's own console before you depend on a number.

## 1. Free API tiers

| Provider | What you get | Free limits (approx.) | Good for |
|---|---|---|---|
| **Google Gemini (AI Studio)** | Strong Flash models, long context, and free embeddings | Flash only. About 10–15 requests/min and somewhere from about **20 to 1,500 requests/day** (sources disagree a lot, so check your console). Pro models are paid-only or tightly capped. Limits apply per project, and the day resets at midnight Pacific | Main model for all projects |
| **Groq** | Open models (Llama, Qwen and others) on very fast hardware | About **30 requests/min**, about **1,000–14,400 requests/day**, and about 6K–30K tokens/min. The numbers vary by model and apply to your whole organisation, so extra keys don't help | Chat, agents, tool calling |
| **GitHub Models** | Several models with just a GitHub account | Low-tier models: **15/min, 150/day**. High-tier models: **10/min, 50/day**. About 8K input / 4K output tokens per request, and 5 requests at once. Limits apply per model | Trying different models |
| **OpenRouter** | Many `:free` models behind one API | **20/min**. **50/day**, or **1,000/day** once you've bought $10 of credit in total | Switching models without changing code |
| **Cloudflare Workers AI** | Hosted open models plus embeddings | **10,000 "Neurons" a day**, shared across all models (big models use more). Resets at 00:00 UTC | Small apps you deploy |
| **Mistral, Cerebras, Hugging Face** | Free or trial tiers | Varies; check each console | Backups when you hit a limit |

## 2. Free cloud GPUs: run your own models on Colab

| Platform | Free limits (approx.) |
|---|---|
| **Google Colab** | T4 GPU (15 GB). The session stops after about **90 minutes idle**, or about **12 hours** at most. A GPU isn't guaranteed at busy times. Remote-access use is restricted |
| **Kaggle** | T4 GPU, about **30 GPU hours a week**. Sessions are up to about 12 hours. You need to verify a phone number to use the GPU and turn Internet on |

Colab notebooks only run Python, so the **setup cells** below use Python and shell commands. Your **app code on your PC** can be JS or .NET.

### Step 1: Open Colab and turn on the GPU

1. Go to **colab.research.google.com** and sign in with Google.
2. Click **New notebook**.
3. Open the menu **Runtime → Change runtime type**, pick **T4 GPU**, then **Save**.

### Step 2: Setup cells (run once per session)

```python
# 1. Install Ollama
!apt-get install -y zstd > /dev/null
!curl -fsSL https://ollama.com/install.sh | sh

# 2. Start the server in the background
import subprocess, time
subprocess.Popen(["ollama", "serve"])
time.sleep(5)

# 3. Download a model (7–8B fits easily on the T4; llama3.1:8b and gemma2:9b also work)
!ollama pull qwen2.5:7b

# 4. Quick test inside Colab
!curl -s http://localhost:11434/api/chat -d '{"model":"qwen2.5:7b","messages":[{"role":"user","content":"hi"}],"stream":false}'

# 5. Open a tunnel so your PC can reach it
!wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -O cloudflared
!chmod +x cloudflared
subprocess.Popen(
    ["./cloudflared", "tunnel", "--url", "http://localhost:11434",
     "--http-host-header", "localhost:11434"],
    stdout=open("cf.log", "w"), stderr=subprocess.STDOUT,
)
time.sleep(8)
!grep -o 'https://[a-z0-9-]*\.trycloudflare\.com' cf.log
```

The last line prints a URL like `https://abc-xyz.trycloudflare.com`. Copy it.

Before you rely on the tunnel, know three things:

- **Anyone who has the URL can use the model.** Don't share it.
- **The URL changes every session.**
- **Colab's free-tier rules restrict remote-access use.** Long-running tunnels can get your session cut off. Use the tunnel for short sessions only.

### Step 3: Call it from your PC with JavaScript (Node 18+)

**Plain `fetch`, no packages:**

```js
// chat.mjs  →  run: node chat.mjs
const BASE = "https://abc-xyz.trycloudflare.com"; // your tunnel URL

const res = await fetch(`${BASE}/api/chat`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "qwen2.5:7b",
    messages: [{ role: "user", content: "Explain RAG in 3 lines" }],
    stream: false,
  }),
});

const data = await res.json();
console.log(data.message.content);
```

**Or with the official `ollama` package** (run `npm i ollama` first):

```js
import { Ollama } from "ollama";

const ollama = new Ollama({ host: "https://abc-xyz.trycloudflare.com" });
const r = await ollama.chat({
  model: "qwen2.5:7b",
  messages: [{ role: "user", content: "Explain RAG in 3 lines" }],
});
console.log(r.message.content);
```

### Step 3 (alternative): Call it from your PC with .NET (6+)

```bash
dotnet new console -n OllamaTest
cd OllamaTest
```

Replace the contents of `Program.cs` with:

```csharp
using System.Net.Http.Json;
using System.Text.Json;

var http = new HttpClient
{
    BaseAddress = new Uri("https://abc-xyz.trycloudflare.com") // your tunnel URL
};

var res = await http.PostAsJsonAsync("/api/chat", new
{
    model = "qwen2.5:7b",
    messages = new[] { new { role = "user", content = "Explain RAG in 3 lines" } },
    stream = false
});
res.EnsureSuccessStatusCode();

var json = await res.Content.ReadFromJsonAsync<JsonElement>();
Console.WriteLine(json.GetProperty("message").GetProperty("content").GetString());
```

Then run `dotnet run`.

### Kaggle setup

Go to kaggle.com, then **Create → Notebook**. In the side panel under **Settings**, set the **Accelerator to GPU T4** and turn **Internet on**. After that, run the same setup cells.

### Colab and Kaggle problems and fixes

| Issue | Fix |
|---|---|
| Session ended | Rerun the setup cells. It takes about 3 minutes, and you get a new tunnel URL |
| Model gone after a restart | Download it again (7B is about 5 GB and quick), or save models to Google Drive |
| No GPU available | Try later, or switch to Kaggle |

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
  if (res.status === 429) throw new Error("Rate limited: switch provider or wait");
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

| Provider | `LLM_BASE_URL` |
|---|---|
| Colab Ollama | `https://abc-xyz.trycloudflare.com/v1` |
| Groq | `https://api.groq.com/openai/v1` |
| OpenRouter | `https://openrouter.ai/api/v1` |
| Gemini | `https://generativelanguage.googleapis.com/v1beta/openai` |

In .NET, store the base URL, key and model in `appsettings.json` or user secrets.

## 4. Free storage and hosting

| Service | Free limits (approx.) |
|---|---|
| **Supabase** (Postgres + pgvector) | **500 MB** database, 1 GB file storage, 5 GB bandwidth, 2 projects. The project **pauses after 7 days of no activity**; your data is kept and you restore it from the dashboard |
| **Vercel Hobby** | Personal, **non-commercial** use only. About 100 GB transfer, 1M edge requests a month, and 100 deploys a day |
| **Render Free** | The app **sleeps after 15 minutes idle** and takes about **1 minute** to wake. 750 instance hours a month. The disk is wiped on every restart |
| **GitHub** | Free public and private repos for your code and CV |

## The catches

- **Rate limits:** free APIs return "429 too many requests". Handle it with retry and backoff, which is a real AI-engineering skill anyway.
- **Privacy:** some free tiers may use your prompts for training. Use only test data, never anything private.
- **Change:** free tiers change or disappear. Check the current limits before you depend on one.
- **Anthropic and Claude:** there's no permanent free API tier. The free Claude.ai chat is fine for learning and asking questions, but your code can't call it.

## Suggested setup for $0

1. Use **Gemini's free tier** as your main model and embeddings. Keep **Groq** as a backup; it has the highest daily limits.
2. Use **GitHub Models** and **OpenRouter** to try other models. Their daily limits are small (50–150 requests).
3. Use **Colab with Ollama** for eval runs that would use up an API's daily quota.
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
