# Domain Services

## Overview

A Domain Service is a stateless operation that represents domain logic which doesn't naturally fit within a single Entity or Value Object. Unlike Application Services which orchestrate workflows, Domain Services contain fundamental business rules that span multiple aggregates or require external dependencies.

**Core characteristics:**
- **Stateless** - No mutable state, operations are pure functions
- **Domain knowledge** - Contains business logic, not just orchestration
- **Ubiquitous Language** - Named using domain terminology
- **Cross-aggregate coordination** - Manages interactions between aggregates
- **Returns `Result<T>`** - Operations that can fail return Result types

```mermaid
flowchart LR
    subgraph ES["Entity / Aggregate"]
        E1[Account]
        E2[Order]
        E3[Customer]
    end

    subgraph DS["Domain Service"]
        D1[FundsTransferService]
        D2[ExchangeRateService]
        D3[OrderPricingService]
    end

    E1 --> D1
    E2 --> D3
    E3 --> D3
    E1 --> D2
    E2 --> D2

    style D1 fill:#e1f5e1
    style D2 fill:#e1f5e1
    style D3 fill:#e1f5e1
```

## When to Use Domain Services

### Decision Framework

```mermaid
flowchart TD
    A[Domain Logic] --> B{Fits in single Entity/VO?}
    B -->|Yes| C[Place in Entity/VO]
    B -->|No| D{Is orchestration?}

    D -->|Yes - workflow| E[Application Service]
    D -->|No - business logic| F{Involves multiple aggregates?}

    F -->|Yes| G[Domain Service]
    F -->|No| H{Needs external dependencies?}

    H -->|Yes| G
    H -->|No| I[Reconsider design]

    style C fill:#e1f5e1
    style E fill:#fff4e1
    style G fill:#e1f5e1
    style I fill:#f5e1e1
```

### Use Cases

| Scenario | Solution | Example |
|----------|----------|---------|
| **Operation involves multiple aggregates** | Domain Service | `FundsTransferService.transfer(from, to, amount)` |
| **Logic requires external service** | Domain Service + Port | `ExchangeRateService.getCurrentRate()` |
| **Calculation spanning aggregates** | Domain Service | `OrderPricingService.calculateDiscount(order, customer)` |
| **Natural operation but no single owner** | Domain Service | `PasswordEncryptionService.encrypt(password)` |
| **Workflow coordination** | Application Service | `ProcessPaymentUseCase.execute(command)` |

### Domain Service vs. Application Service

```mermaid
flowchart TB
    subgraph Comparison["Domain Service vs Application Service"]
        direction TB

        subgraph DS["Domain Service"]
            DS1[Contains Business Logic]
            DS2[Part of Ubiquitous Language]
            DS3[Used by other domain components]
            DS4[Stateless operations]
        end

        subgraph AS["Application Service"]
            AS1[Orchestrates Workflow]
            AS2[Not domain language]
            AS3[Called by UI/Controllers]
            AS4[Manages transactions]
        end
    end

    style DS fill:#e1f5e1
    style AS fill:#fff4e1
```

| Aspect | Domain Service | Application Service |
|--------|---------------|---------------------|
| **Purpose** | Execute business logic | Orchestrate workflow |
| **Knowledge** | Domain rules and entities | DTOs, repositories, security |
| **Location** | Domain Layer | Application Layer |
| **Called by** | Other domain components, app services | Controllers, UI |
| **State** | Stateless | Stateless |
| **Returns** | `Result<T>` for validation | `Result<T>` for operation status |
| **Example** | `FundsTransferService` | `ProcessPaymentUseCase` |

### Domain Service vs. Method on Entity

```mermaid
flowchart TD
    A[Domain Operation] --> B{Does it belong on one entity?}
    B -->|Yes| C[Entity Method]
    B -->|No| D{Would it couple entities?}

    D -->|Yes - inappropriate| E[Domain Service]
    D -->|No| F[Entity Method]

    C --> G[Example: account.withdraw]
    E --> H[Example: transferService.transfer]
    F --> I[Example: order.addItem]

    style C fill:#e1f5e1
    style E fill:#e1f5e1
    style G fill:#e1f5e1
    style H fill:#e1f5e1
    style I fill:#e1f5e1
```

**Example: Funds Transfer**

```mermaid
classDiagram
    class Account {
        <<Aggregate Root / Entity>>
        -Id: AccountId
        -Balance: Money
        +Withdraw(amount: Money) Result
        +Deposit(amount: Money) Result
    }

    class FundsTransferService {
        <<Domain Service>>
        -AccountRepository: IAccountRepository
        +Transfer(from: AccountId, to: AccountId, amount: Money) Result
        -ValidateTransfer(from: Account, to: Account, amount: Money) Result
    }

    note for FundsTransferService "Domain Service because:\n• Coordinates two Account aggregates\n• Transfer is a domain concept\n• Belongs to neither Account\n• Contains business logic"
```

