# ADR-0005: Kotlin Multiplatform Architecture

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0005-kotlin_multiplatform

---

## Context

Koin targets multiple platforms: JVM (server-side), Android, iOS (via Kotlin/Native), JavaScript, and other Kotlin/Native targets. The core DI functionality must work identically across all platforms, while platform-specific integrations (Android scopes, Compose, Ktor) are additive.

## Decision

Structure the project as a Kotlin Multiplatform library:

- **`commonMain`**: All core APIs — `Koin`, `Module`, `Scope`, `BeanDefinition`, `CoreResolver`, registries, DSL. This is the majority of the codebase.
- **`jvmMain`/`androidMain`/`iosMain`/`jsMain`**: Platform-specific implementations of:
  - `KoinPlatformTools` — thread-safe collections, locking mechanisms
  - `KoinPlatform` — default Koin context holder
  - Logging implementations
- **Platform modules**: Separate Gradle modules for Android (`koin-android`), Compose (`koin-compose`), Ktor (`koin-ktor`)

Key multiplatform design decisions:
- `Lockable` base class with `expect`/`actual` for thread safety
- `safeHashMap()` / `safeSet()` platform functions for concurrent collections
- No reflection in `commonMain` — JVM-only verification uses reflection
- Reified inline functions for type-safe resolution across platforms

## Consequences

### Positive
- **Single codebase**: Core logic is written once, tested once, deployed everywhere
- **Consistent behavior**: All platforms use the same resolution, scoping, and module system
- **Type safety**: Reified generics provide type-safe injection without reflection

### Negative
- **Platform abstraction cost**: `expect`/`actual` declarations for platform primitives add complexity
- **Lowest common denominator**: Some JVM features (reflection-based verification) are JVM-only
- **Build complexity**: Multi-target Gradle setup with multiple source sets

### Neutral
- Platform-specific modules (Android, Compose, Ktor) are separate Gradle subprojects with their own source sets
