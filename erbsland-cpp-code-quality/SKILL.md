---
name: erbsland-cpp-code-quality
description: c++20 code quality guidance for Erbsland libraries and applications. Use when writing, reviewing, refactoring, or extending C++ code for Erbsland projects, especially for design decisions, module splitting, testability, safety, and readability conventions.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---

# Erbsland C++ Code Quality Guidelines

Apply these rules when changing C++ code in Erbsland libraries and applications.

## Core Priorities

Prefer, in order:

1. Safe
2. Reliable
3. Readable
4. Testable
5. Fast enough

Optimize for correctness and maintainability first. Trust the compiler to optimize straightforward code. Do not use clever or compact constructs just to reduce line count.

When forced to choose, prefer explicit failure over silent failure or undefined behavior.

## Design First

Do not patch new functionality onto a weak structure.

When extending code:

1. Re-evaluate the design.
2. Refactor responsibilities if needed.
3. Add functionality only after the structure is clear.

Prefer designs that stay understandable and extensible. Avoid short-term hacks that make future work harder.

## Preferred Design Style

### Prefer data-centered types

Keep behavior with the data it operates on.

- Add related operations to the owning class or struct instead of scattering external helpers.
- If an `enum class` accumulates conversions, formatting, parsing, validation, or domain logic, consider replacing it with a small wrapper type that encapsulates the enum and exposes those operations explicitly.

### Prefer proven patterns

Prefer conventional, well-known design patterns over ad-hoc inventions. Choose patterns that make ownership, flow, and responsibilities easier to understand.

### Prefer explicit code

Prefer direct, verbose code over dense or clever expressions when both compile to equally good machine code.

Prefer:
- Explicit control flow
- Named intermediate values
- Clear ownership
- Narrow interfaces

Avoid:
- Obscure template tricks without strong justification
- Over-compressed one-liners
- Hidden control flow
- Implicit ownership assumptions

## Red Flags for Design Issues

- Classes or structs without data and only static methods
- Tiny internal methods with one caller that only wrap another call (acceptable in public API for BWC)
- Inlined lambda functions that would work as private class members

## Safety Rules

Default to safe code.

- Do not use manual memory management; avoid raw pointers.
- Prefer smart pointers over raw owning pointers.
- Prefer references to pointers where absence is impossible.
- Add bounds, range, and validity checks where they protect correctness.
- Make illegal states hard to represent.
- Keep ownership explicit.
- Use `std::optional` for optional values.

Any raw pointer use must be documented and justified in detail and reviewed by a human senior developer.

## Testability Rules

Write unit tests for all modules and all important functions.

Prefer designs testable through clear public interfaces. If details need isolation, move them into a named `impl` namespace with well-designed interfaces instead of hiding them completely.

When code interacts with OS or platform APIs and the path is not performance-critical:

- Add a thin abstraction layer.
- Isolate the system dependency behind an interface or narrow adapter.
- Make behavior mockable in unit tests.

Do not hide logic in places that are hard to exercise from tests.

## Avoid Anonymous Namespaces

Do not use anonymous namespaces as a routine organization tool.

They are a last resort because they:

- Hide logic that should belong to a type
- Encourage file-local helper sprawl
- Reduce testability
- Make reuse harder

Prefer instead:

- Private member functions for small helpers
- A named `impl` namespace for cohesive internal logic
- A small internal type when behavior needs state or structure

Use an anonymous namespace only when true translation-unit isolation is required and no better design exists.

## File and Module Organization

Prefer one manually maintained type per module.

- Usually use one header per class, struct, or major enum-related type.
- Add a `.cpp` file if total method/class implementation exceeds 20 lines.
- Keep source files below 500 lines; refactor if exceeded.
- If a file grows too large, refactor responsibilities before splitting mechanically.

If a `.cpp` still needs splitting after refactoring, use:

- `<type_name>.hpp`
- (`<type_name>.cpp`)
- `<type_name>_<logical_unit1>.cpp`
- `<type_name>_<logical_unit2>.cpp`

Always use an underscore to clearly mark files as part of the same logical unit.

Split code only along clear responsibility boundaries.

## Refactoring Guidance

When improving existing code:

1. Clarify each type's responsibility.
2. Move behavior closer to the data it belongs to.
3. Remove duplicated helper logic.
4. Introduce an internal structure before adding more features.
5. Reduce hidden coupling.
6. Add or improve tests alongside the refactor.

Always leave the code in better shape than you found it.

## Documentation Rules

Take documentation seriously.

Document every public interface, including internal public interfaces used across modules.

No documentation needed for:
- Default ctors, dtors, operators
- Methods derived from base classes; place them in a separate `public: // derived from <ClassName>` block
- Private methods, structs, and enums used only internally when their purpose is clear from the name

Document at least:
- Purpose
- Parameters and return values
- Failure behavior and exceptions where relevant
- Valid input ranges where relevant

Relaxed or minimal documentation:
- Obvious getters and setters without side effects
- Public methods used only internally

When implementation logic becomes complex, add deeper maintainer documentation:

- Create an `Implementation Notes` chapter with an additional page in the Sphinx documentation.
- Explain structure, intent, invariants, and extension points rather than repeating code line by line.
- For complex subsystems, target senior developers who need to extend or maintain the code.

## Modern C++ Expectations

Prefer modern C++20 style that works well with Clang, GCC, and MSVC.

Use modern language and library features when they improve clarity, safety, or expressiveness. Do not keep older patterns when better standard alternatives exist.

Preferred examples:
- `std::span`, `std::string_view`, `std::format`, `std::optional`, `std::variant`, `std::filesystem`, `std::chrono::...`, `std::ranges::...`, `std::views::...`, `std::concepts`
- Ranges where they improve readability
- RAII wherever ownership or cleanup matters
- Strong typing over primitive flags and loosely related parameters

Do not use modern features just for style; use them only when they make code simpler, safer, or clearer.

## Decision Heuristics

When choosing between alternatives, prefer the option that:

1. Makes invalid use harder
2. Makes responsibilities clearer
3. Is easier to test
4. Is easier to extend without redesign
5. Avoids surprising behavior
6. Keeps performance acceptable without sacrificing clarity

## Review Checklist

When writing or reviewing code, check:

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