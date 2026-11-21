# Free Cloud GPUs: Google Colab and Kaggle

Part of the $0 setup in [11-free-setup-overview.md](11-free-setup-overview.md). This note shows how to get a free GPU on Colab or Kaggle. When you have one, go to [16-run-ollama-on-free-gpu.md](16-run-ollama-on-free-gpu.md) to run a model on it and get the URL for your code.

> Limits were looked up in Nov 2026 and change often. Check each site before you depend on a number.

## 1. What they are, and how they relate

Colab and Kaggle are **two separate free services**. They are not connected to each other. Both offer the same kind of thing: a free computer with a GPU that you use through a notebook in your browser.

|                   | **Google Colab**                        | **Kaggle**                                                           |
| ----------------- | --------------------------------------- | -------------------------------------------------------------------- |
| Who runs it       | Google                                  | Kaggle (also owned by Google, but a separate site and account setup) |
| Website           | colab.research.google.com               | kaggle.com                                                           |
| Free GPU          | T4 (15 GB)                              | T4 (sometimes others)                                                |
| Limit style       | Hidden: Google decides how much you get | Clear: about 30 GPU hours a week, shown on screen                    |
| Notebook saved in | Your Google Drive                       | Your Kaggle account                                                  |

**Think of them as two free gyms.** You go to gym A (Colab). If it's full that day, you go to gym B (Kaggle) and do the same workout.

- They share nothing: no files, no sessions, no time quota. Using one doesn't reduce your time on the other, which is why having both gives you more free GPU time in total.
- The **setup cells are the same** on both (Steps A to F in [16-run-ollama-on-free-gpu.md](16-run-ollama-on-free-gpu.md)).
- **Your JS or .NET code doesn't care** which one you use. It only sees a URL (section 2 of [16-run-ollama-on-free-gpu.md](16-run-ollama-on-free-gpu.md)).

## 2. Google Colab

**All steps, from account to GPU (first time):**

1. **Get a Google account.** If you already use Gmail, you have one. If not, go to **accounts.google.com/signup**, fill in your name, birthday, a Gmail address and a password, and verify your phone number if it asks.
2. **Open Colab.** Go to **colab.research.google.com**. If you're not signed in, click **Sign in** (top right) and use your Google account.
3. **Create a notebook.** In the window that opens, click **New notebook**. If no window opens, use the menu **File → New notebook in Drive**.
4. **Give it a name.** Click the name at the top left (like `Untitled0.ipynb`) and rename it, for example `ollama-server.ipynb`. It's saved automatically in your Google Drive, in a folder called **Colab Notebooks**.
5. **Turn on the GPU.** Open the menu **Runtime → Change runtime type**. Under **Hardware accelerator**, pick **T4 GPU**, then click **Save**.
6. **Connect.** Click **Connect** (top right). When it's connected, it shows small **RAM** and **Disk** bars instead.
7. **Check that you got a GPU.** Click **+ Code** to add a cell, paste the line below, and press **Shift + Enter** to run it:

   ```python
   !nvidia-smi
   ```

   You should see a table that mentions **Tesla T4**. If you see an error like "NVIDIA-SMI has failed", you didn't get a GPU. Go back to step 5, or use Kaggle today.
8. **Continue with [16-run-ollama-on-free-gpu.md](16-run-ollama-on-free-gpu.md)** (Steps A to F). These steps get you the URL.

**Next time (a new session):** open the same notebook. Go to **colab.research.google.com**, use **File → Open notebook**, and pick it from the list. Then click **Connect**, and run all cells with **Runtime → Run all** (Ctrl + F9). Copy the new URL from the last cell.

**Limits:**

| Limit                      | What it means                                                                                                                                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **About 90 minutes idle**  | If you stop using the notebook for about 90 minutes, the session ends                                                                                                                                                |
| **About 12 hours at most** | Even if you're busy, a session can't run longer than about 12 hours. It's a maximum, not a promise: on the free tier, Google can end it earlier when GPUs are in demand or if you've used a lot of GPU time recently |
| **Hidden usage limits**    | Google doesn't publish how much GPU time you get. After heavy use you may see **"Cannot connect to GPU backend"** and have to wait, sometimes a few hours and sometimes about a day                                  |
| **Remote access**          | The free-tier rules restrict using Colab as a remote server, so keep the tunnel short                                                                                                                                |

**Can I start a new session straight after one ends?** Usually yes. Click **Connect** and rerun the setup cells. But if you've used a lot of GPU time recently, Colab may refuse you a GPU for a while. You can still connect **without a GPU** (CPU only), which is fine for writing code but too slow to run a 7B model. When that happens, switch to Kaggle.

**Save your work as you go.** Mount Google Drive and write important files, like eval results, there. Anything saved only on the Colab computer is lost when the session ends.

