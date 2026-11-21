# Common C# `using` Directives

## What each one gives you

```csharp
// ── Core ──
using System;                              // Basic types: Console, Math, DateTime, Guid, Exception
using System.Collections.Generic;          // List<T>, Dictionary<K,V>, HashSet<T>, Queue<T>
using System.Linq;                         // Query collections: Where, Select, OrderBy, FirstOrDefault
using System.Threading.Tasks;              // Task and async/await (like Promise in JS)
using System.Text;                         // StringBuilder, Encoding (UTF8 etc.)
using System.IO;                           // File, Directory, Path, Stream (like fs/path in Node)
using System.Net.Http;                     // HttpClient for calling APIs (like fetch/axios)
using System.Text.Json;                    // JsonSerializer: object <-> JSON

// ── ASP.NET Core (Web API) ──
using Microsoft.AspNetCore.Mvc;            // ControllerBase, [ApiController], [HttpGet], Ok(), NotFound()
using Microsoft.AspNetCore.Authorization;  // [Authorize], [AllowAnonymous] to protect endpoints
using Microsoft.AspNetCore.Http;           // HttpContext, IFormFile, StatusCodes
using Microsoft.Extensions.DependencyInjection; // AddScoped / AddSingleton / AddTransient to register services
using Microsoft.Extensions.Logging;        // ILogger<T> for logging
using Microsoft.Extensions.Configuration;  // IConfiguration to read appsettings.json and env vars

// ── Database (EF Core) ──
using Microsoft.EntityFrameworkCore;       // DbContext, DbSet<T>, Include(), ToListAsync()
using System.ComponentModel.DataAnnotations; // [Required], [MaxLength], [Key], [EmailAddress] for validation

// ── Your own code ──
using MyApp.Models;                        // Your entity classes / DTOs
using MyApp.Services;                      // Your business logic classes
```

You'll see these most often:
- **Controllers:** `Microsoft.AspNetCore.Mvc`
- **Database code:** `Microsoft.EntityFrameworkCore`
- **Everyday code:** `System.Linq` and `System.Threading.Tasks`

In .NET 6 and later, the core ones are usually already included through implicit usings, so you won't see them written at the top of the file.

## Most used methods from each

