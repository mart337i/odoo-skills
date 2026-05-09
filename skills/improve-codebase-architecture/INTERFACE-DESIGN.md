# Interface Design

When the user wants to explore alternative interfaces for a chosen deepening candidate, use this process. Based on "Design It Twice": the first idea is unlikely to be best.

## Process

### 1. Frame The Problem Space

Before exploring options, explain:

- Constraints any new interface must satisfy.
- Dependencies it relies on, and which category they fall into from [DEEPENING.md](DEEPENING.md).
- A rough illustrative code sketch to ground the constraints, not a proposal.

### 2. Generate Alternatives

Create at least three radically different interfaces for the deepened module:

- Minimize the interface: aim for 1-3 entry points max.
- Maximize flexibility: support many use cases and extension.
- Optimize for the most common caller: make the default case trivial.
- If applicable, design around ports and adapters for cross-seam dependencies.

Each design should include:

- Interface: types, methods, parameters, invariants, ordering, and error modes.
- Usage example showing how callers use it.
- What implementation details it hides behind the seam.
- Dependency strategy and adapters.
- Trade-offs: where leverage is high and where it is thin.

### 3. Present And Compare

Present designs sequentially, then compare by depth, locality, and seam placement. Give an opinionated recommendation. If elements from different designs combine well, propose a hybrid.
