# What Companies Expect from a 10-Month / 1-Year Developer

At this level you're usually hired as a **junior developer**. Companies aren't expecting an expert. They want someone who can **finish small-to-medium tasks on their own** and doesn't need hand-holding on the basics.

## Technical skills

**Language and framework (C# / .NET here)**
- Comfortable with C# basics: classes, interfaces, generics, LINQ, async/await, exceptions
- Can build a **REST API in ASP.NET Core**: controllers, routing, model binding, validation, status codes
- Understands **dependency injection**: AddScoped, AddSingleton, AddTransient and when to use each
- Reads config from `appsettings.json`, uses logging

**Database**
- **SQL basics:** SELECT, JOIN, GROUP BY, indexes, and what a primary key or foreign key is
- **EF Core:** DbContext, migrations, relationships, `Include`, and avoiding N+1 queries
- Can design a simple schema, for example users, orders and products

**Auth and security basics**
- JWT authentication, `[Authorize]`, roles
- Knows about password hashing, SQL injection, CORS, and keeping secrets out of the code

**Tools**
- **Git:** branch, commit, pull, merge, resolve conflicts, open a PR
- **Debugging:** breakpoints, reading stack traces, reading logs
- Postman or Swagger for testing APIs
- Basic **unit tests** (xUnit or NUnit) are a strong plus

**Nice to have (stands out at this level)**
- Docker basics, deploying to Azure or IIS
- Clean architecture or the repository pattern, DTOs, AutoMapper
- Some frontend skill (React or Angular) if the role is full-stack

## Work habits (often matter more than tech)

- **Take a ticket and finish it** without being told every step
- **Ask good questions** after trying first: "I tried X and Y, and Z happened"
- **Estimate roughly** and tell people early when you're stuck or will be late
- **Read existing code** and follow the project's style instead of rewriting it
- Take PR review feedback without getting defensive
- Write clear commit messages and PR descriptions

## What they don't expect yet

- Designing system architecture or microservices
- Deep performance tuning
- Leading other developers
- Knowing every library

## Interview focus at this level

1. C# / OOP questions: interface vs abstract class, value vs reference types, `async`/`await`
2. Building or explaining a CRUD API
3. SQL queries, often with JOINs
4. A small coding problem (arrays, strings, dictionaries)
5. **"Explain a project you built"**: be ready to walk through one real project end to end

**Tip:** one solid project helps you more than many tutorials. For example, a .NET API with auth, EF Core, validation, tests, and a deployment.
