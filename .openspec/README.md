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

These specs follow a traceability metamodel:
- **IMPLEMENTED_BY**: Each requirement links to source files via `**Implementation**:` lines
- **ADDRESSED_BY**: Requirements reference ADRs via `ADR-XXXX` notation
- **TESTED_BY**: Scenarios link to test functions (inferred from code structure)

## Origin

These specifications were reverse-engineered from the Koin 4.1.2 codebase using Claude Code Opus 4.5. They document the current implemented behavior, not aspirational features.

---

*Last updated: 2026-02-21*
