# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **DDD Specification Marketplace** - a Claude Code plugin for creating domain design documents using tactical Domain-Driven Design patterns. The plugin provides skills that guide users through creating structured `DOMAIN.md` documents that serve as blueprints for implementing domain models.

**Target Audience:** The domain.md documents are primarily for coding agents that will generate code from these specifications. Reviewers are professional software developers familiar with DDD, so explanations should not cover basic DDD concepts.

## Plugin Architecture

This is a **Claude Code Plugin** that extends Claude's capabilities with domain-driven design expertise. The plugin is registered as a local marketplace and contains skills, slash commands, and agents.

### Directory Structure

```
ddd-spec/
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace manifest
├── plugins/
│   └── ddd/                      # DDD Plugin
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin manifest (v0.2.0)
│       ├── commands/             # Slash commands
│       │   ├── ddd-design.md     # Create domain design from specs
│       │   ├── ddd-update.md     # Update existing domain design
│       │   ├── ddd-extract.md    # Extract domain from code
│       │   └── ddd-sync.md       # Sync docs with code changes
│       ├── agents/               # Autonomous agents
│       │   ├── codebase-analyzer.md        # Analyze code for domain concepts
│       │   └── code-change-analyzer.md    # Analyze changes for sync
│       └── skills/
│           └── ddd-domain-design/
│               ├── SKILL.md       # Main skill definition
│               ├── guides/        # Comprehensive DDD pattern guides
│               │   ├── INDEX.md   # Guide index
│               │   ├── value-objects.md
│               │   ├── entities.md
│               │   ├── aggregates.md
│               │   ├── domain-events.md
│               │   ├── domain-services.md
│               │   ├── repositories.md
│               │   ├── ports.md
│               │   ├── factories.md
│               │   ├── specifications.md
│               │   └── policies.md
│               └── examples/      # Example domain designs
│                   ├── order-domain-example.md
│                   └── invariants-examples.md
└── .claude/
    └── settings.local.json       # Local Claude settings
```

## Core Skills

### `ddd-domain-design` Skill

**When to use:** Invoke when users request:
- "Create a domain.md" or "design a domain model"
- "Create a DOMAIN.md document"
- References to tactical DDD patterns (aggregates, entities, value objects, invariants, ubiquitous language)

**What it does:** Creates structured domain design documents following tactical DDD principles with:
- Ubiquitous Language glossary
- Mermaid class diagrams with proper DDD stereotypes
- Type descriptions (aggregates, entities, value objects)
- Design explanations
- Business invariants documentation
- Future considerations section

**Key principle:** Always invoke this skill via the Skill tool when users request domain design work - do not attempt to create domain documents without it.

## Slash Commands

The plugin provides user-facing slash commands for common domain design workflows. All commands invoke the `ddd-domain-design` skill to ensure DDD best practices.

### `/ddd-design` - Create Domain Design

Create a new DOMAIN.md document from user specifications.

**Usage:**
- `/ddd-design` - Interactive mode
- `/ddd-design [domain-name]` - With domain name
- `/ddd-design [domain-name] [context]` - With additional context

**What it does:**
- Guides user through domain design with clarifying questions
- Invokes `ddd-domain-design` skill for DDD expertise
- Creates structured DOMAIN.md with all required sections
- Ensures proper Mermaid diagram syntax with DDD stereotypes

**When to use:** Starting new domain models, creating bounded context designs

---

### `/ddd-update` - Update Existing Design

Update an existing DOMAIN.md with new concepts or changes.

**Usage:**
- `/ddd-update` - Interactive mode
- `/ddd-update [section] [change-description]` - Specific update

**What it does:**
- Reads existing DOMAIN.md
- Asks what sections need updating
- Invokes `ddd-domain-design` skill
- Applies targeted updates while preserving structure
- Shows diff summary of changes

**When to use:** Adding aggregates/entities/VOs, updating invariants, modifying diagrams

---

