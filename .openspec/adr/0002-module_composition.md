# ADR-0002: Composable Module Architecture

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0002-module_composition

---

## Context

Large applications have many dependency definitions. A flat list of definitions in a single module becomes unmanageable. The system needs a way to organize definitions into logical groups that can be composed, reused, and loaded independently.

## Decision

`Module` is the unit of organization. Modules can:
- **Include other modules** via `includes()` — creating a directed acyclic graph of module dependencies
- **Be combined** via the `+` operator for ad-hoc composition
- **Be loaded/unloaded dynamically** at runtime via `Koin.loadModules()` / `unloadModules()`
- **Be flattened** recursively via DFS traversal for registration

Each `Module` maintains:
- `mappings`: A map of `IndexKey` to `InstanceFactory` for all definitions
- `scopes`: Scope-qualified definition sets
- `includes`: References to included child modules
- `eagerInstances`: Definitions marked `createdAtStart = true`

Module flattening uses DFS with visited tracking to handle diamond dependencies (module A includes B and C, both include D — D is only registered once).

## Consequences

### Positive
- **Modular architecture**: Feature modules map to Koin modules, enabling clean separation of concerns
- **Reusability**: Shared modules (networking, logging) can be included by multiple feature modules
- **Dynamic loading**: Server and plugin architectures can load modules at runtime
- **No circular module dependencies**: DFS flattening naturally detects and handles DAG structure

### Negative
- **Override complexity**: When multiple modules define the same type, override behavior depends on load order
- **Flatten cost**: Deep module graphs require full DFS traversal on load (mitigated by caching)

### Neutral
- Module flatten is the most complex function in koin-core (cyclomatic complexity 7)
