# ADR-0001: Pure Kotlin DSL for Dependency Declaration

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0001-pure_kotlin_dsl

---

## Context

Dependency injection frameworks for Kotlin/JVM historically relied on annotation processing and code generation (Dagger, Hilt) or runtime reflection (Spring, Guice). These approaches have trade-offs:

- **Annotation processing**: Compile-time safety but complex build setup, slow compilation, and opaque generated code.
- **Runtime reflection**: Simple setup but slow startup, no compile-time safety, and limited Kotlin Multiplatform support.

Koin needed an approach that was Kotlin-native, lightweight, multiplatform-compatible, and required no build plugin.

## Decision

Use a **pure Kotlin DSL** (Domain-Specific Language) based on Kotlin's receiver lambdas and extension functions for all dependency declarations. No annotation processing, no code generation, and no runtime reflection on non-JVM targets.

The DSL uses:
- `module { }` blocks with `Module` as receiver
- `single { }`, `factory { }`, `scoped { }` for definition kinds
- `get()` within definition lambdas for dependency resolution
- `scope<T> { }` for scoped definition groups
- `includes()` for module composition

### Key design elements:
- `KoinApplication` builder configures the container via fluent API
- `Module` class aggregates `BeanDefinition` entries
- Type resolution uses `KClass` and reified generics, not string identifiers
- `startKoin { }` is the entry point that wires everything together

## Consequences

### Positive
- **No build plugin required**: Works with any Kotlin project out of the box
- **Kotlin Multiplatform compatible**: No JVM-only reflection or annotation processing
- **Readable declarations**: Module definitions read as plain Kotlin code
- **Fast startup**: No classpath scanning or annotation discovery at runtime
- **IDE support**: Standard Kotlin navigation, refactoring, and type checking

### Negative
- **No compile-time graph validation**: Definition errors are caught at runtime or via `verify()` in tests, not at compile time (unlike Dagger)
- **Manual wiring**: Developers must explicitly call `get()` for dependencies (mitigated by `koin-annotations` for those who prefer annotation style)
- **Reified generics limitation**: Some patterns require explicit `KClass` parameters on non-JVM targets

### Neutral
- The `koin-annotations` companion project adds optional annotation processing for developers who prefer it, generating the DSL code as output
