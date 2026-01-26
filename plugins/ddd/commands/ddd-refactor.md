---
description: Suggest alternative DDD designs for existing DOMAIN.md while preserving original intent
argument-hint: [domain-md-path] [focus-area]
allowed-tools: ["Skill", "Read", "AskUserQuestion"]
---

You are analyzing an existing DOMAIN.md file to suggest alternative DDD design approaches that improve the design while preserving the original intent and purpose.

**IMPORTANT**: This command is NOT about finding problems - it's about suggesting BETTER ways to achieve the same goals. Think "refactoring opportunities" not "design mistakes."

Determine DOMAIN.md location:
- If $1 provided: Use that file path
- If no argument: Look for DOMAIN.md in current directory

Determine focus area:
- If $2 provided: Focus on that area (aggregates, services, events, policies, specifications, etc.)
- If no argument: Analyze entire design for refactoring opportunities

Read the DOMAIN.md file using the Read tool.

Invoke the ddd-domain-design skill to analyze the design and suggest alternative approaches across all tactical DDD patterns.

The analysis will identify refactoring opportunities that:

**1. Extract to Domain Services:**
- **Current**: Logic in aggregate that coordinates multiple aggregates
- **Suggestion**: Extract to domain service for cross-aggregate coordination
- **Benefit**: Cleaner aggregate boundaries, clearer responsibilities
- **Preserves**: Same business logic execution, just better organized

**2. Extract to Domain Events:**
- **Current**: Aggregate directly calls other aggregates/services
- **Suggestion**: Emit domain event, use eventual consistency
- **Benefit**: Decoupling, better scalability, clearer triggers
- **Preserves**: Same end result (all updates happen), just async

**3. Extract to Policies:**
- **Current**: Varying algorithms in aggregate methods (if/else chains)
- **Suggestion**: Extract to policy pattern (Strategy)
- **Benefit**: Open/closed principle, easier to add variations
- **Preserves**: Same algorithm selection logic, more extensible

**4. Extract to Specifications:**
- **Current**: Business queries embedded in repository methods
- **Suggestion**: Extract to specification objects
- **Benefit**: Reusable, composable, testable business rules
- **Preserves**: Same query logic, better organized

**5. Extract to Factories:**
- **Current**: Complex creation logic in aggregate constructor or static method
- **Suggestion**: Extract to factory class or builder
- **Benefit**: Cleaner aggregate, separation of concerns
- **Preserves**: Same creation steps, better encapsulated

**6. Extract to Driven Ports:**
- **Current**: External system calls directly in domain logic
- **Suggestion**: Extract port interface, implement in infrastructure
- **Benefit**: Testability, flexibility, clear dependency
- **Preserves**: Same external interactions, inverted dependency

**7. Split Aggregates:**
- **Current**: Large aggregate managing many concepts
- **Suggestion**: Split into multiple smaller aggregates with events for consistency
- **Benefit**: Better concurrency, clearer boundaries, simpler aggregates
- **Preserves**: Same business invariants (via eventual consistency)

**8. Extract Value Objects:**
- **Current**: Primitives used for domain concepts
- **Suggestion**: Extract to value objects
- **Benefit**: Type safety, encapsulated validation, self-documenting code
- **Preserves**: Same data, better modeled

**9. Introduce Specifications for Invariants:**
- **Current**: Complex invariant validation in aggregate methods
- **Suggestion**: Extract parts to specifications for reusability
- **Benefit**: Reusable rules, composable validation
- **Preserves**: Same invariant enforcement

**10. Introduce Domain Events for State Changes:**
- **Current**: Aggregate methods just return success/failure
- **Suggestion**: Emit events for significant state changes
- **Benefit**: Extensibility, decoupling side effects
- **Preserves**: Same state changes, plus notification

Present refactoring suggestions to user:

For each suggestion, provide:

**Current Design:**
- What exists now
- Current pattern being used
- How it works currently

**Suggested Alternative:**
- What to change to
- Alternative pattern to use
- How it would work

**Comparison:**
| Aspect | Current | Alternative |
|--------|---------|-------------|
| Complexity | [How complex] | [How complex] |
| Coupling | [Tight/Loose] | [Tighter/Looser] |
| Testability | [Easy/Hard] | [Easier/Harder] |
| Extensibility | [Easy/Hard] | [Easier/Harder] |
| Performance | [Characteristic] | [Characteristic] |

**Impact Analysis:**
- **What changes**: Code structure affected
- **What stays same**: Business intent, behavior from user perspective
- **Migration effort**: How hard to refactor (Low/Medium/High)
- **Breaking changes**: Whether public API changes (Yes/No)
- **Confidence**: How certain this is an improvement (High/Medium/Low)

**When to Use This:**
- If you need better testability
- If you anticipate more variations
- If coupling is causing problems
- If aggregates are getting too large
- If you need better performance

**When to Stay with Current:**
- If simplicity is more important
- If performance is critical and current is optimal
- If changes would be too disruptive
- If current design meets all needs

Organize suggestions by:
1. **High Impact, Low Effort** - Quick wins that significantly improve design
2. **High Impact, High Effort** - Major improvements worth the investment
3. **Low Impact, Low Effort** - Nice-to-have improvements
4. **Low Impact, High Effort** - Probably not worth it

Ask user which suggestions they want to:
1. Apply automatically (update DOMAIN.md with alternative design)
2. See detailed implementation guidance
3. Discuss trade-offs further
4. Skip for now
