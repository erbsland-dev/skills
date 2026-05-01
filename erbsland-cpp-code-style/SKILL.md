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
  - No empty lines around function definitions in headers. Separated by a `/// API doc`.
  - One empty line around ``struct``, ``class``, or ``enum`` definitions
  - One empty line around function implementations in ``.cpp`` files
- **.clang-format**: If exist in project root, the result is accepted. Still some manual adjustments may be necessary.
- **pre_commit.py**: If exist (in `utilities` or `tools`), authorative but slow tool
  for formatting before any commit (called via `python3 utilities/pre_commit.py`).

## Naming Conventions
- **Classes**: PascalCase (e.g., ``Controller``, ``TestClassBase``)
- **Methods and Functions**: camelCase (e.g., ``addTestClass``, ``parseCommandLine``)
- **Member Variables**: ``_underscorePrefix`` (e.g., ``_console``, ``_testClasses``)
- **Global Constants**: ``cExample`` if not in dedicated namespace.
- **Namespaces**: all lowercase with nested structure (e.g., ``erbsland::unittest``)
- **Preprocessor Macros**: all uppercase ``EXAMPLE``; prefix ``ERBSLAND_<LIB_NAME>_`` for Erbsland apps/libs.

## File Organization
- **Copyright Block**: Two line comments `// Copyright ...` at the begin of the file.
- **Header Guards**: Use ``#pragma once`` directly after copyright block (no empty line!).
- **Include Order** (sort each block individually, separate blocks with empty lines):
  1. Corresponding header (only in ``.cpp``) — *followed by two empty lines*, or two empty lines after `#pragma once`.
  2. Local includes: ``#include "Example.hpp"``
  3. Local subdirectory includes: ``#include "sub/sub/Example.hpp"``
  4. Local adjoint includes: ``#include "../Example.hpp"`` (exact one `..`)
  5. Local relative includes: ``#include "../../Example.hpp"`` (two and more `..`)
  6. Project libraries: ``#include <erbsland/unittest/UnitTest.hpp>``
  7. Standard library headers: ``#include <vector>``
  8. *Two* empty lines after the include block
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
- Group related functions in classes with additional `public:` sections.

## Class organization.
Each of the following is considered as a "section", separated by: empty line, `public/private/protected:`.
Omitted if not applicable.
1. Private types: (no marker, order depends on dependencies)
   - `friend class`
   - `struct`/`class`
   - `enum class`/`enum`
   - aliases/usages `using`
2. Public types: (`public:` no comment, order depends on dependencies)
   - `struct`/`class`
   - `enum class`/`enum`
   - `using`
3. Public constructors: (`public:` no comment)
   - ctor with no arguments first (even when default)
   - ctors with arguments
   - non default copy / move ctors
   - non default assignment / move operators.
   - line comment `// defaults` or `// defaults/delections`
   - `= default` and `= delete` copy ctors
   - `= default` and `= delete` assign/move operators
4. Main methods: (`public:` no comment)
   - e.g. `create() -> MyClassPtr`
5. Implemented methods from superclasses: (`public: // implement <Superclass>`, on section for each class)
   - Overridden methods in order.
6. Operators: (`public: // operators`)
   - comparison operators
   - arithmetic operators
   - logical operators
   - others
7. Accessors: (`public: // accessors`, getters/setters/tests)
   - tests (`isValue()`)
   - getters/setters grouped per attribute
     (`value()`, `setValue(value)`, `convenienceGetter()`, `setConvenienceSetter(value)`)
8. Tools / Everything Else: (`public:`, only add comment if group is distinct)
9. Conversion Methods: (`public: // conversion`)
   - Conversion to methods `auto toStringList() const`, `to...`
   - Conversion from builders `static auto from...() -> MyClass`
10. Private methods: (`private:` no comment)
   - Private helper methods for implementation details
   - Private constructors for specific use cases
11. Public/Protected/Private attributes: (`public:`, `protected:`, `private:` no comment)
   - e.g. `int _value; ///< The value`

## Constants & Literals
- Use ``constexpr`` or ``const`` where applicable
- Never use macros for constants!
- Use lazy initialization when possible (like `auto myData() { static auto data = ...; return data; }`)

## Modern C++ Usage
- Prefer a modern C++20 syntax
- Always use **trailing return types** for all *non-void* functions: ``auto create() -> std::string;``
- Use ``auto`` when the type is obvious or improves readability,
  *but* specify types in arguments for return types for non-generic lambdas and functions
- Use structured bindings ``const auto &[a, b] =``
- Use ``[[nodiscard]]`` and ``noexcept`` where applicable
- Use ``[[maybe_unused]]`` for unused parameters, except for private tags - here omit the parameter name.
- Use ``override`` for overwritten/implemented functions
- Use ``final`` for final classes
- Use **designated initialization** if it improves the readability of the code
- Use ``std::unique_ptr`` / ``std::shared_ptr`` instead of raw pointers
- Prefer range-based ``for`` loops
- Use ``std::format`` and ``std::chrono`` for formatted output and timing
- Prefer ``std::ranges`` and ``std::views`` for expressive algorithms


