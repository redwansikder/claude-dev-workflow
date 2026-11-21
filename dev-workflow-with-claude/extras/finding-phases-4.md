# Finding Your Phases: Example 4

A fourth worked example of the method in [Finding Your Phases](finding-phases-1.md). This one adds a new idea: **some links can't be removed or done by hand. When that happens, test the riskiest one first.**

---

## Example 4: Bus ticket booking (Dhaka ↔ other cities)

### Step 1: List what users must be able to do

Write it all down, in any order:

**Passenger**
- search buses by route and date (Dhaka → Chattogram, 15 Dec)
- see bus times, operator, AC/non-AC and price
- see the seat map and pick seats
- pay with bKash / Nagad / card
- get an e-ticket by SMS and email
- cancel a ticket and get a refund
- see past trips
- sign up and log in

**Bus operator (Green Line, Hanif, …)**
- add buses, routes and schedules
- set prices
- see booked seats for each trip
- print the passenger list for the supervisor
- block seats sold at their own counter

**Admin**
- add and approve operators
- see total sales and commission
- handle complaints

### Step 2: Find the final thing

Ask: **what is the one thing a passenger comes for?**

> A passenger **gets a confirmed seat** on a specific bus.

### Step 3: Work backwards from it

```
Passenger gets a confirmed seat
   ↑ needs: payment succeeds
      ↑ needs: seat is held while they pay
         ↑ needs: seat map shows free and booked seats
            ↑ needs: search shows the trip
               ↑ needs: operator has added bus, route, schedule, price
                  ↑ needs: login + database + deploy
```

### Step 3b: Check each link and try to remove it

Use the questions from [example 2](finding-phases-2.md) and [example 3](finding-phases-3.md): **"Can it wait?"** and **"Can a person do it by hand?"**

| Link | Can it wait? | By hand? | Decision |
|---|---|---|---|
| Online payment | **No.** Cash on delivery doesn't work for a seat: people would "book" and not show up. | No | **Must stay** |
| Seat hold while paying | **No.** Without it, two people pay for the same seat. | No | **Must stay** |
| Seat map | Mostly no, since passengers pick window or aisle | — | Keep |
| Operator adds schedules | — | **Yes.** At first, *you* enter the schedules for 1–2 operators from their paper timetable. | Operator panel moves out |
| Passenger login | Yes | Phone number is enough | Moves out |
| SMS e-ticket | Partly | A confirmation page plus a screenshot works at first | Simple version |

Unlike the shop in example 3, **two links can't be removed or done by hand**: payment and seat holding. They're also the **riskiest** parts:

- **Payment:** bKash's merchant API needs an approved account, takes weeks, and has callbacks and failures to handle.
- **Seat holding:** two people clicking the same seat at the same second. If that's wrong, the business fails on day one.

### Step 4: Put the risky parts first

The new rule: **if a link can't be removed, and you're not sure it will work, prove it first.** A small, ugly test of the riskiest parts goes right after setup:

```
Phase 0: Setup              → database, deploy, admin login
Phase 1: Risk test          → seat hold (10-minute lock) + bKash sandbox payment
                              on ONE hard-coded trip, no design
Phase 2: Trips              → admin enters operators, buses, schedules, prices
Phase 3: Search             → route + date search, trip list
Phase 4: Booking            → seat map → hold → pay → confirmation page   ← the final thing
```

Phase 1 looks backwards: the ugliest page is built before the search page. But if bKash approval takes 4 weeks, or seat locking has a bug, you find out in **week 1**, not in week 6.

Apply for the bKash merchant account on **day 1**, before writing any code.

### Step 5: Put the leftovers after the final thing

| Leftover | Needs first | Why this order |
|---|---|---|
| SMS / email e-ticket | booking | Passengers lose screenshots, so do this soon |
| Operator panel (own schedules, counter seats) | trips | **Counter seats are a risk**: if a seat sells at the counter *and* online, two people show up for it |
| Passenger list for the supervisor | bookings | Operators need it at the bus |
| Cancel + refund | payment | Refunds through bKash are another risk, so test them early in this phase |
| Passenger accounts + past trips | bookings | Nice to have |
| Admin sales + commission report | bookings | Needed by month-end payout |
| Complaints | bookings | Email works at first |

```
Phase 5: E-ticket by SMS + passenger list for operators
Phase 6: Operator panel (schedules, counter seat blocking)
Phase 7: Cancel + refund
Phase 8: Sales + commission report
Phase 9: Passenger accounts, complaints
```

### The final Roadmap

```md
## Roadmap
- [ ] Phase 0: Setup (DB, deploy, admin login)  + apply for bKash merchant account
- [ ] Phase 1: Risk test: seat hold + bKash sandbox on one trip
- [ ] Phase 2: Admin enters operators, buses, schedules
- [ ] Phase 3: Search by route + date
- [ ] Phase 4: Seat map → hold → pay → confirm   ← tickets sell here
- [ ] Phase 5: SMS e-ticket + passenger list
- [ ] Phase 6: Operator panel + counter seat blocking
- [ ] Phase 7: Cancel + refund
- [ ] Phase 8: Sales + commission report
- [ ] Phase 9: Passenger accounts, complaints
```

---

## What this example adds

- **Some links can't be removed.** "Can it wait?" and "by hand?" don't always work. For tickets, payment and seat locking are the product.
- **Test risky links first, even before the "normal" order.** A Phase 1 risk test with one hard-coded trip feels strange, but it saves you from building five phases on something that doesn't work.
- **Start slow outside tasks on day 1.** Merchant accounts, API keys and operator agreements take weeks. Start them before the code needs them.
- **Use the same lens on leftovers.** Counter seat blocking and refunds are risky, so they come before nice-to-haves like accounts.

## All four examples together

| Example | Idea it adds |
|---|---|
| [1. Doctor appointments](finding-phases-1.md) | The basic method: list, then work backwards, then leftovers |
| [2. Course platform](finding-phases-2.md) | Shorten the chain: can this link wait? |
| [3. Clothing shop](finding-phases-3.md) | Manual workarounds, and leftovers ordered by pain |
| 4. Bus tickets | Links that must stay: test the riskiest first |

**Next:** [Example 5: Home services marketplace](finding-phases-5.md) shows how to handle an app with two kinds of users.
