# DDD Pattern Guides - Index

This directory contains comprehensive guides for each tactical Domain-Driven Design pattern. Each guide provides detailed rules, examples, diagrams, and best practices for designing domain models in DOMAIN.md files.

## Quick Reference by Concept

### Core Building Blocks

| Pattern | Guide | Key Concepts |
|---------|-------|--------------|
| **Value Objects** | [`value-objects.md`](./value-objects.md) | Immutability, structural equality, validation, preventing primitive obsession |
| **Entities** | [`entities.md`](./entities.md) | Identity strategies, lifecycle management, behavioral encapsulation |
| **Aggregates** | [`aggregates.md`](./aggregates.md) | Consistency boundaries, four rules of design, concurrency control |

### Domain Logic

| Pattern | Guide | Key Concepts |
|---------|-------|--------------|
| **Domain Services** | [`domain-services.md`](./domain-services.md) | Cross-aggregate logic, stateless operations, port pattern, service vs entity methods |

### Communication Patterns

| Pattern | Guide | Key Concepts |
|---------|-------|--------------|
| **Domain Events** | [`domain-events.md`](./domain-events.md) | Eventual consistency, transactional outbox, idempotent consumers, saga pattern |

### Domain Interfaces

| Pattern | Guide | Key Concepts |
|---------|-------|--------------|
| **Repositories** | [`repositories.md`](./repositories.md) | Repository interfaces (domain concern), specification pattern, collection-like semantics |
| **Driven Ports** | [`ports.md`](./ports.md) | Port interfaces for external dependencies, port vs adapter distinction |

### Supporting Concepts

| Pattern | Guide | Key Concepts |
|---------|-------|--------------|
| **Factories** | [`factories.md`](./factories.md) | Creation encapsulation, factory methods vs factory classes, complex creation with Result pattern |
| **Specifications** | [`specifications.md`](./specifications.md) | Encapsulating business queries, composability, reusable business rules, domain vs technical names |
| **Policies** | [`policies.md`](./policies.md) | Encapsulating algorithms and business rules, Strategy pattern in DDD, policy vs specification, EventStorming automation |

## Guide Contents

### Value Objects (`value-objects.md`)

**Sections:**
- Overview and characteristics
- Core Rules (Result Pattern for Creation, Immutability, Structural Equality, Always Valid, No Identity)
- Common Value Object Patterns (Money, Address, DateRange, EmailAddress, Quantity)
- Aggregate Integration
- Anti-Patterns

**Key Diagrams:**
- Result pattern flow for Create methods
- Immutability flow
- Structural equality comparison
- Money, Address, DateRange, EmailAddress, Quantity class diagrams with Create methods

---

### Entities (`entities.md`)

**Sections:**
- Overview and identity principles
- Identity Generation Strategies (Surrogate Keys, UUID/GUID, Natural Keys, Identity Value Objects)
- Mutability and Behavioral Encapsulation
- Lifecycle Management (Creation with Result pattern, State Transitions with Result pattern, Archival)
- Entities vs. Aggregates (Aggregate Root vs Internal Entity)
- Entity Relationships (One-to-Many, Many-to-One, One-to-One)
- Common Anti-Patterns

**Key Diagrams:**
- Identity continuity over lifecycle
- Identity generation strategy decision tree
- Result pattern for entity creation
- State machine diagrams with Result returns
- Entity vs Value Object comparison
- Aggregate structure with internal entities

---

### Aggregates (`aggregates.md`)

**Sections:**
- Overview (Root, Boundary, Invariants)
- The Four Rules of Aggregate Design (Vaughn Vernon)
- Aggregate Structure (Root, Internal Entities, Value Objects, Domain Events)
- Concurrency Control (Optimistic, Pessimistic)
- Aggregate Lifecycle (Creation, Modification with Result pattern, Deletion)
- Repository Interfaces (One Repository Interface Per Aggregate Root, Result returns)
- Aggregate Design Examples (Correct vs Anti-Patterns)
- Aggregate Invariants
- Advanced Concepts (Dynamic Consistency Boundaries, Aggregate vs Bounded Context)

**Key Diagrams:**
- Aggregate boundary visualization
- Mega-aggregate anti-pattern vs small aggregate
- Cross-aggregate references (by ID only)
- Result pattern for aggregate modifications
- Optimistic concurrency sequence diagram
- Proper aggregate design examples (Order, Payment, Shipment)
- Repository interface diagram

---

### Domain Services (`domain-services.md`)

**Sections:**
- Overview and characteristics
- When to Use Domain Services (decision framework)
- Domain Service vs. Application Service (comparison)
- Domain Service vs. Method on Entity (when to use which)
- Domain Service Design (naming conventions, operation signatures, design patterns)
- Domain Services and External Dependencies (port pattern)
- Common Domain Service Examples (calculation, coordination, validation, domain logic)
- Testing Domain Services
- Common Anti-Patterns

**Key Diagrams:**
- Domain service decision framework
- Domain Service vs Application Service comparison
- Domain Service vs Entity Method decision tree
- Funds transfer service example (sequence diagram)
- Domain Service with Ports (class diagram)
- Calculation, coordination, validation service examples

---

### Domain Events (`domain-events.md`)

**Sections:**
- Overview and benefits (Decoupling, Eventual Consistency)
- Event Design (Naming, Structure, Minimal Data Principle)
- Event Generation (When to Emit, Generation Pattern with Result, Event vs State)
- Common Anti-Patterns

