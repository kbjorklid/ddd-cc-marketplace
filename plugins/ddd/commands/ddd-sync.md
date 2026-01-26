---
description: Sync DOMAIN.md with recent code changes to keep documentation current
argument-hint: [commit-range] [optional-sections]
allowed-tools: ["Skill", "Read", "Grep", "Glob", "Bash", "Task", "AskUserQuestion"]
---

You are synchronizing DOMAIN.md with recent code changes to keep the domain design documentation aligned with the implementation.

**IMPORTANT**: This command is for code that ALREADY follows DDD patterns. The code-change-analyzer agent will detect changes across all tactical DDD pattern types and update DOMAIN.md accordingly.

Determine what to analyze:
- If $1 provided (commit hash/range): Use git diff for that range
- If no argument: Analyze recent uncommitted changes

Use Bash tool to get git diff: git diff $1

Launch the code-change-analyzer agent using the Task tool to analyze changes across ALL DDD patterns:

The agent will detect changes in:
- **Core Building Blocks**: New/modified/deleted aggregates, entities, value objects
- **Domain Logic**: New/modified/deleted domain services, domain events
- **Domain Interfaces**: New/modified/deleted repository interfaces, driven ports
- **Supporting Concepts**: New/modified/deleted factories, specifications, policies

The agent will:
- Read existing DOMAIN.md FIRST to understand current domain context
- Detect new domain concepts added to code
- Find modified domain concepts (behavior, state, invariants changed)
- Identify deleted domain concepts
- Analyze relationship changes between concepts
- Parse invariant changes (validation logic updates)
- Extract ubiquitous language evolution (new terms, renamed concepts)

Agent will return structured change analysis with recommended updates.

Invoke the ddd-domain-design skill to update DOMAIN.md:
- Add new patterns found in code (all 10 types)
- Update relationships between concepts
- Add/remove invariants based on validation changes
- Update ubiquitous language with new/refactored terms
- Update Mermaid diagrams to reflect changes

Apply updates to DOMAIN.md and show synchronization summary:
- What domain concepts changed (by pattern type)
- What updates were applied to each section
- Any conflicts or ambiguities found
- Manual review recommendations (if confidence is low)
