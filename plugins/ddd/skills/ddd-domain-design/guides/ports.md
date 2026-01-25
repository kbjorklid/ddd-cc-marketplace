# Driven Ports (Domain Interfaces)

## Overview

In Hexagonal Architecture (also known as Ports and Adapters), **Driven Ports** are interfaces defined in the Domain layer that represent contracts the domain needs from external dependencies. The domain layer defines the port (interface), while the infrastructure layer provides the adapter (implementation).

**Core principles:**
- **Port in Domain** - Interface defined in Domain layer
- **Adapter in Infrastructure** - Implementation in Infrastructure layer
- **Domain depends on abstraction** - Domain depends on port, not concrete implementation
- **Adapters depend on domain** - Infrastructure implements domain interface

```mermaid
flowchart TB
    subgraph Domain["Domain Layer (This Guide)"]
        Port[IPaymentGateway Port Interface]
        DomainService[PaymentProcessor Domain Service]
    end

    subgraph Infrastructure["Infrastructure Layer (Not Covered)"]
        Adapter[StripePaymentAdapter Adapter]
        PayPalAdapter[PayPalPaymentAdapter Adapter]
    end

    subgraph External["External World"]
        Stripe[Stripe API]
        PayPal[PayPal API]
    end

    DomainService -->|Uses| Port
    Adapter -->|Implements| Port
    PayPalAdapter -->|Implements| Port
    Adapter -->|Calls| Stripe
    PayPalAdapter -->|Calls| PayPal

    style Domain fill:#e1f5e1
    style Infrastructure fill:#fff4e1
    style External fill:#ffe1e1
```

## Port vs Adapter Distinction

### Ports (Domain Concern)

**Ports are interfaces** defined in the Domain layer that represent:

- Contracts the domain needs from external services
- Abstractions over infrastructure concerns
- Domain-specific vocabulary (ubiquitous language)

**Examples of Ports:**
- `IEmailGateway` - sending emails
- `ISmsGateway` - sending SMS messages
- `IPaymentGateway` - processing payments
- `IExchangeRateService` - getting currency exchange rates
- `INotificationService` - sending notifications

### Adapters (Infrastructure Concern)

**Adapters are implementations** that:

- Implement port interfaces
- Handle integration with external services
- Contain infrastructure-specific code

**Examples of Adapters:**
- `SendGridEmailAdapter` - implements `IEmailGateway` using SendGrid
- `TwilioSmsAdapter` - implements `ISmsGateway` using Twilio
- `StripePaymentAdapter` - implements `IPaymentGateway` using Stripe
- `FixerExchangeRateAdapter` - implements `IExchangeRateService` using Fixer.io

**Note:** This guide covers Ports (domain interfaces). Adapters (implementations) are infrastructure concerns and are not covered.

## When to Use Driven Ports

Use driven ports when domain logic requires:

- **External service calls** - Email, SMS, payment gateways
- **Data from other bounded contexts** - External APIs, microservices
- **Infrastructure operations** - File system, logging (sometimes)
- **Third-party integrations** - Shipping providers, geocoding services

### Decision Flow

```mermaid
flowchart TD
    A[Domain needs external service?] -->|Yes| B{Does service\ncontain\nbusiness logic?}
    A -->|No| C[Don't create port]

    B -->|Yes| D[Create Driven Port]
    B -->|No - pure infra| E[Handle in infrastructure]

    D --> F[Define interface in Domain layer]
    F --> G[Use in Domain Service]
    G --> H[Implement in Infrastructure layer]

    style D fill:#e1f5e1
    style F fill:#e1f5e1
    style G fill:#e1f5e1
```

**Examples:**

| External Service | Contains Business Logic? | Create Port? |
|-----------------|------------------------|--------------|
| Payment gateway | ✅ Yes (charge, refund) | ✅ Yes |
| Email service | ❌ No (delivery mechanism) | ✅ Yes (if domain needs "send email") |
| Database access | ❌ No (persistence detail) | ❌ No (use repository) |
| Exchange rate API | ✅ Yes (currency conversion) | ✅ Yes |
| Logging framework | ❌ No (infrastructure) | ❌ No |

