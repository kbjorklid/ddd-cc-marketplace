# Aggregates

## Overview

An Aggregate is a cluster of associated objects (Entities and Value Objects) treated as a unit for data changes. It is the primary pattern for enforcing consistency boundaries in tactical DDD.

**Core components:**
- **Aggregate Root** - The single entry point and gatekeeper
- **Boundary** - Defines what is inside the consistency unit
- **Invariants** - Business rules that must always hold true

```mermaid
flowchart TB
    subgraph Aggregate["Aggregate Boundary"]
        direction TB
        Root["Aggregate Root\n(External access only)"]
        Internal["Internal Entities\n(No external access)"]
        VO["Value Objects\n(Immutable)"]
    end

    External["External World"] -->|Can only reference| Root
    Root -->|Contains| Internal
    Root -->|Contains| VO
    Internal -->|Contains| VO

    style Root fill:#e1f5e1,stroke:#333,stroke-width:3px
    style External fill:#f0f0f0
    style Aggregate fill:#fafafa,stroke:#333,stroke-width:2px
```

## The Four Rules of Aggregate Design

Vaughn Vernon's canonical rules from *Implementing Domain-Driven Design*:

### Rule 1: Model True Invariants in Consistency Boundaries

**Rule:** Group objects that must be consistent with each other immediately after a transaction.

```mermaid
flowchart TD
    A{Must two objects be\nconsistent immediately?}
    A -->|Yes| B[Same Aggregate]
    A -->|No| C[Different Aggregates]

    D[Order and OrderTotal] -->|Yes| B
    E[Order and Customer] -->|No| C

    style B fill:#e1f5e1
    style C fill:#e1f5e1
```

**Examples:**

| Objects | Same Aggregate? | Rationale |
|---------|-----------------|-----------|
| Order + OrderItems | Yes | Total must always equal sum of items |
| Order + Customer | No | Order can exist independently |
| Invoice + InvoiceItems | Yes | Invoice total invariant |
| Invoice + Payment | No | Payment is separate lifecycle |

**Implication:** Transaction boundaries should align with Aggregate boundaries. One transaction = one Aggregate instance modified.

### Rule 2: Design Small Aggregates

**Rule:** Keep Aggregates as small as possible—ideally just the Root and a few Value Objects.

```mermaid
flowchart TB
    subgraph Bad["❌ Mega-Aggregate (Anti-pattern)"]
        A[Order] --> B[Customer object]
        A --> C[Items list]
        A --> D[Shipments list]
        A --> E[Payments list]
        A --> F[Invoices list]
        A --> G[Reviews list]
    end

    subgraph Good["✅ Small Aggregate (Correct)"]
        H[Order] --> I[Items list]
        H --> J[CustomerId only]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Problems with Mega-Aggregates:**

- **Performance:** Loading thousands of child objects causes massive I/O
- **Concurrency:** High contention—any update locks entire graph
- **Complexity:** Difficult to reason about invariants
- **Distribution:** Cannot shard or scale independently

**Benefits of Small Aggregates:**

- **Fast loading:** Minimal database queries
- **Low contention:** Updates target specific roots
- **Clear boundaries:** Easy to understand responsibilities
- **Scalability:** Can distribute across services

### Rule 3: Reference Other Aggregates by Identity Only

**Rule:** Aggregates should hold only the ID (not object reference) of other Aggregate Roots.

**Why identity-only references:**

1. **Decoupling:** Prevents massive connected object graphs
2. **Memory Management:** Enables garbage collection
3. **Distribution:** Referenced aggregate can be in different database/service
4. **Lazy Loading:** Avoids accidental N+1 query problems
5. **Consistency:** Clear boundary of what's inside/outside

**Implementation:**

```mermaid
classDiagram
    %% Order Aggregate
    class Order {
        <<Aggregate Root>>
        -Id: OrderId
        -CustomerId: CustomerId  %% Value Object, not Customer entity
        -Items: List~OrderItem~
        +AddItem()
        +Confirm()
    }

    %% Customer Aggregate (separate)
    class Customer {
        <<Aggregate Root>>
        -Id: CustomerId
        -Name: String
        -Email: EmailAddress
    }

    class CustomerId {
        <<Value Object>>
        -Value: String
    }

    Order "1" *-- "1" CustomerId : references by ID
    Customer "1" *-- "1" CustomerId : identified by
