# Policies

## Overview

A **Policy** is an object that encapsulates a specific business rule, algorithm, or decision-making process. By isolating varying logic from the domain objects that use it, Policies clarify the main domain object's responsibility and make the specific rule explicit and testable.

In Domain-Driven Design, we prefer the term "Policy" over the technical "Strategy Pattern" because it emphasizes the business rule or motivation behind the logic rather than the implementation mechanism.

**Core characteristics:**
- **Encapsulates algorithms** - Separates the rule from the behavior governed by that rule
- **Explicit business concept** - Named using ubiquitous language
- **Interchangeable** - Multiple variations of a rule can be substituted
- **Immutable** - Policies are typically value objects or stateless services
- **Testable** - Complex rules isolated in one place

```mermaid
flowchart LR
    subgraph Problem["Complex Conditional Logic"]
        P1[Entity with if-else chains]
        P2[Multiple algorithms embedded]
        P3[Business rules hidden in code]
    end

    subgraph Solution["Policy Pattern"]
        S1[Domain Object]
        S2[Delegates to Policy]
        S3[Explicit business rule]
    end

    P1 --> S1
    P2 --> S2
    P3 --> S3

    style Solution fill:#e1f5e1
```

**Policy vs Specification:**

While similar, they serve different purposes:

| Pattern | Purpose | Returns | Example |
|---------|---------|---------|---------|
| **Specification** | Predicate evaluation | Boolean | `IsSatisfied(customer)` - does this meet criteria? |
| **Policy** | Execute algorithm/process | Value/Result | `CalculatePrice(order)` - how should this be done? |

```mermaid
flowchart TB
    subgraph Spec["Specification Pattern"]
        S1[Question: Is this valid?]
        S2[Answer: Yes/No]
        S3[Use: Query/Validation]
    end

    subgraph Policy["Policy Pattern"]
        P1[Question: How should this work?]
        P2[Answer: Executed algorithm]
        P3[Use: Calculation/Decision]
    end

    style Spec fill:#fff4e1
    style Policy fill:#e1f5e1
```

## When to Use Policies

### Decision Framework

```mermaid
flowchart TD
    A[Domain Logic] --> B{Does it belong in one entity?}
    B -->|Yes| C[Entity Method]
    B -->|No| D{Is it a simple predicate?}

    D -->|Yes - boolean check| E[Specification]
    D -->|No - algorithm/process| F{Does the rule vary?}

    F -->|Yes - multiple ways| G[Policy]
    F -->|No - single rule| H{Is it complex?}

    H -->|Yes - obscures entity| G
    H -->|No - simple logic| I[Keep in Domain Service]

    style C fill:#e1f5e1
    style E fill:#fff4e1
    style G fill:#e1f5e1
    style I fill:#e1f5e1
```

### Use Cases

| Scenario | Example | Why Policy helps |
|----------|---------|------------------|
| **Varying algorithms** | Pricing: Standard, Discount, VIP rules | Swap algorithm without changing domain objects |
| **Obscured domain logic** | Entity swamped with `if-else` for different rules | Clarifies main object's responsibility |
| **Independent evolution** | Rule changes frequently independently of object | Prevents constant "surgery" on stable objects |
| **EventStorming automation** | Policy listens to Domain Event, decides which Command to execute | `PaymentFailed` → `RetryPayment` command |
| **Complex calculations** | Business priority formula with multiple factors | Centralizes calculation logic |

### Policy vs Domain Service

```mermaid
flowchart TB
    subgraph DS["Domain Service"]
        DS1[Operation spanning aggregates]
        DS2[Stateless business logic]
        DS3[One implementation]
    end

    subgraph Policy["Policy"]
        P1[Encapsulated algorithm]
        P2[Multiple variations]
        P3[Injected into operation]
    end

    style DS fill:#fff4e1
    style Policy fill:#e1f5e1
```

| Aspect | Domain Service | Policy |
|--------|---------------|--------|
| **Purpose** | Stateless operation | Encapsulated algorithm |
| **Variations** | Typically one implementation | Multiple interchangeable implementations |
| **Usage** | Called directly | Injected as parameter |
| **State** | Stateless | Immutable/stateless |
| **Example** | `FundsTransferService` | `OverbookingPolicy`, `RoutingPolicy` |