**Why not `Account.transferTo(other: Account)`?**

- **Tight coupling** - Account would know about other Account instances
- **Violation of aggregate boundaries** - Spans two consistency boundaries
- **Asymmetric responsibility** - Should both accounts handle transfers?
- **Domain concept** - "Transfer" is a fundamental banking operation, not an account behavior

## Domain Service Design

### Naming Conventions

```mermaid
flowchart LR
    A[Domain Service Names] --> B[Verb + Noun]
    A --> C[Domain Terminology]
    A --> D[Service Suffix]

    B --> E[Transfer, Calculate, Convert]
    C --> F[Funds, Exchange, Pricing]
    D --> G[Service, Calculator, Engine]

    style A fill:#e1f5e1
    style E fill:#e1f5e1
    style F fill:#e1f5e1
    style G fill:#e1f5e1
```

| Pattern | Example | When to Use |
|---------|---------|-------------|
| **Verb + Noun + Service** | `FundsTransferService` | Standard domain operations |
| **Noun + Calculator** | `TaxCalculator`, `PriceCalculator` | Pure calculations |
| **Noun + Engine** | `RecommendationEngine` | Complex algorithms |
| **Verb + Noun** | `PasswordEncryptionService` | Single-responsibility service |

**Good names** (from ubiquitous language):
- ✅ `FundsTransferService` - Banking domain language
- ✅ `ExchangeRateService` - Currency trading language
- ✅ `OrderPricingService` - E-commerce language
- ✅ `InventoryAvailabilityService` - Retail language

**Poor names** (technical jargon):
- ❌ `AccountManager` - Too vague, could be app service
- ❌ `HelperService` - Not domain language
- ❌ `UtilityService` - No domain meaning

### Operation Signature Design

```mermaid
classDiagram
    class DomainService {
        <<abstract>>
        <<Domain Service>>
    }

    class FundsTransferService {
        <<Domain Service>>
        +Transfer(from: AccountId, to: AccountId, amount: Money) Result
    }

    class ExchangeRateService {
        <<Domain Service>>
        +Convert(amount: Money, from: Currency, to: Currency) Result~Money~
        +GetCurrentRate(from: Currency, to: Currency) Result~ExchangeRate~
    }

    class OrderPricingService {
        <<Domain Service>>
        +CalculateDiscount(order: Order, customer: Customer) Result~Money~
        +ApplyPricingRules(order: Order) Result~Order~
    }

    DomainService <|-- FundsTransferService
    DomainService <|-- ExchangeRateService
    DomainService <|-- OrderPricingService

    note for DomainService "Return Result~T~ for\noperations that can fail"
```

**Signature principles:**

1. **Accept aggregates or IDs** - Domain services work with domain objects
2. **Return `Result<T>`** - Operations can fail with business rules
3. **Stateless** - No instance state, all dependencies injected
4. **Domain types** - Use value objects, not primitives

### Operation Design Patterns

```mermaid
flowchart TD
    A[Domain Service Operation] --> B{Needs data from aggregates?}
    B -->|Yes| C[Load aggregates via repository]
    B -->|No| D[Accept as parameters]

    C --> E{Validates invariants?}
    D --> E

    E -->|Yes| F[Return Result.Error if invalid]
    E -->|No - just calculation| G[Return Result.Success with value]

    F --> H{Modifies state?}
    G --> I[Pure calculation]

    H -->|Yes| J[Update aggregates, save via repository]
    H -->|No - read only| K[Return calculated value]

    J --> L[Publish domain events]
    L --> M[Return Result.Success]

    style F fill:#f5e1e1
    style G fill:#e1f5e1
    style I fill:#e1f5e1
    style M fill:#e1f5e1
```

**Pattern 1: State Transition Service** (modifies aggregates)

```mermaid
sequenceDiagram
    participant AS as App Service
    participant DS as Domain Service
    participant R1 as Repository A
    participant R2 as Repository B
    participant DB as Database

    AS->>DS: transfer(fromId, toId, amount)
    DS->>R1: getById(fromId)
    R1-->>DS: Result<Account>
    DS->>R2: getById(toId)
    R2-->>DS: Result<Account>

    DS->>DS: Validate business rules

    alt Business rules violated
        DS-->>AS: Result.Error("Insufficient funds")
    else Rules pass
        DS->>DS: from.Withdraw(amount)
        DS->>DS: to.Deposit(amount)
        DS->>R1: Save(from)
        DS->>R2: Save(to)
        DS-->>AS: Result.Success
    end
```

