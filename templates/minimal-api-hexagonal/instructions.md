# .NET Clean Architecture + Hexagonal API — Project Instructions

> **How to use this file**
> This is a GitHub Copilot / VS Code Copilot Chat instruction file (`.github/copilot-instructions.md` or `instructions.md` at repo root).
> Replace every occurrence of `{{ProjectName}}` and `{{SharedPrefix}}` with the values for your new project.
> The AI must use these rules as the single source of truth for every file it generates.

---

## 0. Project variable

| Token | Description | Example |
|---|---|---|
| `{{ProjectName}}` | Root namespace and project prefix | `Whatsapp.Historial` |
| `{{SharedPrefix}}` | Prefix for cross-cutting shared libraries | `MyCompany` → `MyCompany.Shared.*` |
| `{{TargetFramework}}` | .NET target moniker | `net8.0` (or `net10.0` if toolchain supports it) |
| `{{DbContextName}}` | Name of the EF Core DbContext | `HistorialDataContext` |
| `{{ConnectionStringKey}}` | appsettings section key for primary DB | `ConnectionStringHistorialKey` |

---

## 1. Solution layout

Create one Visual Studio solution file: `{{ProjectName}}.sln`

The solution must contain **three solution folders**:

```
Backend/           ← all domain-specific projects
Shared/            ← shared cross-cutting-concern libraries
Consumers/         ← entry points (Minimal API, Workers, etc.)
```

### 1.1 Project list

#### Shared (solution folder `Shared/`)
These libraries are domain-agnostic and must **never** reference any Backend project.

| Project | SDK | Purpose |
|---|---|---|
| `{{SharedPrefix}}.Shared.Abstractions` | `Microsoft.NET.Sdk` | Base interfaces, custom attributes, exception keys |
| `{{SharedPrefix}}.Shared.ExceptionHandler` | `Microsoft.NET.Sdk` + `FrameworkReference Microsoft.AspNetCore.App` | Global middleware exception handler |
| `{{SharedPrefix}}.Shared.MessagesLocalizer` | `Microsoft.NET.Sdk` | Localized messages dictionary for exception codes |

#### Backend (solution folder `Backend/`)

| Project | SDK | Purpose |
|---|---|---|
| `{{ProjectName}}.Abstractions` | `Microsoft.NET.Sdk` | Domain entities, DTOs, models, interface contracts |
| `{{ProjectName}}.Core` | `Microsoft.NET.Sdk` + `FrameworkReference Microsoft.AspNetCore.App` | Application services (use-cases), Minimal API endpoints |
| `{{ProjectName}}.Infrastructure` | `Microsoft.NET.Sdk` | EF Core DbContext, repository implementations (queries) |
| `{{ProjectName}}.Validators` | `Microsoft.NET.Sdk` | Business rule validators injected into services |

#### Consumers (solution folder `Consumers/`)

| Project | SDK | Purpose |
|---|---|---|
| `Web.Api` | `Microsoft.NET.Sdk.Web` | ASP.NET Core entry point (Program.cs only) |

---

## 2. Dependency graph

Follow this strict dependency direction (no circular references):

```
Web.Api
  └── {{ProjectName}}.Core
  └── {{ProjectName}}.Infrastructure
  └── {{ProjectName}}.Validators
  └── {{ProjectName}}.Abstractions
  └── {{SharedPrefix}}.Shared.*

{{ProjectName}}.Core
  └── {{ProjectName}}.Abstractions
  └── {{SharedPrefix}}.Shared.Abstractions

{{ProjectName}}.Infrastructure
  └── {{ProjectName}}.Abstractions

{{ProjectName}}.Validators
  └── {{ProjectName}}.Abstractions
  └── {{SharedPrefix}}.Shared.Abstractions

{{ProjectName}}.Abstractions
  (no project references — pure domain)

{{SharedPrefix}}.Shared.ExceptionHandler
  └── {{SharedPrefix}}.Shared.Abstractions

{{SharedPrefix}}.Shared.MessagesLocalizer
  └── {{SharedPrefix}}.Shared.Abstractions
```

---

## 3. Target framework & common csproj settings

Every `.csproj` must target **`{{TargetFramework}}`** (default `net8.0`; use `net10.0` only if the installed SDK supports it without breaking changes).

```xml
<PropertyGroup>
  <TargetFramework>{{TargetFramework}}</TargetFramework>
  <ImplicitUsings>enable</ImplicitUsings>
  <Nullable>enable</Nullable>  <!-- enable per project; Abstractions may disable to support EF scaffold -->
</PropertyGroup>
```

Use `global using` statements in `GlobalUsings.cs` files inside each project instead of per-file `using` statements wherever possible.

---

## 4. NuGet package baseline