```

### Rule 4: Use Eventual Consistency Outside the Boundary

**Rule:** When operations span multiple Aggregates, use Domain Events for eventual consistency rather than atomic transactions.

```mermaid
sequenceDiagram
    participant U as User
    participant OA as Order Aggregate
    participant DB as Database
    participant EB as Event Bus
    participant CA as Customer Aggregate

    U->>OA: Place Order
    OA->>OA: Validate business rules
    OA->>OA: Create Order
    OA->>DB: Save Order (transaction)
    OA->>EB: Publish OrderPlaced event
    Note over OA,EB: Transaction commits here

    EB->>CA: Consume OrderPlaced event
    CA->>CA: Update customer stats
    CA->>DB: Save Customer (separate transaction)

    Note over OA,CA: Orders and Customers are eventually consistent
```

**Benefits of Eventual Consistency:**

- **Scalability:** No distributed transactions (2PC)
- **Performance:** No locking across aggregates
- **Resilience:** Message queues handle failures
- **Decoupling:** Aggregates evolve independently

**Trade-offs:**

- **Complexity:** Must handle async messaging
- **Consistency window:** Data may be stale temporarily
- **Idempotency:** Consumers must handle duplicate messages

## Aggregate Structure

### Components

```mermaid
classDiagram
    class AggregateRoot {
        <<Aggregate Root>>
        -Id: AggregateId
        -Version: Int
        +GetId() AggregateId
        +GetVersion() Int
        +BehaviorMethod() Result
        +GetDomainEvents() List~DomainEvent~
        +ClearDomainEvents()
    }

    class Entity {
        <<Entity>>
        -Id: EntityId
        +GetId() EntityId
    }

    class ValueObject {
        <<Value Object>>
        -Field1: Type1
        -Field2: Type2
    }

    class DomainEvent {
        <<Domain Event>>
        -OccurredAt: DateTime
        +GetOccurredAt() DateTime
    }

    AggregateRoot "1" *-- "*" Entity : contains
    AggregateRoot "1" *-- "*" ValueObject : uses
    AggregateRoot "0..*" --> "*" DomainEvent : generates
```

**Rules for each component:**

- **Aggregate Root:**
  - Only entity accessible from outside aggregate
  - Holds reference to internal entities
  - Enforces all invariants
  - Generates domain events
  - Has repository

- **Internal Entities:**
  - Accessible only through Root
  - No external repositories
  - Mutable behavior encapsulated by Root

- **Value Objects:**
  - Immutable descriptors
  - Shared within aggregate
  - No identity

- **Domain Events:**
  - Record of significant state changes
  - Published after transaction commits

## Aggregate Lifecycle

### Creation

```mermaid
flowchart TD
    A[Create Aggregate] --> B{Complex creation?}
    B -->|Simple| C[Static factory on Root]
    B -->|Complex| D[Separate Factory class]

    C --> E[Validate invariants]
    D --> E
    E --> F{Valid?}
    F -->|Yes| G[Return valid Aggregate]
    F -->|No| H[Throw exception / Return error]

    style G fill:#e1f5e1
    style H fill:#f5e1e1
```

**Creation strategies:**

1. **Constructor** - Simple aggregates with few dependencies
2. **Static Factory Method** - Common approach, self-contained
3. **Factory Class** - Complex creation with external dependencies

### Modification

```mermaid
flowchart LR
    A[Client request] --> B[Load Aggregate]
    B --> C[Call behavior method]
    C --> D[Root validates invariant]
    D --> E{Valid?}
    E -->|Yes| F[Modify state]
    E -->|No| G[Return Result.Error]
    F --> H[Generate Domain Event]
    H --> I[Return Result.Success]
    I --> J[Save via Repository]

    style F fill:#e1f5e1
    style G fill:#f5e1e1
    style I fill:#e1f5e1
```

**Result pattern for aggregate modifications:**

- **Behavior methods return `Result`** - Operations that can fail return Result type
- **Invariant violations** - Return `Result.Error` with descriptive message
- **State transitions** - Return `Result` if transition may be invalid
- **Success** - Return `Result.Success` after valid state change

**Key principles:**

- All modifications go through Root
- Behavior methods (not setters) return Result
- Invariants checked before modification
- Validation failures return Result.Error, not exceptions
- Domain events generated for significant changes
- Repository persists entire Aggregate

### Deletion

```mermaid
flowchart TD
    A{Delete Aggregate?}
    A -->|Audit required| B[Soft Delete / Archive]
    A -->|No audit needed| C[Hard Delete]

    B --> D[Set status to Archived]
    C --> E[DELETE from database]

    style D fill:#e1f5e1
    style E fill:#e1f5e1
