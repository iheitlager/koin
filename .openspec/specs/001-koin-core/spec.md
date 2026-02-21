---
domain: koin-core
version: 4.1.2
status: draft
date: 2026-02-21
---

# Koin Core — Dependency Injection Container

## Overview

Koin Core is the foundational module of the Koin dependency injection framework. It provides a pure Kotlin DSL for declaring dependency graphs, a lightweight runtime container for resolving instances, and a scoping system for managing instance lifecycles. The core is implemented as a Kotlin Multiplatform library targeting JVM, Android, iOS, JS, and native platforms.

## Philosophy

- **Pure Kotlin DSL**: No annotation processing, no code generation, no reflection (on non-JVM targets). Dependencies are declared as Kotlin code.
- **Pragmatic simplicity**: Minimal API surface — `single`, `factory`, `scoped` cover most use cases.
- **Composable modules**: Modules can include other modules, enabling modular application architecture.
- **Scope-aware lifecycle**: Instances can be bound to scopes with automatic cleanup.
- **Multiplatform first**: Core APIs are in `commonMain`, platform-specific implementations are minimal.

## Requirements

### Requirement 1: Koin Application Initialization

The system MUST provide a builder API (`KoinApplication`) for configuring and starting the DI container. The builder MUST support fluent configuration of modules, properties, logging, and options.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/KoinApplication.kt, ADR-0001

#### Scenario: Start Koin with modules
- GIVEN a Kotlin application
- WHEN `startKoin { modules(appModule) }` is called
- THEN a `Koin` instance is created and registered as the global default

#### Scenario: Configure logging level
- GIVEN a KoinApplication being configured
- WHEN `printLogger(Level.DEBUG)` is called
- THEN the Koin container uses the specified log level for diagnostic output

### Requirement 2: Module Declaration DSL

The system MUST provide a DSL for declaring dependency definitions within modules. The DSL MUST support `single` (singleton), `factory` (new instance per resolution), and `scoped` (instance per scope) definition types.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/module/Module.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/dsl/ModuleDSL.kt, ADR-0002

#### Scenario: Declare a singleton definition
- GIVEN a module block
- WHEN `single { MyService() }` is declared
- THEN a single instance of `MyService` is created on first resolution and reused for all subsequent resolutions

#### Scenario: Declare a factory definition
- GIVEN a module block
- WHEN `factory { MyPresenter(get()) }` is declared
- THEN a new instance of `MyPresenter` is created on every resolution

#### Scenario: Declare a scoped definition
- GIVEN a module with a scope block `scope<MyScope> { scoped { MyScopedService() } }`
- WHEN the scope is opened and the service is resolved
- THEN a single instance exists within that scope, and is destroyed when the scope closes

#### Scenario: Module composition via includes
- GIVEN module A that `includes(moduleB)`
- WHEN module A is loaded into Koin
- THEN all definitions from both module A and module B are registered

### Requirement 3: Dependency Resolution

The system MUST resolve dependencies by type and optional qualifier. Resolution MUST follow a defined strategy chain: injected parameters, instance registry, stacked parameters, scope source value, linked scopes, and extension resolvers.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/resolution/CoreResolver.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/registry/InstanceRegistry.kt, ADR-0003

#### Scenario: Resolve by type
- GIVEN a module with `single<UserRepository> { UserRepositoryImpl() }`
- WHEN `koin.get<UserRepository>()` is called
- THEN the `UserRepositoryImpl` instance is returned

#### Scenario: Resolve with qualifier
- GIVEN two definitions with different qualifiers: `single(named("local")) { ... }` and `single(named("remote")) { ... }`
- WHEN `koin.get<DataSource>(named("remote"))` is called
- THEN the remote data source instance is returned

#### Scenario: Resolution failure
- GIVEN no definition registered for type `MissingService`
- WHEN `koin.get<MissingService>()` is called
- THEN a `NoDefinitionFoundException` is thrown

### Requirement 4: Scope Management

The system MUST support named scopes with lifecycle management. Scopes MUST be creatable, retrievable by ID, and closeable. Closing a scope MUST destroy all scoped instances and invoke registered callbacks.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/scope/Scope.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/registry/ScopeRegistry.kt, ADR-0004

