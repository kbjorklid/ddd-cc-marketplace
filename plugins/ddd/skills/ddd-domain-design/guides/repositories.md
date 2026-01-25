# Repository Interfaces (Domain Layer)

## Overview

In Tactical DDD, a **Repository Interface** is a domain concern that provides an abstraction simulating a collection of Aggregates in memory. The interface is defined in the Domain layer, while implementations are Infrastructure concerns.

**Core principles:**
- **Collection-like interface** - Mimics in-memory collection semantics
- **Aggregate Root only** - Repository interfaces exist only for Aggregate Roots
- **Domain interface** - Interface defined in Domain layer
- **Infrastructure implementation** - Implementation in Infrastructure layer (not covered in this guide)

```mermaid
flowchart TB
    subgraph Domain["Domain Layer (This Guide)"]
        RI[IAggregateRepository Interface]
    end

    subgraph Infrastructure["Infrastructure Layer (Not Covered)"]
        RIImpl[SqlAggregateRepository Implementation]
        DB[(Database)]
    end

    subgraph Application["Application Layer"]
        AS[Application Service]
    end

    AS -->|Depends on| RI
    RI -->|Implemented by| RIImpl
    RIImpl -->|Accesses| DB

    style Domain fill:#e1f5e1
    style Infrastructure fill:#fff4e1
    style Application fill:#e1f5e1
```

## Repository Interface Rules

### Rule 1: One Repository Interface Per Aggregate Root

**Rule:** Only Aggregate Roots have repository interfaces. Internal entities never have repositories.

```mermaid
flowchart TB
    subgraph Correct["✅ Correct"]
        C1[IOrderRepository]
        C2[ICustomerRepository]
        C3[IProductRepository]
    end

    subgraph Wrong["❌ Wrong"]
        W1[IOrderItemRepository]
        W2[IOrderStatusRepository]
    end

    style Correct fill:#e1f5e1
    style Wrong fill:#f5e1e1
```

**Rationale:**

- Internal entities are accessed through their Aggregate Root
- Direct access to internal entities bypasses invariant enforcement
- Repository represents a consistency boundary
- Only Roots can be independently loaded and saved

**Example:**

```mermaid
classDiagram
    class IOrderRepository {
        <<Interface>> <<Domain Layer>>
        +Add(order: Order) Result~void~
        +Update(order: Order) Result~void~
        +Remove(order: Order) Result~void~
        +GetById(id: OrderId) Result~Order~
        +Find(spec: ISpecification) List~Order~
        +Exists(id: OrderId) Result~bool~
    }

    class Order {
        <<Aggregate Root>>
        -Id: OrderId
        -Items: List~OrderItem~
        +AddItem() Result
        +RemoveItem() Result
    }

    class OrderItem {
        <<Entity>>
        -Id: OrderItemId
        +GetTotal() Money
    }

    note for OrderItem "No repository!\nAccessed via Order Root"
```

### Rule 2: Interface in Domain Layer

**Rule:** Repository interface defined in Domain layer, implementation in Infrastructure layer.

```mermaid
classDiagram
    class ICustomerRepository {
        <<Domain Layer>> <<Interface>>
        +Add(customer: Customer) Result~void~
        +GetById(id: CustomerId) Result~Customer~
        +FindByEmail(email: EmailAddress) Result~Customer~
    }

    class SqlCustomerRepository {
        <<Infrastructure Layer>> <<Implementation>>
        -DbContext: DbContext
        +Add(customer: Customer) Result~void~
        +GetById(id: CustomerId) Result~Customer~
        +FindByEmail(email: EmailAddress) Result~Customer~
    }

    class InMemoryCustomerRepository {
        <<Testing Layer>> <<Test Double>>
        -Customers: Dictionary~CustomerId,Customer~
        +Add(customer: Customer) Result~void~
        +GetById(id: CustomerId) Result~Customer~
    }

    ICustomerRepository <|.. SqlCustomerRepository
    ICustomerRepository <|.. InMemoryCustomerRepository
```

**Benefits:**

- **Dependency Inversion** - Domain doesn't depend on Infrastructure
- **Testability** - Swap in fake repository for unit tests
- **Flexibility** - Change persistence strategy without affecting domain

**Layer dependencies:**

```mermaid
flowchart LR
    A[Domain Layer] -->|Defines Interface| B[Repository Interface]
    C[Infrastructure Layer] -->|Implements| B
    D[Application Layer] -->|Uses| B

    style A fill:#e1f5e1
    style C fill:#fff4e1
    style D fill:#e1f5e1
```

