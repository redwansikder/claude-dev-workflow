# Avatar Upload App on .NET 10: Step-by-Step Guide

This guide builds a .NET 10 ASP.NET Core Web API, using controllers, that creates a user with an avatar image. The browser uploads the file straight to Supabase Storage, so the file bytes never pass through your server.

The design uses an **upload intent**: the database records each upload before it happens. The database stays the source of truth, a failed upload leaves nothing broken, and abandoned uploads get cleaned up.

Commands are written for PowerShell on Windows.

## Step 0: Prerequisites

1. Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0), then open a new terminal and check it:

   ```powershell
   dotnet --list-sdks
   ```

   You should see a line starting with `10.`.

2. Install the Entity Framework command-line tool:

   ```powershell
   dotnet tool install --global dotnet-ef
   ```

3. Create a free project at [supabase.com](https://supabase.com).

## Step 1: How the app works

```
Browser                     .NET API                    Supabase
   |  POST /intents            |                           |
   |-------------------------->|  insert row (Pending)     |
   |                           |-------------------------->|
   |                           |  request signed URL       |
   |                           |-------------------------->|
   |  intentId + uploadUrl     |                           |
   |<--------------------------|                           |
   |  PUT file directly to Storage                         |
   |------------------------------------------------------>|
   |  POST /{id}/finalize      |                           |
   |-------------------------->|  check file exists        |
   |                           |-------------------------->|
   |                           |  insert user, mark        |
   |                           |  intent Completed         |
   |  user                     |-------------------------->|
   |<--------------------------|                           |
```

If the user closes the tab, the intent stays `Pending`. A background job later deletes the row and any file that was uploaded.

## Step 2: Set up Supabase

1. Go to **Storage** and create a bucket:
   - Name: `avatars`
   - Public bucket: on
   - File size limit: `10 MB`
   - Allowed MIME types: `image/jpeg, image/png, image/webp, image/gif`

   These bucket settings are the real enforcement. The browser talks to Storage directly, so checks in your API alone can be bypassed.

2. Copy these values from your project settings:
   - **Project URL**, such as `https://abcd1234.supabase.co`
   - **Service role key** (or a secret key). Keep it on the server. Anyone holding it has full access to your project.
   - **Session pooler connection details**, from the **Connect** dialog. Use the Session pooler because the direct connection needs IPv6.

## Step 3: Create the project

```powershell
dotnet new web -n AvatarUploads -f net10.0
cd AvatarUploads
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Design
```

## Step 4: Store the configuration

User secrets keep credentials out of source control. Replace the placeholder values with your own:

```powershell
dotnet user-secrets init
dotnet user-secrets set "Supabase:Url" "https://abcd1234.supabase.co"
dotnet user-secrets set "Supabase:ServiceRoleKey" "YOUR_SERVICE_ROLE_KEY"
dotnet user-secrets set "ConnectionStrings:Default" "Host=YOUR_POOLER_HOST;Port=5432;Database=postgres;Username=postgres.YOUR_PROJECT_REF;Password=YOUR_DB_PASSWORD;SSL Mode=Require"
```

In production, set the same values as environment variables, such as `Supabase__Url` and `ConnectionStrings__Default`.

## Step 5: Add the data model

Create `Data.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

namespace AvatarUploads;

public enum UploadStatus
{
    Pending,
    Completed
}

public sealed class UploadIntent
{
    public Guid Id { get; set; }
    public string Username { get; set; } = "";
    public string FileKey { get; set; } = "";
    public UploadStatus Status { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
}

public sealed class AppUser
{
    public Guid Id { get; set; }
    public string Username { get; set; } = "";
    public string AvatarUrl { get; set; } = "";
    public Guid UploadIntentId { get; set; }
}

public sealed class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<UploadIntent> UploadIntents => Set<UploadIntent>();
    public DbSet<AppUser> Users => Set<AppUser>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<UploadIntent>(entity =>
        {
            entity.Property(x => x.Status).HasConversion<string>().HasMaxLength(20);
            entity.HasIndex(x => x.FileKey).IsUnique();
            entity.HasIndex(x => new { x.Status, x.CreatedAt });
        });

        // One intent can only ever create one user.
        modelBuilder.Entity<AppUser>().HasIndex(x => x.UploadIntentId).IsUnique();
    }
}
```

The username is stored on the intent, so the finalize step doesn't have to trust anything new from the browser.

## Step 6: Add the storage client

Create `SupabaseStorageClient.cs`:

```csharp
using System.Net;
using Microsoft.Extensions.Options;

namespace AvatarUploads;

public sealed class SupabaseOptions
{
    public string Url { get; set; } = "";
    public string ServiceRoleKey { get; set; } = "";
    public string Bucket { get; set; } = "avatars";
}

public sealed class SupabaseStorageClient(HttpClient http, IOptions<SupabaseOptions> options)
{
    private readonly SupabaseOptions _options = options.Value;

    private string StorageRoot => $"{_options.Url.TrimEnd('/')}/storage/v1";

    public async Task<string> CreateSignedUploadUrlAsync(string key, CancellationToken ct)
    {
        using var response = await http.PostAsJsonAsync(
            $"object/upload/sign/{_options.Bucket}/{key}", new { }, ct);
        response.EnsureSuccessStatusCode();

        var body = await response.Content.ReadFromJsonAsync<SignedUrlResponse>(ct)
            ?? throw new InvalidOperationException("Empty response from Supabase Storage.");

        // Supabase returns a relative URL that already contains the token.
        return StorageRoot + body.Url;
    }

    public async Task<bool> ObjectExistsAsync(string key, CancellationToken ct)
    {
        using var response = await http.GetAsync($"object/info/{_options.Bucket}/{key}", ct);

        // Storage can report a missing object as either 404 or 400.
        if (response.StatusCode is HttpStatusCode.NotFound or HttpStatusCode.BadRequest)
        {
            return false;
        }

        response.EnsureSuccessStatusCode();
        return true;
    }

    public async Task DeleteObjectAsync(string key, CancellationToken ct)
    {
        using var request = new HttpRequestMessage(HttpMethod.Delete, $"object/{_options.Bucket}")
        {
            Content = JsonContent.Create(new { prefixes = new[] { key } })
        };

        using var response = await http.SendAsync(request, ct);
        response.EnsureSuccessStatusCode();
    }

    public string GetPublicUrl(string key) =>
        $"{StorageRoot}/object/public/{_options.Bucket}/{key}";

    private sealed record SignedUrlResponse(string Url);
}
```

## Step 7: Add the API controller

Create a `Controllers` folder, then create `Controllers/UploadsController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace AvatarUploads.Controllers;

public sealed record CreateIntentRequest(string? Username, string? MimeType, long Size);
public sealed record CreateIntentResponse(Guid IntentId, string UploadUrl);

[ApiController]
[Route("api/uploads")]
public sealed class UploadsController(AppDbContext db, SupabaseStorageClient storage) : ControllerBase
{
    private const long MaxFileSize = 10L * 1024 * 1024;

    private static readonly Dictionary<string, string> AllowedTypes =
        new(StringComparer.OrdinalIgnoreCase)
        {
            ["image/jpeg"] = ".jpg",
            ["image/png"] = ".png",
            ["image/webp"] = ".webp",
            ["image/gif"] = ".gif"
        };

    [HttpPost("intents")]
    public async Task<IActionResult> CreateIntent(CreateIntentRequest request, CancellationToken ct)
    {
        var username = request.Username?.Trim();
        if (string.IsNullOrEmpty(username) || username.Length > 50)
        {
            return BadRequest(new { error = "Username must be 1 to 50 characters." });
        }

        if (!AllowedTypes.TryGetValue(request.MimeType ?? "", out var extension))
        {
            return BadRequest(new { error = "Invalid file type." });
        }

        if (request.Size is <= 0 or > MaxFileSize)
        {
            return StatusCode(StatusCodes.Status413PayloadTooLarge, new { error = "File must be 10 MB or smaller." });
        }

        var id = Guid.NewGuid();
        var intent = new UploadIntent
        {
            Id = id,
            Username = username,
            FileKey = $"{id}{extension}",
            Status = UploadStatus.Pending,
            CreatedAt = DateTimeOffset.UtcNow
        };

        // Record the intent first, so every file in Storage can be traced to a row.
        db.UploadIntents.Add(intent);
        await db.SaveChangesAsync(ct);

        var uploadUrl = await storage.CreateSignedUploadUrlAsync(intent.FileKey, ct);
        return Ok(new CreateIntentResponse(id, uploadUrl));
    }

    [HttpPost("{id:guid}/finalize")]
    public async Task<IActionResult> FinalizeUpload(Guid id, CancellationToken ct)
    {
        await using var transaction = await db.Database.BeginTransactionAsync(ct);

        // The row lock makes concurrent finalize calls wait for each other.
        var rows = await db.UploadIntents
            .FromSql($"""SELECT * FROM "UploadIntents" WHERE "Id" = {id} FOR UPDATE""")
            .ToListAsync(ct);
        var intent = rows.SingleOrDefault();

        if (intent is null)
        {
            return NotFound(new { error = "Upload intent not found." });
        }

        // A retry returns the user that was already created.
        if (intent.Status == UploadStatus.Completed)
        {
            var existing = await db.Users.SingleAsync(u => u.UploadIntentId == id, ct);
            return Ok(ToResponse(existing));
        }

        if (!await storage.ObjectExistsAsync(intent.FileKey, ct))
        {
            return BadRequest(new { error = "Upload not found in storage." });
        }

        var user = new AppUser
        {
            Id = Guid.NewGuid(),
            Username = intent.Username,
            AvatarUrl = storage.GetPublicUrl(intent.FileKey),
            UploadIntentId = id
        };

        db.Users.Add(user);
        intent.Status = UploadStatus.Completed;
        await db.SaveChangesAsync(ct);
        await transaction.CommitAsync(ct);

        return Ok(ToResponse(user));
    }

    private static object ToResponse(AppUser user) =>
        new { user.Id, user.Username, user.AvatarUrl };
}
```

## Step 8: Add the cleanup job

Create `UploadCleanupService.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

namespace AvatarUploads;

public sealed class UploadCleanupService(
    IServiceScopeFactory scopeFactory,
    ILogger<UploadCleanupService> logger) : BackgroundService
{
    private static readonly TimeSpan Interval = TimeSpan.FromMinutes(15);

    // Supabase signed upload URLs last 2 hours. Waiting longer means a slow
    // upload can't land after its intent has already been removed.
    private static readonly TimeSpan MaxAge = TimeSpan.FromHours(3);

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var timer = new PeriodicTimer(Interval);
        do
        {
            try
            {
                await CleanupAsync(stoppingToken);
            }
            catch (Exception ex) when (ex is not OperationCanceledException)
            {
                logger.LogError(ex, "Upload cleanup failed.");
            }
        }
        while (await timer.WaitForNextTickAsync(stoppingToken));
    }

    private async Task CleanupAsync(CancellationToken ct)
    {
        using var scope = scopeFactory.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        var storage = scope.ServiceProvider.GetRequiredService<SupabaseStorageClient>();

        var cutoff = DateTimeOffset.UtcNow - MaxAge;
        var expiredIds = await db.UploadIntents
            .Where(x => x.Status == UploadStatus.Pending && x.CreatedAt < cutoff)
            .Select(x => x.Id)
            .Take(100)
            .ToListAsync(ct);

        foreach (var id in expiredIds)
        {
            await using var transaction = await db.Database.BeginTransactionAsync(ct);

            // SKIP LOCKED leaves alone any intent that is being finalized right now.
            var rows = await db.UploadIntents
                .FromSql($"""SELECT * FROM "UploadIntents" WHERE "Id" = {id} AND "Status" = 'Pending' FOR UPDATE SKIP LOCKED""")
                .ToListAsync(ct);
            var intent = rows.SingleOrDefault();

            if (intent is null)
            {
                continue;
            }

            await storage.DeleteObjectAsync(intent.FileKey, ct);
            db.UploadIntents.Remove(intent);
            await db.SaveChangesAsync(ct);
            await transaction.CommitAsync(ct);
        }
    }
}
```

## Step 9: Wire everything together

Replace the contents of `Program.cs`:

```csharp
using System.Net.Http.Headers;
using AvatarUploads;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Options;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.Configure<SupabaseOptions>(builder.Configuration.GetSection("Supabase"));

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddHttpClient<SupabaseStorageClient>((services, http) =>
{
    var supabase = services.GetRequiredService<IOptions<SupabaseOptions>>().Value;
    http.BaseAddress = new Uri($"{supabase.Url.TrimEnd('/')}/storage/v1/");
    http.DefaultRequestHeaders.Add("apikey", supabase.ServiceRoleKey);
    http.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Bearer", supabase.ServiceRoleKey);
});

builder.Services.AddHostedService<UploadCleanupService>();

var app = builder.Build();

app.UseDefaultFiles();
app.UseStaticFiles();
app.MapControllers();

app.Run();
```

## Step 10: Create the database tables

1. Create and apply the migration:

   ```powershell
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

2. In the Supabase **SQL Editor**, turn on Row Level Security. Skipping this leaves these tables readable and writable through Supabase's public Data API.

   ```sql
   alter table public."UploadIntents" enable row level security;
   alter table public."Users" enable row level security;
   alter table public."__EFMigrationsHistory" enable row level security;
   ```

   Your app connects as the table owner, so it keeps working without any policies.

## Step 11: Add the upload page

Create `wwwroot/index.html`:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Avatar Upload</title>
</head>
<body>
  <h1>Create user</h1>
  <form id="form">
    <p><label>Username <input id="username" required maxlength="50"></label></p>
    <p><input id="file" type="file" accept="image/jpeg,image/png,image/webp,image/gif" required></p>
    <button>Upload</button>
  </form>
  <p id="status"></p>
  <img id="avatar" alt="" width="160" hidden>

  <script>
    const status = document.getElementById("status");
    const avatar = document.getElementById("avatar");

    async function postJson(url, body) {
      const response = await fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body ?? {})
      });
      const data = await response.json();
      if (!response.ok) throw new Error(data.error ?? "Request failed");
      return data;
    }

    document.getElementById("form").addEventListener("submit", async event => {
      event.preventDefault();
      const file = document.getElementById("file").files[0];
      const username = document.getElementById("username").value;

      try {
        status.textContent = "Preparing upload...";
        const intent = await postJson("/api/uploads/intents", {
          username,
          mimeType: file.type,
          size: file.size
        });

        status.textContent = "Uploading...";
        const upload = await fetch(intent.uploadUrl, {
          method: "PUT",
          headers: { "Content-Type": file.type },
          body: file
        });
        if (!upload.ok) throw new Error("Upload was rejected by storage.");

        status.textContent = "Finalizing...";
        const user = await postJson(`/api/uploads/${intent.intentId}/finalize`);

        status.textContent = `Created user ${user.username}.`;
        avatar.src = user.avatarUrl;
        avatar.hidden = false;
      } catch (error) {
        status.textContent = error.message;
      }
    });
  </script>
</body>
</html>
```

## Step 12: Run and test

Start the app, and note the `http://localhost:PORT` address it prints:

```powershell
dotnet run
```

In a second PowerShell window, set that address:

```powershell
$base = "http://localhost:PORT"
```

| Test | How | Expected result |
| ---- | --- | --------------- |
| Normal upload | Open the address in a browser and upload an image | The image appears, with a new row in `Users` and the intent `Completed` |
| Wrong file type | Run test A below | HTTP 400, `Invalid file type.` |
| Bypass the size check | Run test B below | Storage rejects the file, even though the API accepted the claimed size |
| Finalize twice | Run test C below | Both calls return the same user `Id` |
| Finalize with no file | Create an intent, then finalize without uploading | HTTP 400, `Upload not found in storage.` |
| Abandoned upload | Create an intent and upload, but never finalize | Row and file are removed after `MaxAge`. Set `MaxAge` and `Interval` to 1 minute temporarily to test quickly |

**Test A: wrong file type**

```powershell
Invoke-RestMethod -Method Post -Uri "$base/api/uploads/intents" -ContentType "application/json" -Body '{"username":"john","mimeType":"application/pdf","size":1000}'
```

**Test B: lie about the size, then send 12 MB**

```powershell
$intent = Invoke-RestMethod -Method Post -Uri "$base/api/uploads/intents" -ContentType "application/json" -Body '{"username":"john","mimeType":"image/png","size":1000}'
$bytes = New-Object byte[] (12MB)
Invoke-WebRequest -Method Put -Uri $intent.uploadUrl -ContentType "image/png" -Body $bytes
```

**Test C: finalize twice**

Upload an image with the page first, then copy its intent `Id` from the `UploadIntents` table:

```powershell
$id = "PASTE_INTENT_ID"
Invoke-RestMethod -Method Post -Uri "$base/api/uploads/$id/finalize"
Invoke-RestMethod -Method Post -Uri "$base/api/uploads/$id/finalize"
```

## Step 13: Before going to production

1. **Add authentication.** These endpoints are open, so anyone can create users and use your storage. Require a login with ASP.NET Core authentication, for example by validating Supabase Auth JWTs, and store the user ID on each intent.
2. **Add rate limiting** to `/api/uploads/intents` with the built-in `AddRateLimiter` middleware, since each call creates a row and a signed URL.
3. **Check the file contents if you need to.** The bucket checks the declared MIME type, not the real bytes. For untrusted users, add image inspection or a scanning step before marking an intent `Completed`.
4. **Run one cleanup job.** Multiple app instances are safe because of `SKIP LOCKED`, but they duplicate the polling work.
