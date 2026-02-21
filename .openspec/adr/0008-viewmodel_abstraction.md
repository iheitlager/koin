# ADR-0008: Multiplatform ViewModel Abstraction

**Status**: Accepted

**Date**: 2026-02-21

**Decision ID**: 0008-viewmodel_abstraction

---

## Context

ViewModel is an Android architecture component tied to `androidx.lifecycle`. With Kotlin Multiplatform, Koin needs to support ViewModel injection on Android (via `ViewModelProvider`) and potentially on other platforms (Compose Desktop, iOS). The ViewModel resolution must integrate with platform lifecycle management while keeping the DSL surface (`viewModel { }`) uniform.

JetBrains has introduced `androidx.lifecycle` as a multiplatform library, making `ViewModel` available in `commonMain`.

## Decision

Create a **dedicated `koin-core-viewmodel` module** in `commonMain` that:

1. **`KoinViewModelFactory`**: Implements `ViewModelProvider.Factory`, resolving ViewModels from Koin's container. It receives scope, qualifier, and parameters, and delegates to `scope.getWithParameters()`.
2. **`viewModel { }` / `viewModelOf()` DSL**: Extension functions on `Module` that declare ViewModel definitions (backed by `factory` with special handling).
3. **`AndroidParametersHolder`**: Wraps both Koin `ParametersDefinition` and AndroidX `CreationExtras` into a unified parameter source for the definition lambda.
4. **ViewModel Scope Factory** (experimental): When `viewModelScopeFactory` option is enabled, each ViewModel gets its own auto-closing Koin scope. The scope is linked to the ViewModel via `addCloseable(ViewModelScopeAutoCloseable)` and destroyed when the ViewModel is cleared.
5. **Platform integration modules**: `koin-android` provides `by viewModel()` delegates for Activities/Fragments; `koin-compose-viewmodel` provides `koinViewModel()` composables.

### Key types:
- `KoinViewModelFactory` — bridge between Koin and `ViewModelProvider`
- `ViewModelScopeArchetype` — experimental scope archetype for ViewModel-scoped DI
- `ScopeViewModel` — base class for ViewModels that own a Koin scope

## Consequences

### Positive
- **Multiplatform**: Core ViewModel resolution logic lives in `commonMain`, shared across Android and Desktop
- **Lifecycle-safe**: Uses `ViewModelProvider.Factory` ensuring ViewModels survive configuration changes
- **Composable**: Works in both View-based (Activity/Fragment delegates) and Compose (`koinViewModel()`) contexts
- **Scope isolation**: Optional `viewModelScopeFactory` gives each ViewModel its own DI scope with auto-cleanup

### Negative
- **AndroidX dependency**: `koin-core-viewmodel` depends on `androidx.lifecycle` (multiplatform version)
- **Factory complexity**: `KoinViewModelFactory.create()` has branching logic for scope factory mode vs. direct resolution
- **Multiple entry points**: ViewModels can be resolved via `by viewModel()`, `koinViewModel()`, or direct `get()`, which can confuse users

### Neutral
- Scoped ViewModel support (`ViewModelScopeArchetype`) is experimental and may change