| Package | Version | Used in |
|---|---|---|
| `Microsoft.EntityFrameworkCore.SqlServer` | `8.0.x` (or latest stable for target framework) | Abstractions, Infrastructure |
| `Microsoft.Extensions.DependencyInjection.Abstractions` | `8.0.x` | Shared.Abstractions, Shared.MessagesLocalizer |
| `Swashbuckle.AspNetCore` | `6.x` | Web.Api |
| `Newtonsoft.Json` | `13.x` | Abstractions, Core (if needed) |

> Do **not** use any package that requires .NET < 7. Prefer `System.Text.Json` over `Newtonsoft.Json` for new code; use `Newtonsoft.Json` only when interoperability with existing serialised data requires it.

---

## 5. Shared library patterns

### 5.1 `{{SharedPrefix}}.Shared.Abstractions`

```
{{SharedPrefix}}.Shared.Abstractions/
  Attributes/
    HttpStatusCodeAttribute.cs
  Constants/
    MessagesKeys.cs
  Interfaces/
    IMessagesLocalizer.cs
  GlobalUsings.cs
```

**`HttpStatusCodeAttribute.cs`** — marks domain exceptions with an HTTP status code:
```csharp
namespace {{SharedPrefix}}.Shared.Abstractions.Attributes;

[AttributeUsage(AttributeTargets.Class, Inherited = false, AllowMultiple = false)]
public sealed class HttpStatusCodeAttribute : Attribute
{
    public int StatusCode { get; }
    public HttpStatusCodeAttribute(int statusCode) => StatusCode = statusCode;
}
```

**`IMessagesLocalizer.cs`** — indexer-based localiser interface:
```csharp
namespace {{SharedPrefix}}.Shared.Abstractions.Interfaces;

public interface IMessagesLocalizer
{
    string this[string key] { get; }
}
```

**`MessagesKeys.cs`** — one `public const string` per exception class, using `nameof`:
```csharp
namespace {{SharedPrefix}}.Shared.Abstractions.Constants;

public static class MessagesKeys
{
    public const string ExampleException = nameof(ExampleException);
    // Add one entry per domain exception
}
```

**`GlobalUsings.cs`**:
```csharp
global using {{SharedPrefix}}.Shared.Abstractions.Attributes;
```

---

### 5.2 Domain exceptions

Every domain exception lives in `{{SharedPrefix}}.Shared.Abstractions/Exceptions/` inside a folder that matches its domain group:

```csharp
namespace {{SharedPrefix}}.Shared.Abstractions.Exceptions.<DomainGroup>;

[HttpStatusCode(400)]   // or 404, 409, 500 as appropriate
public sealed class MyDomainException : Exception
{
    public MyDomainException() { }
    public MyDomainException(string? message) : base(message) { }
    public MyDomainException(string? message, Exception? innerException)
        : base(message, innerException) { }
}
```

Rules:
- Use `[HttpStatusCode(4xx)]` for business/validation errors.
- Use `[HttpStatusCode(500)]` for infrastructure errors.
- Name the class exactly as the key in `MessagesKeys`.

---

### 5.3 `{{SharedPrefix}}.Shared.MessagesLocalizer`

```
{{SharedPrefix}}.Shared.MessagesLocalizer/
  MessagesLocalizer.cs
  DependencyContainer.cs
  GlobalUsings.cs
```

**`MessagesLocalizer.cs`** — internal, registered as singleton, maps exception class names to localized strings:
```csharp
namespace {{SharedPrefix}}.Shared.MessagesLocalizer;

internal sealed class MessagesLocalizer : IMessagesLocalizer
{
    private readonly Dictionary<string, string> _messages = new()
    {
        { MessagesKeys.ExampleException, "Este es un ejemplo de error." },
        // Add one entry per domain exception key
    };

    public string this[string key] =>
        _messages.TryGetValue(key, out var value) ? value : key;
}
```

**`DependencyContainer.cs`**:
```csharp
namespace {{SharedPrefix}}.Shared.MessagesLocalizer;

public static class DependencyContainer
{
    public static IServiceCollection AddMessagesLocalizer(this IServiceCollection services)
    {
        services.AddSingleton<IMessagesLocalizer, MessagesLocalizer>();
        return services;
    }
}
```

**`GlobalUsings.cs`**:
```csharp
global using {{SharedPrefix}}.Shared.Abstractions.Interfaces;
global using {{SharedPrefix}}.Shared.Abstractions.Constants;
global using Microsoft.Extensions.DependencyInjection;
```

---

### 5.4 `{{SharedPrefix}}.Shared.ExceptionHandler`

```
{{SharedPrefix}}.Shared.ExceptionHandler/
  ExceptionHandler.cs
  DependencyContainer.cs
  GlobalUsings.cs
```

