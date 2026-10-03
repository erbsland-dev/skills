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
- Apply these rules manually when `.clang-format` does not enforce them:
  - Bodies of `if`, `else`, `while`, `for`, and `do` statements **must** be enclosed in `{}`.
  - There **must** be empty lines around namespace opening and closing lines, between function definitions, and after
    `#pragma once`.
  - Data blocks **should** use manual formatting when this improves readability; surround such blocks with
    `// clang-format off` and `// clang-format on`.
  - Classes **must** use logically ordered access sections according to [Class organization](#class-organization).

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
- You *must* only keep one primary class, struct, enum, alias, or logical free method collection in each `.hpp`/`.cpp` module.
  - **except** type traits that can share one header, and hash or format helpers following a class or struct. 
  - Implementation helpers shall be placed in a `impl` namespace in individual modules.
- Match the primary type and filename, and mirror namespaces in the source directory structure.
  Subdivide a large namespace with directories without adding another namespace.
- Put private implementation details in an `impl` directory and matching `impl` namespace.
- Directly include every declaration a file uses; do not rely on unrelated transitive includes.
  Include a `cpp` file's corresponding header first.
- Split implementations beyond roughly 500 code lines excluding comments, by logical responsibility.
  - Name parts `Class_part.cpp`, `Class_part.hpp`, or `Class_part.tpp`.
  - Include `tpp` parts at *the bottom* of the owning header without an include back to that header.
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
- You *must not* use `/* ... */`, except if required for *generated* code.
- Use `@seedoc{/path}` to link to a documentation page and `@seeref{id}` to link to a reference target.
- Treat `@wip` as work in progress and ask the project owner before modifying the marked API.

### Required documentation

- In public and internal APIs, you **must** document every class, struct, enum, public type alias, public constant,
  namespace-scope function, and public method, **except** comparison operators and methods or operators that are
  overridden, explicitly defaulted, or deleted.
- Each required API documentation block **must** contain:
  - one brief opening line;
  - a description of every parameter (`@param`) and template parameter (`@tparam`), **except** for trivial setters;
  - a description of the non-void return value (`@return`), **except** for getters and fluent methods;
  - a description of exceptions thrown at runtime (`@throws`).
- For non-trivial methods, you **should** also document applicable parameter ranges, thread safety, limits, edge cases,
  error conditions, and side effects.
- Move extensive explanations to linked reference or topic documentation.
- Give every data member and enum member a brief trailing `///<` description.
  If this would make the line too long, place a regular `///` description immediately before the member instead.

### Test status

- You **should** end every documented class, struct, and namespace-scope function API block with exactly one
  test-status marker.
  **Do not** mark inline types, constructors, methods, operators, or other members.
- Use `@tested{ExampleTest OtherTest}` for one or more test-suite class names separated by spaces.
  End every name in `Test`; do not use paths or method selectors.
- Use `@needtest{reason}` to identify missing coverage in one line.
- Use `@notest{reason}` to explain in one line why a test is not applicable.
  `@notest` means the type **is not or cannot be tested** for `reason`.
- Prefer `@needtest` before `@notest` if missing coverage is temporary.
- Prefer `@tested` before `@notest` if a type is implicitly tested through another type.

## Class organization

Divide a class into logical sections, repeating access specifiers as needed.
Use one empty line before each section and none between its declarations, except before a defaults group.
Add lowercase `//` labels according to the rules below.
Structs with no methods, or whose entire declaration is fewer than eight lines excluding comments, **must not** use this
section layout.

Label placement:

- Type and data-member sections **must not** have a label, regardless of access.
- Public method sections **must** have a label, except for the construction and operator sections.
- Protected method sections **should** have a label.
- Private method sections **may** have a label only when the class declaration exceeds 100 lines and the label improves
  navigation.

Use this usual section order:

1. Place private friends, nested types, enums, and aliases in dependency order.
2. Place public types in dependency order.
3. Order the construction section as follows: default constructor, other constructors, destructor, copy constructor,
   move constructor, copy assignment, then move assignment.
   Put explicitly defaulted or deleted members at the end of this section in a defaults group, as described below.
4. Place operators immediately after construction and defaults, ordered as comparison, arithmetic, logical, then other
   operators.
5. Place accessors and condition tests next, with condition tests first, then each attribute's accessors together.
6. Public overrides **must** be grouped in one `public: // implement Base` section per base class.
   This grouping takes precedence over the operator and accessor sections; keep constructors and destructors in the
   construction section.
   Overrides **should** omit API documentation to avoid duplicating the base class's documentation, and **should** follow
   the declaration order of methods in that base class.
7. Place all remaining public methods that do not match other points in this list.
   Divide them into additional logical groups only when this improves navigation.
8. Place public `to...` methods, followed by public static `from...` methods and other factories.
9. Place any remaining public static helper methods.
10. Repeat points 6, 7, and 9 for protected methods first, then private methods, using the corresponding access specifiers.
    Protected methods **must** have API documentation unless exempt under [Required documentation](#required-documentation).
    Private helper methods **should** have API documentation.
11. Group data members by access, in the order public, protected, then private.

Defaults grouping:

- Precede the group with one empty line and one of these comment lines:
  - `// defaults` if every declaration in the group uses `= default`.
  - `// defaults/deletions` if any declaration in the group uses `= delete`.

When rules conflict, choose the most readable layout; use maintainability as the next criterion.

Read [templates](references/templates.md) for complete annotated templates.

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
