# 📋 Documentación del Proyecto — Full Stack (.NET 9 + Angular)

---

## 📌 Tabla de Contenidos

1. [Descripción General](#descripción-general)
2. [Tecnologías Utilizadas](#tecnologías-utilizadas)
3. [Arquitectura del Sistema](#arquitectura-del-sistema)
4. [Domain-Driven Design (DDD)](#domain-driven-design-ddd)
5. [Estructura del Proyecto](#estructura-del-proyecto)
6. [Capas de la Aplicación](#capas-de-la-aplicación)
7. [Patrones de Diseño Utilizados](#patrones-de-diseño-utilizados)
8. [API REST — Endpoints](#api-rest--endpoints)
9. [Frontend — Angular](#frontend--angular)
10. [Base de Datos](#base-de-datos)
11. [Configuración y Ejecución](#configuración-y-ejecución)

---

## Descripción General

Este proyecto es una aplicación **Full Stack** desarrollada con **.NET 9.0** en el backend y **Angular** en el frontend. Su objetivo es gestionar usuarios y sus ítems de trabajo (**Work Items**), permitiendo asignación, seguimiento y control de estado de tareas dentro de un equipo.

El sistema fue construido siguiendo los principios de **Clean Architecture** y **Domain-Driven Design (DDD)**, garantizando un código mantenible, desacoplado y escalable. Se aplicaron principios **SOLID** en cada capa para asegurar la separación de responsabilidades y facilitar la extensión del sistema sin modificar el código existente.

---

## Tecnologías Utilizadas

### Backend
| Tecnología | Versión | Propósito |
|---|---|---|
| .NET | 9.0 | Framework principal del backend |
| ASP.NET Core | 9.0 | API REST |
| Entity Framework Core | 9.x | ORM para acceso a datos |
| MediatR | 12.x | Implementación del patrón CQRS |
| AutoMapper | 13.x | Mapeo entre entidades y DTOs |
| FluentValidation | 11.x | Validación de comandos y queries |
| PostgreSQL | 16.x | Motor de base de datos |

### Frontend
| Tecnología | Versión | Propósito |
|---|---|---|
| Angular | 17+ | Framework SPA del frontend |
| TypeScript | 5.x | Lenguaje principal |
| Angular Material | 17+ | Componentes de UI |
| RxJS | 7.x | Programación reactiva |
| HttpClient | — | Consumo de la API REST |

---

## Arquitectura del Sistema

El proyecto sigue el modelo de **Clean Architecture** propuesto por Robert C. Martin (*Uncle Bob*). Este modelo organiza el código en capas concéntricas donde las dependencias siempre apuntan hacia adentro, es decir, hacia el dominio.

```
┌──────────────────────────────────────────────┐
│                   API Layer                  │  ← Controllers, Middlewares
├──────────────────────────────────────────────┤
│              Application Layer               │  ← Commands, Queries, Handlers, DTOs
├──────────────────────────────────────────────┤
│               Domain Layer                   │  ← Entities, Value Objects, Interfaces
├──────────────────────────────────────────────┤
│            Infrastructure Layer              │  ← EF Core, Repositories, DbContext
└──────────────────────────────────────────────┘
```

> **Regla principal:** Las capas internas no conocen las capas externas. El dominio no depende de infraestructura ni de la API.

---

## Domain-Driven Design (DDD)

El diseño del sistema está guiado por el dominio del negocio. Se identificaron los siguientes conceptos clave:

### Agregados y Entidades

| Entidad | Rol | Descripción |
|---|---|---|
| `User` | Agregado raíz | Representa al usuario del sistema con sus datos personales y credenciales |
| `WorkItem` | Agregado raíz | Representa una tarea o ítem de trabajo con código, descripción y estado |
| `UserWorkItem` | Entidad de relación | Tabla de unión que relaciona un usuario con sus ítems de trabajo asignados |

### Entidad Base

Todas las entidades del dominio heredan de `BaseEntity`, que centraliza el identificador único:

```csharp
public abstract class BaseEntity
{
    public int Id { get; protected set; }
}
```

### Value Objects

Los Value Objects representan conceptos del dominio sin identidad propia. Son inmutables y se comparan por valor:

- **`WorkItemStatus`** — Encapsula el estado de un ítem (`P` = Pendiente, `A` = Activo, `C` = Completado).
- **`Email`** — Encapsula y valida el formato del correo electrónico del usuario.

### Invariantes del Dominio

Las entidades protegen sus propias reglas de negocio mediante constructores privados y métodos de fábrica, evitando estados inválidos:

```csharp
// El dominio no permite crear un User sin los datos mínimos requeridos
public static User Create(string name, string lastName, string email, ...) { ... }
```

---

## Estructura del Proyecto

```
📦 Solution
├── 📁 src
│   ├── 📁 Domain
│   │   ├── Entities
│   │   │   ├── User.cs
│   │   │   ├── WorkItem.cs
│   │   │   └── UserWorkItem.cs
│   │   ├── ValueObjects
│   │   │   ├── Email.cs
│   │   │   └── WorkItemStatus.cs
│   │   └── Interfaces
│   │       ├── IUserRepository.cs
│   │       └── IWorkItemRepository.cs
│   │
│   ├── 📁 Application
│   │   ├── Common
│   │   │   ├── ApiResponse.cs
│   │   │   └── MappingProfiles
│   │   ├── Users
│   │   │   ├── Queries
│   │   │   │   ├── GetAllUsers
│   │   │   │   │   ├── GetAllUsersQuery.cs
│   │   │   │   │   └── GetAllUsersQueryHandler.cs
│   │   │   │   └── GetWorkItemsByUserId
│   │   │   │       ├── GetWorkItemsByUserIdQuery.cs
│   │   │   │       └── GetWorkItemsByUserIdQueryHandler.cs
│   │   │   └── DTOs
│   │   │       └── UserListDto.cs
│   │   └── WorkItems
│   │       ├── Queries
│   │       │   └── GetWorkItemsByStatus
│   │       │       ├── GetWorkItemsByStatusQuery.cs
│   │       │       └── GetWorkItemsByStatusQueryHandler.cs
│   │       └── DTOs
│   │           └── WorkItemDto.cs
│   │
│   ├── 📁 Infrastructure
│   │   ├── Persistence
│   │   │   ├── AppDbContext.cs
│   │   │   └── Configurations
│   │   │       ├── UserConfiguration.cs
│   │   │       └── WorkItemConfiguration.cs
│   │   └── Repositories
│   │       ├── UserRepository.cs
│   │       └── WorkItemRepository.cs
│   │
│   └── 📁 Api
│       ├── Controllers
│       │   ├── UserController.cs
│       │   └── WorkItemController.cs
│       ├── Middlewares
│       └── Program.cs
│
└── 📁 frontend (Angular)
    ├── src
    │   ├── app
    │   │   ├── core
    │   │   ├── features
    │   │   │   ├── users
    │   │   │   └── work-items
    │   │   └── shared
    │   └── environments
    └── angular.json
```

---

## Capas de la Aplicación

### 1. Domain Layer
Es el núcleo del sistema. Contiene las entidades, value objects e interfaces de repositorios. **No tiene dependencias externas.**

- Define los contratos (`IUserRepository`, `IWorkItemRepository`) que la infraestructura debe implementar.
- Protege las invariantes del negocio dentro de las entidades.

### 2. Application Layer
Orquesta los casos de uso del sistema usando el patrón **CQRS** a través de **MediatR**.

- **Queries** — Consultas de solo lectura que retornan datos.
- **Commands** — Operaciones que modifican el estado del sistema.
- **Handlers** — Procesan queries y commands coordinando repositorios y mappers.
- **DTOs** — Objetos de transferencia de datos definidos como `record` para garantizar inmutabilidad.

### 3. Infrastructure Layer
Implementa los contratos definidos en el dominio.

- **AppDbContext** — Configuración de Entity Framework Core con PostgreSQL.
- **Repositories** — Implementan `IUserRepository` e `IWorkItemRepository` usando EF Core.
- Usa `AsNoTracking()` en todas las queries de solo lectura para mejorar el rendimiento.

### 4. Api Layer
Expone los casos de uso como endpoints REST.

- Los **Controllers** son delgados: solo reciben la petición, delegan a **MediatR** y retornan la respuesta.
- Usa `ApiResponse<T>` como wrapper estándar para todas las respuestas.

---

## Patrones de Diseño Utilizados

| Patrón | Capa | Propósito |
|---|---|---|
| **CQRS** | Application | Separar operaciones de lectura y escritura |
| **Mediator** | Application | Desacoplar handlers de controllers mediante MediatR |
| **Repository** | Domain / Infrastructure | Abstraer el acceso a datos del dominio |
| **DTO (Record)** | Application | Transferencia de datos inmutable entre capas |
| **Mapping Profile** | Application | Transformación entre entidades y DTOs con AutoMapper |
| **Wrapper Response** | Application / Api | Estandarizar el formato de respuesta con `ApiResponse<T>` |

---

## API REST — Endpoints

### Users

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/api/users` | Obtiene todos los usuarios |
| `GET` | `/api/users/{userId}/work-items` | Obtiene los work items de un usuario |

### Work Items

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/api/work-items/status/{status}` | Obtiene work items por estado (`P`, `A`, `C`) |

### Formato de Respuesta

Todas las respuestas siguen el wrapper `ApiResponse<T>`:

```json
{
  "success": true,
  "data": [...],
  "message": null
}
```

---

## Frontend — Angular

El frontend consume la API REST y presenta la información al usuario final.

### Estructura de Módulos

```
features/
├── users/
│   ├── components/
│   │   ├── user-list/
│   │   └── user-work-items/
│   └── services/
│       └── user.service.ts
└── work-items/
    ├── components/
    │   └── work-item-list/
    └── services/
        └── work-item.service.ts
```

### Servicio de ejemplo

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly apiUrl = `${environment.apiUrl}/users`;

  constructor(private http: HttpClient) {}

  getWorkItemsByUserId(userId: number): Observable<ApiResponse<WorkItemDto[]>> {
    return this.http.get<ApiResponse<WorkItemDto[]>>(`${this.apiUrl}/${userId}/work-items`);
  }
}
```

---

## Base de Datos

El sistema utiliza **PostgreSQL 16** como motor de base de datos.

### Diagrama de Tablas Principal

```
users
├── id_us         (PK)
├── name
├── last_name
├── email
├── username
├── password_hash
└── ...

work_items
├── id_wi         (PK)
├── code_wi
├── description_wi
├── status_wi     (P | A | C)
├── relevance
├── created_at
└── expiration_date

user_work_items
├── id_uwi           (PK)
├── id_us            (FK → users)
├── id_wi            (FK → work_items)
├── assignment_date
└── status
```

---

## Configuración y Ejecución

### Backend

```bash
# Restaurar dependencias
dotnet restore

# Aplicar migraciones
dotnet ef database update

# Ejecutar la API
dotnet run --project src/Api
```

### Frontend

```bash
# Instalar dependencias
npm install

# Ejecutar en desarrollo
ng serve

# Build para producción
ng build --configuration production
```

### Variables de Entorno — Backend (`appsettings.json`)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=workitems_db;Username=postgres;Password=yourpassword"
  }
}
```

### Variables de Entorno — Frontend (`environment.ts`)

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:5000/api'
};
```

---

> Documentación generada para el proyecto Full Stack — .NET 9.0 + Angular.
> Arquitectura: Clean Architecture + Domain-Driven Design (DDD).