#### Scenario: Create and use a scope
- GIVEN a module with `scope<SessionScope> { scoped { SessionService() } }`
- WHEN a scope is created via `koin.createScope<SessionScope>("session1")`
- THEN `scope.get<SessionService>()` returns an instance bound to that scope

#### Scenario: Close a scope destroys instances
- GIVEN an active scope with resolved scoped instances
- WHEN `scope.close()` is called
- THEN all scoped instances are dropped and scope callbacks are invoked

#### Scenario: Scope linking for hierarchical resolution
- GIVEN scope A linked to scope B
- WHEN scope A cannot resolve a definition locally
- THEN it falls back to resolving from linked scope B

### Requirement 5: Bean Definition and Instance Factory

The system MUST represent each dependency declaration as a `BeanDefinition` with metadata (primary type, qualifier, scope qualifier, kind). Each definition MUST be backed by an `InstanceFactory` that manages instance creation and lifecycle according to its kind (Singleton, Factory, Scoped).

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/definition/BeanDefinition.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/instance/InstanceFactory.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/instance/SingleInstanceFactory.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/instance/FactoryInstanceFactory.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/instance/ScopedInstanceFactory.kt

#### Scenario: Singleton instance lifecycle
- GIVEN a `single` definition
- WHEN the instance is resolved multiple times
- THEN the same object reference is returned each time

#### Scenario: Factory instance lifecycle
- GIVEN a `factory` definition
- WHEN the instance is resolved multiple times
- THEN a different object reference is returned each time

### Requirement 6: KoinComponent Interface

The system SHOULD provide a `KoinComponent` marker interface that gives any class access to `get()` and `inject()` extension functions for resolving dependencies without direct Koin reference.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/component/KoinComponent.kt

#### Scenario: Use KoinComponent for injection
- GIVEN a class implementing `KoinComponent`
- WHEN `val service: MyService by inject()` is used
- THEN the dependency is lazily resolved from the global Koin container

### Requirement 7: Qualifier System

The system MUST support qualifiers for disambiguating multiple definitions of the same type. Both string-based (`named("qualifier")`) and type-based (`_q<MyQualifier>()`) qualifiers MUST be supported.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/qualifier/Qualifier.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/qualifier/StringQualifier.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/qualifier/TypeQualifier.kt

#### Scenario: Named qualifier disambiguation
- GIVEN `single(named("A")) { ServiceA() }` and `single(named("B")) { ServiceB() }`
- WHEN `get<Service>(named("A"))` is called
- THEN `ServiceA` is returned

### Requirement 8: Parameter Injection

The system MUST support passing runtime parameters during resolution via `ParametersHolder`. Parameters SHOULD be accessible by index or type within the definition lambda.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/parameter/ParametersHolder.kt

#### Scenario: Pass parameters at resolution time
- GIVEN `factory { (id: String) -> UserService(id) }`
- WHEN `get<UserService> { parametersOf("user-123") }` is called
- THEN a `UserService` is created with the id "user-123"

### Requirement 9: Property System

The system SHOULD support key-value properties that can be loaded at startup and retrieved at resolution time.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/registry/PropertyRegistry.kt

#### Scenario: Load and retrieve properties
- GIVEN `properties(mapOf("db.url" to "jdbc:..."))` is configured
- WHEN `koin.getProperty<String>("db.url")` is called
- THEN the value "jdbc:..." is returned

### Requirement 10: Extension Mechanism

The system MAY support custom extensions via the `KoinExtension` interface and `ExtensionManager`. Extensions can hook into Koin events and provide additional resolution strategies.

**Implementation**: projects/core/koin-core/src/commonMain/kotlin/org/koin/core/extension/KoinExtension.kt, projects/core/koin-core/src/commonMain/kotlin/org/koin/core/extension/ExtensionManager.kt

#### Scenario: Register a custom extension
- GIVEN a class implementing `KoinExtension`
- WHEN it is registered via `koin.extensionManager.registerExtension(...)`
- THEN the extension participates in the Koin lifecycle
