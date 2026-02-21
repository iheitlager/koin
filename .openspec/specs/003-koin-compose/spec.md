---
domain: koin-compose
version: 4.1.2
status: draft
date: 2026-02-21
---

# Koin Compose — Jetpack Compose & Compose Multiplatform Integration

## Overview

Koin Compose provides dependency injection integration for Jetpack Compose and Compose Multiplatform applications. It leverages `CompositionLocal` to propagate the Koin context through the composition tree and provides composable functions for type-safe dependency resolution that is recomposition-aware.

## Philosophy

- **Composition-native**: Koin context and scopes flow through `CompositionLocal`, following Compose conventions.
- **Recomposition-safe**: Injected instances are `remember`-ed to avoid unnecessary re-resolution on recomposition.
- **Multiplatform**: Core compose APIs are in `commonMain`, usable across Android, Desktop, and iOS.
- **Preview-friendly**: Isolated contexts for Compose previews without affecting the global Koin state.

## Requirements

### Requirement 1: Compose Koin Context

The system MUST provide `CompositionLocal` values (`LocalKoinApplication`, `LocalKoinScope`) for propagating the Koin instance and current scope through the composition tree.

**Implementation**: projects/compose/koin-compose/src/commonMain/kotlin/org/koin/compose/KoinApplication.kt

#### Scenario: Access Koin in a Composable
- GIVEN a Composable within a `KoinApplication` block
- WHEN `getKoin()` or `currentKoinScope()` is called
- THEN the current Koin instance or scope is returned from the composition

### Requirement 2: KoinApplication Composable

The system MUST provide a `KoinApplication` composable that initializes a Koin instance for the composition subtree. The system SHOULD also provide `KoinIsolatedContext` for isolated Koin instances.

**Implementation**: projects/compose/koin-compose/src/commonMain/kotlin/org/koin/compose/KoinApplication.kt

#### Scenario: Start Koin in Compose
- GIVEN a Compose application
- WHEN `KoinApplication(application = { modules(appModule) }) { Content() }` is used
- THEN Koin is started and available to all descendant Composables

#### Scenario: Isolated Koin context
- GIVEN a need for an isolated DI context (e.g., an SDK within a host app)
- WHEN `KoinIsolatedContext(koinApplication = { modules(sdkModule) }) { ... }` is used
- THEN the isolated context does not affect or see the global Koin instance

### Requirement 3: Composable Dependency Injection

The system MUST provide a `koinInject<T>()` composable function for resolving dependencies within Composables. Resolution MUST be recomposition-safe via `remember`.

**Implementation**: projects/compose/koin-compose/src/commonMain/kotlin/org/koin/compose/Inject.kt

#### Scenario: Inject dependency in Composable
- GIVEN a module with `single { UserRepository() }`
- WHEN `val repo = koinInject<UserRepository>()` is used in a Composable
- THEN the singleton instance is returned and cached across recompositions

### Requirement 4: Compose ViewModel Resolution

The system MUST provide composable functions for resolving ViewModels with proper lifecycle management in Compose.

**Implementation**: projects/compose/koin-compose-viewmodel/src/commonMain/kotlin/org/koin/compose/viewmodel/KoinViewModel.kt

#### Scenario: Resolve ViewModel in Composable
- GIVEN a module with `viewModel { MyViewModel(get()) }`
- WHEN `val vm = koinViewModel<MyViewModel>()` is used in a Composable
- THEN the ViewModel is resolved with Compose lifecycle awareness

### Requirement 5: Compose Preview Support

The system SHOULD provide a `KoinApplicationPreview` composable for lightweight Koin initialization in Compose previews.

**Implementation**: projects/compose/koin-compose/src/commonMain/kotlin/org/koin/compose/KoinApplication.kt

#### Scenario: Use Koin in Compose Preview
- GIVEN a `@Preview` composable
- WHEN `KoinApplicationPreview { modules(previewModule) }` wraps the preview content
- THEN dependencies are resolvable without affecting the global Koin state
