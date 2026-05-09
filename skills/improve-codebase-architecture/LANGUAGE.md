# Language

Shared vocabulary for every suggestion this skill makes. Use these terms exactly.

## Terms

**Module**:
Anything with an interface and an implementation. Applies equally to a function, class, package, or tier-spanning slice.
_Avoid_: unit, component, service.

**Interface**:
Everything a caller must know to use the module correctly. Includes type signature, invariants, ordering constraints, error modes, required configuration, and performance characteristics.
_Avoid_: API, signature.

**Implementation**:
What is inside a module: its body of code. Reach for **Adapter** when the seam is the topic; **Implementation** otherwise.

**Depth**:
Leverage at the interface: the amount of behavior a caller or test can exercise per unit of interface they must learn. A module is **deep** when a large amount of behavior sits behind a small interface. A module is **shallow** when the interface is nearly as complex as the implementation.

**Seam**:
A place where behavior can be altered without editing in that place. The location at which a module's interface lives.
_Avoid_: boundary.

**Adapter**:
A concrete thing that satisfies an interface at a seam. Describes role, not substance.

**Leverage**:
What callers get from depth: more capability per unit of interface they must learn.

**Locality**:
What maintainers get from depth: change, bugs, knowledge, and verification concentrated in one place.

## Principles

- Depth is a property of the interface, not the implementation.
- Deletion test: if deleting the module makes complexity vanish, it was a pass-through; if complexity reappears across many callers, it was earning its keep.
- The interface is the test surface.
- One adapter means a hypothetical seam. Two adapters means a real one.
- Do not introduce a seam unless something actually varies across it.