## Core Rules

### Rule 1: Policies Encapsulate Varying Business Rules

**Rule:** Extract logic into a Policy when an operation can be executed in multiple ways.

```mermaid
classDiagram
    class IPricingPolicy {
        <<Interface>>
        <<Policy>>
        +Calculate(order: Order) Money
    }

    class StandardPricingPolicy {
        <<Policy>>
        +Calculate(order: Order) Money
    }

    class DiscountPricingPolicy {
        <<Policy>>
        -DiscountPercentage: decimal
        +Calculate(order: Order) Money
    }

    class VipPricingPolicy {
        <<Policy>>
        -LoyaltyBonus: Money
        +Calculate(order: Order) Money
    }

    class Order {
        <<Aggregate Root>>
        +CalculatePrice(policy: IPricingPolicy) Money
    }

    IPricingPolicy <|-- StandardPricingPolicy
    IPricingPolicy <|-- DiscountPricingPolicy
    IPricingPolicy <|-- VipPricingPolicy
    Order ..> IPricingPolicy : uses

    note for Order "Order delegates to\ninjected pricing policy"
```

**Key principles:**

- Policy defines **how** a calculation or decision is made
- Multiple implementations for different business scenarios
- Domain object delegates to policy, doesn't implement the rule

### Rule 2: Policies Use Ubiquitous Language

**Rule:** Policy names reflect business concepts, not technical implementation.

```mermaid
flowchart TB
    subgraph Bad["❌ Technical Names"]
        B1[OverbookingAlgorithm]
        B2[RoutingCalculation]
        B3[PricingStrategy]
    end

    subgraph Good["✅ Domain Names"]
        G1[OverbookingPolicy]
        G2[RoutingPolicy]
        G3[PricingPolicy]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Examples:**

| Business Concept | Policy Name |
|-----------------|-------------|
| Allow 10% overbooking on voyages | `OverbookingPolicy` |
| Find fastest vs cheapest route | `RoutingPolicy` |
| Calculate business priority | `BusinessPriorityCalculator` (or Policy) |
| Determine payment retry timing | `PaymentRetryPolicy` |

### Rule 3: Policies Are Injected as Parameters

**Rule:** Pass the Policy into the method that needs it, allowing the domain object to delegate the specific calculation or decision.

```mermaid
sequenceDiagram
    participant C as Client
    participant O as Order
    participant P as PricingPolicy

    C->>O: CalculatePrice(DiscountPricingPolicy)
    O->>P: Calculate(order)
    P-->>O: discountedPrice
    O-->>C: Result.Success(price)

    Note over P: Policy encapsulates\nthe calculation logic
```

**Injection patterns:**

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **Method parameter** | Policy varies per call | `order.CalculatePrice(policy)` |
| **Constructor parameter** | Policy fixed for object lifetime | `RoutingService(policy)` |
| **Domain Service dependency** | Policy coordinates across aggregates | `PricingService` uses `PricingPolicy` |

### Rule 4: Policies Can Be Value Objects or Services

**Rule:** Policies are often implemented as Value Objects (immutable configuration) or stateless Services.

```mermaid
classDiagram
    class OverbookingPolicy {
        <<Value Object>>
        -MaxOverbookingPercentage: decimal
        +IsAllowed(cargo: Cargo, voyage: Voyage) Boolean
    }

    class RoutingPolicy {
        <<Domain Service>>
        <<Policy>>
        +FindRoute(cargo: Cargo) Route
        -CalculateCost(route: Route) Money
        -CalculateTime(route: Route) Duration
    }

    class BusinessPriorityCalculator {
        <<Value Object / Policy>>
        +Calculate(value: Money, cost: Money, risk: Risk) PriorityScore
    }

    note for OverbookingPolicy "Value Object:\nImmutable configuration"
    note for RoutingPolicy "Domain Service:\nStateless algorithm"
    note for BusinessPriorityCalculator "Value Object:\nEncapsulates formula"