## Port Interface Rules

### Rule 1: Define Interface in Domain Layer

**Rule:** Port interfaces are defined in the Domain layer using domain terminology.

```mermaid
classDiagram
    class IPaymentGateway {
        <<Domain Layer>> <<Port Interface>>
        +Charge(amount: Money, paymentMethod: PaymentMethod) Result~Payment~
        +Refund(payment: Payment, amount: Money) Result~Refund~
        +GetStatus(payment: Payment) Result~PaymentStatus~
    }

    class StripePaymentAdapter {
        <<Infrastructure Layer>> <<Adapter>>
        -ApiKey: string
        +Charge(amount: Money, paymentMethod: PaymentMethod) Result~Payment~
        +Refund(payment: Payment, amount: Money) Result~Refund~
        +GetStatus(payment: Payment) Result~PaymentStatus~
    }

    IPaymentGateway <|.. StripePaymentAdapter
```

**Benefits:**

- Domain controls the contract it needs
- Infrastructure adapts to domain requirements
- Easy to swap implementations
- Testable with fake adapters

### Rule 2: Use Domain Terminology

**Rule:** Port interfaces use domain language, not technical jargon.

```mermaid
flowchart TB
    subgraph Good["✅ Domain Terminology"]
        G1[IPaymentGateway.Charge]
        G2[IEmailGateway.SendOrderConfirmation]
        G3[IShippingService.CalculateShippingCost]
    end

    subgraph Bad["❌ Technical Jargon"]
        B1[IPostRequest.Execute]
        B2[ISmtpClient.SendMail]
        B3[IHttpWrapper.Get]
    end

    style Good fill:#e1f5e1
    style Bad fill:#f5e1e1
```

### Rule 3: Return Result for Operations That Can Fail

**Rule:** Port operations return `Result<T>` for operations that can fail (network errors, validation, business rules).

```mermaid
classDiagram
    class IEmailGateway {
        <<Port Interface>>
        +Send(message: EmailMessage) Result~void~
        +SendBatch(messages: List~EmailMessage~) Result~EmailSummary~
    }

    class IPaymentGateway {
        <<Port Interface>>
        +Charge(amount: Money) Result~Payment~
        +Refund(payment: Payment, amount: Money) Result~Refund~
    }

    class Result {
        <<Result Pattern>>
        +IsSuccess: bool
        +IsError: bool
        +Error: string
        +Value: T
    }
```

**Examples:**
- `Charge(amount)` → `Result<Payment>` (network failure, declined card = error)
- `Send(email)` → `Result<void>` (delivery failure = error)
- `GetExchangeRate(from, to)` → `Result<ExchangeRate>` (API unavailable = error)

### Rule 4: Ports Return Domain Types

**Rule:** Port methods accept and return domain types (Value Objects, Entities), not primitives.

```mermaid
classDiagram
    class IPaymentGateway {
        <<Port Interface>>
        +Charge(amount: Money, paymentMethod: PaymentMethod) Result~Payment~
    }

    class Money {
        <<Value Object>>
        +Amount: decimal
        +Currency: Currency
    }

    class PaymentMethod {
        <<Value Object>>
        +Type: PaymentType
        +Token: string
    }

    class Payment {
        <<Entity>>
        +Id: PaymentId
        +Amount: Money
        +Status: PaymentStatus
    }

    IPaymentGateway --> Money : uses
    IPaymentGateway --> PaymentMethod : uses
    IPaymentGateway --> Payment : returns
```

## Domain Services Using Ports

### Pattern: Domain Service Coordinates Port Calls

Domain Services use ports when they need to coordinate domain logic with external service calls.

