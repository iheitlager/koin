---
domain: koin-android
version: 4.1.2
status: draft
date: 2026-02-21
---

# Koin Android — Android Platform Integration

## Overview

Koin Android provides Android-specific integrations for the Koin DI framework. It bridges Koin with the Android component lifecycle (Activities, Fragments, Services), provides ViewModel resolution, and integrates with AndroidX libraries including Navigation, WorkManager, and App Startup.

## Philosophy

- **Lifecycle-aware scoping**: Android component scopes are tied to their lifecycle — scopes are created on start and destroyed on stop.
- **ViewModel first-class support**: ViewModels are resolved through Koin with proper AndroidX integration.
- **Minimal boilerplate**: Extension functions and base classes reduce ceremony for common patterns.
- **Backward compatibility**: Java-friendly compat layer for mixed codebases.

## Requirements

### Requirement 1: Android Context Registration

The system MUST provide a way to register the Android `Context` (or `Application`) into the Koin container so it can be injected into components that need it.

**Implementation**: projects/android/koin-android/src/main/java/org/koin/android/ext/koin/KoinExt.kt

#### Scenario: Register Android context
- GIVEN a Koin application being configured
- WHEN `androidContext(application)` is called in the Koin builder
- THEN the `Application` and `Context` instances are available for injection throughout the app

### Requirement 2: Activity Scope Management

The system MUST provide scope integration for Android Activities. Scopes SHOULD be automatically created and destroyed with the Activity lifecycle.

**Implementation**: projects/android/koin-android/src/main/java/org/koin/androidx/scope/ScopeActivity.kt, projects/android/koin-android/src/main/java/org/koin/android/scope/AndroidScopeComponent.kt

#### Scenario: Activity-scoped injection
- GIVEN an Activity extending `ScopeActivity`
- WHEN the Activity is created
- THEN a Koin scope is lazily created and scoped dependencies are resolvable

#### Scenario: Activity scope cleanup
- GIVEN an Activity with an active scope
- WHEN the Activity is destroyed
- THEN the scope is closed and all scoped instances are released

### Requirement 3: Fragment Scope Management

The system MUST provide scope integration for Android Fragments with lifecycle-aware scope creation and destruction.

**Implementation**: projects/android/koin-android/src/main/java/org/koin/androidx/scope/ScopeFragment.kt, projects/android/koin-android/src/main/java/org/koin/androidx/scope/FragmentExt.kt

#### Scenario: Fragment-scoped injection
- GIVEN a Fragment extending `ScopeFragment`
- WHEN the Fragment's view is created
- THEN a Koin scope is created and scoped dependencies are resolvable

### Requirement 4: ViewModel Resolution

The system MUST provide extension functions for resolving AndroidX ViewModels from the Koin container. ViewModel resolution MUST support qualifiers, custom `CreationExtras`, and injected parameters.

**Implementation**: projects/android/koin-android/src/main/java/org/koin/androidx/viewmodel/ext/android/ActivityVM.kt, projects/core/koin-core-viewmodel/src/commonMain/kotlin/org/koin/viewmodel/factory/KoinViewModelFactory.kt

#### Scenario: Resolve ViewModel in Activity
- GIVEN a module with `viewModel { MyViewModel(get()) }`
- WHEN `val vm: MyViewModel by viewModel()` is used in an Activity
- THEN the ViewModel is resolved from Koin with proper AndroidX lifecycle management

#### Scenario: Shared ViewModel between Activity and Fragment
- GIVEN a ViewModel declared in Koin
- WHEN both an Activity and its child Fragment resolve the same ViewModel type
- THEN they share the same ViewModel instance (Activity-scoped lifecycle)

### Requirement 5: Navigation Graph ViewModel Scoping

The system SHOULD support resolving ViewModels scoped to a Navigation Graph backstack entry.

**Implementation**: projects/android/koin-androidx-navigation/src/main/java/org/koin/androidx/navigation/NavGraphExt.kt

#### Scenario: Navigation-scoped ViewModel
- GIVEN a Fragment within a navigation graph
- WHEN `val vm: NavVM by koinNavGraphViewModel(R.id.nav_graph)` is used
- THEN the ViewModel is scoped to the navigation graph's backstack entry

### Requirement 6: WorkManager Integration

The system SHOULD provide a `KoinWorkerFactory` that resolves `Worker` instances from the Koin container.

**Implementation**: projects/android/koin-androidx-workmanager/src/main/java/org/koin/androidx/workmanager/factory/KoinWorkerFactory.kt

#### Scenario: Resolve Worker from Koin
- GIVEN a module with a Worker definition using the worker class name as qualifier
- WHEN WorkManager creates a work request
- THEN `KoinWorkerFactory` resolves the Worker from Koin with injected dependencies

### Requirement 7: App Startup Integration

The system MAY support Android App Startup Initializer for early Koin initialization.

**Implementation**: projects/android/koin-androidx-startup/src/main/java/org/koin/androix/startup/KoinInitializer.kt, projects/android/koin-androidx-startup/src/main/java/org/koin/androix/startup/KoinStartup.kt

#### Scenario: Initialize Koin via App Startup
- GIVEN an Application implementing `KoinStartup`
- WHEN the app starts via AndroidX App Startup
- THEN Koin is initialized before any Activity is created

### Requirement 8: Android Properties Loading

The system MAY support loading properties from an Android assets file (`koin.properties`).

**Implementation**: projects/android/koin-android/src/main/java/org/koin/android/ext/koin/KoinExt.kt

#### Scenario: Load properties from assets
- GIVEN a `koin.properties` file in the Android assets folder
- WHEN `androidFileProperties()` is called during Koin configuration
- THEN the properties are loaded and available via `koin.getProperty()`