```

**When to use which:**

| Use Value Object when | Use Domain Service when |
|----------------------|-------------------------|
| Rule is simple data/configuration | Rule involves complex multi-step logic |
| No dependencies on external services | Requires repositories or ports |
| Pure calculation based on parameters | Needs domain data to make decisions |

### Rule 5: Policies in EventStorming

**Rule:** In process modeling, a Policy often listens to a Domain Event and determines which Command to execute next.

```mermaid
flowchart LR
    E[PaymentFailed Event] --> P[PaymentRetryPolicy]
    P --> C{Should retry?}

    C -->|Yes| R[RetryPayment Command]
    C -->|No| F[CancelPayment Command]

    style P fill:#e1f5e1
```

**EventStorming pattern:**

1. **Domain Event occurs** (e.g., `PaymentFailed`)
2. **Policy evaluates** (e.g., `PaymentRetryPolicy` checks retry count, amount)
3. **Command executes** (e.g., `RetryPayment` or `CancelPayment`)

## Policy Design Examples

### Example 1: Overbooking Policy (Validation)

**Scenario:** A shipping `Voyage` can accept cargo. The business allows 10% overbooking to account for cancellations.

```mermaid
classDiagram
    class Voyage {
        <<Aggregate Root>>
        +Id: VoyageId
        +Capacity: CargoCapacity
        +CurrentLoad: CargoWeight
        +BookCargo(cargo: Cargo, policy: OverbookingPolicy) Result
    }

    class OverbookingPolicy {
        <<Policy>>
        -MaxOverbookingPercentage: Percentage
        +IsAllowed(cargo: Cargo, voyage: Voyage) Boolean
    }

    class Cargo {
        <<Value Object>>
        +Weight: CargoWeight
        +Size: CargoSize
    }

    Voyage ..> OverbookingPolicy : uses

    note for OverbookingPolicy "Business rule:\nAllow booking if\nCurrentLoad + CargoWeight\n<= Capacity * (1 + MaxOverbooking%)"
```

**Business rule documented:**

> "A voyage may be overbooked by up to 10% to account for last-minute cancellations."

**Without Policy (anti-pattern):**
```typescript
// ❌ Hidden business rule
if (voyage.CurrentLoad + cargo.Weight > voyage.Capacity * 1.1) {
  return Result.Error("Exceeds capacity");
}
```

**With Policy:**
```typescript
// ✅ Explicit business rule
if (!policy.IsAllowed(cargo, voyage)) {
  return Result.Error("Exceeds overbooking policy limit");
}
```

### Example 2: Routing Policy (Algorithm Selection)

**Scenario:** A shipping system needs to find routes for cargo. Different optimization goals: "Fastest Route," "Cheapest Route," or "Avoid Restricted Areas."

```mermaid
classDiagram
    class IRoutingPolicy {
        <<Interface>>
        <<Policy>>
        +FindRoute(cargo: Cargo, availableRoutes: Routes) Route
    }

    class FastestRoutePolicy {
        <<Policy>>
        +FindRoute(cargo: Cargo, availableRoutes: Routes) Route
        -CalculateTransitTime(route: Route) Duration
    }

    class CheapestRoutePolicy {
        <<Policy>>
        +FindRoute(cargo: Cargo, availableRoutes: Routes) Route
        -CalculateCost(route: Route) Money
    }

    class RestrictedAreaAvoidancePolicy {
        <<Policy>>
        -RestrictedAreas: List~Area~
        +FindRoute(cargo: Cargo, availableRoutes: Routes) Route
    }

    class RoutingService {
        <<Domain Service>>
        +RouteCargo(cargo: Cargo, policy: IRoutingPolicy) Result~Route~
    }

    IRoutingPolicy <|-- FastestRoutePolicy
    IRoutingPolicy <|-- CheapestRoutePolicy
    IRoutingPolicy <|-- RestrictedAreaAvoidancePolicy
    RoutingService ..> IRoutingPolicy : uses

    note for FastestRoutePolicy "Optimizes for minimum\ntransit time"
    note for CheapestRoutePolicy "Optimizes for lowest\ncost"
    note for RestrictedAreaAvoidancePolicy "Avoids specific\ngeographic areas"