**`ExceptionHandler.cs`** (internal) — reads `HttpStatusCodeAttribute` from the thrown exception and writes an RFC-7807 `ProblemDetails` response:
```csharp
namespace {{SharedPrefix}}.Shared.ExceptionHandler;

internal static class ExceptionHandler
{
    public static async Task<bool> WriteResponseAsync(
        HttpContext context, IMessagesLocalizer localizer)
    {
        var feature = context.Features.Get<IExceptionHandlerFeature>();
        if (feature?.Error is not { } exception) return true;

        var attr = exception.GetType().GetCustomAttribute<HttpStatusCodeAttribute>();
        int statusCode = attr?.StatusCode ?? 500;
        string key = exception.GetType().Name;
        string title = localizer[key] == key ? exception.Message : localizer[key];

        var problem = new ProblemDetails
        {
            Status = statusCode,
            Type = statusCode >= 500
                ? "https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.1"
                : "https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.1",
            Title = title,
            Instance = $"problemDetails/{key}"
        };

        context.Response.ContentType = "application/problem+json";
        context.Response.StatusCode = statusCode;
        await JsonSerializer.SerializeAsync(context.Response.Body, problem);
        return false;
    }
}
```

**`DependencyContainer.cs`**:
```csharp
namespace {{SharedPrefix}}.Shared.ExceptionHandler;

public static class DependencyContainer
{
    public static IApplicationBuilder Use{{SharedPrefix}}ExceptionHandler(
        this IApplicationBuilder app)
    {
        app.UseExceptionHandler(builder =>
            builder.Run(async ctx =>
                await ExceptionHandler.WriteResponseAsync(
                    ctx,
                    app.ApplicationServices.GetRequiredService<IMessagesLocalizer>())));
        return app;
    }
}
```

**`GlobalUsings.cs`**:
```csharp
global using {{SharedPrefix}}.Shared.Abstractions.Attributes;
global using {{SharedPrefix}}.Shared.Abstractions.Interfaces;
global using Microsoft.AspNetCore.Builder;
global using Microsoft.AspNetCore.Diagnostics;
global using Microsoft.AspNetCore.Http;
global using Microsoft.AspNetCore.Mvc;
global using Microsoft.Extensions.DependencyInjection;
global using System.Reflection;
global using System.Text.Json;
```

---

## 6. Domain abstractions — `{{ProjectName}}.Abstractions`

```
{{ProjectName}}.Abstractions/
  Configuration/           ← Options classes (appsettings binding)
  Entities/
    <SchemaName>/          ← EF Core entity classes per DB schema
  Dtos/                    ← Input/output transfer objects
  Models/                  ← Aggregated read models (e.g. join projections)
  Interfaces/
    Validators/            ← Validator interface contracts
  Repositories/
    Transaccion/           ← Generic repository interfaces
    I<Entity>Repository.cs
  Services/
    I<Entity>Service.cs
  GlobalUsings.cs
```

### 6.1 Generic repository contracts (`Repositories/Transaccion/`)

These define the inner building blocks for every repository:

```csharp
// IAdd.cs
public interface IAdd<TEntity> where TEntity : class
{
    Task AddAsync(TEntity entity, CancellationToken ct = default);
}

// IEdit.cs
public interface IEdit<TEntity> where TEntity : class
{
    Task EditAsync(TEntity entity, CancellationToken ct = default);
}

// IDelete.cs
public interface IDelete<TEntity> where TEntity : class
{
    Task DeleteAsync(TEntity entity, CancellationToken ct = default);
}

// IFind.cs
public interface IFind<TEntity> where TEntity : class
{
    Task<TEntity?> FindAsync(Expression<Func<TEntity, bool>> predicate, CancellationToken ct = default);
    IQueryable<TEntity> FindMany(Expression<Func<TEntity, bool>> predicate);
}

// ITransaccion.cs
public interface ITransaccion<TEntity> where TEntity : class
{
    Task SaveChanges(CancellationToken ct = default);
}

// ICommandRepository.cs
public interface ICommandRepository<TEntity>
    : IAdd<TEntity>, IEdit<TEntity>, IDelete<TEntity>,
      IFind<TEntity>, ITransaccion<TEntity>, IDisposable
    where TEntity : class { }
```

### 6.2 Domain repository interfaces

Each entity gets a domain-specific interface that **inherits** `ICommandRepository<TEntity>` and adds query-specific signatures:

```csharp
namespace {{ProjectName}}.Abstractions.Repositories;

public interface I<Entity>Repository : ICommandRepository<<Entity>>
{
    IQueryable<<Entity>> Search(string search, bool isActive,
        int pageSize, int pageNumber,
        out int totalRecords, out int totalPages);
}
```

### 6.3 Service contracts

```csharp
namespace {{ProjectName}}.Abstractions.Services;

public interface I<Entity>Service
{
    Task Register(<Entity>Dto dto, CancellationToken ct = default);
    Task Edit(int id, <Entity>Dto dto, CancellationToken ct = default);
    Task Remove(int id, CancellationToken ct = default);
    IQueryable<<Entity>> ListAll();
    PagerResultsRecords<<Entity>> Search(string search, bool isActive,
        int? pageSize, int? pageNumber);
}
```

