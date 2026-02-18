---
name: erbsland-unit-test
description: Write, structure, and run C++ unit tests with the Erbsland Unit Test framework in this repository. Use in C++ projects that use the Erbsland Unit Test framework, when creating new `*Test.cpp` suites, adding `test...` methods, wiring tests in CMake with `erbsland_unittest`, choosing assertion/context macros (`REQUIRE`, `CHECK`, `WITH_CONTEXT`, `runWithContext`), or running/filtering tests by `name:`, `target:`, and `tag:`.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---

# Erbsland Unit Test

Use this workflow to implement or update tests in this repository with conventions that match the framework docs and existing test suites.

## Quick Workflow

1. Place or update tests, usually in `test/unittest/src`, `test/src` or `unitttest/src`.
2. Name suite files/classes `SomethingTest` and methods `testSomething`.
3. Inherit from `el::UnitTest` or from a helper class via `UNITTEST_SUBCLASS(HelperClass)`.
4. Use `REQUIRE_*`/`CHECK_*` macros and `WITH_CONTEXT(...)` for loops/helpers.
5. Add `TESTED_TARGETS(...)` (and `TAGS(...)` or `SKIP_BY_DEFAULT()` when needed).
6. Ensure sources are added by the relevant `CMakeLists.txt` (directly or via module subdirectories).
7. Build and run the target test executable, then narrow failures via `name:` or `target:` filters.

## Author Tests

Use this suite skeleton:

```cpp
#include <erbsland/unittest/UnitTest.hpp>

TESTED_TARGETS(MyClass)
class MyClassTest final : public el::UnitTest {
public:
    void testConstruction() {
        REQUIRE(true);
    }
};
```

Apply these rules:

- Keep test methods `public`, `void`, no arguments, and prefixed with `test`.
- Do not prefix helper methods with `test`.
- Prefer small test methods with clear setup/assert sections.
- Prefer `REQUIRE_*` for hard stop checks and `CHECK_*` only when continuing is useful.
- For table-style loops, wrap helper calls with `WITH_CONTEXT(...)` to add context to the testoutput on failure.
- For local diagnostics, use `runWithContext(SOURCE_LOCATION(), testFn, diagnoseFn)`.
- For suite-wide diagnostics, override `additionalErrorMessages()` and catch all exceptions in that method.

## Use Framework Features Correctly

Read `references/api-and-macros.md` when selecting assertions, metadata macros, or file helper APIs.

Use metadata macros intentionally:

- `TESTED_TARGETS(TargetName)` to enable `target:TargetName` filtering.
- `TAGS(tag1, tag2, ...)` for thematic grouping.
- `SKIP_BY_DEFAULT()` for expensive/optional tests that should run only when explicitly enabled (`+name`, `+target`, `+tag`).

## Test Auto-Registration

- Test suites (class ending in `..Test`) and test methods (prefix `void test...()`) are automatically scanned and registered via CMake build system. No manual registration is required.

## Integrate With CMake

Use the existing pattern in `<unittest-path>/CMakeLists.txt`:

- Keep one `unittest` executable.
- Add module directories with `add_subdirectory(...)` from `<unittest-path>/src/CMakeLists.txt`.
- Let module `CMakeLists.txt` files attach test sources to target `unittest`.
- Keep `erbsland_unittest(TARGET unittest ...)` enabled.

For data files, prefer `COPY_TEST_DATA` in CMake and `erbsland::unittest::fh::{resolveDataPath,readDataText,readDataLines}` in tests.

## Run and Debug in This Repository

Primary commands (if unittest are in `test/unittest/`):

```bash
cmake --build cmake-build-debug --target unittest && cmake-build-debug/test/unittest/unittest
cmake-build-debug/test/unittest/unittest --list
cmake-build-debug/test/unittest/unittest name:ErrorCategory
cmake-build-debug/test/unittest/unittest name:ToCode
cmake-build-debug/test/unittest/unittest target:Parser
cmake-build-debug/test/unittest/unittest --verbose --no-color
```

When adding tests, run a narrow filter first, then the full executable.

## Reuse Real Patterns

Read `references/common-patterns.md` before writing non-trivial tests. It captures common proven patterns including:

- `UNITTEST_SUBCLASS(TestHelper)` helper inheritance.
- Stateful diagnostics through `additionalErrorMessages()`.
- `WITH_CONTEXT(...)` and `runWithContext(...)` in loops/map checks.
- `TESTED_TARGETS(...)` usage for meaningful target filters.
