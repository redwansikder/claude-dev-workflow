# Finding Your Phases

How to choose the phases for a new project's Roadmap (see [chapter 2](../02-NEW-PROJECT-GUIDE.md)).

The method has two parts:

1. **List what users must be able to do.** This gives you the *pieces*, in any order.
2. **Work backwards.** This puts the pieces in *order*.

Here's one project worked through from start to finish.

---

## Example: Doctor appointment app for a clinic

### Step 1: List what users must be able to do

Write down everything that comes to mind, in any order, as plain sentences, one per user type:

**Patient**
- book an appointment with a doctor
- see a doctor's available times
- search doctors by specialty
- get an SMS reminder before the appointment
- see their past appointments
- sign up and log in

**Doctor**
- set their weekly available hours
- see today's appointments
- write a short note after a visit

**Admin (clinic staff)**
- add doctors to the system
- cancel or move an appointment

The list is messy and unordered. That's fine.

### Step 2: Find the final thing

Ask: **what is the one thing the clinic is paying for?**

> A patient **books an appointment** with a doctor.

Everything else either supports it or comes after it.

### Step 3: Work backwards from it

Pick the final thing and keep asking **"what must exist before this works?"** Each answer comes from your Step 1 list:

```
Patient books an appointment
   ↑ needs: patient can see available times
      ↑ needs: doctor has set available hours
         ↑ needs: doctor exists in the system
            ↑ needs: admin can add doctors
               ↑ needs: login + database
```

### Step 4: Read the chain in reverse and group it into phases

```
Phase 0: Setup              → login + database + deploy
Phase 1: Doctors            → admin adds doctors, doctor sets hours
Phase 2: Find a doctor      → search by specialty, see available times
Phase 3: Booking            → patient books   ← the final thing
```

### Step 5: Put the leftovers after the final thing

Some items from Step 1 aren't in the chain. They're extras that build on booking:

| Leftover from Step 1 | Why it comes after booking |
|---|---|
| SMS reminder | needs a booking to remind about |
| past appointments | needs bookings to list |
| today's appointments (doctor) | needs bookings to show |
| note after a visit | needs an appointment to write on |
| cancel or move (admin) | needs a booking to change |

Group them by who uses them:

```
Phase 4: After booking      → patient's past list, admin cancel/move
Phase 5: Doctor's day       → today's list, visit notes
Phase 6: Reminders          → SMS before appointment
```

### The final Roadmap

```md
## Roadmap
- [ ] Phase 0: Setup (login, DB, deploy)
- [ ] Phase 1: Doctors (admin adds, doctor sets hours)
- [ ] Phase 2: Find a doctor (search, see times)
- [ ] Phase 3: Booking            ← the core; clinic can start using it here
- [ ] Phase 4: Manage bookings (history, cancel/move)
- [ ] Phase 5: Doctor's day (today's list, notes)
- [ ] Phase 6: SMS reminders
```

---

## What to remember

- **Step 1 gives the pieces.** Don't worry about order yet.
- **Working backwards orders the pieces** up to the core feature.
- **The leftovers become later phases**, because they all need the core feature to exist.
- **After Phase 3 the app is already useful.** The clinic can take real bookings while you build Phases 4–6. That's the sign of a good phase order: the most important thing ships first.

**Next:** [Example 2: Online course platform](finding-phases-2.md) shows how to make the chain shorter.