```

**Business value:**

- **Explicitness** - Each routing strategy has a clear name
- **Testability** - Test each routing algorithm in isolation
- **Flexibility** - Add new routing strategies without modifying service
- **Decoupling** - Generic "routing" separated from specific criteria

### Example 3: Business Priority Calculator

**Scenario:** In an Agile Project Management context, calculating the "Business Priority" of a backlog item involves weighing value, cost, and risk.

```mermaid
classDiagram
    class BusinessPriorityCalculator {
        <<Policy / Value Object>>
        +Calculate(value: Money, cost: Money, risk: RiskLevel) PriorityScore
    }

    class BacklogItem {
        <<Entity>>
        +Id: ItemId
        +BusinessValue: Money
        +EstimatedCost: Money
        +RiskLevel: RiskLevel
        +CalculatePriority(calculator: BusinessPriorityCalculator) PriorityScore
    }

    class PriorityScore {
        <<Value Object>>
        +Score: decimal
        +IsHigherThan(other: PriorityScore) Boolean
    }

    BacklogItem ..> BusinessPriorityCalculator : uses
    BusinessPriorityCalculator --> PriorityScore : returns

    note for BusinessPriorityCalculator "Encapsulates formula:\nPriority = Value / (Cost + Risk)"
```

**Benefits:**

- **Single place to update** - Formula changes only in the calculator
- **Testable** - Verify calculation logic without setting up complex backlog state
- **Explicit** - The formula is documented in code, not hidden in calculations

### Example 4: Payment Retry Policy

**Scenario:** When a payment fails, the system must decide whether to retry and when.

```mermaid
classDiagram
    class IPaymentRetryPolicy {
        <<Interface>>
        <<Policy>>
        +ShouldRetry(payment: Payment, attemptNumber: int) Boolean
        +GetRetryDelay(payment: Payment, attemptNumber: int) Duration
    }

    class ExponentialBackoffRetryPolicy {
        <<Policy>>
        -MaxRetries: int
        -BaseDelay: Duration
        +ShouldRetry(payment: Payment, attemptNumber: int) Boolean
        +GetRetryDelay(payment: Payment, attemptNumber: int) Duration
    }

    class FixedIntervalRetryPolicy {
        <<Policy>>
        -MaxRetries: int
        -RetryInterval: Duration
        +ShouldRetry(payment: Payment, attemptNumber: int) Boolean
        +GetRetryDelay(payment: Payment, attemptNumber: int) Duration
    }

    class PaymentService {
        <<Domain Service>>
        -RetryPolicy: IPaymentRetryPolicy
        +ProcessPayment(payment: Payment) Result
        -HandleFailure(payment: Payment, attemptNumber: int) Result
    }

    IPaymentRetryPolicy <|-- ExponentialBackoffRetryPolicy
    IPaymentRetryPolicy <|-- FixedIntervalRetryPolicy
    PaymentService ..> IPaymentRetryPolicy : uses

    note for PaymentService "PaymentService delegates\nretry decision to policy"
```

**EventStorming integration:**

```mermaid
flowchart LR
    E[PaymentFailed Event] --> P[PaymentRetryPolicy]
    P --> Q{Should retry?}

    Q -->|Yes - attempt 1| D1[Wait 1 minute]
    D1 --> R1[RetryPayment Command]

    Q -->|Yes - attempt 2| D2[Wait 5 minutes]
    D2 --> R2[RetryPayment Command]

    Q -->|No - exceeded attempts| C[CancelPayment Command]

    style P fill:#e1f5e1
```

## Policies vs Alternatives

### Policy vs Specification

```mermaid
flowchart TB
    subgraph Spec["Specification: Predicate"]
        S1[Is this valid?]
        S2[Returns Boolean]
        S3[Declarative]
    end

    subgraph Policy["Policy: Algorithm"]
        P1[How should this work?]
        P2[Returns Value/Result]
        P3[Imperative logic]
    end

    style Spec fill:#fff4e1
    style Policy fill:#e1f5e1