### 6.4 Validator contracts

```csharp
namespace {{ProjectName}}.Abstractions.Interfaces.Validators;

public interface I<Entity>Validator
{
    Task ThrowIfNotExist<Entity>ByIdAsync(int id, CancellationToken ct = default);
    Task ThrowIfDuplicated<Entity>ByNameAsync(string name, CancellationToken ct = default);
    // Add domain-specific validation operations
}
```

### 6.5 DTOs

Simple record or POCO classes with Data Annotations:
```csharp
namespace {{ProjectName}}.Abstractions.Dtos;

public sealed record <Entity>Dto(
    [Required] string Name,
    bool IsActive = true
);
```

### 6.6 Models (read-models / projections)

```csharp
namespace {{ProjectName}}.Abstractions.Models;

public sealed class PagerResultsRecords<T>
{
    public int TotalRecords { get; init; }
    public int TotalPages { get; init; }
    public IQueryable<T>? Records { get; init; }
}
```

### 6.7 Options (appsettings binding)

```csharp
namespace {{ProjectName}}.Abstractions.Configuration;

public sealed class <Name>Options
{
    public const string <Name>Key = nameof(<Name>Key);
    public string ConnectionString { get; set; } = string.Empty;
}
```

**`GlobalUsings.cs`**:
```csharp
global using System.ComponentModel.DataAnnotations;
global using System.ComponentModel.DataAnnotations.Schema;
global using System.Linq.Expressions;
global using {{ProjectName}}.Abstractions.Dtos;
global using {{ProjectName}}.Abstractions.Entities.<SchemaName>;
global using {{ProjectName}}.Abstractions.Models;
global using {{ProjectName}}.Abstractions.Repositories.Transaccion;
```

---

## 7. Infrastructure — `{{ProjectName}}.Infrastructure`

```
{{ProjectName}}.Infrastructure/
  DataContext/
    {{DbContextName}}.cs
    ContextExtensions.cs
  Interfaces/
    I{{DbContextName}}Context.cs  ← internal interface for DI
  Options/
    <Name>Options.cs
  Querys/
    <Entity>Repository/
      <Entity>ManagerQuery.cs    ← partial class
  DependencyContainer.cs
  GlobalUsings.cs
```

### 7.1 Internal DbContext interface

```csharp
namespace {{ProjectName}}.Infrastructure.Interfaces;

internal interface I{{DbContextName}}Context : I{{DbContextName}}Entities, IDisposable
{
    Task<int> SaveChangesAsync(CancellationToken ct = default);
    EntityEntry<TEntity> Remove<TEntity>(TEntity entity) where TEntity : class;
    EntityEntry Entry(object entity);
    DbSet<TEntity> Set<TEntity>() where TEntity : class;
    DatabaseFacade Database { get; }
}
```

### 7.2 DbContext

```csharp
namespace {{ProjectName}}.Infrastructure.DataContext;

public partial class {{DbContextName}} : DbContext, I{{DbContextName}}Context
{
    public {{DbContextName}}(DbContextOptions<{{DbContextName}}> options)
        : base(options) { }

    public virtual DbSet<<Entity>> <Entities> { get; set; }
    // One DbSet per entity
}
```

### 7.3 `CommandRepository<TEntity>` base class

Lives inside `{{ProjectName}}.Infrastructure` (internal, not exposed):

```csharp
namespace {{ProjectName}}.Infrastructure.Querys;

internal abstract class CommandRepository<TEntity> : ICommandRepository<TEntity>
    where TEntity : class
{
    protected readonly I{{DbContextName}}Context context;

    protected CommandRepository(I{{DbContextName}}Context context)
        => this.context = context;

    public async Task AddAsync(TEntity entity, CancellationToken ct = default)
        => await context.Set<TEntity>().AddAsync(entity, ct);

    public Task EditAsync(TEntity entity, CancellationToken ct = default)
    {
        context.Entry(entity).State = EntityState.Modified;
        return Task.CompletedTask;
    }

    public Task DeleteAsync(TEntity entity, CancellationToken ct = default)
    {
        context.Remove(entity);
        return Task.CompletedTask;
    }

    public async Task<TEntity?> FindAsync(
        Expression<Func<TEntity, bool>> predicate, CancellationToken ct = default)
        => await context.Set<TEntity>().FirstOrDefaultAsync(predicate, ct);

    public IQueryable<TEntity> FindMany(Expression<Func<TEntity, bool>> predicate)
        => context.Set<TEntity>().Where(predicate);

    public async Task SaveChanges(CancellationToken ct = default)
        => await context.SaveChangesAsync(ct);

    public void Dispose() => context.Dispose();
}
```

### 7.4 Repository query implementation (partial class pattern)

