---
domain: koin-core-coroutines
version: 4.1.2
status: draft
date: 2026-02-21
---

# Koin Core Coroutines — Lazy Module Loading & Background Initialization

## Overview

Koin Core Coroutines provides coroutine-based background module loading for the Koin DI framework. It enables modules to be declared lazily and loaded asynchronously during application startup, reducing cold start time by deferring module initialization off the main thread. The module plugs into the core via the `KoinExtension` mechanism.

## Philosophy

- **Startup performance**: Defer module initialization to background coroutines, reducing blocking time on the main thread.
- **Extension-based**: Integrates via `KoinExtension` rather than modifying core — the coroutines engine is an optional add-on.
- **Lazy-by-default**: `LazyModule` wraps `Lazy<Module>`, deferring all definition evaluation until the module is actually loaded.
- **Await semantics**: Callers can `awaitAllStartJobs()` to block until background loading completes, or poll via `isAllStartedJobsDone()`.

## Requirements

### Requirement 1: Lazy Module Declaration

The system MUST provide a `LazyModule` class that wraps a module initializer lambda as a `Lazy<Module>`. The system MUST provide a `lazyModule { }` DSL function for declaring lazy modules.

**Implementation**: projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/module/LazyModule.kt, projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/dsl/LazyModuleDSL.kt

#### Scenario: Declare a lazy module
- GIVEN a module declaration using `lazyModule { single { HeavyService() } }`
- WHEN the `lazyModule` function is invoked
- THEN a `LazyModule` is returned without initializing the inner module

#### Scenario: Compose lazy modules
- GIVEN two lazy modules `lazyA` and `lazyB`
- WHEN `lazyA + lazyB` is evaluated
- THEN a list of both lazy modules is returned for batch loading

### Requirement 2: Background Module Loading

The system MUST provide `KoinApplication.lazyModules()` functions that load lazy modules asynchronously using coroutines. The loading MUST use the `KoinCoroutinesEngine` extension, which is auto-registered if not already present.

**Implementation**: projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/KoinApplicationLazyExt.kt, ADR-0007

#### Scenario: Load lazy modules in background
- GIVEN a Koin application with `lazyModules(lazyModuleA, lazyModuleB)`
- WHEN the application starts
- THEN the lazy modules are loaded asynchronously via coroutine jobs on the configured dispatcher

#### Scenario: Custom dispatcher for background loading
- GIVEN a Koin application calling `lazyModules(lazyModuleA, dispatcher = Dispatchers.IO)`
- WHEN the lazy modules are loaded
- THEN the loading runs on the specified IO dispatcher

### Requirement 3: Coroutines Engine Extension

The system MUST provide a `KoinCoroutinesEngine` that implements `KoinExtension` and `CoroutineScope`. The engine MUST manage background startup jobs via a `SupervisorJob` and MUST clean up coroutines when Koin is closed.

**Implementation**: projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/coroutine/KoinCoroutinesEngine.kt, projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/extension/KoinCoroutinesExtension.kt

#### Scenario: Engine lifecycle with Koin
- GIVEN a KoinCoroutinesEngine registered as an extension
- WHEN `Koin.close()` is called
- THEN the engine's coroutine scope is cancelled and all pending jobs are terminated

#### Scenario: Register coroutines engine
- GIVEN a Koin application
- WHEN `coroutinesEngine()` is called on the application builder
- THEN a `KoinCoroutinesEngine` is registered as a Koin extension (idempotent — skips if already registered)

### Requirement 4: Await Start Jobs

The system MUST provide `Koin.awaitAllStartJobs()` to suspend until all background module loading is complete. The system SHOULD provide `Koin.onKoinStarted { }` as a convenience for running code after all start jobs finish. The system SHOULD provide `Koin.isAllStartedJobsDone()` to poll completion status.

**Implementation**: projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/KoinLazyExt.kt

#### Scenario: Await background loading before use
- GIVEN lazy modules being loaded in the background
- WHEN `koin.awaitAllStartJobs()` is called
- THEN the call suspends until all background module loading completes

#### Scenario: Run code after startup
- GIVEN lazy modules being loaded in the background
- WHEN `koin.onKoinStarted { koin -> startApp(koin) }` is called
- THEN `startApp` executes only after all start jobs have finished

#### Scenario: Poll completion status
- GIVEN lazy modules being loaded in the background
- WHEN `koin.isAllStartedJobsDone()` is called
- THEN it returns `true` only when all background jobs are no longer active

### Requirement 5: Module Configuration

The system MAY provide a `ModuleConfiguration` class that groups regular modules and lazy modules into a single configuration unit, with an optional dispatcher override.

**Implementation**: projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/module/ModuleConfiguration.kt

#### Scenario: Configure modules and lazy modules together
- GIVEN a `moduleConfiguration { modules(coreModule); lazyModules(featureModule); dispatcher(Dispatchers.IO) }`
- WHEN applied to a KoinApplication via `moduleConfiguration(config)`
- THEN regular modules are loaded synchronously and lazy modules are loaded in the background on the IO dispatcher

### Requirement 6: Global Context Lazy Loading

The system MAY support loading lazy modules into the global Koin context via `loadKoinModules(lazyModule)`.

**Implementation**: projects/core/koin-core-coroutines/src/commonMain/kotlin/org/koin/core/context/LoadLazyModules.kt

#### Scenario: Load lazy module into global context
- GIVEN a running global Koin context
- WHEN `loadKoinModules(myLazyModule)` is called
- THEN the lazy module is evaluated and its definitions are registered in the global context
