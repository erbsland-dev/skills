---
name: erbsland-cpp-code-style
description: Apply the portable Erbsland C++20 code style and API documentation rules. Use when writing, editing, refactoring, or reviewing C++ in Erbsland libraries and applications, or whenever a project requests the Erbsland Code Style.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---

# Erbsland C++ Code Style

## Workflow and authority

1. Inspect the project's instructions and local guidelines before changing code.
2. Treat the project's `.clang-format` as authoritative for mechanical formatting.
3. Apply this skill as the semantic baseline; let project-specific guidelines refine or override it for their domain.
4. Format every changed C++ file and run the project's validation command, if one exists.

Interpret **must** as required, **should** as the expected default unless there is a concrete reason to deviate, and
**may** as optional.

## Formatting

- Use empty lines to separate logical blocks.
- Do not add empty lines between adjacent documented declarations or inline definitions in a header.
- Use one empty line around namespace-scope class, struct, enum, and function definitions.
- Accept `.clang-format` decisions for indentation, line length, braces, wrapping, spacing, and includes.

## Naming

- Name types and public type aliases in `PascalCase`; methods, free functions, local variables, and parameters in
  `camelCase`; and member variables in `_camelCase`.
- Name namespace-scope and static constants in `cCamelCase`.
  Name enum-like static constants in `PascalCase` when appropriate, as in `Color::Red`.
- Use `T` for a single straightforward template parameter and `tCamelCase` for multiple or descriptive parameters.
- Use lowercase nested namespace names, such as `erbsland::unittest`.
- Write preprocessor macros in `UPPER_CASE` with an `ERBSLAND_<LIBRARY>_` prefix in Erbsland libraries and applications.
  Do not use macros for constants.
- Treat initialisms as words, such as `HttpServer` and `parseUtf8`.
  Keep the documented spelling of domain-specific abbreviations.

## Files and includes

- Begin each C++ file with the project's two-line copyright block.
  Put `#pragma once` directly after it in headers.
- Use `.hpp` for headers, `.cpp` for implementations, and `.tpp` for extracted templates.
- Keep one primary class, struct, enum, alias, or logical method collection in each `hpp/cpp` module.
  Allow closely related implementation helpers to share a module.
- Match the primary type and filename, and mirror namespaces in the source directory structure.
  Subdivide a large namespace with directories without adding another namespace.
- Put private implementation details in an `impl` directory and matching `impl` namespace.
- Directly include every declaration a file uses; do not rely on unrelated transitive includes.
  Include a `cpp` file's corresponding header first.
- Split implementations beyond roughly 500 lines by logical responsibility.
  Name parts `Class_part.cpp`, `Class_part.hpp`, or `Class_part.tpp`.
  Include `tpp` parts at the bottom of the owning header without an include back to that header.
- Do not edit generated files directly.
  Modify their source or generator and regenerate them.

## CMake

- Use one `CMakeLists.txt` in each directory that contains source files.
- Add only local files with one flat `target_sources(<target> PRIVATE ...)` block.
  Add nested directories with `add_subdirectory(...)` before `target_sources`.
- Sort files and subdirectories alphabetically.

## Comments and API documentation

### Comment format

- Write API documentation with `///` and Doxygen `@` commands, without empty comment lines.
- Use `//` for short implementation notes.
  Use `/* ... */` only when an inline annotation or generated layout makes it clearer than a line comment.
- Use `@seedoc{/path}` to link to a documentation page and `@seeref{id}` to link to a reference target.
- Treat `@wip` as work in progress and ask the project owner before modifying the marked API.

### Required documentation

- In public and internal APIs, document every class, struct, enum, public type alias, public constant, namespace-scope
  function, and public method.
- Start with one brief line and document every parameter, non-void return value, thrown exception, and relevant edge or
  error case.
- Move extensive explanations to linked reference or topic documentation.
- Give every data member and enum member a brief trailing `///<` description.
- Group undocumented, explicitly defaulted or deleted special members under `// defaults` or `// defaults/deletions`.
- Give a trivial getter or setter only a one-line description without `@param` or `@return`.

### Test status

- End every documented class, struct, and namespace-scope function API block with exactly one test-status marker.
  Do not mark constructors, methods, operators, or other members.
- Use `@tested{ExampleTest OtherTest}` for one or more test-suite class names separated by spaces.
  End every name in `Test`; do not use paths or method selectors.
- Use `@notest{reason}` to explain in one line why a test is not applicable.
- Use `@needtest{reason}` to identify missing coverage in one line.

## Class organization

Group a class with repeated access specifiers.
Use one empty line before each section, none between its declarations, and an optional lowercase `//` label.
Allow simple value structs and dependency constraints to use a smaller or different layout.

Use this usual section order:

1. Place private friends, nested types, enums, and aliases in dependency order.
2. Place public types in dependency order.
3. Order the default and other constructors, destructor, copy and move constructors, then copy and move assignment.
   Put explicitly defaulted or deleted members in a final defaults group.
4. Place main public operations.
5. Group overrides in one `public: // implement Base` section per base.
6. Order comparison, arithmetic, logical, then other operators.
7. Place condition tests first, then each attribute's accessors together.
8. Group other public tools only when this improves navigation.
9. Place `to...` methods before static `from...` and other factories.
10. Place private and protected methods.
11. Place data members grouped by access.

## Modern C++

- Use portable C++20 features.
  Prefer concepts, structured bindings, designated initialization, and range-based loops when clearer.
- Use trailing return types for non-void functions where the syntax permits, such as
  `auto create() -> std::string`.
  Use `void function()` for ordinary void functions and explicit `-> void` on non-generic lambdas.
  Use concrete return and parameter types for non-generic functions and lambdas.
- Use `auto` for values when the type is apparent or clearer.
  Use `const` for immutable values and `constexpr` when usable at compile time.
- Add `[[nodiscard]]` when silently discarding a result is likely to be a mistake.
  Add `noexcept` only when the operation is guaranteed not to propagate an exception.
- Mark intentionally unused named parameters `[[maybe_unused]]`.
  Omit an unused private overload-disambiguation tag's name.
- Mark overriding functions `override` and classes deliberately closed to extension `final`.
- Express ownership explicitly.
  Use values or references by default, `std::unique_ptr` for unique ownership, `std::shared_ptr` only for shared
  ownership, raw pointers for deliberate non-owning or native boundaries, and `std::optional` for an absent value.
- Expose raw pointers through public APIs only at unavoidable interoperability boundaries.
  Use an explicitly named `Unsafe...` alias or wrapper and document ownership, lifetime, nullability, and mutability.
- Use lazy initialization for immutable or expensive data whose construction should be deferred.

## Erbsland Core integration

When Erbsland Core is available:

- Prefer `String` for read-only strings, `""_el` for literals, and `StringFormat` for formatting.
- Prefer `StringEditor` for construction and `AnyStringBuilder` for width-agnostic construction.
- Prefer Erbsland Core types and algorithms before `std::` types and algorithms.

Without Erbsland Core, use `std::format` and `std::chrono` for formatting and time handling, and prefer `std::ranges`
and `std::views` for expressive algorithms.
