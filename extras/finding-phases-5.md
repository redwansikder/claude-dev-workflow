# Finding Your Phases: Example 5

A fifth worked example of the method in [Finding Your Phases](finding-phases-1.md). This one adds a new idea: **when the app needs two kinds of users, build for one side first and fill the other side yourself.** This is the "chicken and egg" problem.

---

## Example 5: Home services marketplace (Dhaka)

Customers book electricians, plumbers and AC technicians. Service providers get jobs.

### Step 1: List what users must be able to do

Write it all down, in any order:

**Customer**
- choose a service (AC servicing, plumbing, electrical)
- describe the problem, with a photo
- pick a date and time slot
- see the price or an estimate
- get a provider assigned
- see the provider's name, photo and rating
- pay cash or bKash after the job
- rate the provider
- book again with the same provider
- sign up and log in

**Service provider (technician)**
- sign up with NID, skills and area
- set the days they're available
- get job requests near them
- accept or reject a job
- mark a job done
- see their earnings

**Admin**
- verify provider NID and skills
- assign jobs when no provider accepts
- handle complaints and refunds
- take a commission from each job

### Step 2: Find the final thing

Ask: **what has to happen for *both* sides to be happy?**

> A customer's **job gets done** by a provider, and the provider **gets paid**.

### Step 3: Work backwards from it

```
Job done, provider paid
   ↑ needs: provider arrives at the right time
      ↑ needs: provider accepted the job
         ↑ needs: job request sent to nearby available providers
            ↑ needs: customer booked a service + slot
            ↑ needs: providers signed up, verified, available   ← a second chain!
```

The chain **splits into two**, one per side. Both must exist before a single job can happen.

### Step 3b: The new idea, which side first?

This is the chicken-and-egg problem:
- Customers won't use an app with no providers.
- Providers won't join an app with no jobs.

The usual answer: **real users come on the hard side; you play the other side by hand.**

| Side | Hard to get? | Decision |
|---|---|---|
| Providers | **Easier.** Technicians want more jobs, and you can personally recruit 10 good ones in a week. | **Recruit by hand.** You add them, verify them by phone, and keep them in the admin panel. No provider sign-up yet. |
| Customers | **Harder.** They need trust, and they're the reason the app exists. | **Build the real app for them.** |
| Job matching | Complex: distance, availability, accept/reject | **Admin assigns by hand.** You see each booking and call a provider. |
| Provider app | Lots of work (requests, accept, done) | **Not yet.** Providers get jobs by phone or WhatsApp from you. |

This uses the manual-workaround idea from [example 3](finding-phases-3.md), applied to a **whole user type** instead of one feature.

The short chain is now:

```
Job done (you confirm by phone)
   ↑ you assign a provider by hand
      ↑ customer books service + slot
         ↑ service list + prices
            ↑ admin adds providers (recruited by you)
               ↑ database + admin login + deploy
```

### Step 4: Read it in reverse and group it into phases

```
Phase 0: Setup              → database, admin login, deploy
Phase 1: Admin basics       → add services + prices, add providers
Phase 2: Customer booking   → pick service, describe problem, pick slot, confirm
Phase 3: Admin dispatch     → see bookings, assign a provider, mark done   ← the final thing
```

After Phase 3 **real jobs happen**. Customers use the real app, and you are the "matching system" with a phone. Running it by hand also teaches you how matching should work before you automate it.

### Step 5: Put the leftovers after the final thing

Bring the second side into the app **once doing it by hand gets too slow**:

| Leftover | Needs first | When |
|---|---|---|
| Provider app: see jobs, accept, mark done | providers + bookings | When you're making **20+ calls a day** |
| Auto-matching to nearby providers | provider app | When the provider app is in use |
| Provider self sign-up + NID verification | provider app | When you need **more providers than you can recruit** |
| Ratings and reviews | finished jobs | After the first 50–100 jobs |
| bKash payment + commission | finished jobs | When cash collection becomes a problem |
| Provider earnings page | payments | With payments |
| Book again with the same provider | ratings | Nice to have |
| Customer accounts | bookings | Phone number works until then |

```
Phase 4: Provider app (see jobs, accept, mark done)
Phase 5: Auto-matching by area + availability
Phase 6: Ratings + reviews
Phase 7: bKash payment + commission + earnings
Phase 8: Provider self sign-up + NID check
Phase 9: Customer accounts, book again
```

### The final Roadmap

```md
## Roadmap
- [ ] Phase 0: Setup (DB, admin login, deploy)  + recruit 10 providers by hand
- [ ] Phase 1: Admin adds services, prices, providers
- [ ] Phase 2: Customer booking (service, problem, slot)
- [ ] Phase 3: Admin dispatch by hand   ← real jobs start here
- [ ] Phase 4: Provider app (jobs, accept, done)
- [ ] Phase 5: Auto-matching
- [ ] Phase 6: Ratings + reviews
- [ ] Phase 7: bKash + commission + earnings
- [ ] Phase 8: Provider self sign-up + NID check
- [ ] Phase 9: Customer accounts, book again
```

---

## What this example adds

- **Two-sided apps have two chains.** Both sides must exist before the final thing can happen.
- **Pick one side to build for, and play the other side yourself.** Usually you build for the side that's harder to get (here, customers), and handle the easier side by hand.
- **Being the "algorithm" by hand teaches you the rules.** After assigning 100 jobs yourself, you'll know how auto-matching should work.
- **Bring the second side into the app when doing it by hand hurts.** "20+ calls a day" is a clear signal, unlike "someday".

## All five examples together

| Example | Idea it adds |
|---|---|
| [1. Doctor appointments](finding-phases-1.md) | The basic method: list, then work backwards, then leftovers |
| [2. Course platform](finding-phases-2.md) | Shorten the chain: can this link wait? |
| [3. Clothing shop](finding-phases-3.md) | Manual workarounds, and leftovers ordered by pain |
| [4. Bus tickets](finding-phases-4.md) | Links that must stay: test the riskiest first |
| 5. Home services | Two-sided apps: build one side, play the other by hand |
