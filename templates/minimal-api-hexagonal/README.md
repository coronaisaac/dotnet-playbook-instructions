# 📦 Template: minimal-api-hexagonal

REST API template for .NET 8/10 following **Clean Architecture**, **Hexagonal (Ports & Adapters)**, **SOLID**, and **Minimal API**.

---

## When to use this template

- You need a new **REST API** backed by SQL Server.
- You want a consistent, multi-layer structure that scales with the team.
- You want the AI to scaffold the full project from a single conversation.

---

## Solution structure generated

```
{{ProjectName}}.sln
│
├── Shared/
│   ├── {{SharedPrefix}}.Shared.Abstractions        ← Base interfaces, attributes, exception keys
│   ├── {{SharedPrefix}}.Shared.ExceptionHandler    ← RFC-7807 global error middleware
│   └── {{SharedPrefix}}.Shared.MessagesLocalizer   ← Dictionary-based error messages
│
├── Backend/
│   ├── {{ProjectName}}.Abstractions    ← Entities, DTOs, Models, Ports (interfaces)
│   ├── {{ProjectName}}.Core            ← Use-cases (services) + Minimal API endpoints
│   ├── {{ProjectName}}.Infrastructure  ← EF Core DbContext + Repository adapters
│   └── {{ProjectName}}.Validators      ← Business rule validators
│
└── Consumers/
    └── Web.Api                         ← Program.cs only — composition root
```

---

## Configurable tokens

| Token | Description | Example |
|---|---|---|
| `{{ProjectName}}` | Domain project prefix | `Whatsapp.Historial` |
| `{{SharedPrefix}}` | Shared libraries prefix | `MyCompany` |
| `{{TargetFramework}}` | .NET target | `net8.0` / `net10.0` |
| `{{DbContextName}}` | EF Core DbContext name | `HistorialDataContext` |
| `{{ConnectionStringKey}}` | appsettings section key | `ConnectionStringHistorialKey` |

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
3. Answer the 5 configuration questions the AI will ask.
4. The AI generates the full solution with:
   - All `.csproj` + solution file with correct folder nesting
   - `GlobalUsings.cs` per project
   - Generic repository interfaces (`ICommandRepository<T>`)
   - `DependencyContainer.cs` per layer (extension methods)
   - Partial class services (constructor + CRUD operations split)
   - Minimal API endpoints with `WithTags()` for Swagger
   - `Program.cs` with full middleware pipeline
   - Sample entity, DTO, exception, validator, repository, service, and endpoint

---

## Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) or higher
- SQL Server (or SQL Server LocalDB for development)
- Any AI assistant: GitHub Copilot, Cursor AI, Claude, ChatGPT

---

## Tech stack

| Layer | Technology |
|---|---|
| Web framework | ASP.NET Core Minimal API |
| ORM | Entity Framework Core 8+ |
| Database | SQL Server |
| Serialization | System.Text.Json (primary) / Newtonsoft.Json (compat) |
| DI | Microsoft.Extensions.DependencyInjection |
| API docs | Swashbuckle (Swagger) |
| Error handling | Custom RFC-7807 ProblemDetails middleware |