```csharp
namespace {{ProjectName}}.Infrastructure.Querys.<Entity>Repository;

internal partial class <Entity>ManagerQuery
    : CommandRepository<<Entity>>, I<Entity>Repository
{
    public <Entity>ManagerQuery(I{{DbContextName}}Context context) : base(context) { }

    public IQueryable<<Entity>> Search(
        string search, bool isActive,
        int pageSize, int pageNumber,
        out int totalRecords, out int totalPages)
    {
        var query = context.<Entities>
            .Where(e => e.IsActive == isActive
                     && (string.IsNullOrEmpty(search) || e.Name.Contains(search)));

        totalRecords = query.Count();
        totalPages = (int)Math.Ceiling((double)totalRecords / pageSize);

        return query
            .OrderByDescending(e => e.Id)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize)
            .AsNoTracking();
    }
}
```

### 7.5 `DependencyContainer.cs`

```csharp
namespace {{ProjectName}}.Infrastructure;

public static class DependencyContainer
{
    public static IServiceCollection Add{{ProjectName}}DatabaseService(
        this IServiceCollection services,
        Action<<Name>Options> configure)
    {
        var options = new <Name>Options();
        configure(options);
        services.AddSingleton(options);

        services.AddDbContext<I{{DbContextName}}Context, {{DbContextName}}>(opt =>
            opt.UseSqlServer(options.ConnectionString)
               .UseQueryTrackingBehavior(QueryTrackingBehavior.TrackAll));

        // Register repositories (Scoped)
        services.AddScoped<I<Entity>Repository, <Entity>ManagerQuery>();

        return services;
    }
}
```

**`GlobalUsings.cs`**:
```csharp
global using Microsoft.EntityFrameworkCore;
global using Microsoft.EntityFrameworkCore.ChangeTracking;
global using Microsoft.EntityFrameworkCore.Infrastructure;
global using Microsoft.Extensions.DependencyInjection;
global using System.Diagnostics.CodeAnalysis;
global using System.Linq.Expressions;
global using {{ProjectName}}.Abstractions.Entities.<SchemaName>;
global using {{ProjectName}}.Abstractions.Repositories;
global using {{ProjectName}}.Abstractions.Repositories.Transaccion;
global using {{ProjectName}}.Infrastructure.Interfaces;
global using {{ProjectName}}.Infrastructure.Options;
global using {{ProjectName}}.Infrastructure.Querys;
```

---

## 8. Core (Application Services) — `{{ProjectName}}.Core`

```
{{ProjectName}}.Core/
  Services/
    <Entity>/
      Endpoint.cs              ← Minimal API endpoint registration (static class)
      <Entity>ManagerService.cs   ← partial class: constructor + fields
      <Entity>CrudOperations.cs   ← partial class: method implementations
    DependencyContainer.cs    ← AddScoped registrations for all services
    Endpoints.cs              ← Root endpoint router
  GlobalUsings.cs
```

### 8.1 Service — partial class split

**Constructor file** (`<Entity>ManagerService.cs`):
```csharp
namespace {{ProjectName}}.Core.Services.<Entity>;

internal partial class <Entity>ManagerService : I<Entity>Service
{
    private readonly I<Entity>Repository _repository;
    private readonly I<Entity>Validator _validator;

    public <Entity>ManagerService(
        I<Entity>Repository repository,
        I<Entity>Validator validator)
    {
        _repository = repository;
        _validator = validator;
    }
}
```

**Operations file** (`<Entity>CrudOperations.cs`):
```csharp
namespace {{ProjectName}}.Core.Services.<Entity>;

internal partial class <Entity>ManagerService
{
    public async Task Register(<Entity>Dto dto, CancellationToken ct = default)
    {
        await _validator.ThrowIfDuplicated<Entity>ByNameAsync(dto.Name, ct);
        await _repository.AddAsync(new <Entity>
        {
            Name    = dto.Name,
            IsActive = true,
            CreatedAt = DateTime.UtcNow
        }, ct);
        await _repository.SaveChanges(ct);
    }

    public async Task Edit(int id, <Entity>Dto dto, CancellationToken ct = default)
    {
        var entity = await _validator.ThrowIfNotExist<Entity>ByIdAsync(id, ct);
        entity.Name      = dto.Name;
        entity.UpdatedAt = DateTime.UtcNow;
        await _repository.EditAsync(entity, ct);
        await _repository.SaveChanges(ct);
    }

    public async Task Remove(int id, CancellationToken ct = default)
    {
        var entity = await _validator.ThrowIfNotExist<Entity>ByIdAsync(id, ct);
        entity.IsActive  = false;
        entity.DeletedAt = DateTime.UtcNow;
        await _repository.EditAsync(entity, ct);
        await _repository.SaveChanges(ct);
    }

    public IQueryable<<Entity>> ListAll()
        => _repository.FindMany(e => e.IsActive);

    public PagerResultsRecords<<Entity>> Search(
        string search, bool isActive, int? pageSize, int? pageNumber)
    {
        int size   = pageSize   ?? 50;
        int number = pageNumber ?? 1;
        int totalRecords, totalPages;

        var records = _validator.ThrowIfNoElementsFound(
            () => _repository.Search(search, isActive, size, number,
                                     out totalRecords, out totalPages),
            () => throw new <Entity>NotFoundException());

        return new PagerResultsRecords<<Entity>>
        {
            TotalRecords = totalRecords,
            TotalPages   = totalPages,
            Records      = records
        };
    }
}
```