### Rule 3: Collection-Like Semantics

**Rule:** Repository interface should feel like an in-memory collection, not a database access layer.

```mermaid
flowchart TB
    subgraph Good["✅ Collection-like"]
        G1[repo.Add(order)]
        G2[repo.Update(order)]
        G3[repo.GetById(id)]
        G4[repo.Find(spec)]
        G5[repo.Remove(order)]
    end

    subgraph Bad["❌ Database-like"]
        B1[repo.ExecuteSql(sql)]
        B2[repo.SelectFromWhere(query)]
        B3[repo.CallStoredProcedure(name)]
    end

    style Good fill:#e1f5e1
    style Bad fill:#f5e1e1
```

**Standard operations:**

| Operation | Semantic | Example |
|-----------|----------|---------|
| `Add()` | Insert new aggregate | `repo.Add(order)` |
| `Update()` | Modify existing aggregate | `repo.Update(order)` |
| `Remove()` | Delete aggregate | `repo.Remove(order)` |
| `GetById()` | Find by ID | `repo.GetById(orderId)` |
| `Find()` | Query with specification | `repo.Find(spec)` |
| `Exists()` | Check existence | `repo.Exists(orderId)` |
| `Count()` | Count matching aggregates | `repo.Count(spec)` |

### Rule 4: Return Result for Operations That Can Fail

**Rule:** Repository operations return `Result<T>` for operations that can fail (not found, concurrency violations, etc.).

```mermaid
classDiagram
    class IOrderRepository {
        <<Interface>>
        +GetById(id: OrderId) Result~Order~
        +Add(order: Order) Result~void~
        +Update(order: Order) Result~void~
        +Remove(order: Order) Result~void~
    }

    class Result {
        <<Result Pattern>>
        +IsSuccess: bool
        +IsError: bool
        +Error: string
        +Value: T
    }

    note for IOrderRepository "GetById returns Result<Order>\nnot found → Result.Error"
```

**Examples:**
- `GetById(id)` → `Result<Order>` (not found = error)
- `Update(order)` → `Result<void>` (concurrency violation = error)
- `Add(order)` → `Result<void>` (constraint violation = error)

## Query Patterns

### Specification Pattern

**Problem:** Repository interfaces become cluttered with specific query methods.

```mermaid
flowchart TB
    A[Repository] --> B[GetById]
    A --> C[GetByName]
    A --> D[GetByStatus]
    A --> E[GetByNameAndStatus]
    A --> F[GetByDateRange]
    A --> G[GetByCustomerAndStatusAndDate]

    style A fill:#f5e1e1
    style G fill:#f5e1e1
```

**Solution:** Encapsulate query logic in Specification objects.

```mermaid
classDiagram
    class ISpecification {
        <<Interface>> <<Domain Layer>>
        +ToExpression() Expression~Func~T,Boolean~~
        +IsSatisfied(entity: T) bool
    }

    class OverdueOrdersSpecification {
        <<Specification>> <<Domain Layer>>
        +ToExpression() Expression
        +IsSatisfied(order: Order) bool
    }

    class CustomerActiveOrdersSpecification {
        <<Specification>> <<Domain Layer>>
        -CustomerId: CustomerId
        +ToExpression() Expression
        +IsSatisfied(order: Order) bool
    }

    class IOrderRepository {
        <<Interface>> <<Domain Layer>>
        +Find(spec: ISpecification) Result~List~Order~~
        +FindOne(spec: ISpecification) Result~Order~
        +Count(spec: ISpecification) Result~int~
    }

    IOrderRepository ..> ISpecification : uses
```

**Benefits:**

- Encapsulates business rules in Domain layer
- Keeps repository interface stable
- Composable specifications
- Testable query logic

### Query vs. Command

**CQRS Principle:** Separate read and write models.

```mermaid
flowchart TB
    subgraph WriteSide["Command Side (Write Model)"]
        W1[Repository Interface]
        W2[Aggregate Root]
        W3[Domain Events]
    end

    subgraph ReadSide["Query Side (Read Model)"]
        R1[Query Object/DAO]
        R2[DTO/Read Model]
        R3[Denormalized Table]
    end

    W1 -->|Loads for modification| W2
    W2 -->|Emits| W3
    R1 -->|Reads from| R3
    R1 -->|Returns| R2

    style WriteSide fill:#e1f5e1
    style ReadSide fill:#fff4e1
```

