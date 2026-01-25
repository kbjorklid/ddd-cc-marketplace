# Domain Events

## Overview

A Domain Event is a record of something significant that happened in the domain. It represents a completed action or state change that other parts of the system may need to react to.

**Core characteristics:**
- **Immutable** - Historical fact that cannot be changed
- **Past-tense naming** - Describes what happened (e.g., `OrderPlaced`, not `PlaceOrder`)
- **Minimal data** - Contains only what consumers need
- **Publish-subscribe** - Decouples producers from consumers

```mermaid
flowchart LR
    A[Order Aggregate] -->|Publishes| B[OrderPlaced Event]
    B -->|Consumed by| C[Email Service]
    B -->|Consumed by| D[Inventory Service]
    B -->|Consumed by| E[Analytics Service]

    C --> F[Sends confirmation email]
    D --> G[Reserves inventory]
    E --> H[Updates metrics]

    style B fill:#e1f5e1
```

## Purpose and Benefits

### Decoupling Side Effects

**Problem:** Business operations often trigger side effects (emails, notifications, external system updates).

**Without Domain Events (Tightly Coupled):**

```mermaid
flowchart TD
    A[Order.Confirm] --> B[Validate Order]
    B --> C[Update Status]
    C --> D[Send Email Service]
    D --> E[Update Inventory Service]
    E --> F[Update Analytics Service]
    F --> G[Log to Audit Service]

    style A fill:#f5e1e1
    style C fill:#f5e1e1
    style D fill:#f5e1e1
    style E fill:#f5e1e1
    style F fill:#f5e1e1
    style G fill:#f5e1e1
```

**With Domain Events (Loosely Coupled):**

```mermaid
flowchart TD
    A[Order.Confirm] --> B[Validate Order]
    B --> C[Update Status]
    C --> D[Publish OrderConfirmed Event]
    D --> E[Message Bus]

    E --> F[Email Handler]
    E --> G[Inventory Handler]
    E --> H[Analytics Handler]
    E --> I[Audit Handler]

    style A fill:#e1f5e1
    style C fill:#e1f5e1
    style D fill:#e1f5e1
    style E fill:#fff4e1
```

**Benefits:**

- **Single Responsibility** - Aggregate focuses on core logic only
- **Open/Closed Principle** - Add new consumers without modifying producer
- **Testability** - Test aggregate in isolation without side effects
- **Resilience** - Message queues buffer consumers if they're down

### Eventual Consistency Across Aggregates

**Challenge:** Business operations often span multiple aggregates.

**Anti-Pattern:** Distributed transaction (2PC):

```mermaid
flowchart TD
    A[Begin Transaction] --> B[Update Order]
    B --> C[Update Customer]
    C --> D[Update Inventory]
    D --> E{All succeed?}
    E -->|Yes| F[Commit Transaction]
    E -->|No| G[Rollback Transaction]

    style F fill:#e1f5e1
    style G fill:#f5e1e1
```

**Problems:**
- Locks multiple resources simultaneously
- Reduces throughput
- Doesn't scale across microservices
- Complex failure handling

**With Domain Events (Eventual Consistency):**

```mermaid
sequenceDiagram
    participant OA as Order Aggregate
    participant DB as Database
    participant EB as Event Bus
    participant CA as Customer Aggregate

    OA->>OA: Place Order
    OA->>DB: Save Order (transaction 1)
    OA->>EB: Publish OrderPlaced

    Note over OA,EB: Transaction 1 commits

    EB->>CA: Handle OrderPlaced
    CA->>CA: Update customer stats
    CA->>DB: Save Customer (transaction 2)

    Note over CA,DB: Transaction 2 commits\n(eventually consistent)
```

**Benefits:**
- Each aggregate updates in its own transaction
- No distributed locks
- Scales across microservices
- Message queues provide reliability

## Event Design

### Naming Conventions

**Rule:** Use past tense to indicate completed action.

```mermaid
flowchart LR
    subgraph Bad["❌ Poor Names"]
        A1[PlaceOrder]
        A2[Confirm]
        A3[ProcessPayment]
    end

    subgraph Good["✅ Good Names"]
        B1[OrderPlaced]
        B2[OrderConfirmed]
        B3[PaymentProcessed]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Examples:**

| Context | Event Name |
|---------|------------|
| Customer registers | `CustomerRegistered` |
| Order placed | `OrderPlaced` |
| Payment authorized | `PaymentAuthorized` |
| Payment captured | `PaymentCaptured` |
| Order shipped | `OrderShipped` |
| Inventory reserved | `InventoryReserved` |
| Email sent | `EmailSent` |

### Event Structure

```mermaid
classDiagram
    class DomainEvent {
        <<Abstract Base>>
        #EventId: EventId
        #AggregateId: AggregateId
        #OccurredAt: DateTime
        #Version: Int
        +GetEventId() EventId
        +GetAggregateId() AggregateId
        +GetOccurredAt() DateTime
        +GetVersion() Int
    }

    class OrderPlaced {
        <<Domain Event>>
        -OrderId: OrderId
        -CustomerId: CustomerId
        -TotalAmount: Money
        -Items: List~OrderItemPlaced~
        -ShippingAddress: Address
        +OrderPlaced(orderId, customerId, total, items, address)
    }

    class PaymentAuthorized {
        <<Domain Event>>
        -PaymentId: PaymentId
        -OrderId: OrderId
        -Amount: Money
        -PaymentMethod: PaymentMethod
        -AuthorizedAt: DateTime
    }

    DomainEvent <|-- OrderPlaced
    DomainEvent <|-- PaymentAuthorized