### 8.2 Minimal API Endpoint

```csharp
namespace {{ProjectName}}.Core.Services.<Entity>;

public static class Endpoint
{
    /// <summary>Registers all <Entity> HTTP endpoints.</summary>
    public static IEndpointRouteBuilder Use<Entity>Endpoints(
        this IEndpointRouteBuilder app)
    {
        app.MapGet("/api/<entity>", (I<Entity>Service svc) =>
            Results.Ok(svc.ListAll()))
            .WithTags("<Entity>");

        app.MapGet("/api/<entity>/search",
            (I<Entity>Service svc,
             [FromQuery] string search,
             [FromQuery] bool isActive,
             [FromQuery] int? pageSize,
             [FromQuery] int? pageNumber) =>
                Results.Ok(svc.Search(search, isActive, pageSize, pageNumber)))
            .WithTags("<Entity>");

        app.MapPost("/api/<entity>/register",
            async (I<Entity>Service svc, [FromBody] <Entity>Dto dto,
                   CancellationToken ct) =>
            {
                await svc.Register(dto, ct);
                return Results.Created();
            }).WithTags("<Entity>");

        app.MapPut("/api/<entity>/edit",
            async (I<Entity>Service svc,
                   [FromQuery] int id, [FromBody] <Entity>Dto dto,
                   CancellationToken ct) =>
            {
                await svc.Edit(id, dto, ct);
                return Results.Ok();
            }).WithTags("<Entity>");

        app.MapDelete("/api/<entity>/remove",
            async (I<Entity>Service svc, [FromQuery] int id, CancellationToken ct) =>
            {
                await svc.Remove(id, ct);
                return Results.Ok();
            }).WithTags("<Entity>");

        return app;
    }
}
```

### 8.3 Root endpoint router (`Endpoints.cs`)

```csharp
namespace {{ProjectName}}.Core.Services;

public static class Endpoints
{
    public static IEndpointRouteBuilder Use{{ProjectName}}Endpoints(
        this IEndpointRouteBuilder app)
    {
        app.Use<Entity1>Endpoints();
        app.Use<Entity2>Endpoints();
        // Add one line per entity
        return app;
    }
}
```

### 8.4 DependencyContainer (`Services/DependencyContainer.cs`)

```csharp
namespace {{ProjectName}}.Core.Services;

public static class DependencyContainer
{
    public static IServiceCollection Add{{ProjectName}}CoreService(
        this IServiceCollection services)
    {
        services.AddScoped<I<Entity1>Service, <Entity1>ManagerService>();
        services.AddScoped<I<Entity2>Service, <Entity2>ManagerService>();
        // One AddScoped per service
        return services;
    }
}
```

**`GlobalUsings.cs`**:
```csharp
global using Microsoft.AspNetCore.Builder;
global using Microsoft.AspNetCore.Http;
global using Microsoft.AspNetCore.Mvc;
global using Microsoft.AspNetCore.Routing;
global using Microsoft.Extensions.DependencyInjection;
global using {{ProjectName}}.Abstractions.Dtos;
global using {{ProjectName}}.Abstractions.Interfaces.Validators;
global using {{ProjectName}}.Abstractions.Models;
global using {{ProjectName}}.Abstractions.Repositories;
global using {{ProjectName}}.Abstractions.Services;
global using {{ProjectName}}.Core.Services.<Entity>;   // repeat for each entity namespace
global using Dalton.Shared.Abstractions.Exceptions;
```

---

## 9. Validators — `{{ProjectName}}.Validators`

```
{{ProjectName}}.Validators/
  Validators/
    <Entity>Validator.cs
  DependencyContainer.cs
  GlobalUsings.cs
```

### 9.1 Validator implementation

```csharp
namespace {{ProjectName}}.Validators.Validators;

internal sealed class <Entity>Validator : I<Entity>Validator
{
    private readonly I<Entity>Repository _repository;

    public <Entity>Validator(I<Entity>Repository repository)
        => _repository = repository;

    public async Task<<Entity>> ThrowIfNotExist<Entity>ByIdAsync(
        int id, CancellationToken ct = default)
    {
        var entity = await _repository.FindAsync(e => e.Id == id, ct)
            ?? throw new <Entity>NotFoundException();
        return entity;
    }

    public async Task ThrowIfDuplicated<Entity>ByNameAsync(
        string name, CancellationToken ct = default)
    {
        var exists = await _repository.FindAsync(e => e.Name == name, ct);
        if (exists is not null)
            throw new <Entity>DuplicatedException();
    }
}
```

