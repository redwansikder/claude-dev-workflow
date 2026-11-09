# Types of .NET App

## How they stack

```
                    C# language + .NET runtime + built-in library
                                       │
                               Console App   ← the base: Program.cs, runs, ends
                                       │
          ┌────────────────────────────┼───────────────────────────┐
          │                            │                           │
   + a web server               + runs forever             + a window
   (ASP.NET Core)               in the background          (desktop UI)
          │                            │                           │
   ┌──────┼──────────┬──────────┐  Worker Service       WPF · WinForms · MAUI
   │      │          │          │
Minimal  Web API    MVC /     Blazor
 API    (Controllers) Razor Pages  (UI in C#)
(JSON)   (JSON)     (HTML pages)

Side pieces used by all of them:
   Class Library  → shared code, can't run alone (like an npm package you write)
   Test Project   → xUnit / NUnit, tests your other projects
```

## Apps that run on a server (users reach them over the internet)

**1. Web API** (Minimal API or Controllers)
- **What it is:** a backend with no screen. It sends and receives JSON.
- **Real use:** the backend behind a mobile app or a React website. A food delivery app sends "show me restaurants near me", and the Web API answers with a list.
- **Like:** Minimal API = **Express**, Controllers = **NestJS**

**2. MVC / Razor Pages**
- **What it is:** a website where the server builds the HTML pages.
- **Real use:** a company admin panel, a university result portal, a hospital appointment site.
- **Like:** **Next.js ≤12** (`pages/` + `getServerSideProps`)

**3. Blazor**
- **What it is:** an interactive website UI written in C# instead of JavaScript.
- **Real use:** internal dashboards and business tools, built by teams that only want to write C#.
- **Like:** **Next.js 13+** (server-rendered, with interactive parts like `"use client"`)

**4. Worker Service**
- **What it is:** a program that runs in the background forever, with no user talking to it.
- **Real use:** send SMS reminders every morning, process uploaded images, sync data every hour.
- **Like:** **NestJS** `@Cron()` jobs

## Apps that run on the user's device

**5. Desktop** (WPF, WinForms)
- **What it is:** a Windows program with windows and buttons.
- **Real use:** billing software on a shop counter, accounting software, hospital reception software.
- **Like:** **Electron**

**6. Mobile + desktop** (.NET MAUI)
- **What it is:** one C# codebase that becomes Android, iOS, Windows and Mac apps.
- **Real use:** a field-staff app for a company, a simple business app on phones.
- **Like:** **React Native**

**7. Games** (Unity)
- **What it is:** Unity uses C# for game code.
- **Real use:** mobile and PC games.
- **Like:** nothing in the JS world comes close

## Supporting types (never used alone)

**8. Console App:** runs, does a job, ends. Used for learning, scripts and small tools. **Like:** `node script.js`

**9. Class Library:** shared code that other projects use. **Like:** an npm package you write

**10. Test Project:** checks that your other code works. **Like:** **Jest**

## Where the jobs are

```
Web API + MVC/Blazor  ███████████████  most jobs (web/backend)
Desktop               ████████         many jobs, mostly maintaining existing company software
Unity games           ████             separate game industry
Worker Service        (part of backend jobs)
MAUI                  (small)
```

These bars aren't measured. They roughly follow the JetBrains 2025 survey: 68% of .NET developers work on web, 41% on desktop, 19% on Unity.