```

**Approaches:**

- **Soft Delete** - Set status to Archived/Deleted (preferred for audit)
- **Hard Delete** - Actual database row removal (rare)
- **Archive** - Move to separate table/storage for compliance

## Repository Pattern

### One Repository Per Aggregate Root

```mermaid
flowchart TB
    subgraph Domain["Domain Layer"]
        AR[Aggregate Root Interface]
    end

    subgraph Infrastructure["Infrastructure Layer"]
        ARImpl[Aggregate Root Repository]
        DB[(Database)]
    end

    subgraph Application["Application Layer"]
        AS[Application Service]
    end

    AS -->|depends on| AR
    AR -->|implemented by| ARImpl
    ARImpl -->|accesses| DB

    style Domain fill:#e1f5e1
    style Infrastructure fill:#fff4e1
    style Application fill:#e1f5e1
```

**Rules:**

- Only Aggregate Roots have repositories
- Internal entities never have repositories
- Repository interface in Domain layer
- Repository implementation in Infrastructure layer
- Repository simulates in-memory collection

**Repository operations:**

```mermaid
classDiagram
    class IOrderRepository {
        <<Interface>>
        +Add(order: Order) Result
        +Update(order: Order) Result
        +Remove(order: Order) Result
        +GetById(id: OrderId) Result~Order~
        +Find(spec: Specification) List~Order~
        +Exists(id: OrderId) Boolean
    }
    note for IOrderRepository "Domain layer abstraction\nReturns Result for operations that can fail"
```

## Aggregate Design Examples

### Example 1: Order Aggregate (Correct)

```mermaid
classDiagram
    %% Order Aggregate - small and focused
    class Order {
        <<Aggregate Root>>
        -Id: OrderId
        -CustomerId: CustomerId
        -Status: OrderStatus
        -Items: List~OrderItem~
        -ShippingAddress: Address
        -Version: Int
        +AddItem(productId, quantity, price) Result
        +RemoveItem(itemId) Result
        +Confirm() Result
        +MarkAsPaid() Result
        +Ship(trackingNumber) Result
    }
    class OrderItem {
        <<Entity>>
        -Id: OrderItemId
        -ProductId: ProductId
        -Quantity: Quantity
        -UnitPrice: Money
        +GetTotal() Money
    }
    class OrderId {
        <<Value Object>>
        -Value: String
    }
    class CustomerId {
        <<Value Object>>
        -Value: String
    }
    class Address {
        <<Value Object>>
        -Street: String
        -City: String
        -PostalCode: String
    }

    Order "1" *-- "*" OrderItem : contains
    Order "1" *-- "1" CustomerId : references by ID
    Order "1" *-- "1" Address : contains
```

**Why this is correct:**
- Small aggregate (Root + one entity type)
- Customer referenced by ID only
- Clear behavior methods
- Enforces invariants (e.g., cannot confirm without items)
- Version field for optimistic concurrency

### Example 2: Mega-Aggregate Anti-Pattern

```mermaid
classDiagram
    %% ANTI-PATTERN: Mega-Aggregate
    class Order {
        <<Aggregate Root>>
        -Id: OrderId
        -Customer: Customer  %% WRONG: Direct object reference
        -Items: List~OrderItem~
        -Shipments: List~Shipment~  %% WRONG: Too many entities
        -Payments: List~Payment~  %% WRONG: Separate lifecycle
        -Invoices: List~Invoice~  %% WRONG: Separate aggregate
        -Reviews: List~Review~  %% WRONG: Separate aggregate
    }
    class Customer {
        <<Entity>>
        -OrderHistory: List~Order~  %% WRONG: Circular reference
    }
    class Shipment {
        <<Entity>>
        -Items: List~ShipmentItem~
    }
    class Payment {
        <<Entity>>
        -Transactions: List~Transaction~
    }

    Order "1" *-- "*" Customer : owns
    Order "1" *-- "*" OrderItem : owns
    Order "1" *-- "*" Shipment : owns
    Order "1" *-- "*" Payment : owns
    Order "1" *-- "*" Invoice : owns
    Order "1" *-- "*" Review : owns
