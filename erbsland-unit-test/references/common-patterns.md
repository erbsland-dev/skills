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

Example:
```cpp
class ExampleTest final : public el::UnitTest {
public:
    MyTestedClass myTestedClass;

    auto additionalErrorMessages() -> std::string override {
        try {
            std::string result;
            result += std::format("myTestedClass.x(): {}\n", myTestedClass.x());
            result += std::format("myTestedClass.y(): {}\n", myTestedClass.y());
            return result;
        } catch(...) {
            return "Unexpected exception";
        }
    }
    
    void testFirst() {
        myTestedClass = MyTestedClass{1, 3};
        // ...
    }
};
```

## Context Pattern for Iteration

Two common approaches are:

1. Wrap helper assertions methods with `WITH_CONTEXT(...)`, this adds the file and line in diagnostic output on failure.
2. Use `runWithContext(SOURCE_LOCATION(), testLambda, diagnoseLambda)` in table-driven tests for diagnostics about the failed test case.

### Test Case Driven Example
When calling the same test helper method from multiple locations `WITH_CONTEXT` add the exact line to the test output.
When an assert inside the test helper is failing, the full stack of nested `WITH_CONTEXT` gets visible and allows
to locate the source and therefore the failing test values.
```cpp
class ExampleTest final : public el::UnitTest {
    void requireAdd(int a, int b, int expected) {
        REQUIRE_EQUAL(a + b, expected); 
    }
    void testExample() {
        WITH_CONTEXT(requireAdd(0, 0, 0));
        WITH_CONTEXT(requireAdd(2, 2, 4));
        WITH_CONTEXT(requireAdd(100, -50, 50));
        WITH_CONTEXT(requireAdd(-10, 10, 0));
        // ...
    }
};
```

### Table Driven Example
With six or more test cases, a vector with test values keeps the test extensible, readable and maintainable.
Using `runWithContext` to get failing values in the test output.
```cpp
class ExampleTest final : public el::UnitTest {
    void testExample() {
        struct TestCase {
            int a;
            int b;
            int expected;
        };
        const auto testCases = std::vector<TestCase>{{
            {0, 0, 0},
            {2, 2, 4},
            {100, -50, 50},
            // ...
        }};
        for (const auto &[a, b, expected] : testCases) {
            runWithContext(SOURCE_LOCATION(), [&]() -> void {
                REQUIRE_EQUAL(a + b, expected);
            }, [&]() -> std::string {
                return std::format("failed for a:{} b:{}", a, b);
            });
        }
    }
};
```

### Combined Test Case with Table
Multiple test cases inside a loop require `WITH_CONTEXT` to pin-down the failed test in the loop, and `runWithContext`
to get the failed test values.
```cpp
// ...
for (const auto &[a, b, expected] : testCases) {
    runWithContext(SOURCE_LOCATION(), [&]() -> void {
        WITH_CONTEXT(requireFirst(a, b, expected));
        WITH_CONTEXT(requireFirst(a, -b, -expected));
        WITH_CONTEXT(requireFirst(-a, -b, -expected));        
    }, [&]() -> std::string {
        return std::format("failed for a:{} b:{}", a, b);
    });
}
```

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
