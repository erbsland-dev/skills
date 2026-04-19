---
name: erbsland-unit-test
description: Write, structure, and run C++ unit tests with the Erbsland Unit Test framework in this repository. Use in C++ projects that use the framework, when creating `*Test.cpp` suites, adding `test...` methods, wiring tests in CMake with `erbsland_unittest`, choosing assertion/context/file/text helpers (`REQUIRE_*`, `WITH_CONTEXT`, `fh::readDataText`, `REQUIRE_EQUAL_LINES`), or running/filtering tests by `name:`, `target:`, and `tag:`.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---

# Erbsland Unit Test

Use this workflow for Erbsland Unit Test suites in this repository.

## Quick Workflow

1. Put suites in `test/unittest/src`, `test/src`, `test/<project>-unittest/src`, or `unittest/src`.
2. Name suite files/classes `SomethingTest` and methods `testSomething`.
3. Inherit from `el::UnitTest` or from a helper class via `UNITTEST_SUBCLASS(HelperClass)`.
4. Add `TESTED_TARGETS(...)`; add `TAGS(...)` or `SKIP_BY_DEFAULT()` only when useful.
5. Prefer `REQUIRE_*`, `WITH_CONTEXT(...)`, `runWithContext(...)`, and small helper methods over repeated inline logic.
6. Use framework file/text helpers for fixtures and multi-line output instead of ad-hoc parsing.
7. Ensure the suite is wired into `CMakeLists.txt`, run a narrow filter first, then the full executable.

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
- Prefer small suites and extract shared helpers before the file becomes hard to scan.
- Prefer `REQUIRE_*` for normal assertions; use `CHECK_*` only when continuing after failure adds signal.

## Diagnostics And Helpers

Read `references/api-and-macros.md` when choosing assertion, metadata, file, or text helpers.

Use these patterns:

- Wrap nested helper calls with `WITH_CONTEXT(functionCall(...))`.
- Use `runWithContext(SOURCE_LOCATION(), testFn, diagnoseFn)` inside loops and table-driven tests.
- Store interesting state in members and expose it via `additionalErrorMessages()`.
- Prefer comparison macros such as `REQUIRE_EQUAL(a, b)` so failures print values.
- For test data, use `fh::unitTestExecutablePath()`, `fh::resolveDataPath()`, `fh::readDataText()`, and `fh::readDataLines()`.
- For text-heavy checks, use `th::splitLineViews()`, `th::splitLines()`, `REQUIRE_EQUAL_LINES(...)`, and `REQUIRE_VALID_UTF8(...)`.

## Integrate With CMake

Follow the local pattern in `<unittest-path>/CMakeLists.txt`:

- Keep one `unittest` executable.
- Add module directories with `add_subdirectory(...)` from `<unittest-path>/src/CMakeLists.txt`.
- Let module `CMakeLists.txt` files attach sources to target `unittest`.
- Keep `erbsland_unittest(TARGET unittest ...)` enabled.
- For copied fixtures, use `COPY_TEST_DATA` and `ENABLE_DATA_DEPS`.

## Run And Debug

Typical commands when tests live in `test/unittest/`:

```bash
cmake --build cmake-build-debug --target unittest && cmake-build-debug/test/unittest/unittest
cmake-build-debug/test/unittest/unittest --list
cmake-build-debug/test/unittest/unittest name:ErrorCategory
cmake-build-debug/test/unittest/unittest name:ToCode
cmake-build-debug/test/unittest/unittest target:Parser
cmake-build-debug/test/unittest/unittest --verbose --no-color
```

## Reuse Patterns

Read `references/common-patterns.md` before writing non-trivial tests. It captures common proven patterns including:

- `UNITTEST_SUBCLASS(TestHelper)` helper inheritance.
- Stateful diagnostics through `additionalErrorMessages()`.
- `WITH_CONTEXT(...)` and `runWithContext(...)` for loop/table diagnostics.
- `TESTED_TARGETS(...)` for useful `target:` filters.

Starting a new test suite from scratch? Have a look at these real-world examples:

- `references/trivial-function-test.md`
- `references/target-with-many-values-test.md`
