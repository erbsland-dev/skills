# Erbsland Unit Test API and Macros

## Include and Entry Point

- Include framework headers with either:
  - `#include <erbsland/unittest/UnitTest.hpp>`
  - `#include <erbsland/unittest/all.hpp>`
- Define test executable entry point in `main.cpp`:
  - `ERBSLAND_UNITTEST_MAIN();`

## Base Class API (`UnitTest`)

Main overridable/public members:

- `virtual auto additionalErrorMessages() -> std::string;`
- `virtual void setUp();`
- `virtual void tearDown();`
- `void runWithContext(const SourceLocation&, const std::function<void()>&, const std::function<std::string()>& = nullptr);`
- `void consoleWriteLine(const std::string&);`
- `auto unitTestExecutablePath() -> std::filesystem::path;`

Guidance:

- Prefer `additionalErrorMessages()` for suite-state diagnostics.
- Keep `setUp()/tearDown()` minimal; prefer explicit setup helpers in each test.
- Use `runWithContext(...)` when local variable capture improves failure output.

## Assertion Macros

Hard-fail macros:

- `REQUIRE(expr)`
- `REQUIRE_FALSE(expr)`
- `REQUIRE_THROWS(expr)`
- `REQUIRE_THROWS_AS(ExceptionType, expr)`
- `REQUIRE_NOTHROW(expr)`
- `REQUIRE_EQUAL(a, b)`
- `REQUIRE_NOT_EQUAL(a, b)`
- `REQUIRE_LESS(a, b)`
- `REQUIRE_LESS_EQUAL(a, b)`
- `REQUIRE_GREATER(a, b)`
- `REQUIRE_GREATER_EQUAL(a, b)`

Non-fatal check macros:

- `CHECK(expr)`
- `CHECK_FALSE(expr)`
- `CHECK_THROWS(expr)`
- `CHECK_THROWS_AS(ExceptionType, expr)`
- `CHECK_NOTHROW(expr)`
- `CHECK_EQUAL(a, b)`
- `CHECK_NOT_EQUAL(a, b)`
- `CHECK_LESS(a, b)`
- `CHECK_LESS_EQUAL(a, b)`
- `CHECK_GREATER(a, b)`
- `CHECK_GREATER_EQUAL(a, b)`

Context helpers:

- `WITH_CONTEXT(functionCall(x, y))`
- `SOURCE_LOCATION()`

## Metadata Macros

- `TESTED_TARGETS(name_list)`
- `TAGS(tags)`
- `SKIP_BY_DEFAULT()`
- `UNITTEST_SUBCLASS(BaseClass)`

Use `TESTED_TARGETS(...)` consistently; it drives `target:<target>` filters in CLI runs.

## File Helpers (`erbsland::unittest::fh`)

- `unitTestExecutablePath()`
- `resolveDataPath(relativePath)`
- `readDataText(relativePath, maximumSize=10000000)`
- `readDataLines(relativePath, maximumSize=10000000)`

Use these with CMake `erbsland_unittest(... COPY_TEST_DATA ... ENABLE_DATA_DEPS)`.

## CLI Filters and Options

- `--help`
- `--verbose`
- `-e`
- `--list`
- `--no-color`
- `--no-summary`
- `name:<name>`, `+name:<name>`, `-name:<name>`
- `target:<target>`, `+target:<target>`, `-target:<target>`
- `tag:<tag>`, `+tag:<tag>`, `-tag:<tag>`

Processing precedence is `<opt>`, then `+<opt>`, then `-<opt>`.
