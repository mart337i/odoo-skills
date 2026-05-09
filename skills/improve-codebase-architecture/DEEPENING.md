# Deepening

How to deepen a cluster of shallow modules safely, given its dependencies. Assumes the vocabulary in [LANGUAGE.md](LANGUAGE.md).

## Dependency Categories

### 1. In-Process

Pure computation, in-memory state, no I/O. Always deepenable. Merge the modules and test through the new interface directly. No adapter needed.

### 2. Local-Substitutable

Dependencies with local test stand-ins, such as PGLite for Postgres or an in-memory filesystem. Deepenable if the stand-in exists. The seam is internal; no port at the module's external interface.

### 3. Remote But Owned

Your own services across a network boundary. Define a port at the seam. The deep module owns the logic; transport is injected as an adapter. Tests use an in-memory adapter. Production uses an HTTP, gRPC, or queue adapter.

### 4. True External

Third-party services you do not control. The deepened module takes the external dependency as an injected port; tests provide a mock adapter.

## Seam Discipline

- One adapter means a hypothetical seam. Two adapters means a real one.
- Do not expose internal seams through the interface just because tests use them.
- Old unit tests on shallow modules become waste once tests at the deepened module's interface exist; delete them.
- Tests assert observable outcomes through the interface, not internal state.
