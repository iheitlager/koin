# ADR-0006: Type-Indexed Instance Registry

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0006-type_indexed_registry

---

## Context

Dependency resolution is the hot path of the DI container. Every `get<T>()` call must find the correct `InstanceFactory` for the requested type, qualifier, and scope. This lookup must be fast — O(1) — for production applications with hundreds of definitions.

## Decision

Use a **composite index key** (`IndexKey`) for O(1) definition lookup:

- `IndexKey` is composed of: `primaryType (KClass<*>)` + `qualifier (Qualifier?)` + `scopeQualifier (Qualifier)`
- `InstanceRegistry` maintains a `HashMap<IndexKey, InstanceFactory<*>>` for all definitions
- Secondary types (supertypes bound via `bind<T>()`) create additional index entries pointing to the same factory
- `saveMapping()` handles duplicates with configurable override behavior

The registry supports:
- `resolveDefinition()` — find factory by key
- `resolveInstance()` — find factory and create/retrieve instance
- `getAll<T>()` — collect all factories matching a type across scopes

## Consequences

### Positive
- **O(1) resolution**: HashMap lookup by composite key
- **Secondary type binding**: A `single<Impl> { ... } bind Interface::class` creates entries for both `Impl` and `Interface`
- **Override control**: Duplicate detection with configurable allow/deny via `OptionRegistry`

### Negative
- **Memory overhead**: Secondary type bindings multiply index entries
- **Key equality complexity**: `IndexKey` must correctly implement equals/hashCode across the three components

### Neutral
- The `saveMapping()` function has cyclomatic complexity 5 due to override/duplicate handling
