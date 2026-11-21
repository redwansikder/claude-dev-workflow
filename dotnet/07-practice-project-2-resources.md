# Resources for Practice Project 2

Free resources for the Helpdesk Ticket System in [06-practice-project-2.md](06-practice-project-2.md), one or two per phase.

| Phase | Resource | Best for |
|---|---|---|
| 1 | [.NET Aspire docs](https://aspire.dev) | AppHost, adding Postgres/Redis, dashboard, deploy |
| 1, 3 | [Minimal APIs overview](https://learn.microsoft.com/aspnet/core/fundamentals/minimal-apis) | `MapGroup`, `TypedResults`, validation |
| 2 | [Npgsql EF Core provider](https://www.npgsql.org/efcore/) | EF Core with PostgreSQL, full-text search |
| 2, 7 | [EF Core interceptors](https://learn.microsoft.com/ef/core/logging-events-diagnostics/interceptors) | Auto timestamps, audit log |
| 4 | [JWT introduction](https://jwt.io/introduction) | How access tokens work (refresh tokens build on this) |
| 5 | [Upload files in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/file-uploads) | `IFormFile`, size limits, security checks |
| 5 | [Azurite emulator](https://learn.microsoft.com/azure/storage/common/storage-use-azurite) | Azure Blob Storage locally |
| 6 | [SignalR introduction](https://learn.microsoft.com/aspnet/core/signalr/introduction) | Hubs, groups, real-time messages |
| 6 | [Background tasks with hosted services](https://learn.microsoft.com/aspnet/core/fundamentals/host/hosted-services) | `BackgroundService`, timed jobs |
| 6 | [System.Threading.Channels](https://learn.microsoft.com/dotnet/core/extensions/channels) | An in-memory queue for emails |
| 6 | [MailKit](https://github.com/jstedfast/MailKit) + [Mailpit](https://mailpit.axllent.org) | Sending email, plus a fake inbox for testing |
| 7 | [HybridCache](https://learn.microsoft.com/aspnet/core/performance/caching/hybrid) | Caching the reports in memory + Redis |
| 8 | [Testcontainers for .NET](https://dotnet.testcontainers.org) | Integration tests against a real Postgres |

**Videos:** the channels in [05-free-resources.md](05-free-resources.md) cover these topics. Search "Milan Jovanović SignalR", "Milan Jovanović refresh tokens", "Nick Chapsas BackgroundService" and "Julio Casal Aspire".

**Note:** the Aspire docs moved to aspire.dev. If that link doesn't open, use [learn.microsoft.com/dotnet/aspire](https://learn.microsoft.com/dotnet/aspire).
