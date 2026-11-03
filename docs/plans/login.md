# Task: Login & signup

## Goal
Users can sign up, log in, and log out with email and password.

## Steps
- [x] 1. Add NextAuth with the credentials provider
- [x] 2. Create `/login` and `/signup` pages
- [x] 3. Hash passwords with bcrypt before saving
- [x] 4. Protect logged-in pages with middleware
- [x] 5. Add a logout button to the header

## Progress notes
- 2026-10-27: Step 1 done. Session uses JWT, no database sessions.
- 2026-10-27: Step 2 done. Both pages share the `AuthForm` component.
- 2026-10-28: Step 3 done.
- 2026-10-29: Step 4 done. Middleware redirects to `/login?next=<page>`.
- 2026-10-29: Step 5 done. Task finished and archived here.

## Open questions
- (none left: "Allow Google login?" was moved to a later task)
