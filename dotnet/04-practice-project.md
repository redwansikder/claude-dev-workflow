# Practice Project: Event Booking API (.NET 10)

A project for a developer with about one year of experience. Organizers create events. Users browse them and book tickets. Admins manage everything.

**Why this project:** it has real business rules (seat limits, cancellations), more than one role, relationships between tables, and a concurrency problem: two people booking the last seat at the same time. Those make good interview stories.

**Which version:** use **.NET 10**. It's LTS, supported until November 2028, and most companies will be on it. .NET 11 is a short-term release. Switching later only means changing `<TargetFramework>` in the `.csproj` files.

**Stack:** ASP.NET Core 10 Web API (controllers), EF Core 10, SQL Server or PostgreSQL, JWT, xUnit v3, Docker, OpenAPI + Scalar

Parts marked 🆕 are what changed in recent .NET versions.

## Phase 1: Setup (1–2 days)
1. 🆕 Create the API: `dotnet new webapi -n EventBooking.Api --use-controllers`. The template uses minimal APIs by default, so the flag gets you controllers.
2. 🆕 Create the solution: `dotnet new sln` now makes a **`.slnx`** file (the new XML format). Add projects with `dotnet sln add`.
3. Projects: `EventBooking.Core` (entities, interfaces), `EventBooking.Infrastructure` (EF Core), `EventBooking.Tests`
4. 🆕 **API docs:** Swagger is no longer in the template. It uses built-in OpenAPI: `builder.Services.AddOpenApi()` + `app.MapOpenApi()`. For a UI, add **Scalar** (`Scalar.AspNetCore` → `app.MapScalarApiReference()`), or add Swashbuckle yourself.
5. Set up Git with a `.gitignore`, `appsettings.Development.json`, and a `README.md` that explains how to run the project

**Done when:** the API runs and the Scalar UI opens at `/scalar`.

## Phase 2: Database and entities (2–3 days)
1. Entities: `User`, `Event`, `Category`, `Booking`
2. Relationships:
   - User → Bookings (one-to-many)
   - Event → Bookings (one-to-many)
   - Category → Events (one-to-many)
3. `AppDbContext` configured with the Fluent API (keys, max lengths, indexes)
4. Migrations: `dotnet ef migrations add Initial`, then `dotnet ef database update`
5. 🆕 Seed categories and an admin user with `UseSeeding` / `UseAsyncSeeding` in the DbContext options (EF Core 9+), instead of `HasData`
6. 🆕 A **global query filter** for soft delete, so deleted events stay hidden: `HasQueryFilter(e => !e.IsDeleted)`

**Done when:** the tables exist with the right foreign keys and the seed data is loaded.

## Phase 3: CRUD for Events (3–4 days)
1. DTOs: `CreateEventDto`, `UpdateEventDto`, `EventResponseDto`. Never return entities directly. 🆕 Use `record` types: `public record CreateEventDto(string Title, DateTime StartsAt, int Capacity);`
2. `IEventService` / `EventService`, registered with `AddScoped`
3. `EventsController` with GET (list), GET `{id}`, POST, PUT and DELETE
4. Correct status codes: 200, 201 (`CreatedAtAction`), 204, 400, 404
5. Validation: `[Required]`, `[Range]`, plus "the start date must be in the future"
6. List features: **pagination** (`Skip`/`Take`), **filtering** (category, date), **search** (title), **sorting**
7. 🆕 Inject **`TimeProvider`** instead of using `DateTime.UtcNow` directly, so date rules can be tested

**Done when:** all endpoints work in Scalar, with filters and pagination.

## Phase 4: Authentication and roles (3–4 days)
1. `POST /auth/register`: hash passwords with `PasswordHasher<T>`
2. `POST /auth/login`: return a JWT (`Microsoft.AspNetCore.Authentication.JwtBearer`)
3. Roles: `User`, `Organizer`, `Admin`
4. Protect endpoints:
   - Only an Organizer or Admin can create events
   - Only the event's owner can edit or delete it
   - 🆕 Use **policies** for owner checks: `AddAuthorizationBuilder().AddPolicy(...)` plus resource-based checks with `IAuthorizationService`
5. `GET /auth/me`: return the current user from the claims
6. 🆕 Know that `MapIdentityApi<TUser>()` exists as the built-in shortcut, but build JWT by hand here, because interviewers ask how it works.

**Done when:** endpoints return 401 without a token and 403 when the role is wrong.

## Phase 5: Booking logic, the important part (4–5 days)
1. `POST /events/{id}/bookings`: book N seats
2. Rules:
   - You can't book past events
   - You can't book more seats than are left
   - Each user can book at most 5 seats per event
3. `DELETE /bookings/{id}`: cancel, allowed only up to 24 hours before the event
4. `GET /bookings/my`: the user's own bookings, using `Include` to load event details
5. 🆕 **Concurrency, two ways (know both):**
   - **Atomic update:** `ExecuteUpdateAsync` with `WHERE SeatsLeft >= @n`, then check the number of rows affected. This is one SQL statement, so there's no race.
   - **Optimistic:** a `RowVersion` token on `Event` and catching `DbUpdateConcurrencyException`
6. Organizer: `GET /events/{id}/attendees`

**Done when:** two simultaneous bookings for the last seat can't both succeed.

## Phase 6: Quality (3–4 days)
1. 🆕 **Error handling:** `builder.Services.AddProblemDetails()` + a class that implements **`IExceptionHandler`**, registered with `AddExceptionHandler<T>()`. This replaces custom middleware.
2. FluentValidation, optional, to replace data annotations
3. 🆕 **Tests:** **xUnit v3**, with **NSubstitute** or Moq for mocks
4. Unit tests for the `BookingService` rules, using 🆕 `FakeTimeProvider` to control time
5. 🆕 **Integration tests:** `WebApplicationFactory` + **Testcontainers**, so tests run against a real SQL Server or PostgreSQL container instead of the in-memory provider
6. Logging: Serilog, or the built-in logging with OpenTelemetry
7. Check queries for N+1 problems and use `AsNoTracking` for reads

**Done when:** `dotnet test` passes, including integration tests against a real database.

## Phase 7: Deploy (2–3 days)
1. 🆕 **Container:** either write a `Dockerfile`, or use `dotnet publish /t:PublishContainer`, which builds an image without one. Add `docker-compose.yml` for the API and the database.
2. Secrets in environment variables (user secrets locally: `dotnet user-secrets`)
3. GitHub Actions running build and test on every push, using `actions/setup-dotnet` with `dotnet-version: 10.0.x`
4. Deploy to Azure App Service, Render or Railway
5. 🆕 Add health checks: `AddHealthChecks()` + `MapHealthChecks("/health")`
6. Update the README: the live URL, an architecture diagram, and how to run it locally

**Done when:** there's a live URL in the README, CI is green, and `/health` returns Healthy.

## Bonus (pick 1–2 if you have time)
- 🆕 **Rate limiting** is built in: `AddRateLimiter()` + `[EnableRateLimiting]` on login
- 🆕 **HybridCache** (`AddHybridCache()`) for popular events. It combines memory and Redis and prevents many requests hitting the database at once when the cache is empty.
- 🆕 **.NET Aspire:** run the API, database and Redis together with a dashboard. It's in demand on job posts.
- Email confirmation on booking with `BackgroundService` or Hangfire
- Refresh tokens
- A simple React or Angular frontend

**Total:** about 4–5 weeks part-time.

**Interview stories this gives you:** the seat-limit concurrency fix, role-based authorization, pagination and filtering, how you tested the booking rules with a fake clock, and how you deployed it.