### 9.2 DependencyContainer

```csharp
namespace {{ProjectName}}.Validators;

public static class DependencyContainer
{
    public static IServiceCollection Add{{ProjectName}}ValidatorsService(
        this IServiceCollection services)
    {
        services.TryAddScoped<I<Entity>Validator, <Entity>Validator>();
        // One TryAddScoped per validator
        return services;
    }
}
```

**`GlobalUsings.cs`**:
```csharp
global using Microsoft.Extensions.DependencyInjection;
global using Microsoft.Extensions.DependencyInjection.Extensions;
global using {{ProjectName}}.Abstractions.Entities.<SchemaName>;
global using {{ProjectName}}.Abstractions.Interfaces.Validators;
global using {{ProjectName}}.Abstractions.Repositories;
global using {{ProjectName}}.Validators.Validators;
global using Dalton.Shared.Abstractions.Exceptions;
```

---

## 10. Entry point — `Web.Api`

### 10.1 `Program.cs`

```csharp
using {{ProjectName}}.Core.Services;
using {{ProjectName}}.Abstractions.Configuration;
using Dalton.Shared.MessagesLocalizer;

var builder = WebApplication.CreateBuilder(args);

// ── Services ──────────────────────────────────────────────
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddCors(opt =>
    opt.AddDefaultPolicy(pol =>
    {
        pol.AllowAnyHeader();
        pol.AllowAnyMethod();
        pol.AllowAnyOrigin();
    }));

builder.Services
    .AddMessagesLocalizer()
    .Add{{ProjectName}}ValidatorsService()
    .Add{{ProjectName}}DatabaseService(opt =>
        builder.Configuration
               .GetSection(<Name>Options.<Name>Key)
               .Bind(opt))
    .Add{{ProjectName}}CoreService();

// ── Pipeline ──────────────────────────────────────────────
var app = builder.Build();

app.UseDaltonExceptionHandler();

app.UseSwagger();
app.UseSwaggerUI();

app.UseHttpsRedirection();
app.UseRouting();
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();

app.Use{{ProjectName}}Endpoints();
app.MapControllers();

app.Run();
```

### 10.2 Web.Api.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>{{TargetFramework}}</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.4.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\{{ProjectName}}.Abstractions\{{ProjectName}}.Abstractions.csproj" />
    <ProjectReference Include="..\{{ProjectName}}.Core\{{ProjectName}}.Core.csproj" />
    <ProjectReference Include="..\{{ProjectName}}.Infrastructure\{{ProjectName}}.Infrastructure.csproj" />
    <ProjectReference Include="..\{{ProjectName}}.Validators\{{ProjectName}}.Validators.csproj" />
    <ProjectReference Include="..\Dalton.Shared.Abstractions\Dalton.Shared.Abstractions.csproj" />
    <ProjectReference Include="..\Dalton.Shared.ExceptionHandler\Dalton.Shared.ExceptionHandler.csproj" />
    <ProjectReference Include="..\Dalton.Shared.MessagesLocalizer\Dalton.Shared.MessagesLocalizer.csproj" />
  </ItemGroup>
