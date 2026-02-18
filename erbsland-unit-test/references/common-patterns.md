# Common Patterns

Use these common patterns when adding tests in this repository.

## Suite Structure

Preferred style:

- If multiple suites share same test functionality, include/create a local helper header (for example `TestHelper.hpp` or module-specific helper).
- Add `TESTED_TARGETS(...)` before the class.
- Inherit from `UNITTEST_SUBCLASS(TestHelper)` when helper utilities are needed.
- Mark test class `final`.

Example shape:

```cpp
#include "TestHelper.hpp"

TESTED_TARGETS(ErrorCategory)
class ErrorCategoryTest final : public UNITTEST_SUBCLASS(TestHelper) {
public:
    void testDefaultConstruction() {
        REQUIRE(true);
    }
};
```

## Diagnostics Pattern

Store tested state in members variables and override `additionalErrorMessages()`.

Use this when a failure needs structured state output:

- Keep primary tested objects as members.
- Reset them in `setUp()` when required.
- Catch all exceptions in `additionalErrorMessages()` and return a fallback string.

## Context Pattern for Iteration

Two common approaches are:

1. Wrap helper assertions with `WITH_CONTEXT(...)` inside loops, this adds the file and line in diagnostic output on failure.
2. Use `runWithContext(SOURCE_LOCATION(), testLambda, diagnoseLambda)` for detailed diagnostics.

Best practice for table-driven checks and map/set verification, using helper functions, to find the error origin.

## Assertion Style

Recommended style:

- `REQUIRE_*` for regular checks.
- `REQUIRE_NOTHROW` is used around setup/setup-adjacent calls.
- `REQUIRE_THROWS_AS` is used for explicit error-path validation.
- Comparison-specific macros (`REQUIRE_EQUAL`, `REQUIRE_LESS`, ...) are used for clearer intent and good diagnostic output.
- `CHECK_*` has no actual use for real tests.

## Build Integration Pattern

Common layout:

- Root unit-test target: `test/unittest/CMakeLists.txt`
- Module registration: `test/unittest/src/CMakeLists.txt`
- Per-module file lists: `test/unittest/src/<module>/CMakeLists.txt`
- Unit-test library: `test/erbsland-unittest/...`

Use local `CMakeLists.txt` in each subdirectory:

1. Add new suite source to the relevant module `CMakeLists.txt` via `target_sources(unittest PRIVATE ...)`.
2. If creating a new module, add `add_subdirectory(<module>)` in `test/unittest/src/CMakeLists.txt`.

## Main Entry Pattern

`test/unittest/src/main.cpp` uses:

```cpp
#include <erbsland/unittest/UnitTest.hpp>
ERBSLAND_UNITTEST_MAIN();
```

Keep this unchanged unless explicitly requested.

## Repository Run Commands

Use these exact commands for fast feedback:

```bash
cmake --build cmake-build-debug --target unittest && cmake-build-debug/test/unittest/unittest
cmake-build-debug/test/unittest/unittest --list
cmake-build-debug/test/unittest/unittest name:<ClassWithoutTestSuffix>
cmake-build-debug/test/unittest/unittest name:<MethodWithoutTestPrefix>
cmake-build-debug/test/unittest/unittest target:<TargetFromTESTED_TARGETS>
cmake-build-debug/test/unittest/unittest --verbose --no-color
```

Examples:

- `name:ErrorCategory` runs tests from `ErrorCategoryTest`.
- `name:ToCode` matches method `testToCode`.
- `target:Parser` runs suites tagged with `TESTED_TARGETS(Parser)`.