### `/ddd-extract` - Extract from Code

Analyze existing code to create DOMAIN.md for refactoring guidance.

**Usage:**
- `/ddd-extract` - Analyze current directory
- `/ddd-extract [directory]` - Analyze specific directory
- `/ddd-extract [directory] [domain-name]` - With domain name

**What it does:**
- Launches `codebase-analyzer` agent for deep code analysis
- Extracts aggregates, entities, value objects from code
- Identifies invariants and ubiquitous language
- Detects DDD anti-patterns
- Creates DOMAIN.md with refactoring recommendations

**When to use:** Analyzing non-DDD codebases, creating refactoring roadmaps

**Output includes:**
- Extracted Domain (what was found)
- DDD Refactoring (recommendations)
- Proposed Design (restructured model)
- Migration Path (steps to refactor)

---

### `/ddd-sync` - Sync with Code Changes

Update DOMAIN.md to reflect recent code modifications.

**Usage:**
- `/ddd-sync` - Sync uncommitted changes
- `/ddd-sync [commit-range]` - Sync specific commits

**What it does:**
- Analyzes git diff for code changes
- Launches `code-change-analyzer` agent
- Detects new/modified/deleted domain concepts
- Updates DOMAIN.md based on changes
- Shows synchronization summary

**When to use:** Keeping docs synchronized with code, after implementing changes

## Agents

The plugin includes autonomous agents that perform complex analysis tasks. Agents are invoked by commands but can also trigger proactively based on context.

### `codebase-analyzer` Agent

Analyzes existing codebases to extract domain concepts for DDD modeling.

**When it runs:**
- Invoked by `/ddd-extract` command
- Proactively when analyzing code structure
- On explicit request for domain analysis

**What it does:**
- Identifies aggregates, entities, value objects
- Extracts invariants from validation logic
- Builds ubiquitous language glossary
- Detects DDD anti-patterns (anemic models, primitive obsession, etc.)
- Provides refactoring recommendations

