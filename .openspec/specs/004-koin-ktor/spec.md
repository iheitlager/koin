---
domain: koin-ktor
version: 4.1.2
status: draft
date: 2026-02-21
---

# Koin Ktor — Server-Side Ktor Framework Integration

## Overview

Koin Ktor provides dependency injection integration for the Ktor server framework. It installs as a Ktor plugin, manages the Koin lifecycle alongside the Ktor application lifecycle, and provides request-scoped dependency resolution.

## Philosophy

- **Plugin-based**: Koin integrates via the standard Ktor plugin mechanism (`install(Koin)`).
- **Request scoping**: Each HTTP request gets its own Koin scope, enabling request-local state.
- **Dynamic modules**: Modules can be loaded and unloaded at runtime for multi-tenant or plugin-based architectures.

## Requirements

### Requirement 1: Ktor Plugin Installation

The system MUST provide a Ktor plugin (`Koin`) that configures and starts the Koin container with the Ktor application lifecycle. Koin MUST start when the application starts and stop when the application stops.

**Implementation**: projects/ktor/koin-ktor/src/commonMain/kotlin/org/koin/ktor/plugin/KoinPlugin.kt

#### Scenario: Install Koin plugin
- GIVEN a Ktor application
- WHEN `install(Koin) { modules(appModule) }` is called
- THEN Koin is started and accessible via `application.koin()`

#### Scenario: Koin stops with Ktor
- GIVEN a running Ktor application with Koin installed
- WHEN the application is stopped
- THEN Koin is stopped and all instances are released

### Requirement 2: Request-Scoped Dependencies

The system MUST create a Koin scope for each HTTP request and close it after the response is sent. Scoped definitions MUST be resolvable within request handlers.

**Implementation**: projects/ktor/koin-ktor/src/commonMain/kotlin/org/koin/ktor/plugin/RequestScope.kt

#### Scenario: Request scope lifecycle
- GIVEN a module with `scope<RequestScope> { scoped { RequestLogger() } }`
- WHEN an HTTP request is processed
- THEN a scope is created before the route handler, and closed after the response

#### Scenario: Access request scope in route handler
- GIVEN an active request scope
- WHEN `call.scope.get<RequestLogger>()` is used in a route handler
- THEN the request-scoped instance is returned

### Requirement 3: Dynamic Module Loading

The system SHOULD support loading and unloading modules at runtime via `Application.koinModule()` and `Application.koinModules()`.

**Implementation**: projects/ktor/koin-ktor/src/commonMain/kotlin/org/koin/ktor/plugin/KoinPlugin.kt

#### Scenario: Load module at runtime
- GIVEN a running Ktor application with Koin
- WHEN `application.koinModule(newModule)` is called
- THEN definitions from `newModule` are available for injection

### Requirement 4: SLF4J Logger Integration

The system MAY provide an SLF4J-based logger for Koin diagnostic output in server environments.

**Implementation**: projects/ktor/koin-logger-slf4j/src/main/kotlin/org/koin/logger/SLF4JLogger.kt

#### Scenario: Use SLF4J logging
- GIVEN a Ktor application with Koin
- WHEN `logger(SLF4JLogger())` is configured
- THEN Koin log output is routed through SLF4J
