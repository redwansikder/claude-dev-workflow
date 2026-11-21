# Practice Project 2: Helpdesk Ticket System (.NET 10)

Customers open support tickets, agents reply and resolve them, and managers see reports.

**Why this project:** Project 1 ([04-practice-project.md](04-practice-project.md)) covered CRUD, auth, EF Core and concurrency. This one adds the next skills companies ask for: **real-time updates (SignalR), background jobs, file uploads, sending email, audit logs, reports, and Minimal APIs**. Between the two projects you'll have a strong CV.

**Stack:** ASP.NET Core 10 **Minimal APIs**, EF Core 10, PostgreSQL, SignalR, Redis, xUnit v3, Testcontainers, .NET Aspire

## Phase 1: Setup with Aspire (1–2 days)
1. Create the solution with Aspire: `dotnet new aspire-starter -n Helpdesk`. This gives you an AppHost, ServiceDefaults and an API project.
2. In the AppHost, add PostgreSQL and Redis: `builder.AddPostgres("db")`, `builder.AddRedis("cache")`
3. Use **Minimal APIs**, grouping endpoints by feature: `app.MapGroup("/tickets")`
4. Set up OpenAPI + Scalar, Git and a README

**Done when:** the Aspire dashboard shows the API, Postgres and Redis running.

## Phase 2: Database (2–3 days)
1. Entities: `User`, `Ticket`, `Comment`, `Attachment`, `AuditLog`
2. Ticket fields: `Status` (Open, InProgress, Resolved, Closed), `Priority` (Low to Urgent), `AssignedAgentId`, `DueAt`
3. Store enums as strings: `HasConversion<string>()`
4. Add `CreatedAt` / `UpdatedAt` automatically with a **SaveChanges interceptor**
5. Indexes on `Status`, `AssignedAgentId` and `CreatedAt`

**Done when:** migrations run, and timestamps fill in without you setting them in code.

## Phase 3: Tickets and comments (3–4 days)
1. Endpoints: create a ticket, list, get by id, update the status, assign an agent
2. Comments on tickets, with **internal notes** that only agents can see
3. **Status rules:** for example, a Closed ticket can't go back to InProgress. Put these rules in the `Ticket` entity (`ticket.Resolve()`), not in the endpoint.
4. Return errors with `TypedResults` and `Results<Ok<T>, NotFound, ValidationProblem>`
5. 🆕 Validation: use the .NET 10 built-in Minimal API validation, `builder.Services.AddValidation()`
6. **Cursor pagination** for the ticket list, which works better than `Skip`/`Take` on big tables

**Done when:** a full ticket lifecycle works and invalid status changes are rejected.

## Phase 4: Auth, with refresh tokens (2–3 days)
1. Roles: `Customer`, `Agent`, `Manager`
2. Short-lived JWT access tokens (15 minutes) plus **refresh tokens** stored in the database
3. `POST /auth/refresh` and `POST /auth/logout`, which revokes the refresh token
4. Customers see only their own tickets, agents see the ones assigned to them, managers see all

**Done when:** an expired access token can be refreshed, and logout blocks the old refresh token.

## Phase 5: File attachments (2 days)
1. Upload with `IFormFile`: limit the size (5 MB) and allow only certain types (png, jpg, pdf)
2. Check the real file type from its first bytes, not just the extension
3. Store files on disk locally, or in Azure Blob Storage (use the **Azurite** emulator locally)
4. Download endpoint, with an auth check so only people on that ticket can download

**Done when:** uploads are checked, and other users can't download someone else's file.

## Phase 6: Real-time and background jobs (3–4 days)
1. **SignalR hub:** when an agent replies, the customer sees it instantly without refreshing
2. Send to groups: one SignalR group per ticket (`ticket-{id}`)
3. **Background job** with `BackgroundService` + `PeriodicTimer`: every 5 minutes, find tickets past `DueAt` and escalate their priority
4. **Email notifications:** on a new reply, send an email. Use **MailKit**, and test locally with **Mailpit** (a fake inbox, added through Aspire).
5. Use a **channel** (`System.Threading.Channels`) as a queue so sending email doesn't slow the API response

**Done when:** a reply shows up live in a second browser tab, and overdue tickets get escalated by themselves.

## Phase 7: Audit log and reports (2–3 days)
1. **Audit log:** record who changed what and when, for status, assignment and priority. Do it in the SaveChanges interceptor using `ChangeTracker`.
2. Manager reports, built with LINQ `GroupBy`:
   - Tickets per status
   - Average resolution time per agent
   - Tickets created per day (last 30 days)
3. Cache the reports in Redis with **HybridCache** for 5 minutes
4. Export a report as CSV

**Done when:** every change appears in the audit log, and the reports load fast after the first call.

## Phase 8: Tests and deploy (3–4 days)
1. Unit tests for the ticket status rules and the escalation logic (with `FakeTimeProvider`)
2. Integration tests with Testcontainers (Postgres)
3. A SignalR test: connect a test client and check it receives the message
4. GitHub Actions: build, test, and publish a container image
5. Deploy to Azure Container Apps (Aspire can deploy straight there with `azd up`), Render or Railway

**Done when:** CI is green and the live app works with real-time updates.

## Bonus
- A small React or Blazor frontend to show the live updates
- Full-text search on tickets (PostgreSQL full-text search)
- Rate limiting on ticket creation
- OpenTelemetry traces, which show up in the Aspire dashboard

**Total:** about 5–6 weeks part-time.

**Interview stories this gives you:** real-time updates with SignalR, a background job for escalation, safe file uploads, refresh-token auth, an audit log built with an EF Core interceptor, and running everything locally with Aspire.
