# Resources for the Avatar Upload API

Free resources for the guide in [08-avatar-upload-api.md](08-avatar-upload-api.md), matched to its steps.

| Step | Resource | Best for |
|---|---|---|
| 0 | [.NET 10 SDK download](https://dotnet.microsoft.com/download/dotnet/10.0) | Installing the SDK |
| 2 | [Supabase Storage overview](https://supabase.com/docs/guides/storage) | How buckets and files work |
| 2 | [Creating buckets](https://supabase.com/docs/guides/storage/buckets/creating-buckets) | Public buckets, allowed MIME types |
| 2 | [Storage file limits](https://supabase.com/docs/guides/storage/uploads/file-limits) | Bucket and project size limits |
| 2 | [Connecting to Postgres](https://supabase.com/docs/guides/database/connecting-to-postgres) | Session pooler vs direct connection |
| 2, 4 | [Supabase API keys](https://supabase.com/docs/guides/api/api-keys) | Service role and secret keys, what each can access |
| 4 | [Safe storage of app secrets](https://learn.microsoft.com/aspnet/core/security/app-secrets) | `dotnet user-secrets` |
| 5, 10 | [Npgsql EF Core provider](https://www.npgsql.org/efcore/) | EF Core with PostgreSQL |
| 6 | [IHttpClientFactory](https://learn.microsoft.com/aspnet/core/fundamentals/http-requests) | Typed clients like `SupabaseStorageClient` |
| 6, 9 | [Options pattern](https://learn.microsoft.com/aspnet/core/fundamentals/configuration/options) | Binding `SupabaseOptions` from configuration |
| 6, 11 | [`createSignedUploadUrl` reference](https://supabase.com/docs/reference/javascript/storage-from-createsigneduploadurl) | How signed upload URLs behave and how long they last |
| 7 | [Create web APIs with ASP.NET Core](https://learn.microsoft.com/aspnet/core/web-api/) | `[ApiController]`, `ControllerBase`, return types |
| 7 | [Controller-based API tutorial](https://learn.microsoft.com/aspnet/core/tutorials/first-web-api) | A full walkthrough if controllers are new to you |
| 7 | [EF Core transactions](https://learn.microsoft.com/ef/core/saving/transactions) | `BeginTransactionAsync`, commit and rollback |
| 7, 8 | [EF Core SQL queries](https://learn.microsoft.com/ef/core/querying/sql-queries) | `FromSql` and safe parameters |
| 7, 8 | [PostgreSQL locking clause](https://www.postgresql.org/docs/current/sql-select.html#SQL-FOR-UPDATE-SHARE) | `FOR UPDATE` and `SKIP LOCKED` |
| 8 | [Background tasks with hosted services](https://learn.microsoft.com/aspnet/core/fundamentals/host/hosted-services) | `BackgroundService`, timed jobs |
| 10 | [EF Core migrations](https://learn.microsoft.com/ef/core/managing-schemas/migrations/) | `dotnet ef migrations add`, `database update` |
| 10 | [Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security) | Locking tables away from the public Data API |
| 11 | [Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) | Sending JSON and files from the browser |
| 13 | [ASP.NET Core authentication](https://learn.microsoft.com/aspnet/core/security/authentication/) | Adding login to the controller |
| 13 | [Supabase JWTs](https://supabase.com/docs/guides/auth/jwts) | Validating Supabase Auth tokens on your API |
| 13 | [Rate limiting middleware](https://learn.microsoft.com/aspnet/core/performance/rate-limit) | `AddRateLimiter`, `[EnableRateLimiting]` |
| 13 | [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) | What to check before trusting an uploaded file |