```

**Problems:**
- Loading Order loads entire Customer history
- Updating Payment locks entire Order
- Violates single responsibility
- Performance nightmare
- Concurrency conflicts

### Example 3: Correct Split Design

```mermaid
classDiagram
    %% Order Aggregate
    class Order {
        <<Aggregate Root>>
        -Id: OrderId
        -CustomerId: CustomerId
        -Items: List~OrderItem~
        +AddItem()
        +Confirm()
    }

    %% Payment Aggregate (separate)
    class Payment {
        <<Aggregate Root>>
        -Id: PaymentId
        -OrderId: OrderId
        -Amount: Money
        -Status: PaymentStatus
        +Process()
        +Refund()
    }

    %% Shipment Aggregate (separate)
    class Shipment {
        <<Aggregate Root>>
        -Id: ShipmentId
        -OrderId: OrderId
        -Address: Address
        -Status: ShipmentStatus
        +Ship()
        +Deliver()
    }

    Order --> CustomerId : references by ID
    Payment --> OrderId : references by ID
    Shipment --> OrderId : references by ID

    note for Payment "Separate aggregate\nUpdated via domain events"
    note for Shipment "Separate aggregate\nUpdated via domain events"
```

**Benefits:**
- Each aggregate has single responsibility
- Payment can be updated without locking Order
- Shipment can be handled by different service
- Clear transaction boundaries
- Independent scalability

## Aggregate Invariants

### Documenting Invariants

**Template:**
| ID | Invariant | Enforcement Point | Notes |
|----|-----------|-------------------|-------|
| ORD-1 | Order must have at least one item to confirm | `Order.Confirm()` | Prevents empty orders |
| ORD-2 | Order total equals sum of item totals | `Order.AddItem()`, `Order.RemoveItem()` | Calculated, not stored |
| ORD-3 | Cannot modify items after confirmation | `Order.AddItem()`, `Order.RemoveItem()` | Locks order state |

**Enforcement strategies:**

1. **Method guards** - Check at start of behavior method
2. **Constructor validation** - Check on creation
3. **State machine** - Enforce valid transitions
4. **Domain events** - Trigger cross-aggregate validation

### Example Invariants

```mermaid
flowchart TD
    A[Order.Confirm] --> B{Has items?}
    B -->|No| C[Throw exception]
    B -->|Yes| D{Address valid?}
    D -->|No| C
    D -->|Yes| E[Set status to Confirmed]
    E --> F[Publish OrderConfirmed event]

    style C fill:#f5e1e1
    style E fill:#e1f5e1
    style F fill:#e1f5e1
```

## Advanced Concepts

### Dynamic Consistency Boundaries

**Challenge:** Some business rules span multiple aggregates but require immediate validation.

**Traditional approach:** Mega-aggregates (bad)

**Proposed solution:** Use event store to validate invariants before appending events.

```mermaid
flowchart LR
    A[Validate rule] --> B{Query event store}
    B --> C[Check pure events]
    C --> D{Invariant holds?}
    D -->|Yes| E[Append new event]
    D -->|No| F[Reject operation]

    style E fill:#e1f5e1
    style F fill:#f5e1e1
```

### Aggregate vs. Bounded Context

```mermaid
flowchart TB
    subgraph BC["Bounded Context"]
        direction TB
        subgraph A1["Order Aggregate"]
            O[Order]
            OI[OrderItems]
        end
        subgraph A2["Customer Aggregate"]
            C[Customer]
        end
        subgraph A3["Inventory Aggregate"]
            I[Inventory]
        end
    end

    style BC fill:#fafafa,stroke:#333,stroke-width:2px
    style A1 fill:#e1f5e1
    style A2 fill:#e1f5e1
    style A3 fill:#e1f5e1
```

**Relationship:**
- Bounded Context = Large-scale boundary (strategic DDD)
- Aggregate = Small-scale consistency boundary (tactical DDD)
- One Bounded Context contains multiple Aggregates
- Each Aggregate has one Root

## Summary Checklist

When reviewing a DOMAIN.md for Aggregate compliance, ask:

- [ ] Is the aggregate small (Root + few entities/VOs)?
- [ ] Are consistency boundaries clearly defined?
- [ ] Is access restricted to the Aggregate Root?
- [ ] Do references to other aggregates use IDs only?
- [ ] Is there one repository interface per Aggregate Root?
- [ ] Are all invariants documented and enforced?
- [ ] Are there behavior methods instead of public setters?
- [ ] Do behavior methods return `Result` for operations that can fail?
- [ ] Do Create methods return `Result<T>` with validation?
- [ ] Do repository methods return `Result` for operations that can fail?
- [ ] Are domain events used for significant changes?
- [ ] Is there a concurrency control strategy defined?
- [ ] Is eventual consistency used for cross-aggregate operations?
- [ ] Is there a version field for optimistic locking?
- [ ] Are error messages in Result.Error returns descriptive?
