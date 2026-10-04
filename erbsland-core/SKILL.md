---
name: erbsland-core
description: "Use only when developing, reviewing, or refactoring applications that consume Erbsland Core as a dependency. Discover existing Core APIs, choose efficient string operations, and use public includes and consumer namespace spelling. Do not use when working on Erbsland Core itself, including its implementation, tests, demos, documentation, or build system. Standalone Erbsland libraries are outside this skill's scope."
---

# Erbsland Core

## Scope

Use this skill only for applications consuming Core, including their own tests. Do not apply it to work on Core
itself: implementation, internal APIs, tests, demos, documentation, generators, or build configuration.
Reading those files to understand an API is supported; determine scope from the code being developed.

## Consumer defaults

Use the highest-level Core functionality that fits the task. Use the standard library only where Core provides no
suitable functionality or when an external interface requires standard-library objects.

- Build new applications on `el::Application`.
- Use the short `el` alias and flattened names: `el::String`, not `erbsland::text::String`.
- Use the shortest public per-type include: `<erbsland/String.hpp>`, not `<erbsland/text/String.hpp>`.
- Use `String` for ordinary text values and parameters; pass `"text"_el` literals directly when accepted.
- Keep required standard-library and native conversions at boundaries, outside processing loops where possible.

## Detect Core: take the usual path first

Check `<project root>/erbsland/core` first. When present, use that dependency and the consumer defaults above.
Read the consuming project's applicable instructions, then continue with the task. Reuse this location and established
conventions throughout the task.

Only when Core is unavailable at that path, read [locating Core](references/locating-core.md) for fallback detection.
Follow explicit project configuration when it differs from the consumer defaults; consult the conventions reference
for a relevant namespace/include mismatch.

The dependency's actual headers are authoritative for unfamiliar signatures and availability. Use its topics and
demos for consumer patterns, and tests for behavioral evidence. Shared names do not make Core's embedded `cterm`,
configuration, or regular-expression APIs interchangeable with the standalone libraries.

## Discover before implementing

Use already-understood APIs directly. Before adding a helper, manually processing text, or building a `std::`
workaround, consult the relevant part of [the capability map](references/capability-map.md). Search for the task when
the type is unknown, inspect the candidate's contract, and add custom logic only for behavior Core does not cover.
Keep this lookup focused on the operation needed, rather than surveying the library.

With the `erbsland_knowledge` MCP server:

- Use `search_functionality` for tasks and `search_symbols` for names. Follow returned `symbol_id`/`path` values with
  `get_symbol`, `get_context`, or `get_document`; prefer `structuredContent`.
- Check result library, version, and source category. Narrow searches to Core once its library id is known; do not
  repeatedly list libraries. Consumer docs and demos are better usage patterns than internal test helpers.
- Compare related symbols only when their trade-offs matter. Check freshness or versions when a result conflicts
  with the dependency, and confirm against its headers; a missing search result does not prove an API is absent.

Without usable indexed evidence, search inside the dependency's relevant domain with `rg`, for example:

```sh
rg -n 'split|separator|shared slice' doc/topics/text_strings doc/reference/text
rg -n 'StringSplitter|fromJoined|replacedAll' src/erbsland/text
```

Follow aliases, base classes, and included template implementations. `String` aliases `U8String`; the short alias
header alone does not expose its methods. Verify Core naming and units instead of assuming standard-library interfaces:
`List::count()` returns `ItemCount`, rather than providing `size()`.

## Read the reference for the current decision

- Starting a new application or choosing its control flow: [application designs](references/applications.md).
- Text processing, parsing, construction, or string costs: [strings](references/strings.md), before choosing an algorithm.
- Public-header selection, nested namespace exceptions, or adapting test code: [consumer conventions](references/consumer-conventions.md).

Load only references relevant to the task. Preserve an established application's design unless the requested change
calls for another one.

## Review the result

- Does Core already provide the behavior of a newly introduced helper or loop?
- Are units, encoding policy, ownership, allocations, and throwing versus fallback behavior appropriate?
- Do includes and type names follow the consuming application's conventions?