```python
from google.colab import drive
drive.mount('/content/drive')
# then save files under /content/drive/MyDrive/...
```

## 3. Kaggle

**All steps, from account to GPU (first time):**

1. **Create an account.** Go to **kaggle.com** and click **Register** (top right). The easiest way is **Register with Google**. You can also use an email address. Pick a username and accept the terms.
2. **Verify your phone number.** You need this before Kaggle will give you a GPU or turn on Internet in a notebook. Click your profile picture (top right), then **Settings**. Find **Phone verification**, enter your number, then type in the code you get by SMS. You only do this once.
3. **Create a notebook.** Click **+ Create** (top left), then **New Notebook**. A notebook editor opens and a session starts.
4. **Give it a name.** Click the name at the top (like `notebook1a2b3c`) and rename it, for example `ollama-server`.
5. **Turn on the GPU.** Open the **Settings** menu at the top of the notebook, or the **Session options** panel on the right. Set **Accelerator** to **GPU T4 x2** (or **GPU T4**), and confirm when it asks. Your weekly GPU hours start counting from now.
6. **Turn on Internet.** In the same place, switch **Internet** to **On**. Without it, the install and tunnel steps fail.
7. **Check that you got a GPU.** Click **+ Code** to add a cell, paste the line below, and press **Shift + Enter** to run it:

   ```python
   !nvidia-smi
   ```

   You should see a table that mentions **Tesla T4**.
8. **Continue with [16-run-ollama-on-free-gpu.md](16-run-ollama-on-free-gpu.md)** (Steps A to F). These steps get you the URL.

**Next time (a new session):** go to **kaggle.com → Code → Your Work** and open the notebook. Start the session, check that the GPU and Internet are still on, then use **Run All**. Copy the new URL from the last cell.

**When you're done:** stop the session (the power button, or **Run → Stop session**) so you don't waste your weekly GPU hours.

Kaggle changes its menus from time to time. If a button isn't where these steps say, look for the same words (**Accelerator**, **Internet**) in the notebook's settings.

**Limits:**

| Limit                          | What it means                                               |
| ------------------------------ | ----------------------------------------------------------- |
| **About 30 GPU hours a week**  | A clear weekly quota, shown on screen. It resets every week |
| **About 12 hours per session** | A single session stops after about 12 hours                 |
| **Idle timeout**               | A session also stops if it's idle for a while               |

**Can I start a new session straight after one ends?** Yes, as long as you have GPU hours left this week. Kaggle is more predictable than Colab because it shows how much you have left.

**Save your work as you go.** Files in `/kaggle/working` are kept when you **Save Version** the notebook. Anything else is lost when the session ends.

## 4. Using Colab and Kaggle together

**Order to try:**

1. Open **Colab** and run the setup cells. Paste the tunnel URL into your code.
2. If Colab says "Cannot connect to GPU backend", open **Kaggle**, run the **same setup cells**, and paste the new URL into your code.
3. If both are out of GPU time, use the free **API tiers** (Gemini or Groq) through your `chat()` function (section 3 of [11-free-setup-overview.md](11-free-setup-overview.md)). They need no GPU at all.

**Example: about 5 hours of work a day (about 35 hours a week)**

| Option         | What happens                                                                                                                 |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Colab only     | Some days are fine, but 5 GPU hours every day is heavy free use. Expect to be blocked now and then, maybe a few times a week |
| Kaggle only    | 30 hours a week covers about 6 days at 5 hours. You'd be short one day                                                       |
| Colab + Kaggle | Covered. Use Colab first and Kaggle as the backup. You'll rarely be fully blocked                                            |

**Better:** you won't need the GPU for all 5 hours, because much of that time is writing code, reading and debugging. Open a GPU session only when you're actually calling the local model.

| Time       | What you're doing                | Use                                                |
| ---------- | -------------------------------- | -------------------------------------------------- |
| ~3–4 hours | Coding and testing               | Free APIs (Gemini, then Groq when you hit a limit) |
| ~1–2 hours | Evals and open-model experiments | Colab, then Kaggle if Colab is blocked             |

That way you stay well under both limits and avoid restarting sessions all day.

**Problems and fixes:**

| Issue                                    | Fix                                                                                     |
| ---------------------------------------- | --------------------------------------------------------------------------------------- |
| Session ended                            | Rerun the setup cells. It takes about 3 minutes, and you get a new tunnel URL           |
| Model gone after a restart               | Download it again (7B is about 5 GB and quick), or save models to Google Drive on Colab |
| "Cannot connect to GPU backend" on Colab | Switch to Kaggle, or try Colab again later                                              |
| Kaggle weekly hours used up              | Use Colab, or the free APIs, until the weekly reset                                     |

Don't create extra accounts to get around the limits. It breaks the terms of both services and can get the accounts banned.