```csharp
// ── System ──
Console.WriteLine("hi");          // print a line
Console.ReadLine();               // read user input
Math.Max(a, b); Math.Round(x, 2); // math helpers
DateTime.Now; DateTime.UtcNow;    // current time
date.AddDays(7); date.ToString("yyyy-MM-dd");
Guid.NewGuid();                   // unique id
int.Parse("5"); int.TryParse(s, out var n);   // string -> number
str.Split(','); str.Trim(); str.Contains("x"); str.Replace("a","b");
string.IsNullOrEmpty(s);          // null/empty check
$"Hello {name}";                  // string interpolation (like JS template literals)

// ── System.Collections.Generic ──
var list = new List<int>();
list.Add(1); list.Remove(1); list.Contains(1); list.Count;
var dict = new Dictionary<string, int>();
dict["a"] = 1; dict.ContainsKey("a"); dict.TryGetValue("a", out var v);
var set = new HashSet<string>(); set.Add("x");

// ── System.Linq ──  (JS equivalent in comments)
list.Where(x => x > 5)            // filter
list.Select(x => x * 2)           // map
list.FirstOrDefault(x => x.Id==1) // find (null/default if none)
list.Any(x => x > 5)              // some
list.All(x => x > 0)              // every
list.OrderBy(x => x.Name)         // sort ascending  (OrderByDescending)
list.GroupBy(x => x.Category)     // group by
list.Sum(x => x.Price); list.Count(); list.Max(); list.Average();
list.Skip(10).Take(5)             // pagination (slice)
list.Distinct(); list.ToList(); list.ToArray();

// ── System.Threading.Tasks ──
async Task<User> GetUser() { ... } // async method (returns Promise-like Task)
await SomethingAsync();
await Task.WhenAll(t1, t2);        // Promise.all
await Task.Delay(1000);            // setTimeout / sleep
Task.Run(() => HeavyWork());       // run on background thread

// ── System.Text ──
var sb = new StringBuilder();
sb.Append("a"); sb.AppendLine("b"); sb.ToString(); // fast string building
Encoding.UTF8.GetBytes(str); Encoding.UTF8.GetString(bytes);

// ── System.IO ──
File.ReadAllText(path); File.WriteAllText(path, text);
File.Exists(path); File.Delete(path);
await File.ReadAllTextAsync(path);
Directory.CreateDirectory(dir); Directory.GetFiles(dir);
Path.Combine("a", "b.txt"); Path.GetExtension(path);

// ── System.Net.Http ──
var res = await http.GetAsync(url);
var json = await http.GetStringAsync(url);
await http.PostAsJsonAsync(url, obj);   // needs System.Net.Http.Json
res.EnsureSuccessStatusCode();
await res.Content.ReadAsStringAsync();

// ── System.Text.Json ──
JsonSerializer.Serialize(obj);               // JSON.stringify
JsonSerializer.Deserialize<User>(json);      // JSON.parse (typed)

// ── Microsoft.AspNetCore.Mvc ──
[ApiController] [Route("api/[controller]")]  // controller setup
[HttpGet] [HttpPost] [HttpPut("{id}")] [HttpDelete("{id}")]
[FromBody] [FromQuery] [FromRoute]           // where params come from
return Ok(data);          // 200
return Created(uri, obj); // 201   (CreatedAtAction also common)
return NoContent();       // 204
return BadRequest(msg);   // 400
return Unauthorized();    // 401
return NotFound();        // 404

// ── Microsoft.AspNetCore.Authorization ──
[Authorize]                       // login required
[Authorize(Roles = "Admin")]      // role required
[AllowAnonymous]                  // public endpoint

// ── Microsoft.AspNetCore.Http ──
HttpContext.User; HttpContext.Request.Headers["Authorization"];
IFormFile file; file.CopyToAsync(stream);   // file upload
StatusCodes.Status500InternalServerError;

// ── Microsoft.Extensions.DependencyInjection ──  (in Program.cs)
builder.Services.AddScoped<IUserService, UserService>();    // one per request
builder.Services.AddSingleton<ICache, Cache>();             // one for whole app
builder.Services.AddTransient<IEmail, Email>();             // new every time
builder.Services.AddControllers();
builder.Services.AddDbContext<AppDbContext>(o => o.UseSqlServer(conn));

// ── Microsoft.Extensions.Logging ──
_logger.LogInformation("User {Id} logged in", id);
_logger.LogWarning("..."); _logger.LogError(ex, "Failed");

// ── Microsoft.Extensions.Configuration ──
config["Jwt:Key"];                                 // read appsettings value
config.GetConnectionString("Default");
config.GetValue<int>("Settings:PageSize");

// ── Microsoft.EntityFrameworkCore ──
await db.Users.ToListAsync();                      // SELECT *
await db.Users.FindAsync(id);                      // by primary key
await db.Users.FirstOrDefaultAsync(u => u.Email == e);
await db.Users.Where(u => u.Active).ToListAsync();
db.Users.Include(u => u.Orders)                    // JOIN related data
db.Users.Add(user); db.Users.Update(user); db.Users.Remove(user);
await db.SaveChangesAsync();                       // commit changes
await db.Users.AnyAsync(u => u.Email == e);
db.Users.AsNoTracking()                            // faster read-only query

// ── System.ComponentModel.DataAnnotations ──
[Key] public int Id { get; set; }
[Required] [MaxLength(100)] public string Name { get; set; }
[EmailAddress] public string Email { get; set; }
[Range(1, 100)] public int Age { get; set; }
```

If you learn these first, you'll cover most daily work:
1. **LINQ:** `Where`, `Select`, `FirstOrDefault`, `Any`, `OrderBy`, `ToList`
2. **EF Core:** `ToListAsync`, `FirstOrDefaultAsync`, `Add`, `Include`, `SaveChangesAsync`
3. **Controller returns:** `Ok`, `NotFound`, `BadRequest`, `CreatedAtAction`
