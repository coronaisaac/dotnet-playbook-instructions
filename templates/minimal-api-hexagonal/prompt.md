Vamos a crear un nuevo proyecto .NET siguiendo el archivo instructions.md que está en la raíz del repo.
Antes de generar cualquier código, necesito que me hagas las siguientes preguntas y esperes mis respuestas:

1. ¿Cuál es el nombre del dominio/proyecto? (este valor reemplazará {{ProjectName}})
   Ejemplo: Whatsapp.Historial → genera proyectos como Whatsapp.Historial.Core, Whatsapp.Historial.Infrastructure, etc.

2. ¿Cuál es el prefijo de las librerías compartidas (Shared)? (este valor reemplazará {{SharedPrefix}})
   Estas son las DLLs reutilizables entre proyectos (ExceptionHandler, MessagesLocalizer, Abstractions base).
   Ejemplo: MiEmpresa → genera MiEmpresa.Shared.Abstractions, MiEmpresa.Shared.ExceptionHandler, etc.

3. ¿Qué versión de .NET usarás? (reemplaza {{TargetFramework}})
   Opciones: net8.0 (recomendado y estable) o net10.0 (si el SDK lo soporta sin conflictos)
   [Presiona Enter para dejar net8.0 por defecto]

4. ¿Cómo se llamará el DbContext? (reemplaza {{DbContextName}})
   Ejemplo: HistorialDataContext

5. ¿Cuál es la clave de la connection string en appsettings? (reemplaza {{ConnectionStringKey}})
   Ejemplo: ConnectionStringHistorialKey
   [Presiona Enter para usar ConnectionString{{ProjectName sin puntos}}Key por defecto]

Una vez que respondas todas las preguntas, procederé a generar la solución completa con:
- Estructura de solución con carpetas Backend / Shared / Consumers
- Todos los .csproj con sus dependencias correctas
- GlobalUsings.cs por proyecto
- Interfaces genéricas de repositorio (ICommandRepository<T> y sus partes)
- DependencyContainer.cs por capa
- Program.cs completo con el pipeline
- Un ejemplo de entidad de muestra con su DTO, excepción, repositorio, validador, servicio (partial classes) y endpoint Minimal API

Todo el código generado seguirá estrictamente los estándares: SOLID, Clean Architecture, Hexagonal (Ports & Adapters) y modern .NET (≥ net8.0): async/await, CancellationToken, Minimal API, Options pattern, global usings, sin código obsoleto.

¿Empezamos?
