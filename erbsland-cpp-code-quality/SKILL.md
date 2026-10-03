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

- Prefer explicit, conventional designs over clever or speculative abstractions.
- Refactor weak or mixed responsibilities before extending them.
- Keep behavior with the data whose invariants it governs.
  Replace enum-like values with proper types when they accumulate behavior.
- Expose reusable or independently meaningful internal logic through an `impl` interface when this improves testing.
  Keep genuinely local helpers private.
- Keep unsafe and native boundaries narrow, explicit, named, documented, and easy to audit.
- Require evidence before adding complexity for performance.

## Reviews

Report only material issues that conflict with these priorities.
Support each finding with a concrete consequence and recommend the smallest proportionate remedy.
Say clearly when no material quality issue is present.
