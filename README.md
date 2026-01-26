# DDD Specification Marketplace

A Domain-Driven Design marketplace for specifying microservices and modules with DDD tactical patterns.

## Overview

The DDD plugin provides a comprehensive toolkit for creating and maintaining domain design documents using tactical Domain-Driven Design patterns. It includes slash commands, agents, and skills that work together to help you design, document, and refactor domain models.

## Components

- **Skills**: The `ddd-domain-design` skill provides deep DDD expertise for creating structured domain documents
- **Commands**: User-facing slash commands for common domain design workflows
- **Agents**: Autonomous specialists for analyzing codebases and detecting changes

## Slash Commands

### `/ddd-design` - Create Domain Design

Create a new DOMAIN.md document from user specifications.

**Usage:**
```bash
/ddd-design                                    # Interactive mode
/ddd-design Order Management                   # With domain name
/ddd-design Order Management Focus on order lifecycle  # With context
```

**What it does:**
- Guides you through domain design with clarifying questions
- Invokes the `ddd-domain-design` skill for DDD expertise
- Creates a structured DOMAIN.md document with all required sections
- Ensures proper Mermaid diagram syntax with DDD stereotypes
- Documents invariants, ubiquitous language, and design explanations

**When to use:**
- Starting a new domain model from scratch
- Creating a blueprint for implementing a domain
- Designing a bounded context

---

### `/ddd-update` - Update Existing Design

Update an existing DOMAIN.md with new concepts or changes.

**Usage:**
```bash
/ddd-update                                    # Interactive mode
/ddd-update Add Payment aggregate              # Specific update
/ddd-update invariants Add payment validation  # Section-specific
```

**What it does:**
- Reads your existing DOMAIN.md
- Asks what sections need updating
- Invokes the `ddd-domain-design` skill for DDD expertise
- Applies targeted updates while preserving structure
- Shows a diff summary of changes

**When to use:**
- Adding new aggregates, entities, or value objects
- Updating invariants or business rules
- Modifying class diagrams
- Refining ubiquitous language

---

### `/ddd-extract` - Extract from Code

Analyze existing code to create or update DOMAIN.md for refactoring guidance.

**Usage:**
```bash
/ddd-extract                    # Analyze current directory
/ddd-extract src/               # Analyze specific directory
/ddd-extract src/ "Order Management"  # With domain name
```

**What it does:**
- Launches the `codebase-analyzer` agent for deep code analysis
- Extracts aggregates, entities, value objects from code
- Identifies invariants from validation logic
- Builds ubiquitous language glossary
- Detects DDD anti-patterns and violations
- Creates DOMAIN.md with refactoring recommendations

**When to use:**
- Analyzing existing codebases that don't follow DDD
- Creating refactoring roadmaps
- Understanding domain structure in legacy code
- Documenting implicit domain models

**Output includes:**
- Extracted Domain (what was found in code)
- DDD Refactoring (recommendations)
- Proposed Design (restructured model)
- Migration Path (steps to refactor)

---

### `/ddd-sync` - Sync with Code Changes

Update DOMAIN.md to reflect recent code modifications.

**Usage:**
```bash
/ddd-sync              # Sync uncommitted changes
/ddd-sync HEAD~5       # Sync last 5 commits
/ddd-sync abc123..def456  # Sync specific commit range
```

**What it does:**
- Analyzes git diff to detect code changes
- Launches the `code-change-analyzer` agent to identify impact
- Detects new/modified/deleted domain concepts
- Updates DOMAIN.md based on changes
- Shows synchronization summary

**When to use:**
- Keeping documentation synchronized with code
- After implementing domain model changes
- Reviewing impact of code changes on domain design
- Maintaining up-to-date domain documentation

---

## Agents

### `codebase-analyzer`

Analyzes existing codebases to extract domain concepts for DDD modeling.

**When it runs:**
- Invoked by `/ddd-extract` command
- Proactively when analyzing code structure
- On explicit request for domain analysis

**What it does:**
- Identifies aggregates, entities, value objects
- Extracts invariants from validation logic
- Builds ubiquitous language glossary
- Detects DDD anti-patterns
- Recommends refactoring

