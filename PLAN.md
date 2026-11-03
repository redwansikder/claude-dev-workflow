# Task: User profile page

## Goal
A page at `/profile` where a logged-in user can see and edit their name, email, and avatar.

## Steps
- [x] 1. Create page layout at `src/app/profile/page.tsx`
- [x] 2. Add `ProfileForm` component with name, email, avatar fields
- [ ] 3. Connect form to API (`GET /api/profile`, `PUT /api/profile`)
- [ ] 4. Add loading and error states
- [ ] 5. Add form validation (required name, valid email)
- [ ] 6. Test on mobile width and fix layout

## Progress notes
- 2026-10-31: Step 1 done. Layout uses a centered card, max-width 600px.
- 2026-10-31: Step 2 done. Form is UI only, uses mock data for now.

## Open questions
- Should avatar upload go to our server or to Cloudinary?

---

## Roadmap
- [x] Login & signup → `docs/plans/login.md`
- [ ] User profile page  ← current
- [ ] Account settings (password, delete account)
- [ ] Email notifications

## How to work on this plan
- Start a chat: `/resume-work` (shows where you stopped, checks half-done changes)
- Do one step: "Do step N"
- End each step: `/finish-step` (lint, build, tick, commit message)
- Then commit + push yourself