```mermaid
classDiagram
    class PaymentProcessor {
        <<Domain Service>>
        -PaymentGateway: IPaymentGateway
        +Process(order: Order) Result~Payment~
    }

    class IPaymentGateway {
        <<Port Interface>>
        +Charge(amount: Money, paymentMethod: PaymentMethod) Result~Payment~
    }

    class Order {
        <<Aggregate Root>>
        +Total: Money
        +PaymentMethod: PaymentMethod
    }

    PaymentProcessor --> IPaymentGateway : uses
    PaymentProcessor --> Order : processes
```

### Example: Payment Processing with Port

```mermaid
classDiagram
    class IPaymentGateway {
        <<Port Interface>>
        +Charge(amount: Money, paymentMethod: PaymentMethod) Result~Payment~
        +Refund(payment: Payment, amount: Money) Result~Refund~
    }

    class PaymentProcessor {
        <<Domain Service>>
        -PaymentGateway: IPaymentGateway
        +ProcessPayment(order: Order) Result~Payment~
    }

    class Order {
        <<Aggregate Root>>
        +Id: OrderId
        +Total: Money
        +PaymentMethod: PaymentMethod
        +MarkAsPaid(payment: Payment) Result
    }

    class Payment {
        <<Entity>>
        +Id: PaymentId
        +OrderId: OrderId
        +Amount: Money
        +Status: PaymentStatus
    }

    PaymentProcessor --> IPaymentGateway : uses
    PaymentProcessor --> Order : processes
    PaymentProcessor --> Payment : creates
```

**Domain Logic:**

```mermaid
flowchart TD
    A[PaymentProcessor.ProcessPayment] --> B[Validate order state]
    B --> C{Order can be paid?}
    C -->|No| D[Return Result.Error]
    C -->|Yes| E[Call PaymentGateway.Charge]
    E --> F{Charge succeeded?}
    F -->|No| G[Return Result.Error]
    F -->|Yes| H[Create Payment entity]
    H --> I[Order.MarkAsPaid]
    I --> J[Return Result.Success with Payment]

    style D fill:#f5e1e1
    style G fill:#f5e1e1
    style J fill:#e1f5e1
```

## Common Port Examples

### Example 1: Email Gateway

```mermaid
classDiagram
    class IEmailGateway {
        <<Port Interface>> <<Domain Layer>>
        +Send(message: EmailMessage) Result~void~
        +SendBatch(messages: List~EmailMessage~) Result~EmailSummary~
    }

    class EmailMessage {
        <<Value Object>>
        +To: EmailAddress
        +From: EmailAddress
        +Subject: string
        +Body: string
        +Create() Result~EmailMessage~
    }

    class EmailSummary {
        <<Value Object>>
        +Sent: int
        +Failed: int
        +Errors: List~string~
    }

    IEmailGateway --> EmailMessage : accepts
    IEmailGateway --> EmailSummary : returns
```

**Usage in Domain:**

```mermaid
classDiagram
    class NotificationService {
        <<Domain Service>>
        -EmailGateway: IEmailGateway
        +SendOrderConfirmation(order: Order) Result~void~
    }

    class Order {
        <<Aggregate Root>>
        +CustomerEmail: EmailAddress
        +OrderNumber: string
    }

    NotificationService --> IEmailGateway : uses
    NotificationService --> Order : notifies
```

### Example 2: Exchange Rate Service

```mermaid
classDiagram
    class IExchangeRateService {
        <<Port Interface>> <<Domain Layer>>
        +GetRate(from: Currency, to: Currency, date: Date) Result~ExchangeRate~
    }

    class ExchangeRate {
        <<Value Object>>
        +From: Currency
        +To: Currency
        +Rate: decimal
        +Date: Date
        +Convert(money: Money) Money
    }

    class Currency {
        <<Value Object>>
        +Code: string
        +Symbol: string
    }

    IExchangeRateService --> ExchangeRate : returns
    ExchangeRate --> Currency : uses
```

**Usage in Domain:**