**Key capabilities:**
- Cross-language support (TypeScript, C#, Java, Python)
- Heuristic pattern recognition
- DDD violation detection (anemic models, primitive obsession, etc.)
- Actionable refactoring recommendations

---

### `code-change-analyzer`

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
- Recommends specific updates

**Key capabilities:**
- Git diff parsing
- Change impact analysis
- Confidence levels for recommendations
- Structured update instructions

---

## Skill

### `ddd-domain-design`

The core skill that provides DDD expertise for creating and modifying domain design documents.

**What it provides:**
- Comprehensive DDD pattern guides (Value Objects, Entities, Aggregates, etc.)
- DOMAIN.md template structure (7 required sections)
- Mermaid diagram guidelines with DDD stereotypes
- Result pattern integration for type-safe error handling
- Progressive disclosure (quick reference + detailed guides)

**When it's used:**
- Automatically invoked by all slash commands
- Ensures DDD best practices are followed
- Validates domain design decisions
- Provides domain pattern guidance

**Pattern guides available:**
- Value Objects, Entities, Aggregates
- Domain Services, Domain Events
- Repositories, Driven Ports
- Factories, Specifications, Policies
- Result Pattern

---

## Quick Start Examples

### Create a new domain design

```bash
/ddd-design Order Management
```

The command will ask you questions about:
- Domain purpose and boundaries
- Core aggregates and entities
- Business invariants
- Ubiquitous language

Result: A structured `DOMAIN.md` document ready for implementation.

---

### Analyze existing code for refactoring

```bash
/ddd-extract src/
```

The agent will:
1. Scan all code files in `src/`
2. Identify domain concepts (even if not explicitly DDD)
3. Detect anti-patterns and violations
4. Create DOMAIN.md with refactoring recommendations

Result: A roadmap for transforming your code to proper DDD.

---

### Update existing domain design

```bash
/ddd-update Add Payment aggregate with authorization flow
```

The command will:
1. Read your existing DOMAIN.md
2. Guide you through adding the Payment aggregate
3. Update class diagrams with proper relationships
4. Document payment-specific invariants

Result: Updated DOMAIN.md with new Payment aggregate.

---

### Sync docs after code changes

```bash
/ddd-sync HEAD~3
```

The command will:
1. Analyze changes in the last 3 commits
2. Detect new/modified/deleted domain concepts
3. Update DOMAIN.md to reflect changes
4. Show synchronization summary

Result: DOMAIN.md synchronized with your latest code.

---

## File Structure

```
ddd-spec/
├── plugins/
│   └── ddd/
│       ├── commands/              # Slash commands
│       │   ├── ddd-design.md
│       │   ├── ddd-update.md
│       │   ├── ddd-extract.md
│       │   └── ddd-sync.md
│       ├── agents/                # Autonomous agents
│       │   ├── codebase-analyzer.md
│       │   └── code-change-analyzer.md
│       └── skills/                # DDD expertise
│           └── ddd-domain-design/
│               ├── SKILL.md
│               ├── guides/        # Pattern guides
│               └── examples/      # Working examples
```

---

## Workflow Examples

### Greenfield Domain Design

```
1. /ddd-design [domain-name]
   ↓
2. Answer clarifying questions
   ↓
3. Review generated DOMAIN.md
   ↓
4. Implement domain model
```

### Brownfield Refactoring

```
1. /ddd-extract [code-directory]
   ↓
2. Review extracted concepts and violations
   ↓
3. Review proposed DDD refactoring
   ↓
4. Follow migration path
   ↓
5. /ddd-sync to keep docs updated during refactoring
```

### Iterative Domain Evolution

```
1. Start with /ddd-design
   ↓
2. Implement initial model
   ↓
3. /ddd-update to add concepts as you learn
   ↓
4. /ddd-sync after code changes
   ↓
5. Repeat as domain understanding grows
```

---

## Best Practices

1. **Start with /ddd-design** for new domains
2. **Use /ddd-update** when adding/modifying concepts
3. **Run /ddd-extract** before refactoring existing code
4. **Use /ddd-sync** regularly to keep docs current
5. **Review generated DOMAIN.md** before saving
6. **Follow the migration path** when refactoring
7. **Update ubiquitous language** as domain understanding grows

---

## Contributing

This plugin follows tactical DDD principles from:
- *Domain-Driven Design* by Eric Evans
- *Implementing Domain-Driven Design* by Vaughn Vernon
- *Patterns, Principles, and Practices of Domain-Driven Design* by Scott Millett

---

## License

MIT License - See LICENSE file for details
