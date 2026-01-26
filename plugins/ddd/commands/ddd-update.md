---
description: Update existing DOMAIN.md with new domain concepts or changes
argument-hint: [section-to-update] [change-description]
allowed-tools: ["Skill", "Read", "Write", "AskUserQuestion"]
---

You are updating an existing domain design document.

First, read the existing DOMAIN.md file using the Read tool.

If no arguments provided, ask the user using the AskUserQuestion tool:
- Which section needs updating?
  - **ubiquitous-language**: Add/update domain terminology
  - **class-diagrams**: Add new patterns, update relationships
  - **types**: Add new aggregates, entities, value objects, services, events, repositories, ports, factories, specifications, policies
  - **invariants**: Add/update business rules
  - **design-explanations**: Document new design decisions
  - **future-considerations**: Add ideas not yet implemented
- What new concepts to add? (specify which DDD pattern type)
- What existing concepts to modify? (which pattern and what changes)

Invoke the ddd-domain-design skill to ensure updates follow proper DDD patterns for all 10 tactical DDD types.

Apply targeted updates to DOMAIN.md:
- Preserve existing structure
- Add new concepts in appropriate sections (aggregates, entities, VOs, services, events, repositories, ports, factories, specifications, policies)
- Update Mermaid diagrams if types/relationships change
- Maintain invariant ID numbering
- Update ubiquitous language with new terms
- Ensure proper DDD stereotypes in diagrams

After updating, show a diff summary highlighting:
- What domain concepts were added/modified (by pattern type)
- Diagram changes made
- Invariant updates
- Ubiquitous language changes