```

| Aspect | Specification | Policy |
|--------|--------------|--------|
| **Question** | Is this satisfied? | How should this be executed? |
| **Return** | Boolean | Value, Result, Object |
| **Usage** | Query, Validation | Calculation, Decision-making |
| **Composition** | AND, OR, NOT | Strategy selection |
| **Example** | `ActiveCustomerSpecification` | `OverbookingPolicy` |

### Policy vs Domain Service

```mermaid
flowchart TB
    subgraph DS["Domain Service"]
        DS1[Stateless operation]
        DS2[Cross-aggregate logic]
        DS3[Typically one implementation]
    end

    subgraph Policy["Policy"]
        P1[Encapsulated algorithm]
        P2[Multiple variations]
        P3[Injected as dependency]
    end

    style DS fill:#fff4e1
    style Policy fill:#e1f5e1
```

| Aspect | Domain Service | Policy |
|--------|---------------|--------|
| **Responsibility** | Coordinate across aggregates | Encapsulate specific algorithm |
| **Variations** | Usually single implementation | Multiple interchangeable implementations |
| **State** | Always stateless | Stateless or immutable VO |
| **Usage** | Called directly | Injected as parameter |
| **Example** | `FundsTransferService` | `PricingPolicy` (Standard/Discount/VIP) |

**When to use both:**

A Domain Service can use Policies to delegate specific algorithms:

```mermaid
classDiagram
    class OrderPricingService {
        <<Domain Service>>
        -PricingPolicy: IPricingPolicy
        -DiscountPolicy: IDiscountPolicy
        +CalculateFinalPrice(order: Order) Result~Money~
    }

    class IPricingPolicy {
        <<Policy>>
        +Calculate(order: Order) Money
    }

    class IDiscountPolicy {
        <<Policy>>
        +CalculateDiscount(order: Order, customer: Customer) Money
    }

    OrderPricingService ..> IPricingPolicy : uses
    OrderPricingService ..> IDiscountPolicy : uses

    note for OrderPricingService "Domain Service coordinates\npricing using injected policies"
```

### Policy vs Entity Method

```mermaid
flowchart TB
    A[Business Logic] --> B{Does the rule vary?}

    B -->|No - single way| C[Entity Method]
    B -->|Yes - multiple ways| D{Is it complex?}

    D -->|Yes - obscures entity| E[Policy]
    D -->|No - simple| F[Entity Method with parameter]

    style C fill:#e1f5e1
    style E fill:#e1f5e1
    style F fill:#e1f5e1
```

**Use Entity Method when:**
- Single, unchanging way to perform the operation
- Logic is intrinsic to the entity's nature
- Simple calculation

**Use Policy when:**
- Multiple valid ways to perform the operation
- Rule varies by context (customer type, region, time)
- Extracting it clarifies the entity's responsibility

**Example:** Payment calculation

```mermaid
classDiagram
    class Order {
        <<Aggregate Root>>
        +CalculatePayment(method: PaymentMethod) Money
        -AddBaseFee() Money
        -AddTax() Money
    }

    class IPaymentMethodPolicy {
        <<Policy>>
        +CalculateFee(order: Order) Money
    }

    class CreditCardPolicy {
        <<Policy>>
        +CalculateFee(order: Order) Money
    }

    class BankTransferPolicy {
        <<Policy>>
        +CalculateFee(order: Order) Money
    }

    Order ..> IPaymentMethodPolicy : uses

    note for Order "Base calculation in Order\nPayment method fee via Policy"
```

## Policies in Domain Documents

### When to Include Policies in DOMAIN.md

Include policies in your domain design when:

1. **Multiple algorithms exist** for the same operation
2. **Business rules are complex** and would obscure entity logic
3. **Rules change independently** of the objects they apply to
4. **Automation logic** exists (EventStorming: event → policy → command)
5. **Calculation logic** needs to be centralized and explicit

### How to Document Policies

**Option 1: In Types section**

```markdown
## Types

### OverbookingPolicy

Encapsulates the business rule for allowing cargo bookings above vessel capacity. The policy allows overbooking up to 10% of capacity to account for last-minute cancellations. Validates whether new cargo can be added to a voyage.
```

**Option 2: In Class Diagrams**

```mermaid
classDiagram
    class OverbookingPolicy {
        <<Policy>>
        -MaxOverbookingPercentage: Percentage
        +IsAllowed(cargo: Cargo, voyage: Voyage) Boolean
    }
```

**Option 3: In Design Explanations**

```markdown
## Design Explanations

### Overbooking Policy

