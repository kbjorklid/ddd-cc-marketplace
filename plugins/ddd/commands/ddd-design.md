---
description: Create a new domain design document using tactical DDD patterns
argument-hint: [domain-name] [optional-context]
allowed-tools: ["Skill", "AskUserQuestion"]
---

You are creating a new domain design document from scratch using tactical Domain-Driven Design patterns.

This command helps design proper DDD models covering all 10 tactical patterns:
- **Core Building Blocks**: Aggregates, Entities, Value Objects
- **Domain Logic**: Domain Services, Domain Events
- **Domain Interfaces**: Repository Interfaces, Driven Ports
- **Supporting Concepts**: Factories, Specifications, Policies

If no arguments provided, ask the user using the AskUserQuestion tool:
- What is the domain name?
- What is the primary purpose of this domain?
- Any specific DDD patterns to focus on? (aggregates, events, services, etc.)

Invoke the ddd-domain-design skill using the Skill tool to guide through domain design.

At this point, when you have an initial idea of what you're doing, interview the user thoroughly using the AskUserQuestion tool to:
- Remove any ambiguities
- Gather more context about the domain
- Identify all core aggregates and their boundaries
- Understand cross-aggregate interactions (for domain services and events)
- Identify external dependencies (for ports)
- Clarify complex business rules (for specifications and policies)

The skill will ensure:
- Proper DOMAIN.md structure (all required sections)
- Correct Mermaid diagram syntax with DDD stereotypes (<<Aggregate Root>>, <<Entity>>, <<Value Object>>, etc.)
- Comprehensive ubiquitous language glossary
- Documented invariants with IDs
- Design explanations for non-obvious choices
- Coverage of all relevant tactical DDD patterns

After skill completes, save the output as DOMAIN.md in the current directory.

Quality checks before saving:
- All sections from template present
- Mermaid diagram uses proper DDD stereotypes for all pattern types
- Invariants documented with IDs
- Ubiquitous language comprehensive
- Design explanations cover non-obvious decisions
