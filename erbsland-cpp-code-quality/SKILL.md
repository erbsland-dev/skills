---
name: erbsland-cpp-code-quality
description: Erbsland C++20 quality rules for writing, reviewing, refactoring, and extending code.
---

Prioritize safety, reliability, readability/smallness, testability, then adequate speed.
Prefer explicit, conventional designs; refactor weak responsibilities before extending.
Keep behavior with data; wrap enums accumulating logic.
Treat types with static-only methods/variables, one-call internal wrappers, and in-function helper lambdas as smells.

Use RAII; forbid manual memory management and unguarded raw pointers in regular code.
Instead of raw pointers, prefer references, smart pointers, `std::span` or `std::optional`.\
Express ownership/invariants; check boundaries, not guaranteed states.
Prefer defined failure to UB.

Test every module and important function.
Expose logic through public or `impl/` interfaces.
Avoid `.cpp`-only helpers/types, anonymous namespaces, and function embedded lambdas unless isolation is necessary.
Abstract noncritical OS APIs for mocks.

Use one maintained type per header.
Add `.cpp` above 20 implementation lines or to isolate dependencies; keep source below 500 lines.
Refactor before splitting; name parts `<Type>_<part(lowercase)>.cpp`/`.tpp`.

Document public/cross-module/impl interfaces: purpose, contract, failures, ranges.
API docs shall always make purpose, parameter, return values, exceptions, possible errors and special cases clear.
Omit obvious special members/private details; minimize trivial accessors/overrides.
Put complex invariants and extension points in Sphinx Implementation Notes.

Refactor toward clear responsibilities, low duplication/coupling, and tests.

Prefer design/code that: make invalid use hard; responsibilities clear; is easy to test; is modular and easy to extend; has defined behaviour; keeps performance acceptable.

When reviewing code, check if:
- methods/behavior live with the correct (data) type;
- ownership is explicit;
- the code is safe by default;
- the code is testable without hacks;
- design is ready for future extension;
- there are untestable hidden functions;
- files are small and cohesive;
- all public interfaces are documented according to rules;
- an implementation easy to understand and does no clever hacks;
- the code uses modern C++20 appropriately.
