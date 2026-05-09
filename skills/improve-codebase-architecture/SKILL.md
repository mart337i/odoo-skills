---
name: improve-codebase-architecture
description: Find deepening opportunities in a codebase, informed by the domain language in CONTEXT.md and the decisions in docs/adr/. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, or make a codebase more testable and AI-navigable.
---

# Improve Codebase Architecture

Surface architectural friction and propose deepening opportunities: refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability.

## Glossary

Use these terms exactly in every suggestion. Full definitions are in [LANGUAGE.md](LANGUAGE.md).

- Module: anything with an interface and an implementation.
- Interface: everything a caller must know to use the module correctly.
- Implementation: the code inside.
- Depth: leverage at the interface. Deep means high leverage; shallow means the interface is nearly as complex as the implementation.
- Seam: where an interface lives; a place behavior can be altered without editing in place.
- Adapter: a concrete thing satisfying an interface at a seam.
- Leverage: what callers get from depth.
- Locality: what maintainers get from depth.

Key principles:

- Deletion test: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across many callers, it was earning its keep.
- The interface is the test surface.
- One adapter means a hypothetical seam. Two adapters means a real seam.

## Process

### 1. Explore

Read the project's domain glossary and any ADRs in the area first. Then inspect the codebase organically and note where understanding creates friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules shallow?
- Where have pure functions been extracted only for testability, while real bugs hide in how they are called?
- Where do tightly-coupled modules leak across seams?
- Which parts are untested, or hard to test through their current interface?

Apply the deletion test to anything suspected of being shallow.

### 2. Present Candidates

Present a numbered list of deepening opportunities. For each candidate include:

- Files: which files/modules are involved.
- Problem: why current architecture causes friction.
- Solution: plain-English description of what would change.
- Benefits: explain locality, leverage, and how tests would improve.

Use `CONTEXT.md` vocabulary for the domain and [LANGUAGE.md](LANGUAGE.md) vocabulary for architecture. If a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting that ADR.

Do not propose interfaces yet. Ask: "Which of these would you like to explore?"

### 3. Grilling Loop

Once the user picks a candidate, walk the design tree: constraints, dependencies, the shape of the deepened module, what sits behind the seam, and what tests survive.

Side effects happen inline as decisions crystallize:

- If naming a deepened module after a concept not in `CONTEXT.md`, add the term using [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md).
- If sharpening a fuzzy term, update `CONTEXT.md` immediately.
- If the user rejects the candidate with a load-bearing reason, offer an ADR using [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md).
- If exploring alternative interfaces, use [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).
