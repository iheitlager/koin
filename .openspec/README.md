# OpenSpec Specifications — Koin

This directory contains reverse-engineered behavioral specifications and architectural decision records for the [Koin](https://github.com/InsertKoinIO/koin) dependency injection framework.

## What is OpenSpec?

OpenSpec is a specification format designed for:
- **Clear requirements**: Documenting what a system must, should, or may do
- **Testable scenarios**: Using Given-When-Then format for validation
- **Evolution tracking**: Marking features as proposed, implemented, or deprecated
- **Shared understanding**: Bridging product, engineering, and user perspectives

## Reading Specifications

Each specification follows this structure:

1. **Frontmatter**: Domain, version, status, and date
2. **Overview**: High-level description of the subsystem
3. **Philosophy**: Core principles and design values
4. **Requirements**: Detailed requirements with RFC 2119 keywords (MUST, SHOULD, MAY)
5. **Scenarios**: Given-When-Then examples for each requirement
6. **Implementation**: Current status and file references

### RFC 2119 Keywords

- **MUST**: Absolute requirement — core behavior
- **SHOULD**: Recommended but not mandatory
- **MAY**: Optional feature or capability

### Given-When-Then Format

```
GIVEN [precondition or context]
WHEN [action or event]
THEN [expected outcome]
```

## Current Specifications

### 001-koin-core

**Path**: `.openspec/specs/001-koin-core/spec.md`

**Focus**: Core DI container, DSL, scoping, and resolution

Documents the foundational Koin framework covering:
- DI container (Koin, KoinApplication)
- Module DSL for declaring definitions
- Bean definitions (single, factory, scoped)
- Scope management and lifecycle
- Dependency resolution and instance creation
- Qualifier and parameter systems
- Extension mechanism

### 002-koin-android

**Path**: `.openspec/specs/002-koin-android/spec.md`

**Focus**: Android platform integration

Documents Android-specific capabilities:
- Android Context injection
- Activity/Fragment scope management
- ViewModel integration
- AndroidX Navigation support
- WorkManager integration
- App Startup integration

### 003-koin-compose

**Path**: `.openspec/specs/003-koin-compose/spec.md`

**Focus**: Jetpack Compose and Compose Multiplatform integration

### 004-koin-ktor

**Path**: `.openspec/specs/004-koin-ktor/spec.md`

**Focus**: Ktor server-side framework integration

### 005-koin-test

**Path**: `.openspec/specs/005-koin-test/spec.md`

**Focus**: Testing utilities and module verification

### 006-koin-core-coroutines

**Path**: `.openspec/specs/006-koin-core-coroutines/spec.md`

**Focus**: Lazy module loading and background initialization

Documents coroutine-based capabilities:
- LazyModule declaration and composition
- Background module loading via KoinCoroutinesEngine
- Await semantics for start job completion
- ModuleConfiguration for grouping sync/async modules

## Directory Structure

```
.openspec/
├── specs/          # Behavioral specs (current implemented state)
│   └── NNN-name/
│       └── spec.md
├── adr/            # Architectural Decision Records
│   └── NNNN-title.md
├── changes/        # Delta specs (proposed changes)
│   └── NNN-name/
│       └── spec.md
└── README.md
```

## Traceability

These specs follow the traceability metamodel defined in code-analyzer's ADR-0019:
- **IMPLEMENTED_BY**: Each requirement links to source files via `**Implementation**:` lines
- **ADDRESSED_BY**: Requirements reference ADRs via `ADR-XXXX` notation
- **TESTED_BY**: Scenarios link to test functions (inferred from code structure)

## Origin

These specifications were reverse-engineered from the Koin 4.1.2 codebase using [code-analyzer](https://github.com/iheitlager/code-analyzer). They document the current implemented behavior, not aspirational features.

---

*Last updated: 2026-02-21*