```mermaid
classDiagram
    class CurrencyConverter {
        <<Domain Service>>
        -ExchangeRateService: IExchangeRateService
        +Convert(money: Money, targetCurrency: Currency) Result~Money~
    }

    class Money {
        <<Value Object>>
        +Amount: decimal
        +Currency: Currency
    }

    CurrencyConverter --> IExchangeRateService : uses
    CurrencyConverter --> Money : converts
```

### Example 3: Shipping Service

```mermaid
classDiagram
    class IShippingService {
        <<Port Interface>> <<Domain Layer>>
        +CalculateShippingCost(order: Order, address: ShippingAddress) Result~ShippingQuote~
        +CreateShipment(order: Order) Result~Shipment~
        +TrackShipment(shipmentId: ShipmentId) Result~ShipmentStatus~
    }

    class ShippingQuote {
        <<Value Object>>
        +Cost: Money
        +EstimatedDelivery: Date
        +Carrier: string
        +ServiceLevel: string
    }

    class Shipment {
        <<Entity>>
        +Id: ShipmentId
        +OrderId: OrderId
        +Carrier: string
        +TrackingNumber: string
        +Status: ShipmentStatus
    }

    IShippingService --> ShippingQuote : returns
    IShippingService --> Shipment : creates
```

## Ports vs Repositories

Both ports and repositories are interfaces in the domain layer, but they serve different purposes.

```mermaid
flowchart TB
    subgraph Repositories["Repositories"]
        R1[Persist aggregates]
        R2[Database abstraction]
        R3[One per aggregate root]
    end

    subgraph Ports["Driven Ports"]
        P1[External services]
        P2[Integration abstraction]
        P3[As needed by domain]
    end

    Repositories -->|Both are| Both[Domain interfaces]
    Ports -->|Both are| Both
    Both --> Infrastructure[Implemented in infrastructure]

    style Repositories fill:#e1f5e1
    style Ports fill:#fff4e1
```

| Aspect | Repository | Driven Port |
|--------|-----------|-------------|
| **Purpose** | Persist aggregates | Integrate external services |
| **Abstraction** | Database storage | External service/API |
| **Cardinality** | One per aggregate root | As needed |
| **Operations** | CRUD-like | Domain-specific |
| **Returns** | `Result<T>` (aggregate) | `Result<T>` (domain type) |

## Common Anti-Patterns

### Port with Implementation Details
**Problem:** Port interface exposes implementation details like `SendWithSendGrid(apiKey: string, to: string, from: string)` or `SetSmtpConfig(host: string, port: int)`.
**Solution:** Use domain types and terminology (e.g., `Send(message: EmailMessage)` where EmailMessage is a domain value object).

### Port for Pure Infrastructure
**Problem:** Creating ports for pure infrastructure concerns like `ILogger` or `ICache` that contain no business logic.
**Solution:** Handle pure infrastructure concerns in the infrastructure layer, not as domain ports. Ports are for external services with business logic.

### Port Returning Primitive Types
**Problem:** Port methods return primitive types like `Charge(amount: decimal, currency: string) Result<string>`.
**Solution:** Return domain types (Value Objects, Entities) to maintain domain language and type safety.

## Summary Checklist

When designing driven ports in the domain layer:

- [ ] Interface defined in Domain layer
- [ ] Use domain terminology (ubiquitous language)
- [ ] Return `Result<T>` for operations that can fail
- [ ] Accept and return domain types (not primitives)
- [ ] Used by Domain Services (not directly by Aggregates)
- [ ] Abstract external services containing business logic
- [ ] No implementation details in interface
- [ ] Enable testing with fake adapters
- [ ] One port per external service concern
- [ ] Distinguish from repositories (persistence vs integration)

## Port vs Adapter

This guide focuses on **Ports** (domain interfaces).

**Adapters** (implementations) are **infrastructure concerns** and are not covered in this guide. When implementing:

1. Define the port interface in the Domain layer (this guide)
2. Implement the adapter in the Infrastructure layer
3. Use dependency injection to provide the adapter
4. Test domain logic with fake/test adapters
