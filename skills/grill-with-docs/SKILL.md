---
name: grill-with-docs
description: Grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates documentation (CONTEXT.md, ADRs) inline as decisions crystallise. Use when the user wants to stress-test a plan against their project's language and documented decisions.
disable-model-invocation: true
---

# Grill With Docs

Interview the user relentlessly about every aspect of this plan until there is shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one. For each question, provide your recommended answer.

Ask exactly one question at a time, waiting for feedback before continuing.

If a question can be answered by exploring the codebase, inspect the codebase instead of asking.

## Domain Awareness

During codebase exploration, also look for existing documentation:

```text
/
├── CONTEXT.md
├── docs/
│   └── adr/
└── src/
```

If a `CONTEXT-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives.

Create files lazily. If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first ADR is needed.

## During The Session

### Challenge Against The Glossary

When the user uses a term that conflicts with existing language in `CONTEXT.md`, call it out immediately.

### Sharpen Fuzzy Language

When the user uses vague or overloaded terms, propose a precise canonical term.

### Discuss Concrete Scenarios

When domain relationships are being discussed, stress-test them with specific scenarios that probe edge cases and force precise boundaries between concepts.

### Cross-Reference With Code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it.

### Update CONTEXT.md Inline

When a term is resolved, update `CONTEXT.md` immediately. Use [CONTEXT-FORMAT.md](CONTEXT-FORMAT.md).

Do not couple `CONTEXT.md` to implementation details. Only include terms that are meaningful to domain experts.

### Offer ADRs Sparingly

Only offer to create an ADR when all three are true:

1. Hard to reverse: the cost of changing your mind later is meaningful.
2. Surprising without context: a future reader will wonder why it was done this way.
3. Real trade-off: there were genuine alternatives and one was picked for specific reasons.

If any condition is missing, skip the ADR. Use [ADR-FORMAT.md](ADR-FORMAT.md).
