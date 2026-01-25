# Factories

## Overview

Factories encapsulate the creation logic of complex objects, ensuring Aggregates are born in a valid state. While simple Aggregates can use static factory methods on the Aggregate Root, complex creation scenarios benefit from dedicated Factory classes.

**Core principles:**
- **Separation of concerns** - Creation logic separated from representation
- **Always valid** - Objects cannot be created in invalid state (uses Result pattern)
- **Encapsulation** - Complex creation details hidden from clients
- **Dependency injection** - Factories can use Domain Services during creation

```mermaid
flowchart TB
    A[Need to create Aggregate] --> B{Simple creation?}
    B -->|Yes| C[Static factory on Root]
    B -->|No| D[Separate Factory class]
    C --> E[Validate in constructor]
    D --> F[Use Domain Services]
    E --> G[Return Result.Success]
    F --> G
    C --> H[Return Result.Error]
    D --> H

    style G fill:#e1f5e1
    style H fill:#f5e1e1
```

## When to Use Factories

**Use a Factory when:**
- Creation logic is complex (multiple steps, validations)
- Aggregate requires data from external services
- Multiple ways to create the same object (different constructors)
- Creation involves other domain objects
- You need to hide complex creation logic

**Don't use a Factory when:**
- Simple constructor is sufficient
- Object has only one way to be created
- Creation doesn't require domain logic

## Factory Methods

### Static Factory Methods on Aggregate Root

**Pattern:** Static `Create` method on the Aggregate Root that returns `Result<Aggregate>`.

```mermaid
classDiagram
    class Order {
        <<Aggregate Root>>
        -Id: OrderId
        -CustomerId: CustomerId
        -Items: List~OrderItem~
        -Status: OrderStatus
        -Order(id, customerId, items, status)
        +Create(customerId: CustomerId, items: List~OrderItemDto~) Result~Order~
        +AddItem() Result
        +Confirm() Result
    }

    note for Order "Private constructor\nPublic Create factory method\nReturns Result for validation failures"
```

**Benefits:**
- Creation logic co-located with entity
- Access to private constructor
- Self-contained
- Returns `Result<T>` for validation failures

**Example with Result pattern:**

```mermaid
flowchart TD
    A[Order.Create] --> B{CustomerId valid?}
    B -->|No| C[Return Result.Error]
    B -->|Yes| D{Items not empty?}
    D -->|No| C
    D -->|Yes| E{Total > 0?}
    E -->|No| C
    E -->|Yes| F[Create Order with private constructor]
    F --> G[Return Result.Success with Order]

    style C fill:#f5e1e1
    style G fill:#e1f5e1
```

**Implementation pattern:**
1. Private constructor ensures only factory can create
2. All validations in `Create` method
3. Return `Result.Error` with descriptive message for invalid input
4. Return `Result.Success` with instance for valid input

## Factory Classes

### When to Use Separate Factory Classes

**Use a Factory Class when:**
- Multiple creation strategies for same type
- Creation is a domain concept in itself
- Creation logic is complex and would clutter the entity

**Example concept:** An Order may be created from different sources (Quote, Cart, or Reorder), each requiring different validation logic. A Factory encapsulates these distinct creation strategies while keeping each focused and testable.

**Key aspects:**
- Multiple creation methods for different scenarios
- Validates business rules before creating entity
- Returns `Result<T>` for validation failures
- Factory methods are the only way to create instances (private constructor)

## Creation vs. Construction

### Constructor
- Simple initialization
- No business logic
- Minimal validation (only invariant checks)

### Factory
- Complex creation logic
- Multiple creation strategies
- Comprehensive validation with Result pattern

```mermaid
flowchart TB
    subgraph Constructor["Constructor (Private)"]
        C1[Initialize fields]
        C2[Check invariants]
    end

    subgraph Factory["Factory Method (Public)"]
        F1[Validate business rules]
        F2[Call constructor]
        F3[Return Result]
    end

    Factory -->|Calls| Constructor

    style Factory fill:#e1f5e1
    style Constructor fill:#fff4e1
```

## Common Anti-Patterns

### Factory as Entity Manager
**Problem:** Factory doing too much - creating, validating, persisting to repository, publishing events, and sending emails.
**Solution:** Factory should only create objects in valid state.

### Bypassing Validation
**Problem:** Factory creates objects in invalid state when input validation fails, breaking the "Always Valid" principle.
**Solution:** Always return `Result.Error` for invalid input; never create invalid objects.

### Factory Gods
**Problem:** One generic factory creates all domain objects (CreateOrder, CreateCustomer, CreateProduct, etc.).
**Solution:** Use separate factories per aggregate or factory methods on entities.

## Summary Checklist

When designing Factories, ensure:

- [ ] Creation logic separated from representation
- [ ] Factory methods return `Result<T>` for validation failures
- [ ] Private constructor on entity (public factory)
- [ ] Invalid objects cannot be created
- [ ] Error messages are descriptive
- [ ] Creation strategies are clear and focused
