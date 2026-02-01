---
description: Analyze existing DOMAIN.md to identify DDD design problems and anti-patterns
argument-hint: [domain-md-path]
allowed-tools: ["Skill", "Read", "AskUserQuestion"]
---

You are analyzing an existing DOMAIN.md file to identify DDD design problems, anti-patterns, and documentation issues.

**IMPORTANT**: This command validates the design quality and documentation completeness of an existing domain design document. It helps catch common DDD mistakes before implementing the design.

Determine DOMAIN.md location:
- If $1 provided: Use that file path
- If no argument: Look for DOMAIN.md in current directory

Read the DOMAIN.md file using the Read tool.

Perform comprehensive validation by walking through the following checklists for each tactical DDD pattern present in the domain design:

## Value Objects Validation Checklist

For each Value Object in the domain design, check:

- [ ] Is the Value Object immutable (read-only fields, no setters)?
- [ ] Does the Value Object have structural equality (equals/hashCode overridden)?
- [ ] Does the Value Object have no identity (no ID field)?
- [ ] Is the Create method returning `Result<T>` for validation failures?
- [ ] Are all validation rules in the Create method?
- [ ] Does the Value Object have self-contained behavior?
- [ ] Is the domain meaning of the Value Object clear?
- [ ] Does the Value Object prevent primitive obsession?
- [ ] Are error messages in the Result.Error returns descriptive?

