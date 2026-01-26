---
description: Analyze existing legacy code to extract domain concepts and create DOMAIN.md for refactoring
argument-hint: [code-directory] [optional-domain-name]
allowed-tools: ["Skill", "Read", "Grep", "Glob", "Bash", "Task", "AskUserQuestion"]
---

You are analyzing **legacy, tangled code** to extract hidden domain concepts and create a DOMAIN.md document for refactoring guidance.

**IMPORTANT**: This command is for analyzing code that does NOT follow DDD patterns yet. The codebase-analyzer agent will hunt for hidden domain concepts buried in anemic models, god classes, and scattered business logic.

Determine code directory:
- If $1 provided: Use that directory
- If no argument: Use current directory

Verify directory exists and contains code files.

Launch the codebase-analyzer agent using the Task tool to perform deep legacy code analysis:

The agent will identify ALL tactical DDD patterns (even if poorly implemented):
- **Core Building Blocks**: Hidden aggregates (misclassified as services), anemic entities, primitive tuples (should be VOs)
- **Domain Logic**: Scattered business logic (should be domain services), sequential flows (should be domain events)
- **Domain Interfaces**: Missing repository interfaces, tight coupling to external systems (should be ports)
- **Supporting Concepts**: Complex creation (factories), validation rules (specifications), varying algorithms (policies)

The agent will also:
- Extract ubiquitous language from inconsistent naming
- Identify invariants scattered across validation layers
- Detect DDD anti-patterns (anemic models, primitive obsession, god classes, etc.)
- Provide concrete refactoring recommendations

Agent will return structured findings with refactoring roadmap.

Invoke the ddd-domain-design skill with the extracted concepts to:
- Transform findings into proper DDD patterns (all 10 types)
- Create structured DOMAIN.md following template
- Document refactoring recommendations clearly

Present findings to user:
- Show what hidden domain concepts were discovered
- Highlight DDD violations found (anemic models, primitive obsession, etc.)
- Show refactoring roadmap with phases
- Ask for confirmation before writing DOMAIN.md

If user confirms, write DOMAIN.md with:
1. **Extracted Domain**: What was found in the legacy code (current state)
2. **DDD Refactoring**: Anti-patterns detected and how to fix them
3. **Proposed Design**: Restructured model using proper DDD patterns
4. **Migration Path**: Step-by-step refactoring plan (phased approach)

The DOMAIN.md becomes a blueprint for refactoring the legacy codebase to proper DDD.
