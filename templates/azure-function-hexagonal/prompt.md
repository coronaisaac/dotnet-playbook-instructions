Vamos a crear un nuevo proyecto de tipo **Azure Function (Isolated Worker)** siguiendo el archivo `instructions.md` de este template.

Antes de generar cualquier código, necesito que me hagas las siguientes preguntas y esperes mis respuestas:

---

1. ¿Cuál es el nombre de la solución? (reemplaza `{{SolutionName}}`)
   Ejemplo: `Fn_Historial_Backend`

2. ¿Cuál es el nombre del dominio/contexto principal? (reemplaza `{{DomainName}}`)
   Ejemplo: `Historial`

3. ¿Cuál es el prefijo del namespace completo? (reemplaza `{{DomainPrefix}}`)
   Puede incluir empresa. Ejemplo: `Whatsapp.Historial` → genera `Whatsapp.Historial.Core`, `Whatsapp.Historial.Function`, etc.

4. ¿Qué proveedor de base de datos usarás? (reemplaza `{{DatabaseProvider}}` y `{{DatabaseProviderVersion}}`)
   Opciones comunes:
   - `Npgsql.EntityFrameworkCore.PostgreSQL` / `9.0.4`  ← PostgreSQL
   - `Microsoft.EntityFrameworkCore.SqlServer` / `10.0.0`  ← SQL Server
   [Presiona Enter para usar PostgreSQL por defecto]

5. ¿Cómo se llamará el DbContext? (reemplaza `{{DbContextName}}`)
   Ejemplo: `HistorialContext`

6. ¿Cuál es la clave de la variable de entorno para la connection string? (reemplaza `{{ConnectionStringKey}}`)
   Ejemplo: `ConnectionStrings__DefaultDb`
   [Presiona Enter para usar `ConnectionStrings__DefaultDb` por defecto]

---

Una vez que respondas todas las preguntas, generaré la solución completa con:

- ✅ Solución `.sln` con 3 solution folders: `Backend/`, `Shared/`, `Consumers/`
- ✅ `{{DomainPrefix}}.Core` — Dominio puro: entidades, interfaces, excepciones, enums, DTOs, modelos
- ✅ `{{DomainPrefix}}.Infrastructure` — DbContext, CommandRepository base, repositorios por entidad
- ✅ `{{DomainPrefix}}.Services` — Casos de uso, servicios de aplicación
- ✅ `{{DomainPrefix}}.Function` — Azure Functions HTTP Trigger (Isolated Worker), Program.cs, host.json, local.settings.json
- ✅ `{{DomainPrefix}}.Shared.ExceptionHandler` — Middleware `IFunctionsWorkerMiddleware` con ProblemDetails RFC-7807
- ✅ `{{DomainPrefix}}.Shared.MessagesLocalizer` — Localización de mensajes de error
- ✅ `GlobalUsings.cs` en cada proyecto
- ✅ `DependencyContainer.cs` con extension methods en cada capa
- ✅ Ejemplo completo de entidad con repositorio, servicio y función HTTP
- ✅ Todo usando C# 12+: primary constructors, `sealed`, `internal`, `record`, `init`, `required`

Todo el código seguirá estrictamente: **SOLID**, **Clean Architecture**, **Hexagonal (Ports & Adapters)**, **modern .NET 10** (sin Newtonsoft.Json, sin `.Result`/`.Wait()`, con `CancellationToken`, `IHttpClientFactory`, `AsNoTracking()`, logging estructurado).

¿Empezamos?