**Pattern 2: Calculation Service** (pure function)

```mermaid
sequenceDiagram
    participant AS as App Service
    participant DS as Domain Service
    participant P as Port/Interface
    participant Ext as External Service

    AS->>DS: convert(amount, from, to)
    DS->>P: GetCurrentRate(from, to)
    P->>Ext: API call
    Ext-->>P: Rate
    P-->>DS: Result<Rate>

    alt Rate unavailable
        DS-->>AS: Result.Error("Rate not available")
    else Success
        DS->>DS: Calculate converted amount
        DS-->>AS: Result.Success(converted)
    end
```

## Domain Services and External Dependencies

### The Port Pattern

When Domain Services need external dependencies (email, payment gateways, external APIs), use the Port pattern:

```mermaid
classDiagram
    class DomainService {
        <<Domain Service>>
    }

    class EmailDomainService {
        <<Domain Service>>
        -Gateway: IEmailGateway
        +SendWelcomeEmail(user: User) Result
    }

    class IEmailGateway {
        <<Port / Interface>>
        <<Domain Layer>>
        +Send(message: EmailMessage) Result
    }

    class SmtpEmailGatewayAdapter {
        <<Adapter / Implementation>>
        <<Infrastructure Layer>>
        +Send(message: EmailMessage) Result
    }

    EmailDomainService ..> IEmailGateway : depends on
    IEmailGateway <|-- SmtpEmailGatewayAdapter : implements

    note for IEmailGateway "Port defined in Domain\nAdapter in Infrastructure"
```

**Benefits:**

- **Domain remains pure** - No infrastructure dependencies in domain layer
- **Testability** - Mock ports for unit testing
- **Flexibility** - Swap implementations without changing domain
- **Clear separation** - Domain defines contract, infrastructure provides implementation

### Domain Service with Ports

```mermaid
classDiagram
    class FundsTransferService {
        <<Domain Service>>
        -AccountRepo: IAccountRepository
        -ExchangeRate: IExchangeRateService
        +Transfer(from: AccountId, to: AccountId, amount: Money, currency: Currency) Result
    }

    class IAccountRepository {
        <<Port>>
        +GetById(id: AccountId) Result~Account~
        +Save(account: Account) Result
    }

    class IExchangeRateService {
        <<Port>>
        +GetCurrentRate(from: Currency, to: Currency) Result~ExchangeRate~
    }

    FundsTransferService ..> IAccountRepository : uses
    FundsTransferService ..> IExchangeRateService : uses

    note for FundsTransferService "Domain Service uses ports\nfor repositories and external services"
```

## Common Domain Service Examples

### 1. Calculation Services

```mermaid
classDiagram
    class TaxCalculationService {
        <<Domain Service>>
        +CalculateTax(order: Order, region: Region) Result~TaxBreakdown~
        +CalculateTotal(order: Order) Result~Money~
        -GetTaxRate(region: Region, category: ProductCategory) Decimal
        -ApplyRegionalRules(order: Order) Result
    }

    note for TaxCalculationService "Pure calculations\nNo state modification\nBusiness rules encapsulated"
```

**Characteristics:**
- Pure functions (same input = same output)
- No side effects
- No aggregate modification
- Return calculated values

### 2. Coordination Services

```mermaid
classDiagram
    class FundsTransferService {
        <<Domain Service>>
        -AccountRepo: IAccountRepository
        +Transfer(from: AccountId, to: AccountId, amount: Money) Result
        -ValidateAccounts(from: Account, to: Account) Result
        -CheckLimits(account: Account, amount: Money) Result
    }

    class Account {
        <<Aggregate Root>>
        +Withdraw(amount: Money) Result
        +Deposit(amount: Money) Result
    }

    note for FundsTransferService "Coordinates multiple aggregates\nEnforces transfer-specific rules\nManages atomic updates"
```

**Characteristics:**
- Modifies multiple aggregates
- Coordinates across consistency boundaries
- Enforces cross-aggregate invariants
- May publish domain events

### 3. Validation Services

```mermaid
classDiagram
    class UserRegistrationService {
        <<Domain Service>>
        -UserRepo: IUserRepository
        -EmailGateway: IEmailGateway
        +Register(email: EmailAddress, password: Password) Result~User~
        -ValidateEmailUnique(email: EmailAddress) Result
        -ValidatePasswordStrength(password: Password) Result
        -SendVerificationEmail(user: User) Result
    }

    note for UserRegistrationService "Validates across aggregates\nCoordinates creation\nSide effects (email)"
```

