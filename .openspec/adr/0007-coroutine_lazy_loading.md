# ADR-0007: Coroutine-Based Lazy Module Loading

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0007-coroutine_lazy_loading

---

## Context

Applications with many Koin modules experience startup latency because all module definitions are evaluated and registered synchronously during `startKoin`. On Android, this blocks the main thread and increases cold start time. Large module graphs (100+ definitions) can add measurable delay.

The system needed a way to defer module initialization without changing the core module/definition API.

## Decision

Introduce an **optional coroutines module** (`koin-core-coroutines`) that:

1. **`LazyModule`**: Wraps `Lazy<Module>` to defer module initialization until the inner `Module.value` is accessed.
2. **`KoinCoroutinesEngine`**: A `KoinExtension` that manages a `SupervisorJob`-backed `CoroutineScope` for background work. It uses `async` to launch module loading as `Deferred` jobs.
3. **`lazyModules()` on `KoinApplication`**: Accepts `Lazy<Module>` varargs/lists and delegates to the coroutines engine for background loading.
4. **`awaitAllStartJobs()`**: Suspend function to await completion of all background module loading before proceeding with app startup.

The engine auto-registers itself via `coroutinesEngine()` when `lazyModules()` is called. The dispatcher is configurable (defaults to platform default via `KoinPlatformCoroutinesTools.defaultCoroutineDispatcher()`).

### Integration pattern:
```kotlin
startKoin {
    modules(criticalModule)           // loaded synchronously
    lazyModules(featureModuleA,       // loaded in background
                featureModuleB)
}
// Later, when features are needed:
koin.awaitAllStartJobs()
```

## Consequences

### Positive
- **Reduced cold start**: Non-critical modules load in the background, unblocking the main thread
- **Opt-in**: `koin-core-coroutines` is a separate dependency — projects not needing it pay no cost
- **Extension-based**: Uses the `KoinExtension` mechanism rather than modifying core internals
- **SupervisorJob isolation**: A failing module load doesn't crash other background jobs

### Negative
- **Race conditions**: If code accesses a lazily-loaded definition before `awaitAllStartJobs()`, resolution fails with `NoDefinitionFoundException`
- **Additional dependency**: Requires `kotlinx-coroutines-core` transitive dependency
- **Debugging complexity**: Background loading failures surface asynchronously

### Neutral
- `ModuleConfiguration` provides a higher-level API grouping sync and async modules, but is marked `@KoinExperimentalAPI`