The domain extracts overbooking logic into a `OverbookingPolicy` to:
- Make the 10% overbooking rule explicit and named
- Allow the rule to change without modifying the Voyage aggregate
- Enable testing of the rule in isolation
- Add the concept to the ubiquitous language

The Voyage aggregate delegates to the policy when booking cargo.
```

**Option 4: In Invariants section**

```markdown
## Invariants

### OverbookingPolicy Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| OVERBOOK-1 | Maximum overbooking is 10% of voyage capacity | Configured percentage |
| OVERBOOK-2 | Overbooking only allowed for cargo with confirmed status | Prevents speculative bookings |
```

## Common Anti-Patterns

### Policy for Simple Logic

**Problem:** Creating a Policy for trivial calculations that don't vary.

```mermaid
flowchart TB
    subgraph Excessive["❌ Over-Engineered"]
        E1[AddNumbersPolicy]
        E2[SimpleDateFormatPolicy]
        E3[BasicValidationPolicy]
    end

    subgraph Appropriate["✅ Appropriate"]
        A1[OverbookingPolicy]
        A2[RoutingPolicy]
        A3[PricingPolicy]
    end

    style Excessive fill:#f5e1e1
    style Appropriate fill:#e1f5e1
```

**Solution:** Only extract Policies for meaningful business rules that vary or are complex.

### Policy as Anemic Domain

**Problem:** Extracting all entity behavior into Policies, leaving entities with no behavior.

```mermaid
flowchart LR
    subgraph Bad["❌ Anemic with Policies"]
        B1[Account - data only]
        B2[AccountValidationPolicy]
        B3[AccountStateChangePolicy]
        B4[AccountCalculationPolicy]
    end

    subgraph Good["✅ Rich Domain"]
        G1[Account - core behavior]
        G2[OverdraftPolicy for bank-specific rules]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Solution:** Keep core entity behavior in the entity. Use Policies only for varying rules.

### God Policy

**Problem:** One Policy that handles multiple, unrelated concerns.

```mermaid
flowchart TB
    subgraph Bad["❌ God Policy"]
        B1[OrderProcessingPolicy<br/>Validation<br/>Pricing<br/>Discount<br/>Shipping<br/>Notification]
    end

    subgraph Good["✅ Focused Policies"]
        G1[PricingPolicy]
        G2[DiscountPolicy]
        G3[ShippingPolicy]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Solution:** Split by business concept. One policy per cohesive rule.

### Policy That Modifies State

**Problem:** Policies that directly modify aggregate state.

```mermaid
flowchart TB
    subgraph Bad["❌ Policy Modifies State"]
        B1[Policy saves to repository]
        B2[Policy modifies aggregate directly]
    end

    subgraph Good["✅ Policy Returns Decisions"]
        G1[Policy returns calculation/result]
        G2[Domain Service applies change]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Solution:** Policies should calculate or decide, not modify. Let Domain Services or Entities apply state changes based on Policy results.

### Technical Naming

**Problem:** Policy names reflect implementation patterns, not business concepts.

```mermaid
flowchart TB
    subgraph Bad["❌ Technical Names"]
        B1[OverbookingStrategy]
        B2[PricingAlgorithm]
        B3[RoutingCalculator]
    end

    subgraph Good["✅ Domain Names"]
        G1[OverbookingPolicy]
        G2[PricingPolicy]
        G3[RoutingPolicy]
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Solution:** Use ubiquitous language. The word "Policy" signals business intent.

## Summary Checklist

When designing Policies, ensure:

- [ ] Encapsulates varying business rule or algorithm
- [ ] Named using ubiquitous language (domain terminology)
- [ ] Multiple variations exist or are planned
- [ ] Logic is complex enough to warrant extraction
- [ ] Clarifies domain object's responsibility (doesn't create anemic model)
- [ ] Stateless or immutable value object
- [ ] Injected as parameter or dependency
- [ ] Returns decisions/calculations, doesn't modify state
- [ ] Testable in isolation
- [ ] Documented in DOMAIN.md with business rule explanation
- [ ] Distinguished from Specifications (algorithms vs predicates)
- [ ] Distinguished from Domain Services (encapsulated rule vs stateless operation)
