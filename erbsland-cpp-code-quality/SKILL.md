---
name: erbsland-cpp-code-quality
description: Erbsland C++20 quality rules for writing, reviewing, refactoring, and extending code.
---
# Erbsland C++ Code Quality

Read the project instructions, Code Style, API guidelines, and tests before reviewing a design.
Apply normal C++ engineering best practices without restating them.

## Erbsland priorities

Apply these priorities in order:

1. Safety, correctness, and reliability.
2. Defined and diagnosable failure.
3. Portability across all supported platforms.
4. Readability, smallness, and testability.
5. Adequate performance.

Do not trade a higher priority for a lower one without an explicit project requirement.

## Additional notes on the priorities
- **Safety:**
  - Write interfaces in a way *to guide* users toward safe usage but do not babysit them.
  - Do not add defensive machinery without identifying the concrete misuse or failure it prevents.
  - Make ownership, preconditions, and safe usage explicit.
  - Prefer safe defaults and narrow unsafe boundaries.
- **Reliability:**
  - Code should be reliable *by design*.
  - Exception handling, retries, and fallback behavior do not repair broken code.
  - We expose logic errors and test our code until we are confident that our code is correct.
  - Hiding logic errors to make an application look reliable is a serious design issue.
- **Defined failures:**
  - All failures need defined behavior. The following fail-fast guidance concerns programming errors and invalid
    internal states; expected operational failures belong to normal runtime error handling.
  - Classify failures according to the interface contract. Invalid external input is an expected failure;
    violating a documented caller precondition is a programming error.
  - An exception escaping a boundary whose contract forbids propagation may trigger a deliberate fail-fast policy.
  - Treat disguising programming errors as recoverable operational failures as a design issue.
  - Exhausted system resources are fatal failures, even when some of them are technically recoverable.
    If a developer expects such extreme cases, they will implement their own memory/thread management.
    Inside libraries and regular application code, we allow a process to terminate.
- **Diagnosable failures:**
  - Deliberate fail-fast termination is valid for programming errors.
    It should preserve useful diagnostic context through the termination site, a debugger, a crash report,
    or an existing diagnostic mechanism.
  - Do not suppress or obscure the original failure.
- **Readability, smallness, and testability:**
  - Prefer explicit, conventional designs over clever or speculative abstractions.
  - Refactor weak or mixed responsibilities before extending them.
  - Keep behavior with the data whose validity rules it enforces.
  - Replace enum-like values with proper types when they accumulate behavior.
  - Expose reusable, independently meaningful, or significant internal logic through an implementation interface to
    allow proper unittest coverage.
  - Never hide types and logic in namespaces or units that make it difficult to test.
  - Keep unsafe and native boundaries narrow, explicit, named, documented, and easy to audit.
- **Adequate performance:**
  - Require evidence before adding complexity for performance.

## Reviews

Report only material issues that conflict with these priorities.
Support each finding with a concrete consequence and recommend the smallest proportionate remedy.
Say clearly when no material quality issue is present.