**Common Anti-Patterns to Detect:**
- Mutable Value Objects (has setters or mutates state)
- Identity Leaks (Value Object with ID field - should be Entity)
- Deferred Validation (Create method doesn't validate)

## Entities Validation Checklist

For each Entity in the domain design, check:

- [ ] Is there a clear identity strategy (surrogate, UUID, natural)?
- [ ] Is the identity wrapped in a Value Object for type safety?
- [ ] Are all setters private (state changes via behavior methods)?
- [ ] Is business logic encapsulated in entity methods, not services?
- [ ] Is there clear lifecycle management (creation, transitions, archival)?
- [ ] Do Create methods return `Result<T>` with validation?
- [ ] Do behavior methods return `Result` for operations that can fail?
- [ ] Is there a state machine modeled for valid transitions?
- [ ] Is the entity rich with behavior, not an anemic data holder?
- [ ] Is the distinction between Aggregate Root and internal entity clear?
- [ ] Are all cross-aggregate references by ID only?
- [ ] Are invariants enforced before state changes?
- [ ] Are error messages in Result.Error returns descriptive?

**Common Anti-Patterns to Detect:**
- Anemic Domain Model (getters/setters only, no behavior)
- God Entity (too many responsibilities, should be split)
- Identity Leakage (Entity that should be Value Object)

## Aggregates Validation Checklist

For each Aggregate in the domain design, check:

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

**Common Anti-Patterns to Detect:**
- Mega-aggregates (too many entities/value objects)
- Cross-aggregate references (direct object references instead of IDs)
- Missing consistency boundaries (no clear invariants)
- God aggregates (doing too much, should be split)

## Domain Events Validation Checklist

For each Domain Event in the domain design, check:

- [ ] Are Domain Events named in past tense (OrderPlaced, not PlaceOrder)?
- [ ] Are Domain Events immutable (events are facts)?
- [ ] Are Domain Events defined in the Domain Layer?
- [ ] Do Domain Events contain minimal data (only what consumers need)?
- [ ] Do Domain Events have standard metadata (EventId, AggregateId, OccurredAt)?
- [ ] Do Domain Events have clear correlation (events reference aggregate IDs)?

**Common Anti-Patterns to Detect:**
- Wrong naming (PlaceOrder instead of OrderPlaced)
- Fat events (containing entire aggregate state)
- Event god objects (generic XChanged events instead of specific events)
- Missing events (tight coupling where events should decouple)

## Domain Services Validation Checklist

For each Domain Service in the domain design, check:

- [ ] Does the Domain Service logic genuinely not fit in a single Entity/Value Object?
- [ ] Is the Domain Service named using ubiquitous language (verb + noun + service)?
- [ ] Are all Domain Service operations stateless (no instance state)?
- [ ] Do Domain Service operations return `Result<T>` for failures?
- [ ] Does the Domain Service use ports for external dependencies?
- [ ] Is the Domain Service distinguished from Application Services (business logic vs. orchestration)?
- [ ] Does the Domain Service accept domain types (aggregates, value objects), not DTOs?
- [ ] Does the Domain Service keep entities rich (not a procedural script)?
- [ ] Does the Domain Service have a single responsibility (not a "god service")?
- [ ] Is the Domain Service documented in the domain design with a clear purpose?
- [ ] Is the Domain Service testable in isolation (mock ports and repositories)?

**Common Anti-Patterns to Detect:**
- Misplaced domain logic (logic in entity that should be in domain service)
- Anemic domain service (service with no actual business logic - should be application service)
- God domain services (services doing too much, should split)

## Repository Interfaces Validation Checklist

For each Repository Interface in the domain design, check:

- [ ] Is the repository interface defined in the Domain layer?
- [ ] Is there one repository interface per Aggregate Root?
- [ ] Does the repository have collection-like semantics (Add, Update, Remove, GetById, Find)?
- [ ] Are the methods named using the Ubiquitous Language?
- [ ] Do repository operations return `Result<T>` for operations that can fail?
- [ ] Does GetById return `Result<T>` for not-found cases?
- [ ] Is the Specification pattern used for complex queries?
- [ ] Are there no database-specific methods in the interface?
- [ ] Is there no repository for internal entities?
- [ ] Does the interface enable testing with fake implementations?
- [ ] Is the interface kept stable (implementation details hidden)?

**Common Anti-Patterns to Detect:**
- Repository for internal entities (repositories for non-aggregate roots)
- Database-like methods (SQL/exposed persistence methods in interface)
- Missing Result returns (GetById returning null instead of Result<T>)
- No repository for aggregate (aggregate root without repository interface)

## Driven Ports Validation Checklist

For each Driven Port in the domain design, check:

**Design Principles:**
- [ ] Is there interface segregation (specific interfaces for specific clients, avoiding monolithic ports)?
- [ ] Is there intent-based naming (business language, not technical jargon)?
- [ ] Is the port technology agnostic (no HTTP, SQL, REST in method names)?

**Interface Design:**
- [ ] Is the port interface defined in the Domain layer?
- [ ] Does the port use domain terminology (ubiquitous language)?
- [ ] Does the port interface allow for swapping implementations?
- [ ] Do port operations return `Result<T>` for operations that can fail?
- [ ] Does the port accept and return domain types (not primitives)?
- [ ] Are there no implementation details in the port interface?
- [ ] Are there no framework-specific types in port method signatures?
- [ ] Do technical exceptions get caught in the adapter and re-thrown as business exceptions?

**Patterns:**
- [ ] Is the Gateway Pattern used for external service communication (REST, gRPC, SOAP)?
- [ ] Is the Output Port Pattern used for event publishing (write-only, observer pattern)?
- [ ] Is the Repository Pattern distinguished from ports (for persistence)?

**Data Exchange:**
- [ ] Are framework-specific objects (Hibernate proxies, HTTP responses) never passed through ports?

**Usage:**
- [ ] Is the port used by Domain Services (not directly by Aggregates)?
- [ ] Does the port abstract external services containing business logic?
- [ ] Is there one port per external service concern?

**Common Anti-Patterns to Detect:**
- Tight coupling (external dependencies directly in domain logic)
- Missing ports (domain logic calling infrastructure directly)
- Port in wrong layer (port defined in infrastructure instead of domain)

## Factories Validation Checklist

For each Factory in the domain design, check:

- [ ] Is the creation logic separated from representation?
- [ ] Do factory methods return `Result<T>` for validation failures?
- [ ] Is the Factory used for complex assemblies?
- [ ] Are Factory Methods placed on Aggregate Roots (when creating internal parts or closely related Aggregates)?
- [ ] Is the constructor on the entity private (public factory)?
- [ ] Can invalid objects never be created?
- [ ] Are error messages in validation failures descriptive?
- [ ] Are creation strategies clear and focused?

**Common Anti-Patterns to Detect:**
- Missing factories (complex creation logic in constructors)
- Factory as entity manager (doing too much - creating, validating, persisting)
- Bypassing validation (creating objects in invalid state)
- Factory gods (one factory creating all domain objects)

## Specifications Validation Checklist

For each Specification in the domain design, check:

- [ ] Is the Specification named using ubiquitous language (domain terminology)?
- [ ] Does the Specification encapsulate a meaningful business concept?
- [ ] Is the Specification reusable across multiple use cases?
- [ ] Is the Specification composable (can be combined with AND, OR, NOT)?
- [ ] Is the Specification a pure predicate (no side effects)?
- [ ] Is the Specification used for both queries and validation?
- [ ] Is the Specification documented in DOMAIN.md when complex or reused?
- [ ] Is the business rule of the Specification clearly defined?
- [ ] Is the Specification not over-used (avoiding trivial one-off conditions)?
- [ ] Is the Specification distinguished from aggregate behavior methods?
- [ ] Is the Specification distinguished from domain services (stateless operations)?

**Common Anti-Patterns to Detect:**
- Over-specification (creating specs for every simple query)
- Leaking implementation details (technical query conditions in names)
- God specifications (one spec that knows too much)
- Specifications that modify state (should be pure predicates)

## Policies Validation Checklist

For each Policy in the domain design, check:

- [ ] Does the Policy encapsulate a varying business rule or algorithm?
- [ ] Does the Policy decouple the rule from the Entity?
- [ ] Does the Policy trigger commands based on events (if using EventStorming)?
- [ ] Is the Policy named using ubiquitous language (domain terminology)?
- [ ] Do multiple variations of the Policy exist or are planned?
- [ ] Is the logic complex enough to warrant extraction into a Policy?
- [ ] Does the Policy clarify the domain object's responsibility (doesn't create anemic model)?
- [ ] Is the Policy a stateless service or immutable value object?
- [ ] Is the Policy injected as a parameter or dependency?
- [ ] Does the Policy return decisions/calculations, not modify state?
- [ ] Is the Policy testable in isolation?
- [ ] Is the Policy documented in DOMAIN.md with business rule explanation?
- [ ] Is the Policy distinguished from Specifications (algorithms vs predicates)?
- [ ] Is the Policy distinguished from Domain Services (encapsulated rule vs stateless operation)?