**Key Diagrams:**
- Event-driven architecture flow
- Tightly coupled vs loosely coupled (with events)
- Eventual consistency across aggregates (sequence diagram)
- Event generation with Result pattern

---

### Repositories (`repositories.md`)

**Sections:**
- Overview and principles
- Repository Interface Rules (One Per Aggregate Root, Collection-Like Semantics, Domain Layer Definition)
- Query Patterns (Specification Pattern, Query vs Command)
- Result returns for operations that can fail
- Common Anti-Patterns

**Key Diagrams:**
- Repository interface (domain) vs implementation (infrastructure)
- Specification pattern class diagram
- CQRS (Command vs Query side)
- Repository interface examples with Result returns

**Note:** This guide focuses on repository interfaces as domain concerns. Repository implementations are infrastructure concerns and are documented separately.

---

### Driven Ports (`ports.md`)

**Sections:**
- Overview and purpose (Hexagonal Architecture)
- Port vs Adapter distinction
- Port Interface Rules (Defined in Domain, Implemented in Infrastructure)
- When to use ports (external dependencies, infrastructure concerns)
- Result returns for port operations
- Common examples (EmailGateway, PaymentGateway, etc.)
- Common Anti-Patterns

**Key Diagrams:**
- Port (domain) vs Adapter (infrastructure) relationship
- Hexagonal architecture layers
- Port interface examples with Result returns
- Domain Service using ports

**Note:** Ports are interfaces defined in the domain layer for external dependencies. Adapters (implementations) are infrastructure concerns.

---

### Factories (`factories.md`)

**Sections:**
- Overview and purpose
- Factory Methods (static Create methods with Result pattern)
- Factory Classes (complex creation scenarios)
- Creation vs. Construction
- Common Anti-Patterns

**Key Diagrams:**
- Factory method decision flow
- Result pattern for factory creation

---

### Specifications (`specifications.md`)

**Sections:**
- Overview and purpose
- When to Use Specifications (business queries, validation, reusable rules)
- Core Rules (domain language, composability, pure predicates)
- Specification Design Examples (Customer, Order, composed rules)
- Specifications in Domain Documents (how to document in DOMAIN.md)
- Specifications vs Alternatives (vs hardcoded queries, aggregate behavior, domain services)
- Common Anti-Patterns

**Key Diagrams:**
- Specification concept flow
- Domain names vs technical names
- Specification composition (AND, OR, NOT)
- Specification class diagrams
- Composed business rules

---

### Policies (`policies.md`)

**Sections:**
- Overview and purpose (Policy vs Strategy Pattern terminology)
- When to Use Policies (varying algorithms, obscured logic, EventStorming automation)
- Core Rules (varying business rules, ubiquitous language, parameter injection, VO vs Service, EventStorming)
- Policy Design Examples (Overbooking, Routing, Business Priority, Payment Retry)
- Policies in Domain Documents (how to document in DOMAIN.md)
- Policies vs Alternatives (vs Specification, Domain Service, Entity Methods)
- Common Anti-Patterns

**Key Diagrams:**
- Policy pattern concept flow
- Specification vs Policy comparison
- Policy vs Domain Service comparison
- Pricing policy class diagram with multiple implementations
- Overbooking policy for voyage validation
- Routing policy for algorithm selection
- EventStorming: event → policy → command flow

---

## How to Use These Guides

### When Designing a Domain Model

1. **Start with the SKILL.md** - Get the overview and quick reference
2. **Read relevant pattern guides** - Deep dive into specific patterns you're using
3. **Consult examples/** - See working examples of complete domain designs

### Progressive Disclosure Approach

**Quick decisions:** Use the quick reference tables in SKILL.md

**Designing a specific pattern:** Read the full guide for that pattern

**Understanding relationships:** Read multiple guides (e.g., Aggregates + Domain Events for consistency boundaries)

**Troubleshooting:** Consult the anti-patterns sections in each guide

### Diagram Reference

All guides include Mermaid diagrams that visualize:

- **Class diagrams** - Structure of entities, value objects, aggregates, and interfaces
- **Sequence diagrams** - Interaction flows, event publishing
- **Flowcharts** - Decision processes, validation flows
- **State diagrams** - Entity lifecycle, state machines
- **ER diagrams** - Persistence mapping

These diagrams serve as both reference material and examples for your own domain diagrams in DOMAIN.md files.

---

## Domain Layer Focus

These guides focus on the **Domain layer** of clean architecture:

✅ **Included (Domain Concerns):**
- Domain model design (entities, value objects, aggregates)
- Domain logic (domain services, domain logic encapsulation)
- Domain interfaces (repository interfaces, driven ports)
- Business invariants
- Domain events

❌ **Excluded (Infrastructure/Application Concerns):**
- Repository implementations (SQL, NoSQL, in-memory)
- Adapter implementations (email, SMS, payment gateways)
- Application services
- File organization, modules, packaging
- Deployment and infrastructure

For implementation details, refer to infrastructure-specific documentation or implementation guides.

---

## Contributing

When updating or adding to these guides:

1. **Use progressive disclosure** - Start with overview, then details
2. **Include diagrams** - Visualize concepts with Mermaid
3. **Provide examples** - Show both correct and anti-patterns
4. **Keep summaries** - End each guide with a checklist for quick reference
5. **Cross-reference** - Link between related guides for deeper learning
6. **Focus on domain** - Remember this is about designing DOMAIN.md files, not implementing code
