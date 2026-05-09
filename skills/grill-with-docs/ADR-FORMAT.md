# ADR Format

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`, etc.

Create the `docs/adr/` directory lazily, only when the first ADR is needed.

## Template

```md
# {Short title of the decision}

{1-3 sentences: what is the context, what did we decide, and why.}
```

That is enough. An ADR can be a single paragraph. The value is in recording that a decision was made and why, not in filling out sections.

## Optional Sections

Only include these when they add genuine value:

- Status frontmatter: `proposed`, `accepted`, `deprecated`, or `superseded by ADR-NNNN`.
- Considered Options: only when rejected alternatives are worth remembering.
- Consequences: only when non-obvious downstream effects need to be called out.

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.

## When To Offer An ADR

All three must be true:

1. Hard to reverse: the cost of changing your mind later is meaningful.
2. Surprising without context: a future reader will wonder why it was done this way.
3. Real trade-off: there were genuine alternatives and one was picked for specific reasons.
