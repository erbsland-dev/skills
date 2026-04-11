---
name: erbsland-cpp-code-quality
description: c++20 code quality guidance for erbsland libraries and applications. Use when writing, reviewing, refactoring, or extending C++ code for Erbsland projects, especially when making design decisions, splitting modules, improving testability, or enforcing safety and readability conventions.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---

# Erbsland C++ Code Quality Guidelines

Apply these rules when changing C++ code in Erbsland libraries and applications.

## Core Priorities

Prefer code that is, in this order:

1. Safe
2. Reliable
3. Readable
4. Testable
5. Fast enough

Optimize for correctness and maintainability first. Trust the compiler to optimize straightforward code. Do not introduce clever or compact constructs merely to reduce line count.

When forced to choose, prefer an explicit failure over silent failure or undefined behavior.

## Design First

Do not patch new functionality onto a weak structure.

When extending code:

1. Re-evaluate the design first.
2. Refactor responsibilities if needed.
3. Add the new functionality only after the structure is clear.

Prefer designs that remain understandable and extensible after the change. Avoid short-term hacks that make future work harder.

## Preferred Design Style

### Prefer data-centered types

Prefer designs where behavior lives with the data it operates on.

- Add related operations to the owning class or struct instead of scattering helper functions externally.
- When an `enum class` starts accumulating conversions, formatting, parsing, validation, or domain logic, consider replacing it with a small wrapper type that encapsulates the enum and provides those operations explicitly.

### Prefer proven patterns

Prefer well-known, conventional design patterns over ad-hoc inventions. Choose patterns that make ownership, flow, and responsibilities easier to understand.

### Prefer explicit code

Prefer verbose, direct code over dense or clever expressions when both compile to equally good machine code.

Good:
- Explicit control flow
- Named intermediate values
- Clear ownership
- Narrow interfaces

Avoid:
- Obscure template tricks without strong justification
- Over-compressed one-liners
- Hidden control flow
- Implicit ownership assumptions

## Safety Rules

Default to safe code.

- Prefer references over pointers where absence is impossible.
- Prefer smart pointers over raw owning pointers.
- Do not use manual memory management.
- Add bounds, range, and validity checks where they protect correctness.
- Make illegal states hard to represent.
- Keep ownership explicit.
- Use `std::optional` for optional values.

The use of raw pointers must be documented and justified in detail. It must be reviewed by a human senior developer.

## Testability Rules

Write unit tests for all modules and all important functions.

Prefer designs that can be tested through clear public interfaces. If details need isolation, move them into a named `impl` namespace with well-designed interfaces rather than hiding them completely.

When code interacts with OS or platform APIs and the code path is not performance-critical:

- Add a thin abstraction layer.
- Isolate the system dependency behind an interface or narrow adapter.
- Make the behavior mockable in unit tests.

Do not hide logic in places that are difficult to exercise from tests.

## Avoid Anonymous Namespaces

Do not use anonymous namespaces as a routine organization tool.

Anonymous namespaces are a last resort. In most cases they:

- Hide logic that should belong to a type
- Encourage file-local helper sprawl
- Reduce testability
- Make reuse harder

Prefer these alternatives:

- Move small helpers into private member functions.
- Move cohesive internal logic into a named `impl` namespace.
- Introduce a small internal type when behavior needs state or structure.

Use an anonymous namespace only when true translation-unit isolation is required and no better design exists.

## File and Module Organization

Prefer one manually maintained type per module.

- Usually use one file pair per class, struct, or major enum-related type.
- Keep manually written source files below 500 lines.
- If a file grows too large, refactor responsibilities before splitting mechanically.

When a `.cpp` file still needs splitting after refactoring, use this naming pattern:

- `<type_name>.hpp`
- `<type_name>.cpp`
- `<type_name>_<logical_unit>.cpp`

Use underscores only for logical sub-units that remain clearly owned by the same main type.

Do not split code into arbitrary fragments without a clear responsibility boundary.

## Refactoring Guidance

When improving existing code, apply these steps:

1. Clarify the responsibility of each type.
2. Move behavior closer to the data it belongs to.
3. Remove duplicated helper logic.
4. Introduce internal structure before adding more features.
5. Reduce hidden coupling.
6. Add or improve tests alongside the refactor.

Always leave the code in better shape than you found it.

## Documentation Rules

Take documentation seriously.

Document every public interface, including internal public interfaces used across modules.

Document at least:

- Purpose
- Inputs and outputs
- Preconditions and invariants
- Failure behavior where relevant

When implementation logic becomes complex, add deeper design documentation for maintainers.

- Create a new "Implementation Notes" chapter with an additional page in the Sphinx documentation.
- Prefer documentation that explains structure, intent, invariants, and extension points rather than repeating the code line by line.
- For complex subsystems, add implementation notes aimed at senior developers who need to extend or maintain the code.

## Modern C++ Expectations

Prefer modern C++20 style.

Use modern language and library features when they improve clarity, safety, and expressiveness. Do not use older patterns when better standard alternatives exist.

Examples of preferred direction:

- `std::span`
- `std::string_view`
- `std::optional`
- `std::variant`
- ranges where they improve readability
- RAII everywhere ownership or cleanup matters
- strong typing over primitive flags and loosely related parameters

Do not use modern features just for style. Use them when they make the code simpler, safer, or clearer.

## Decision Heuristics

When choosing between alternatives, prefer the option that:

1. Makes invalid use harder
2. Makes responsibilities clearer
3. Is easier to test
4. Is easier to extend without redesign
5. Avoids surprising behavior
6. Keeps performance acceptable without sacrificing clarity

## Review Checklist

When writing or reviewing code, check the following:

- Is the design clean enough for future extension?
- Does behavior live with the correct data type?
- Is ownership explicit?
- Is the code safe by default?
- Is the code testable without hacks?
- Are there unnecessary anonymous namespaces?
- Is the file still small and cohesive enough?
- Are public interfaces documented?
- Is the implementation clear without being clever?
- Does the code use modern C++20 constructs appropriately?
