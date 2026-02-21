---
domain: koin-test
version: 4.1.2
status: draft
date: 2026-02-21
---

# Koin Test — Testing Utilities and Module Verification

## Overview

Koin Test provides utilities for testing Koin-based applications. It includes a test interface for container access, mock injection support, and a module verification system that validates dependency graphs at compile/test time without starting the full container.

## Philosophy

- **Fail fast**: Module verification catches missing definitions and circular dependencies before runtime.
- **Lightweight test access**: `KoinTest` interface gives test classes simple access to the container.
- **Mock-friendly**: Built-in mock provider support for replacing real definitions with test doubles.

## Requirements

### Requirement 1: KoinTest Interface

The system MUST provide a `KoinTest` marker interface that gives test classes access to `get()` and `inject()` functions for resolving dependencies from the Koin container.

**Implementation**: projects/core/koin-test/src/commonMain/kotlin/org/koin/test/KoinTest.kt

#### Scenario: Resolve dependency in test
- GIVEN a test class implementing `KoinTest`
- WHEN `val service = get<UserService>()` is called
- THEN the dependency is resolved from the active Koin container

### Requirement 2: Module Verification

The system MUST provide a `verify()` function that validates module definitions without starting the full container. Verification MUST detect missing definitions and circular dependencies.

**Implementation**: projects/core/koin-test/src/jvmMain/kotlin/org/koin/test/verify/Verification.kt

#### Scenario: Detect missing definition
- GIVEN a module where `ServiceA` depends on `ServiceB` but `ServiceB` is not declared
- WHEN `module.verify()` is called
- THEN a verification error reports the missing `ServiceB` definition

#### Scenario: Detect circular dependency
- GIVEN a module where `A` depends on `B` and `B` depends on `A`
- WHEN `module.verify()` is called
- THEN a verification error reports the circular dependency

#### Scenario: Verify with extra types
- GIVEN a module with definitions that depend on Android or platform types not in the graph
- WHEN `module.verify(extraTypes = listOf(Context::class))` is called
- THEN the verification treats the extra types as satisfied

### Requirement 3: Mock Provider

The system SHOULD provide a `MockProvider` for creating mock instances during testing. The mock provider MUST be configurable with the user's preferred mocking library.

**Implementation**: projects/core/koin-test/src/commonMain/kotlin/org/koin/test/mock/MockProvider.kt

#### Scenario: Configure mock provider
- GIVEN a test setup
- WHEN `MockProvider.register { clazz -> mockk(clazz) }` is called
- THEN Koin test utilities use the registered provider to create mocks

### Requirement 4: Check Modules DSL

The system SHOULD provide a `checkModules` DSL for validating module graphs with parameter bindings.

**Implementation**: projects/core/koin-test/src/commonMain/kotlin/org/koin/test/check/CheckModulesDSL.kt

#### Scenario: Check modules with parameter binding
- GIVEN a module with parameterized definitions
- WHEN `koinApplication { modules(appModule) }.checkModules { withParameter<MyService> { parametersOf("test") } }`
- THEN all definitions are instantiated with the provided parameters to verify they can be created

### Requirement 5: JUnit Integration

The system SHOULD provide JUnit 4 and JUnit 5 rules/extensions for automatic Koin lifecycle management in tests.

**Implementation**: projects/core/koin-test-junit4/src/main/kotlin/org/koin/test/AutoCloseKoinTest.kt, projects/core/koin-test-junit5/src/main/kotlin/org/koin/test/junit5/AutoCloseKoinTest.kt

#### Scenario: Auto-close Koin after test
- GIVEN a test class implementing `AutoCloseKoinTest`
- WHEN a test method completes
- THEN Koin is automatically stopped and cleaned up

### Requirement 6: Android Module Verification

The system SHOULD provide Android-specific verification helpers that pre-register common Android types (Context, Application, SavedStateHandle, etc.) as extra types.

**Implementation**: projects/android/koin-android-test/src/main/java/org/koin/android/test/verify/AndroidVerify.kt

#### Scenario: Verify Android modules
- GIVEN a module with definitions depending on Android types
- WHEN `module.androidVerify()` is called
- THEN common Android types are automatically included as extra types for verification
