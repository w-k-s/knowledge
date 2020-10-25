# Documenting Architecture Decisions

## Problem

- One of the hardest things to track during the life of a project is the motivation behind certain decisions
- When developers don't know the motivation behind a decision, they may
  
  1. **Blindly accept the decision**: OK, if the decision is still valid. Not OK, if the context has changed and the decision should really be revisited.
  2. **Blindly change the decision**: OK if the decision needs to be reversed. Not OK, damaging the project's overall value without realizing it

## Solution

- Keep a collection of records for "architecturally significant" decisions: those that affect the structure, non-functional characteristics, dependencies, interfaces, or construction techniques.
- An architecture decision record is a short text file in the project repository under doc/arch/adr-NNN.md. 

**Rules for ARDs**
1. We should use a lightweight text formatting language like Markdown or Textile
2. ADRs will be numbered sequentially and monotonically. Numbers will not be reused e.g. `0001-record-architecture-decisions.md`
3. If a decision is reversed, we will keep the old one around, but mark it as superseded
4. one or two pages long

**Format**

| Section      | Purpose                                                                                                                                                                                                                                                                     | Example                                      |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------|
| Title        | Short noun phrases describing the decision                                                                                                                                                                                                                                  | e.g. ADR 1: Deployment on Ruby on Rails 3.0. |
| Context      | - Describes the forces at play, includingtechnological, political, social, and project local - The language in this section is value-neutral. It is simply describing fact                                                                                                  |                                              |
| Decision     | - Response to these forces. It is stated in full sentences, with active voice. "We will …"                                                                                                                                                                                  |                                              |
| Status       | One of: - "Proposed": if the project stakeholders haven't agreedwith it yet. - "Accepted": once it is agreed - "Superseded": Decision has been reversed (in this case, provide reference to new decision)                                                                   |                                              |
| Consequences | Describes the resulting context, after applying the decision.  All consequences should be listed here, not just the "positive" ones. A particular decision may have positive, negative, and neutral consequences, but all of them affect the team and project in the future |                                              |

## Further Reading

- [Introduction to Architecture Decision Records](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Homepage of the ADR GitHub organization](https://adr.github.io/)
- [adr-tool to generate architecture decision records](https://github.com/npryce/adr-tools)
