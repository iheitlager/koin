# ADR-0003: Multi-Strategy Resolution Chain

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0003-resolution_strategy_chain

---

## Context

Dependency resolution in a scoped DI container is not a simple map lookup. Multiple sources can provide an instance for a given type: the main registry, injected parameters, scope-local state, linked parent scopes, and external extensions. The system needs a deterministic resolution order.

## Decision

`CoreResolver` implements a **chain of responsibility** with a fixed priority order:

1. **Injected parameters** — runtime parameters passed via `parametersOf()`
2. **Instance registry** — standard definitions registered in modules
3. **Stacked parameters** — scope-local parameters pushed during resolution
4. **Scope source value** — the value bound to the scope itself (e.g., an Activity for an activity scope)
5. **Scope archetype** (experimental) — inherited scope definitions
6. **Parent/linked scopes** — traversal up the scope hierarchy via `linkedScopes`
7. **External resolution extensions** — `ResolutionExtension` implementations

Resolution stops at the first strategy that returns a non-null result. If all strategies fail, `NoDefinitionFoundException` is thrown (or null is returned for `getOrNull()`).

The resolver also supports `getAll<T>()` which collects instances from all sources that match the type.

## Consequences

### Positive
- **Predictable resolution**: Fixed priority order means behavior is deterministic and debuggable
- **Extensible**: `ResolutionExtension` interface allows external systems (Dagger bridge, etc.) to participate
- **Scope hierarchy**: Linked scopes enable parent-child relationships without complex inheritance

### Negative
- **Resolution complexity**: 7-step chain adds cognitive overhead for debugging unexpected resolution
- **Performance**: Each miss cascades through multiple strategies before failing

### Neutral
- `Scope.getOrNull()` is the most complex resolution method (cyclomatic complexity 7)