**Characteristics:**
- Validates uniqueness across aggregates
- Complex creation logic
- Side effects via ports
- Returns new aggregates

### 4. Domain Logic Services

```mermaid
classDiagram
    class PasswordEncryptionService {
        <<Domain Service>>
        +Hash(password: Password) Result~HashedPassword~
        +Verify(password: Password, hash: HashedPassword) Boolean
        +IsStrong(password: Password) Boolean
        -HashAlgorithm(value: String) String
    }

    note for PasswordEncryptionService "Domain-specific algorithm\nNo entities involved\nPure business logic"
```

**Characteristics:**
- Domain-specific algorithm
- No entity/aggregate involvement
- Pure business logic
- Often stateless utilities

## Testing Domain Services

### Unit Testing Strategy

```mermaid
flowchart LR
    A[Domain Service Test] --> B[Arrange]
    A --> C[Act]
    A --> D[Assert]

    B --> E[Mock repositories]
    B --> F[Mock ports]
    B --> G[Create test data]

    C --> H[Call service method]

    D --> I{Expected Result?}
    I -->|Success| J[Verify: aggregates updated]
    I -->|Failure| K[Verify: error message]

    J --> L[Verify: repository saves called]
    K --> M[Verify: repository saves not called]

    style J fill:#e1f5e1
    style K fill:#e1f5e1
```

**Test scenarios:**

1. **Success case** - Valid operation completes successfully
2. **Business rule violation** - Returns `Result.Error` with message
3. **External dependency failure** - Port returns error, service propagates
4. **Aggregate state** - Verify aggregates modified correctly
5. **Domain events** - Verify correct events published

## Common Anti-Patterns

### Anemic Domain Service

**Problem:** Domain Service becomes a procedural script with all business logic extracted from entities.

```mermaid
flowchart LR
    subgraph Bad["❌ Anemic Domain Service"]
        B1[Account Entity<br/>Getters/Setters only]
        B2[FundsTransferService<br/>All business logic]
        B1 --> B2
    end

    subgraph Good["✅ Rich Domain Model"]
        G1[Account Entity<br/>Withdraw/Deposit methods]
        G2[FundsTransferService<br/>Transfer coordination only]
        G1 --> G2
    end

    style Bad fill:#f5e1e1
    style Good fill:#e1f5e1
```

**Solution:** Keep behavior on entities. Use Domain Services only for logic that genuinely spans multiple aggregates.

### Application Service Masquerading as Domain Service

**Problem:** Service orchestrates workflows (loads repositories, saves results, manages transactions) instead of containing business logic.

| Anti-pattern | Correct |
|--------------|---------|
| `PaymentOrchestrator` - manages entire payment flow | `PaymentAuthorizationService` - contains authorization logic |
| Calls repositories directly | Receives aggregates as parameters |
| Manages transactions | Business logic only |
| Returns DTOs | Returns `Result<T>` with domain types |

**Solution:** Move orchestration to Application Services. Domain Services should focus purely on business logic.

### Overused Domain Services

**Problem:** Creating Domain Services for logic that belongs in a single entity.

```mermaid
flowchart TB
    A[domain operation] --> B{One entity owner?}
    B -->|Yes| C[❌ AccountValidationService]
    B -->|Yes| D[✅ Account.Validate()]

    C --> E[Unnecessary indirection]
    D --> F[Logic where it belongs]

    style C fill:#f5e1e1
    style D fill:#e1f5e1
```

**Solution:** If logic naturally belongs to one entity, place it there. Not every service needs to be a "Domain Service."

### God Domain Service

**Problem:** Single service accumulates too many responsibilities.

**Examples:**
- ❌ `AccountService` - Handles transfers, validation, creation, archival, reporting
- ✅ `FundsTransferService`, `AccountValidationService`, `AccountCreationService`

**Solution:** Split by domain concept, following single responsibility principle.

## Summary Checklist

When designing Domain Services, ensure:

- [ ] Logic genuinely doesn't fit in single Entity/Value Object
- [ ] Named using ubiquitous language (verb + noun + service)
- [ ] Stateless operations (no instance state)
- [ ] Returns `Result<T>` for operations that can fail
- [ ] Uses ports for external dependencies
- [ ] Distinguished from Application Services (business logic vs. orchestration)
- [ ] Accepts domain types (aggregates, value objects), not DTOs
- [ ] Keeps entities rich (doesn't become procedural script)
- [ ] Single responsibility (not a "god service")
- [ ] Documented in domain design with clear purpose
- [ ] Testable in isolation (mock ports and repositories)