**Common Anti-Patterns to Detect:**
- Missing policies (varying algorithms in if/else chains)
- Policy for simple logic (extracting trivial calculations)
- God policies (one policy handling multiple unrelated concerns)
- Policy that modifies state (should return decisions only)

## Documentation Completeness Validation

Check for overall document completeness:

- [ ] Are all required sections present (Ubiquitous Language, Class Diagrams, Types, Design Explanations, Invariants, Future Considerations)?
- [ ] Does Introduction provide clear context?
- [ ] Do Mermaid diagrams use proper DDD stereotypes (<<Aggregate Root>>, <<Entity>>, <<Value Object>>)?
- [ ] Do invariants have IDs and are documented?
- [ ] Are relationships between types clear?
- [ ] Is cross-aggregate referencing by ID only in diagrams?

### Ubiquitous Language Quality Checks

- [ ] Are all definitions in **business terms only** (no DDD technical jargon)?
- [ ] Are definitions free of technical terms like "aggregate root", "entity", "value object", "repository", "domain service"?
- [ ] Are definitions free of programming terms like "class", "instance", "reference", "ID", "field"?
- [ ] Do definitions describe **what the concept is** in business terms, not how it's implemented?
- [ ] Would a business stakeholder (product manager, domain expert) understand the definitions?
- [ ] Are definitions concise (1-2 sentences)?
- [ ] Do definitions use language from actual business conversations?

**Common Anti-Patterns to Detect:**
- "Signing Workflow: The **aggregate root** that orchestrates..." - Uses DDD term "aggregate root"
- "Signer: An **entity** that represents a participant..." - Uses DDD term "entity"
- "Expiration Time: A **value object** that stores the deadline..." - Uses DDD term "value object"
- "Customer ID: The **identifier field** for customers..." - Uses programming term "field"

**Good Examples:**
- "Signing Workflow: A document signing process that coordinates multiple signers and tracks their signatures"
- "Signer: A person who needs to sign a document, identified by their email address"
- "Expiration Time: The deadline after which the signing workflow is automatically cancelled"

### Diagram Organization Checks

- [ ] Are diagrams split when a single diagram exceeds 20 classes?
- [ ] Is there an **Aggregate Overview** diagram showing all aggregates and their relationships?
- [ ] Are there **per-aggregate detail diagrams** showing internal structure?
- [ ] Is there a **Domain Events** diagram when 3+ events exist?
- [ ] Is there a **Domain Services and Ports** diagram when 2+ services exist?
- [ ] Are diagrams clearly labeled with descriptive section headings?
- [ ] Do diagrams avoid mixing concerns (e.g., not showing events, services, and aggregates all together)?

**Common Anti-Patterns to Detect:**
- Single massive diagram with 30+ classes that should be split
- All value objects shown in aggregate overview (clutters the high-level view)
- Domain events mixed with aggregates in same diagram
- No clear section headings between multiple diagrams

## Present Results

Present validation results to user in the following format:

### High Severity Issues (Must Fix)
*Violations of core DDD rules that will cause problems*

For each issue:
- **Location**: Which section/pattern has the issue
- **Problem**: What's wrong (referencing specific DDD principle from checklists)
- **Impact**: Why it matters (consequences)
- **Recommendation**: How to fix it (specific action)
- **Confidence**: How certain (High/Med/Low)

### Medium Severity Issues (Should Fix)
*Design improvements that would enhance quality*

For each issue:
- **Location**: Which section/pattern has the issue
- **Problem**: What's wrong (referencing specific DDD principle from checklists)
- **Impact**: Why it matters (consequences)
- **Recommendation**: How to fix it (specific action)
- **Confidence**: How certain (High/Med/Low)

### Low Severity Issues (Nice to Have)
*Minor documentation improvements, stylistic suggestions*

For each issue:
- **Location**: Which section/pattern has the issue
- **Problem**: What's wrong (referencing specific DDD principle from checklists)
- **Impact**: Why it matters (consequences)
- **Recommendation**: How to fix it (specific action)
- **Confidence**: How certain (High/Med/Low)

### Summary Statistics
Provide a summary of:
- Total issues found by severity level
- Issues by pattern type (e.g., 3 Value Object issues, 2 Aggregate issues)
- Completion status of checklists (e.g., Value Objects: 8/9 checks passed)

## Follow-up Options

After presenting results, ask user if they want to:
1. **Auto-fix issues** - Issues that can be automatically corrected
2. **Get guidance on manual fixes** - Detailed instructions for fixes requiring manual intervention
3. **Generate a detailed report** - Full validation report in markdown format
4. **Update DOMAIN.md with fixes** - Apply fixes directly to the domain document
