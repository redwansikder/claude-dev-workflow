# Run a Model on a Free GPU and Call It from Your Code

**Before you start:** you need a Colab or Kaggle notebook with the GPU turned on. If you don't have one yet, follow [15-free-gpu-colab-kaggle.md](15-free-gpu-colab-kaggle.md) (Colab steps 1–7 or Kaggle steps 1–7) first.

This note takes you from that notebook to a URL like `https://abc-xyz.trycloudflare.com`, and then shows how to call the model from JavaScript or .NET on your PC.

## 1. Steps A to F: from notebook to URL (same on Colab and Kaggle, run once per session)

> **What is a "session"?** A session is one period during which Colab or Kaggle lends you a computer with a GPU. It starts when you connect (or run your first cell) and ends when one of these happens:
>
> - you close it yourself (on Colab, **Runtime → Disconnect and delete runtime**)
> - you're idle for too long (about 90 minutes on Colab)
> - you reach the limit of about 12 hours
>
> **When a session ends, that computer is wiped.** Ollama, the downloaded model, the running server and the tunnel are all gone. Only your notebook (the code cells) stays saved.
>
> - **Inside one session:** run the setup cells once. Then you can call the model as many times as you like without rerunning them.
> - **When you come back later** (a new session): run the setup cells again from the top. It takes about 3 minutes, and you'll get a new tunnel URL to paste into your JS or .NET code.
>
> Think of it like a hotel room: each visit you get a clean room and have to unpack again.

The notebooks only run Python, so these cells use Python and shell commands. Your **app code on your PC** can be JS or .NET.

Put each step in its own cell: click **+ Code**, paste the code, and press **Shift + Enter**. Wait for each cell to finish (the spinner stops) before you run the next one.

**Step A: Install Ollama** (about 1 minute)

```python
!apt-get install -y zstd > /dev/null
!curl -fsSL https://ollama.com/install.sh | sh
```

The output should end with a line like `>>> The Ollama API is now available...` or `>>> Install complete`. A warning that systemd isn't running is normal; Step B handles that.

**Step B: Start the Ollama server in the background** (5 seconds)

```python
import subprocess, time
subprocess.Popen(["ollama", "serve"])
time.sleep(5)
```

This prints nothing. That's normal.

**Step C: Download a model** (about 2–5 minutes)

```python
!ollama pull qwen2.5:7b
```

You'll see progress bars, and then `success` at the end. 7–8B models fit easily on the T4. `llama3.1:8b` and `gemma2:9b` also work.

**Step D: Test the model inside the notebook**

```python
!curl -s http://localhost:11434/api/chat -d '{"model":"qwen2.5:7b","messages":[{"role":"user","content":"hi"}],"stream":false}'
```

You should get JSON back with `"message":{"role":"assistant","content":"Hello! ..."}`. If you get "connection refused", the server isn't running yet. Run Step B again, wait 10 seconds, and retry.

**Step E: Open the tunnel and get your URL** (about 10 seconds)

```python
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

The output is one line, like:

```text
https://abc-xyz.trycloudflare.com
```

That's your URL. Select it and copy it.

If nothing prints, the tunnel needs a few more seconds. Run just the last line again:

```python
!grep -o 'https://[a-z0-9-]*\.trycloudflare\.com' cf.log
```

**Step F: Check the URL, then paste it into your code**

1. Open the URL in your PC's browser. You should see the text **"Ollama is running"**. If you do, everything works.
2. Paste the URL into your code, with no slash at the end:

   ```js
   const BASE = "https://abc-xyz.trycloudflare.com";
   ```

3. **Keep the Colab or Kaggle tab open** while you work. If the session ends, the URL stops working, and you go back to Step A in a new session.

Before you rely on the tunnel, know three things:

- **Anyone who has the URL can use the model.** Don't share it.
- **The URL changes every session.**
- **Keep tunnel sessions short.** Colab's free-tier rules restrict remote-access use, and long-running tunnels can get your session cut off.

## 2. Call the model from your PC

This code is the same whether the model runs on Colab or Kaggle. Only the URL changes.

### JavaScript (Node 18+)

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

### .NET (6+)

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

## Next

- To switch between this model and free APIs (Gemini, Groq, OpenRouter) with one function, see section 3 of [11-free-setup-overview.md](11-free-setup-overview.md).
- If Colab or Kaggle stops giving you a GPU, see section 4 of [15-free-gpu-colab-kaggle.md](15-free-gpu-colab-kaggle.md).

> Code not yet tested end to end.
