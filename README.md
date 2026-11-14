# School

Proyecto en **C# (.NET 8)** que implementa las capas de **Dominio**, **Infraestructura** y **Aplicación (servicios)** de un sistema escolar, usando **Entity Framework Core** y **SQL Server**. Estas capas serán consumidas por un API.

## Tecnologías

- C# / .NET 8
- Entity Framework Core 8 (SQL Server)
- Patrón Repositorio

## Estructura del proyecto

```
School.sln
├── School.Domain
│   ├── Core            → BaseEntity, Person
│   ├── Entities        → Course, Department, Instructor, Student
│   └── Repository      → IBaseRepository<T>
└── School.Infrastructure
    ├── Context         → SchoolContext
    ├── Core            → BaseRepository<T>
    ├── Exceptions      → CourseException, DepartmentException
    ├── Interfaces      → ICourseRepository, IDepartmentRepository
    ├── Models          → CursoModel, DepartmentModel (DTOs)
    └── Repositories    → CourseRepository, DepartmentRepository

School.Application
├── Contract        → IDepartmentService, ICourseService
├── Core            → IBaseService, BaseService, ServiceResult
├── Dtos            → DtoBase, Department/*, Course/*
└── Service         → DepartmentService, CourseService
```

### Capa de servicio (School.Application)

- Los servicios devuelven un `ServiceResult` (`Success`, `Message`, `Data`).
- Cada servicio valida todos los campos de su entidad antes de llamar al repositorio.
- Registro en `Program.cs`:

```csharp
builder.Services.AddScoped<IDepartmentService, DepartmentService>();
builder.Services.AddScoped<ICourseService, CourseService>();
```

## Entidades

| Entidad | Descripción | Campos principales |
|---|---|---|
| `BaseEntity` | Clase base de todas las entidades | Id, CreationDate, CreationUser, ModifyDate, ModifyUser, Deleted, DeletedDate, DeletedUser |
| `Person` | Clase base para personas | FirstName, LastName |
| `Student` | Estudiante | EnrollmentDate |
| `Instructor` | Profesor | HireDate |
| `Department` | Departamento | Name, Budget, StartDate, InstructorId |
| `Course` | Curso | Title, Credits, DepartmentId |

## Características

- Repositorio genérico (`BaseRepository<T>`) con operaciones `Exists`, `GetAll`, `GetEntityById`, `Save`, `Update` y `Remove`.
- **Borrado lógico**: los registros se marcan con `Deleted = true`, no se eliminan físicamente.
- Validaciones de negocio en `CourseRepository` y `DepartmentRepository`, que lanzan `CourseException` y `DepartmentException`.
- Modelos (DTOs) para no exponer las entidades directamente al API.

## Requisitos

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- SQL Server (o SQL Server LocalDB / Express)
- Visual Studio 2022 o VS Code

## Cómo ejecutarlo

1. Clonar el repositorio y cambiar a la rama `develop`:

   ```bash
   git clone https://github.com/TU_USUARIO/TU_REPO.git
   cd TU_REPO
   git checkout develop
   ```

2. Compilar la solución:

   ```bash
   dotnet build
   ```

3. En el proyecto del API, agregar la referencia a `School.Infrastructure` y la cadena de conexión en `appsettings.json`:

   ```json
   "ConnectionStrings": {
     "SchoolContext": "Server=(localdb)\\MSSQLLocalDB;Database=School;Trusted_Connection=True;TrustServerCertificate=True"
   }
   ```

4. Registrar los servicios en `Program.cs`:

   ```csharp
   builder.Services.AddDbContext<SchoolContext>(options =>
       options.UseSqlServer(builder.Configuration.GetConnectionString("SchoolContext")));

   builder.Services.AddScoped<ICourseRepository, CourseRepository>();
   builder.Services.AddScoped<IDepartmentRepository, DepartmentRepository>();
   ```

5. Crear la base de datos con las migraciones:

   ```bash
   dotnet ef migrations add InitialCreate --project School.Infrastructure --startup-project <ProyectoApi>
   dotnet ef database update --project School.Infrastructure --startup-project <ProyectoApi>
   ```

## Ejemplo de uso

```csharp
public class CoursesController : ControllerBase
{
    private readonly ICourseRepository _courseRepository;

    public CoursesController(ICourseRepository courseRepository)
    {
        _courseRepository = courseRepository;
    }

    [HttpGet]
    public async Task<IActionResult> Get() => Ok(await _courseRepository.GetCourses());
}
```

## Flujo de trabajo (Git)

- `main`: rama estable.
- `develop`: capas Domain e Infrastructure.
- `feature/application-layer`: capa de servicio (School.Application).

## Autor

Nombre del estudiante — Matrícula