**Key capabilities:**
- Cross-language support (TypeScript, C#, Java, Python)
- Heuristic pattern recognition
- Actionable refactoring recommendations

**Tools:** Read, Grep, Glob, Bash

---

### `code-change-analyzer` Agent

Analyzes code changes to determine DOMAIN.md updates needed.

**When it runs:**
- Invoked by `/ddd-sync` command
- After git commits modifying domain code
- On explicit request to sync documentation

**What it does:**
- Detects new domain concepts
- Identifies modified concepts
- Finds removed concepts
- Analyzes invariant changes
- Recommends specific updates with confidence levels

**Key capabilities:**
- Git diff parsing
- Change impact analysis
- Confidence levels (High/Medium/Low)
- Structured update instructions

**Tools:** Read, Grep, Glob, Bash

## Integration Patterns

### Command → Skill Flow

Commands invoke the `ddd-domain-design` skill via the Skill tool:
1. Command receives user request
2. Command invokes skill with context
3. Skill provides DDD expertise and guidance
4. Command applies skill output to create/update DOMAIN.md

### Command → Agent → Skill Flow

Commands involving code analysis use agents:
1. Command receives user request
2. Command launches agent via Task tool
3. Agent performs analysis (codebase or changes)
4. Agent returns structured findings
5. Command invokes skill with agent findings
6. Skill transforms findings into DDD patterns
7. Command creates/updates DOMAIN.md

### Example Workflow: `/ddd-extract`

```
User: /ddd-extract src/
  ↓
Command validates directory exists
  ↓
Command launches codebase-analyzer agent (Task tool)
  ↓
Agent scans code, extracts domain concepts
  ↓
Agent returns: aggregates, entities, VOs, invariants, violations
  ↓
Command invokes ddd-domain-design skill (Skill tool)
  ↓
Skill transforms concepts into proper DDD patterns
  ↓
Command presents findings to user for confirmation
  ↓
DOMAIN.md written with refactoring recommendations
```

## Document Structure (DOMAIN.md)

Every domain design must follow this exact structure:

```markdown
# [Domain Name] Domain Design

[Introduction - 1-4 sentences]

## Ubiquitous Language

| Term | Definition |
|------|------------|
| [Term 1] | [Clear, concise definition] |
| [Term 2] | [Clear, concise definition] |

## Class Diagrams

[Mermaid class diagram]

## Types

### [AggregateRoot]
[Purpose and responsibilities]

### [SomeEntity]
[Purpose and responsibilities]

## Design Explanations

### [Design choice header]
[Explanation of non-obvious design decisions]

## Invariants

### [Aggregate name] Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| XX-1 | [invariant description] | [Optional clarification] |

## Future Considerations

[Ideas not yet incorporated]
```

## Key DDD Principles

### Aggregate Design (Four Rules)

1. **Model True Invariants** - Group objects requiring immediate consistency
2. **Design Small Aggregates** - Root + few value objects (avoid mega-aggregates)
3. **Reference by Identity Only** - Hold IDs of other aggregates, not object references
4. **Use Eventual Consistency Outside** - Cross-aggregate updates via domain events

### Value Objects

- Immutable (never change after creation)
- Structural equality (same values = same object)
- No identity (defined by attributes, not ID)
- Always valid (validation in constructor/factory)
- Prevents primitive obsession (e.g., `Money`, `Address`, `EmailAddress`)

### Entities

- Identity-based (same ID = same entity)
- Mutable (state changes over lifecycle)
- Behavioral encapsulation (methods, not public setters)
- Private setters (state changes via business methods)

### Domain Events

- Naming: Past tense (e.g., `OrderPlaced`, `PaymentAuthorized`)
- Content: Minimal data required (aggregate ID + changed fields)
- Publishing: Use Transactional Outbox pattern for atomicity
- Consumers: Must be idempotent (handle duplicate messages)

## Mermaid Diagram Guidelines

### Stereotypes (Required)
- `<<Aggregate Root>>` - Root of consistency boundary
- `<<Entity>>` - Object with identity, mutable
- `<<Value Object>>` - Immutable, defined by attributes
- `<<Enumeration>>` - Fixed set of values
- `<<Domain Service>>` - Stateless operation spanning aggregates

### Relationships
- `*--` Composition (aggregate owns child)
- `-->` Association (reference by ID to other aggregate)

### Cardinality
- `"1"` - Exactly one
- `"*"` - Many (zero or more)
- `"0..1"` - Zero or one
- `"1..*"` - One or more

### Example Pattern
```mermaid
classDiagram
    class Order {
        <<Aggregate Root>>
        +Id: OrderId
        +CustomerId: CustomerId
        +Status: OrderStatus
        +Items: List~OrderItem~
        +Create(customerId: CustomerId) Order
        +AddItem(productId: ProductId, quantity: int) Result
    }
    class OrderItem {
        <<Entity>>
        +Id: OrderItemId
        +ProductId: ProductId
        +Quantity: int
    }
    Order "1" *-- "*" OrderItem : contains
```

## Writing Style

**For domain.md documents:**
- Succinct, technical prose (assume DDD expertise)
- No code snippets (abstraction level above code)
- No basic DDD concept explanations
- Focus on design rationale and trade-offs

**For class diagrams:**
- Use verb phrases for methods (e.g., `confirm()`, not `confirmOrder()`)
- Show return types (e.g., `Result`, `Order`)
- Use types, not primitives (e.g., `Money`, not `decimal`)

## Local Development

### Plugin Registration

This plugin is registered in `.claude/settings.local.json` as a local marketplace:

```json
{
  "extraKnownMarketplaces": {
    "ddd-marketplace": {
      "source": {
        "source": "directory",
        "path": "./"
      }
    }
  }
}
```

The plugin is auto-discovered by Claude Code from the local marketplace directory.

### Plugin Manifests

- **Marketplace manifest:** `.claude-plugin/marketplace.json` - Defines the marketplace metadata and plugin list
- **Plugin manifest:** `plugins/ddd/.claude-plugin/plugin.json` - Defines plugin metadata and skills directory

### Local Settings

Permissions for skill development and Mermaid diagrams are configured in `.claude/settings.local.json`. This file also configures the local marketplace registration.

### Testing the Skill

After making changes to skill files or guides:

1. **Reload Claude Code** - The skill will be auto-discovered from the local marketplace
2. **Test invocation** - Use trigger phrases like "create a domain.md" or "design a domain model"
3. **Validate documentation** - Ensure changes in `guides/` are properly referenced in `SKILL.md`

No build step is required - this is a documentation-only plugin with Markdown files.

## Comprehensive DDD Pattern Guides

The `guides/` directory contains extensive documentation for each tactical DDD pattern. These guides provide:

- **Detailed rules and principles** for each pattern
- **Mermaid diagrams** showing structure and relationships
- **Common anti-patterns** to avoid
- **When to use** decision frameworks
- **Design examples** and comparisons

### Available Guides

| Pattern | Guide | Focus |
|---------|-------|-------|
| Value Objects | `value-objects.md` | Immutability, structural equality, Result pattern for creation |
| Entities | `entities.md` | Identity strategies, lifecycle, behavioral encapsulation |
| Aggregates | `aggregates.md` | Four rules of design, consistency boundaries, small aggregates |
| Domain Services | `domain-services.md` | Cross-aggregate logic, stateless operations, port pattern |
| Domain Events | `domain-events.md` | Eventual consistency, minimal data, transactional outbox |
| Repositories | `repositories.md` | Domain interfaces, specification pattern, Result returns |
| Driven Ports | `ports.md` | External dependencies, hexagonal architecture, adapters |
| Factories | `factories.md` | Complex creation, factory methods vs classes |
| Specifications | `specifications.md` | Business queries, composability, reusable rules |
| Policies | `policies.md` | Encapsulating algorithms, business rules, EventStorming automation |

### Progressive Disclosure Approach

When working with users on domain design:

1. **Quick decisions:** Use the quick reference tables in SKILL.md
2. **Designing specific patterns:** Read the full guide for that pattern
3. **Understanding relationships:** Read multiple related guides (e.g., Aggregates + Domain Events)
4. **Troubleshooting:** Consult anti-patterns sections in guides

### Domain Layer Focus

The guides focus exclusively on **domain layer concerns**:
- Domain model design (entities, value objects, aggregates)
- Domain logic (domain services, behavioral encapsulation)
- Domain interfaces (repository interfaces, driven ports)
- Business invariants and domain events

**Excluded** (infrastructure/application concerns):
- Repository implementations (SQL, NoSQL)
- Adapter implementations (email, payment gateways)
- Application services
- File organization and deployment

## Important Design Constraints

### Result Pattern Integration

The skill emphasizes the **Result pattern** for type-safe error handling without exceptions:

- Value Object `Create` methods return `Result<ValueObject>` for validation
- Entity creation returns `Result<Entity>` for validation failures
- Aggregate behavior methods return `Result` for invariant violations
- Repository `GetById` returns `Result<T>` for not-found cases
- Port/Service operations return `Result<T>` for external failures

This pattern is integrated throughout all DDD patterns documented in the guides.

### Critical Design Rules

1. **No primitive types in diagrams** - Wrap primitives in value objects (use `Money`, not `decimal`)
2. **Small aggregates only** - Aggregate root + 1-2 entities max, prefer just root + value objects
3. **Cross-aggregate references by ID** - Never hold direct object references to other aggregates
4. **Business methods, not setters** - Entities expose behavior (`order.confirm()`), not state manipulation
5. **Invariants in the aggregate** - Business rules enforced by aggregate root, not external services
6. **Anemic domain anti-pattern** - Avoid entities that are mere data containers with all logic in services
