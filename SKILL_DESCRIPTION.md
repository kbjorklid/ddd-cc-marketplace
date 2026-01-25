egates, value objects, ubiquitous language, or invariants.
---

# Skill description

This skill is about creating or modifying a domain.md document. The document should describe a microservices or modules domain using tactical domain-driven design patterns such as aggregates, entities, and value objects. This skill should ensure a specific format for the document and adherence to domain-driven design rules.

The main target audience for domain.md is coding agents that will create the code. The abstraction level should be one step above code, so no code snippets or technical details should be included. The reviewers of the document will be professional software developers who have knowledge about domain-driven design, so no need to explain things for non-professionals. Text should be kept succinct, no need to explain things that would be obvious to senior developers.

# When to use

- Designing a new domain model or context (a new DOMAIN.md)
- Updating an existing domain model document with new design
- Updating an existing domain model based on changes made to code (domain model is updated to correspond with the code)
- Creating a domain model to design a refactoring a code base to follow domain-driven design principles


# Document format

## Document structure

1. Title and introduction - what this domain is about
2. Ubiquitous language - Glossary of domain terms
3. Class diagrams - Mermaid diagrams showing the model
4. Types - Descriptions of types in the class diagrams (only those that are not self-evident)
5. Design explanations - External types and cross-context references

## Complete template

[square parenthesis] are used in the template for template instructions / placeholders.

````markdown

# [Domain name] Domain Design

[1-4 sentence description of what this domain model represents and its purpose]

## Ubiquitous Language

| Term | Definition |
|------|------------|
| [Term 1] | [Clear, concise definition of the domain term] |
| [Term 2] | [Clear, concise definition of the domain term] |

## Class Diagrams

[Mermaid diagram]

## Types

[Only describe types whose purpose is not self evident or ]

### [AggregateRoot]

[Purpose and responsibilities] 

### [SomeEntity]

[Purpose and responsibilities] 

### [SomeValueObject]

[Purpose and responsibilities] 

## Design Explanations

[Add third-level headers and plain text explanations of non-obvious design choices to help user read the class diagram]

### [Some design explanation header/some architectural choice header]

[Design explanation - a few sentences. Other structures or mermaid diagrams are allowed, but not encouraged (keep it succinct and to-the-point)]

## Invariants 

### [Aggregate name] Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| XX-1 | [invariant description] | [Optional clarification] |
| XX-2 | [invariant description] | [Optional clarification] |

### [Entity name] Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| YY-1 | [invariant description] | [Optional clarification] |
| YY-2 | [invariant description] | [Optional clarification] |


## Future Considerations

[Free-form content of ideas that have been not yet incorporated to the design]

````