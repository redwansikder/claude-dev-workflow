# Finding Your Phases: Example 2

A second worked example of the method in [Finding Your Phases](finding-phases-1.md). This one adds a new idea: **keep the chain as short as possible**, because some things that look required can wait.

---

## Example 2: Online course platform (like a small Udemy for Bangla courses)

### Step 1: List what users must be able to do

Write it all down, in any order:

**Student**
- sign up and log in
- browse courses
- watch a course's lesson videos
- buy a course (bKash / card)
- see their progress (lessons completed)
- take a quiz after a lesson
- get a certificate when the course is done
- leave a review and rating

**Instructor**
- create a course (title, price, description)
- upload lesson videos
- see how many students enrolled
- see their earnings

**Admin**
- approve new courses before they go live
- pay instructors every month

### Step 2: Find the final thing

Ask: **what is the one thing everyone is here for?**

> A student **watches a lesson** of a course they're enrolled in.

Not certificates, not reviews. If students can't watch lessons, nothing else matters.

### Step 3: Work backwards from it

```
Student watches a lesson
   ↑ needs: student is enrolled in the course
      ↑ needs: student can buy the course?          ← wait, check this
         ↑ needs: course page to buy from
            ↑ needs: course exists with lessons
               ↑ needs: instructor creates course + uploads videos
                  ↑ needs: login + database + video storage
```

### Step 3b: The new idea, checking each link

For every link, ask: **"Is this *really* needed before the final thing, or can it wait?"**

| Link | Really needed first? | Decision |
|---|---|---|
| Buy the course | **No.** Start with free enrollment ("Enroll" button). Payment can come later. | Move out of the chain |
| Admin approves courses | **No.** At first only invited instructors exist, so you trust them. | Move out of the chain |
| Course page | Yes. Students need somewhere to click Enroll. | Keep |
| Instructor uploads videos | Yes. Nothing to watch otherwise. | Keep |

The short chain is now:

```
Watch a lesson
   ↑ enrolled (free)
      ↑ course page
         ↑ instructor creates course + uploads videos
            ↑ login + database + video storage
```

### Step 4: Read it in reverse and group it into phases

```
Phase 0: Setup              → login, database, video storage, deploy
Phase 1: Instructor side    → create course, upload lessons
Phase 2: Student side       → browse courses, course page, free enroll
Phase 3: Learning           → watch lessons   ← the final thing
```

After Phase 3 you can run a **real test**: invite 10 students to a free course and watch whether they actually finish lessons.

### Step 5: Put the leftovers after the final thing

| Leftover | Needs first |
|---|---|
| Buy a course | enrollment working (Phase 2) |
| Progress tracking | lessons being watched (Phase 3) |
| Quizzes | lessons to attach them to |
| Certificate | progress tracking, to know the course is done |
| Reviews and ratings | students who took the course |
| Instructor enroll count + earnings | enrollments and payments |
| Admin approval | many instructors, enough that trust matters |
| Monthly instructor payout | earnings |

Leftovers can depend on each other too: a certificate needs progress tracking. Order them the same way, by working backwards.

```
Phase 4: Payments           → buy course, instructor earnings
Phase 5: Progress           → progress tracking, quizzes
Phase 6: Certificates       → needs progress
Phase 7: Trust & admin      → reviews, course approval, payouts
```

### The final Roadmap

```md
## Roadmap
- [ ] Phase 0: Setup (login, DB, video storage, deploy)
- [ ] Phase 1: Instructor creates course + lessons
- [ ] Phase 2: Browse + course page + free enroll
- [ ] Phase 3: Watch lessons      ← core; test with real students here
- [ ] Phase 4: Payments + earnings
- [ ] Phase 5: Progress + quizzes
- [ ] Phase 6: Certificates
- [ ] Phase 7: Reviews, admin approval, payouts
```

---

## What this example adds to example 1

- **Question every link in the chain.** "Must students pay before watching?" No, not at first. Each link you remove makes the core feature arrive sooner.
- **Free first, paid later** is a common pattern. It lets you test whether people want the product before building payments.
- **Leftovers can have their own chain.** Certificates need progress, so progress comes first.
- **Admin features often come late.** At the start you're the admin, and you can fix things by hand in the database.

**Next:** [Example 3: Online clothing shop](finding-phases-3.md) shows how manual workarounds open the shop sooner.
