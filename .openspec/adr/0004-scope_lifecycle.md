# ADR-0004: Scope-Based Instance Lifecycle

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0004-scope_lifecycle

---

## Context

Not all instances should live for the entire application lifetime. Android Activities are created and destroyed, HTTP requests have a bounded lifecycle, user sessions expire. The DI container needs a way to bind instance lifecycles to arbitrary scopes.

## Decision

Introduce a `Scope` as a named, closeable container that holds scoped instances:

- **Root scope**: Always exists, holds singleton definitions. Lives as long as `Koin` itself.
- **Named scopes**: Created via `koin.createScope<T>(id)`, registered with `ScopeRegistry`.
- **Scope qualifiers**: Definitions declare which scope they belong to via `scope<T> { scoped { ... } }`.
- **Scope linking**: Scopes can link to other scopes for fallback resolution.
- **Scope callbacks**: `ScopeCallback.onScopeClose()` is invoked when a scope closes.
- **Scope closure**: `scope.close()` drops all scoped instances, invokes callbacks, and removes the scope from the registry.

The three instance factory types map to lifecycle:
- `SingleInstanceFactory` — root scope, application lifetime
- `FactoryInstanceFactory` — no caching, new instance per call
- `ScopedInstanceFactory` — scoped, lives until scope closes

## Consequences

### Positive
- **Lifecycle control**: Scoped instances are automatically cleaned up when their context ends
- **Memory management**: No leaks from instances outliving their context (e.g., Activity scopes on Android)
- **Hierarchical**: Scope linking enables parent-child patterns (Activity → Fragment)

### Negative
- **Scope management burden**: Developers must create and close scopes at the right lifecycle points (mitigated by base classes like `ScopeActivity`)
- **Closed scope errors**: Accessing a closed scope throws `ClosedScopeException`

### Neutral
- Android and Compose integrations automate scope lifecycle for common cases