**When to use:**

- **Repository Interface:** For transactional operations (writes)
- **Query Object/DAO:** For reporting and queries (reads) - not a domain concern

## Repository Interface Examples

### Example 1: Order Repository

```mermaid
classDiagram
    class IOrderRepository {
        <<Interface>> <<Domain Layer>>
        +Add(order: Order) Result~void~
        +Update(order: Order) Result~void~
        +Remove(order: Order) Result~void~
        +GetById(id: OrderId) Result~Order~
        +FindByCustomer(customerId: CustomerId) Result~List~Order~~
        +Find(spec: ISpecification) Result~List~Order~~
        +Exists(id: OrderId) Result~bool~
    }

    class Order {
        <<Aggregate Root>>
        +Id: OrderId
        +CustomerId: CustomerId
        +Status: OrderStatus
        +Create() Result~Order~
        +Confirm() Result
        +Cancel() Result
    }

    IOrderRepository ..> Order : manages
```

### Example 2: Customer Repository with Specifications

```mermaid
classDiagram
    class ICustomerRepository {
        <<Interface>> <<Domain Layer>>
        +Add(customer: Customer) Result~void~
        +Update(customer: Customer) Result~void~
        +GetById(id: CustomerId) Result~Customer~
        +GetByEmail(email: EmailAddress) Result~Customer~
        +Find(spec: ISpecification) Result~List~Customer~~
        +ExistsWithEmail(email: EmailAddress) Result~bool~
    }

    class ActiveCustomersSpecification {
        <<Specification>> <<Domain Layer>>
        +ToExpression() Expression
        +IsSatisfied(customer: Customer) bool
    }

    class PremiumCustomersSpecification {
        <<Specification>> <<Domain Layer>>
        -MinPurchaseAmount: Money
        +ToExpression() Expression
        +IsSatisfied(customer: Customer) bool
    }

    ICustomerRepository ..> ISpecification : uses
    ActiveCustomersSpecification ..|> ISpecification
    PremiumCustomersSpecification ..|> ISpecification
```

### Example 3: Repository with Domain Events

```mermaid
classDiagram
    class IOrderRepository {
        <<Interface>> <<Domain Layer>>
        +Add(order: Order) Result~void~
        +Update(order: Order) Result~void~
        +GetById(id: OrderId) Result~Order~
        +GetWithPendingEvents(id: OrderId) Result~Order~
    }

    class Order {
        <<Aggregate Root>>
        -DomainEvents: List~IDomainEvent~
        +Confirm() Result
        +GetUncommittedEvents() List~IDomainEvent~
        +MarkEventsAsCommitted() void
    }

    note for IOrderRepository "Repository loads/stores\naggregate with events"
```

## Common Anti-Patterns

### Repository for Internal Entities
**Problem:** Creating a repository for internal entities (e.g., `IOrderItemRepository`) bypasses aggregate root and invariant enforcement.
**Solution:** Access internal entities only through their aggregate root. Only aggregate roots have repositories.

### Database-Like Interface
**Problem:** Repository exposes database-specific methods like `ExecuteQuery`, `CallStoredProcedure`, `GetRawData`.
**Solution:** Use collection-like semantics (Add, Update, Remove, GetById, Find) that mimic in-memory collections.

### Not Returning Result for Failure Cases
**Problem:** Repository methods like `GetById(id: OrderId) Order` throw exceptions or return null for not-found cases.
**Solution:** Return `Result<T>` for operations that can fail, making failures explicit in the type signature.

## Summary Checklist

When designing repository interfaces in the domain layer:

- [ ] Interface defined in Domain layer
- [ ] One interface per Aggregate Root
- [ ] Collection-like semantics (Add, Update, Remove, GetById, Find)
- [ ] Return `Result<T>` for operations that can fail
- [ ] GetById returns `Result<T>` for not-found cases
- [ ] Use Specification pattern for complex queries
- [ ] No database-specific methods
- [ ] No repository for internal entities
- [ ] Enable testing with fake implementations
- [ ] Keep interface stable (implementation details hidden)

## Repository Interface vs Implementation

This guide focuses on **Repository Interfaces** as domain concerns.

**Repository Implementations** (SQL, NoSQL, in-memory, etc.) are **infrastructure concerns** and are not covered in this guide. When implementing:

- Define the interface in the Domain layer (this guide)
- Implement the interface in the Infrastructure layer
- Use dependency injection to provide the implementation
- Test domain logic with fake/test implementations
