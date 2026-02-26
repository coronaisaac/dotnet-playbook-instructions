# 📦 Template: azure-function-hexagonal

Azure Functions v4 (Isolated Worker) template for .NET 10 following **Clean Architecture**, **Hexagonal (Ports & Adapters)**, **SOLID**, and **C# 12+** modern patterns.

---

## When to use this template

- You need an **Azure Function** (HTTP Trigger or other triggers) with a clean, testable structure.
- You want a `IFunctionsWorkerMiddleware` exception handler with RFC-7807 ProblemDetails.
- You need EF Core persistence with a swappable DB provider (PostgreSQL, SQL Server, etc.).
- You want the AI to scaffold the full project from a single conversation.

---

## Solution structure generated

```
{{SolutionName}}.sln
│
├── Shared/
│   ├── {{DomainPrefix}}.Shared.ExceptionHandler    ← IFunctionsWorkerMiddleware + RFC-7807 ProblemDetails
│   └── {{DomainPrefix}}.Shared.MessagesLocalizer   ← Dictionary-based localized error messages
│
├── Backend/
│   ├── {{DomainPrefix}}.Core            ← Domain: entities, interfaces (ports), exceptions, enums, DTOs
│   ├── {{DomainPrefix}}.Infrastructure  ← EF Core DbContext + CommandRepository base + adapters
│   └── {{DomainPrefix}}.Services        ← Application use-cases (service implementations)
│
└── Consumers/
    └── {{DomainPrefix}}.Function        ← Azure Functions entry point (Program.cs + HTTP Triggers)
```

---

## Dependency graph

```
Function  →  Services  →  Core
Function  →  Infrastructure  →  Core
Function  →  Shared.ExceptionHandler  →  Core
Function  →  Shared.MessagesLocalizer  →  Core

Core  →  (no project references — pure domain)
```

---

## Configurable tokens

| Token | Description | Example |
|---|---|---|
| `{{SolutionName}}` | Solution file name | `Fn_Historial_Backend` |
| `{{DomainName}}` | Domain/context name | `Historial` |
| `{{DomainPrefix}}` | Full namespace prefix | `Whatsapp.Historial` |
| `{{DatabaseProvider}}` | EF Core DB provider package | `Npgsql.EntityFrameworkCore.PostgreSQL` |
| `{{DatabaseProviderVersion}}` | DB provider package version | `9.0.4` |
| `{{DbContextName}}` | EF Core DbContext class name | `HistorialContext` |
| `{{ConnectionStringKey}}` | Environment variable key for connection string | `ConnectionStrings__DefaultDb` |

---

## Files in this template

| File | Purpose |
|---|---|
| `instructions.md` | Full architecture rules + code blueprints for the AI |
| `prompt.md` | Starter prompt — paste this into your AI assistant |
| `README.md` | This file |

---

## How to use

1. Copy `prompt.md` content and paste it into **Copilot Chat**, **Cursor**, **Claude**, or **ChatGPT**.
2. When prompted, reference or attach `instructions.md` as context.
3. Answer the 6 configuration questions the AI will ask.
4. The AI generates the full solution with:
   - All `.csproj` + solution file with correct folder nesting
   - `GlobalUsings.cs` per project
   - `ICommandRepository<T>` generic interfaces + `CommandRepository<T>` base
   - `DependencyContainer.cs` per layer (extension methods)
   - `CommandRepository<T>` with `BeginTransaction`, `FindManyAsync`, `ToPagedListAsync`
   - `IFunctionsWorkerMiddleware` exception handler with RFC-7807
   - `Program.cs` with full Functions pipeline
   - `host.json` + `local.settings.json`
   - Sample entity, DTO, exception, repository, service, and HTTP Trigger function

---

## Requirements

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- [Azure Functions Core Tools v4](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local)
- Chosen database (PostgreSQL or SQL Server)
- Any AI assistant: GitHub Copilot, Cursor AI, Claude, ChatGPT

---

## Tech stack

| Layer | Technology |
|---|---|
| Trigger/Host | Azure Functions v4 (Isolated Worker) |
| ORM | Entity Framework Core 10.x |
| Database | PostgreSQL (default) / SQL Server (configurable) |
| Serialization | System.Text.Json (native — no Newtonsoft.Json) |
| DI | Microsoft.Extensions.DependencyInjection |
| Observability | Application Insights (`WorkerService`) |
| Error handling | Custom `IFunctionsWorkerMiddleware` → RFC-7807 ProblemDetails |
| C# version | C# 12+ (primary constructors, `required`, `init`, `sealed`, `record`) |