```

**Standard fields:**

| Field | Purpose | Example |
|-------|---------|---------|
| `EventId` | Unique identifier for this event instance | `550e8400-...` |
| `AggregateId` | ID of the aggregate that emitted the event | `Order("123")` |
| `AggregateType` | Type name of the aggregate | `"Order"` |
| `EventType` | Type name of the event | `"OrderPlaced"` |
| `OccurredAt` | Timestamp when the event occurred | `2025-01-15T10:30:00Z` |
| `Version` | Optimistic concurrency version | `1` |

### Minimal Data Principle

**Rule:** Include only the data consumers need, not the entire aggregate state.

```mermaid
flowchart TB
    subgraph Excessive["❌ Too Much Data"]
        A1[OrderPlaced Event]
        A1 --> A2[Entire Order object]
        A2 --> A3[All OrderItems]
        A2 --> A4[Customer object]
        A2 --> A5[Product objects]
    end

    subgraph Minimal["✅ Minimal Data"]
        B1[OrderPlaced Event]
        B1 --> B2[OrderId]
        B1 --> B3[CustomerId]
        B1 --> B4[TotalAmount]
        B1 --> B5[Item summaries (ID, qty, price)]
    end

    style Excessive fill:#f5e1e1
    style Minimal fill:#e1f5e1
```

**Guidelines:**

- Include aggregate ID (for querying if needed)
- Include fields that changed
- Include identifiers for referenced aggregates
- Don't include unchanged fields
- Don't include entire child object graphs

**Example: OrderPlaced event**

```mermaid
classDiagram
    class OrderPlaced {
        <<Domain Event>>
        +OrderId: OrderId
        +CustomerId: CustomerId
        +TotalAmount: Money
        +Items: List~OrderItemData~
        +ShippingAddress: Address
        +OccurredAt: DateTime
    }
    class OrderItemData {
        <<Data Transfer Object>>
        +ProductId: ProductId
        +Quantity: int
        +UnitPrice: Money
    }
    OrderPlaced "1" *-- "*" OrderItemData : contains
```

## Event Generation

### When to Emit Events

**Criteria for significance:**

- State change that other systems care about
- Business milestone reached
- Decision point in a process
- Error or exception condition
- Timeout or SLA breach

**Examples:**

| Aggregate State Change | Emit Event? | Reason |
|------------------------|-------------|--------|
| Order created | Yes (`OrderPlaced`) | Triggers inventory, email |
| Order item added | No | Internal to aggregate |
| Order confirmed | Yes (`OrderConfirmed`) | Triggers payment flow |
| Payment status updated | Yes (`PaymentAuthorized`) | Triggers fulfillment |
| Customer address changed | Yes (`CustomerAddressChanged`) | Triggers shipping update |

### Event Generation Pattern

```mermaid
flowchart TD
    A[Call behavior method] --> B[Validate business rules]
    B --> C{Valid?}
    C -->|No| D[Return Result.Error]
    C -->|Yes| E[Modify state]
    E --> F{Significant change?}
    F -->|Yes| G[Create Domain Event]
    F -->|No| H[Return Result.Success]
    G --> I[Add to event collection]
    I --> H
    D --> J[Caller handles error]
    H --> K[Caller processes success]

    style G fill:#e1f5e1
    style I fill:#e1f5e1
    style H fill:#e1f5e1
```

**Implementation with Result pattern:**

```mermaid
classDiagram
    class AggregateRoot {
        <<Abstract Base>>
        #_DomainEvents: List~DomainEvent~
        +GetDomainEvents() List~DomainEvent~
        +ClearDomainEvents() void
        +AddDomainEvent(event: DomainEvent) void
    }

    class Order {
        <<Aggregate Root>>
        +Confirm() Result {
            if (!CanConfirm()) {
                return Result.Error("Order cannot be confirmed: has no items")
            }

            SetStatus(OrderStatus.Confirmed)
            AddDomainEvent(OrderConfirmed(Id, CustomerId, Total))

            return Result.Success
        }
    }

    AggregateRoot <|-- Order
```

### Event vs. State

**Principle:** Events are facts; state is current reality.

```mermaid
flowchart LR
    A[Event Stream] --> B[Current State]
    B --> C[Event Stream]

    subgraph History["Event Store"]
        E1[OrderPlaced]
        E2[OrderConfirmed]
        E3[PaymentAuthorized]
        E4[OrderShipped]
    end

    subgraph Current["Current State"]
        S1[Order<br/>Status: Shipped<br/>Items: ...]
    end

    E1 --> E2 --> E3 --> E4
    E4 -.->|Rebuild| S1

    style History fill:#e1f5e1
    style Current fill:#fff4e1
```

**Event Sourcing pattern:** Store events, derive state on read.

## Common Anti-Patterns

### Event Explosion
**Problem:** Emitting events for every minor state change (e.g., `OrderCreated`, `OrderStatusChangedToDraft`, `OrderItemsCollectionModified`) creates event noise.
**Solution:** Emit events only for significant business state changes that other systems need to react to.

### Event God Object
**Problem:** A generic `OrderChanged` event containing entire Order state, old state, new state, user who changed, timestamp, and change reason.
**Solution:** Use specific event types with minimal data (e.g., `OrderPlaced`, `PaymentAuthorized`) containing only what consumers need.

### Synchronous Event Handlers
**Problem:** Aggregate emits event and immediately calls synchronous handler that invokes external service, blocking the aggregate and coupling it to application concerns.
**Solution:** Emits domain events but delegates handling to external consumers.

## Summary Checklist

When designing Domain Events, ensure:

- [ ] Past-tense naming (OrderPlaced, not PlaceOrder)
- [ ] Immutable (events are facts)
- [ ] Minimal data (only what consumers need)
- [ ] Standard metadata (EventId, AggregateId, OccurredAt)
- [ ] Clear correlation (events reference aggregate IDs)
