# CONTEXT.md Format

## Structure

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Order**:
{A concise description of the term}
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

## Relationships

- An **Order** produces one or more **Invoices**

## Example Dialogue

> **Dev:** "When a **Customer** places an **Order**, do we create the **Invoice** immediately?"
> **Domain expert:** "No, an **Invoice** is only generated once a **Fulfillment** is confirmed."

## Flagged Ambiguities

- "account" was used to mean both **Customer** and **User**; resolved: these are distinct concepts.
```

## Rules

- Be opinionated. When multiple words exist for the same concept, pick the best one and list the others as aliases to avoid.
- Flag conflicts explicitly in `Flagged Ambiguities` with a clear resolution.
- Keep definitions tight. One sentence max. Define what it is, not what it does.
- Show relationships with bold term names and cardinality where obvious.
- Only include terms specific to the project's context. General programming concepts do not belong.
- Group terms under subheadings when natural clusters emerge.
- Write an example dialogue that demonstrates how terms interact naturally.

## Single Vs Multi-Context Repos

Single context: one `CONTEXT.md` at the repo root.

Multiple contexts: a `CONTEXT-MAP.md` at the repo root lists the contexts, where they live, and how they relate.

When multiple contexts exist, infer which one the current topic relates to. If unclear, ask.
