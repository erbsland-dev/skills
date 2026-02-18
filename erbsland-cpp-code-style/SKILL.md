---
name: erbsland-cpp-code-style
description: Learn the C++ code style/guidelines for all Erbsland libraries and applications. Use when writing/editing code of C++ Erbsland libs/apps or when requested to use "Erbsland Code Style" in C++ projects.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---
# Erbsland Code Style and Code Guidelines
## Style
- **Indentation**: 4 spaces, no tabs
- **Line Length**: Target a maximum of *120 characters* per line
- **Braces**: Opening brace stays on the same line
- **Spacing and Separation**:
  - Use empty lines to separate logical blocks
  - One empty line between function definitions if they span >3 lines or have documentation
  - Two empty lines around ``struct``, ``class``, or ``enum`` definitions
  - Two empty lines around function implementations in ``.cpp`` files
- **.clang-format**: Often preconfigured in project root. Result is accepted, manual formating is preferred. 

## Naming Conventions
- **Classes**: PascalCase (e.g., ``Controller``, ``TestClassBase``)
- **Methods and Functions**: camelCase (e.g., ``addTestClass``, ``parseCommandLine``)
- **Member Variables**: ``_underscorePrefix`` (e.g., ``_console``, ``_testClasses``)
- **Global Constants**: ``cExample`` if not in dedicated namespace.
- **Namespaces**: all lowercase with nested structure (e.g., ``erbsland::unittest``)
- **Preprocessor Macros**: all uppercase ``EXAMPLE``; prefix ``ERBSLAND_<LIB_NAME>_`` for Erbsland apps/libs.

## File Organization
- **Copyright Block**: Two line comments `// Copyright ...` at the begin of the file.
- **Header Guards**: Use ``#pragma once`` directly after copyright block.
- **Include Order** (sort each block individually, separate blocks with empty lines):
  1. Corresponding header (only in ``.cpp``) — *followed by two empty lines*, or two empty lines after `#pragma once`.
  2. Local includes: ``#include "Example.hpp"``
  3. Local subdirectory includes: ``#include "sub/sub/Example.hpp"``
  4. Local adjoint includes: ``#include "../Example.hpp"`` (exact one ..)
  4. Local relative includes: ``#include "../../Example.hpp"`` (two and more ..)
  5. Project libraries: ``#include <erbsland/unittest/UnitTest.hpp>``
  6. Standard library headers: ``#include <vector>``
  7. *Two* empty lines after the include block
- **File Extensions**: Use ``.hpp`` for headers, ``.cpp`` for implementations
- **File Scope**: Aim for one class per file, with matching name
- **Maximum File Length**: Try to keep files under 500 lines

## Comments and API Documentation
- **API Docs**: Use ``///`` lines and Doxygen ``@``-syntax
- **Inline Comments**: Use ``//`` for short clarifying notes
- **Block Comments**: Avoid ``/* ... */`` unless necessary
- **Special Doxygen Tags**:
  - ``@tested``: Indicates this is covered by a unit test (name or path to the tests)
  - ``@notest``: Explains why a unit test is not applicable
  - ``@needtest``: Flags untested functionality
  - ``@wip``: This part is work in progress—talk to the author before modifying
- Compact `///` doc blocks with no empty lines are preferred for large header files.

## Constants & Literals
- Use ``constexpr`` or ``const`` where applicable
- Never use macros for constants!
- Use lazy initialization when possible (like `auto myData() { static auto data = ...; return data; }`)

## Modern C++ Usage
- Prefer a modern C++20 syntax
- Always use **trailing return types** for all *non-void* functions: ``auto create() -> std::string;``
- Use ``auto`` when the type is obvious or improves readability,
  *but* specify types in arguments for for return types for non-generic lambdas and functions
- Use structured bindings ``const auto &[a, b] =``
- Use ``[[nodiscard]]`` and ``noexcept`` where applicable
- Use ``override`` for overwritten/implemented functions
- Use ``final`` for final classes
- Use **designated initialization** if it improves the readability of the code
- Use ``std::unique_ptr`` / ``std::shared_ptr`` instead of raw pointers
- Prefer range-based ``for`` loops
- Use ``std::format`` and ``std::chrono`` for formatted output and timing
- Prefer ``std::ranges`` and ``std::views`` for expressive algorithms

## Guidelines: Quality and Safety First
- Prefer code that is: safe, reliable, readable, testable.
- Avoid manual memory management and raw pointers
- Trust the compiler to optimize; prioritize clarity over micro-optimization
- Perform bounds and range checks
- Write unit tests for all modules and key functions
- Ensure good test coverage and fail clearly when things go wrong
- A crash is better than silent failure or undefined behavior
- For helper functions, use anonymous namespaces as a last resort:
  - prefer private static member methods
  - put helper methods in separate modules when there is a chance for reuse
- Even for implementation details, add API comments if the purpose is not obvious.