</Project>
```

---

## 11. Architecture & coding standards (mandatory)

All generated code **must** comply with the following:

### SOLID
| Principle | Rule |
|---|---|
| **S**ingle Responsibility | Each class has one reason to change. Split concerns: service constructor in one file, CRUD operations in another (partial classes). |
| **O**pen/Closed | Use interfaces for all cross-layer communication. Never depend on concrete implementations across project boundaries. |
| **L**iskov Substitution | Repository interfaces are substitutable; any implementation must honour the contract. |
| **I**nterface Segregation | Keep service and repository interfaces small and focused. One interface per entity per concern. |
| **D**ependency Inversion | High-level modules (Core) depend on abstractions (Abstractions), never on Infrastructure classes. |

### Clean Architecture rings
1. **Dalton.Shared.Abstractions** and **{{ProjectName}}.Abstractions** → innermost ring (no external dependencies).
2. **{{ProjectName}}.Core** → application ring (only depends on ring 1 + framework DI).
3. **{{ProjectName}}.Infrastructure / Validators** → outer ring (implements ring-1 contracts).
4. **Web.Api** → composition root only; wires everything via DI, contains zero business logic.

### Hexagonal / Ports & Adapters
- **Ports** = interfaces in `Abstractions` (`IRepository`, `IService`, `IValidator`).
- **Adapters** = implementations in `Infrastructure` (EF Core queries) and `Validators`.
- **Application core** = `Core` services; drives ports, never accesses adapters directly.

### Modern .NET rules (≥ .NET 7)
- Use **Minimal API** (`app.MapGet/Post/Put/Delete`) — no MVC controllers except when scaffolded by the team.
- Use **`global using`** in `GlobalUsings.cs` per project.
- Use **`record`** or **`sealed record`** for DTOs where immutability is desirable.
- Use **`Task<T>`** / `async-await` for all I/O-bound operations. Never use `.Result` or `.Wait()`.
- Use **`CancellationToken`** parameters on all async methods.
- Use **Options pattern** (`IOptions<T>` / binding via `GetSection(...).Bind(...)`) for all configuration.
- Use **`IServiceCollection` extension methods** (`DependencyContainer.cs`) for all DI registration; never call `new` on services.
- Use **`TryAdd*`** in validators to avoid duplicate registration.
- Use **`Results.*`** factory methods in Minimal API handlers.
- Use **`WithTags("...")`** on every Minimal API endpoint for Swagger grouping.
- Use `DateTime.UtcNow` for all timestamps.
- Avoid `nullable` disable globally; restrict to `Entities/` when needed for EF Core scaffold compatibility.
- Avoid obsolete APIs: no `HttpResponseMessage` raw reading, no `StreamReader`, no `.Result` blocking.

---

## 12. Step-by-step recreation guide (for AI)

When asked to create a new project with this template, follow these exact steps in order:

1. **Resolve `{{ProjectName}}`** — ask the user for the project name if not provided.
2. **Create solution** — `dotnet new sln -n {{ProjectName}}`.
3. **Create Shared projects** first (no dependencies):
   - `Dalton.Shared.Abstractions`
   - `Dalton.Shared.MessagesLocalizer`
   - `Dalton.Shared.ExceptionHandler`
4. **Create Backend projects** in dependency order:
   - `{{ProjectName}}.Abstractions`
   - `{{ProjectName}}.Infrastructure`
   - `{{ProjectName}}.Validators`
   - `{{ProjectName}}.Core`
5. **Create Consumer** — `Web.Api`.
6. **Add all projects to the solution** using `dotnet sln add` with the correct solution folder nesting.
7. **Add `GlobalUsings.cs`** to each project with the correct imports from Section 6–10.
8. **Add domain entities** in `{{ProjectName}}.Abstractions/Entities/`.
9. **Add domain exceptions** in `Dalton.Shared.Abstractions/Exceptions/` with `[HttpStatusCode]`.
10. **Add message keys** in `MessagesKeys.cs` and translations in `MessagesLocalizer.cs`.
11. **For each entity**, generate:
    - DTO in `Abstractions/Dtos/`
    - Repository interface in `Abstractions/Repositories/`
    - Service interface in `Abstractions/Services/`
    - Validator interface in `Abstractions/Interfaces/Validators/`
    - DbSet entry in `{{DbContextName}}`
    - Repository query implementation (partial class) in `Infrastructure/Querys/`
    - Validator implementation in `Validators/Validators/`
    - Service (partial class — two files) in `Core/Services/<Entity>/`
    - Endpoint (static class) in `Core/Services/<Entity>/`
12. **Register everything** in the respective `DependencyContainer.cs` files.
13. **Wire `Program.cs`** in `Web.Api` following Section 10.
14. **Verify** with `dotnet build` — zero warnings, zero errors.

---

## 13. File naming conventions

| Artifact | File name pattern |
|---|---|
| Service interface | `I<Entity>Service.cs` |
| Repository interface | `I<Entity>Repository.cs` |
| Validator interface | `I<Entity>Validator.cs` |
| Service constructor | `<Entity>ManagerService.cs` |
| Service operations | `<Entity>CrudOperations.cs` |
| Repository query | `<Entity>ManagerQuery.cs` |
| Validator impl | `<Entity>Validator.cs` |
| Endpoint | `Endpoint.cs` (inside `<Entity>/`) |
| Entity | `<Entity>.cs` (inside `Entities/<Schema>/`) |
| DTO | `<Entity>Dto.cs` |
| Exception | `<Entity>NotFoundException.cs`, `<Entity>DuplicatedException.cs`, etc. |
| DI registration | `DependencyContainer.cs` (one per project) |
| Global usings | `GlobalUsings.cs` (one per project) |
| Options | `<Name>Options.cs` |

---

## 14. What NOT to generate

- No `Controller` classes (use Minimal API).
- No EF Core `Migrations` folder (scaffold DB-first or create manually).
- No `appsettings.json` secrets — use environment variables or user-secrets.
- No `[Authorize]` attributes without being explicitly asked.
- No static helper classes with business logic (put that in services).
- No `async void` methods.
- No `.Result` / `.Wait()` calls.
- No hardcoded connection strings in code.
- No `Console.WriteLine` in production code.
