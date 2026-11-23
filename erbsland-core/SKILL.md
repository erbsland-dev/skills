---
name: erbsland-core
description: "Use when developing, reviewing, or refactoring applications consuming Erbsland Core. Discover Core APIs, choose string operations, time types, and typed sequence coordinates, and use public includes and consumer namespaces. Excludes work on Core itself (implementation, tests, demos, documentation, build system) and standalone Erbsland libraries."
---

# Erbsland Core

## Scope

Use this skill for applications consuming Core, including their tests. Reading Core internals to understand an API
is supported; developing Core itself is outside scope.

## Consumer defaults

Use the highest-level Core functionality that fits the task. Use the standard library only where Core provides no
suitable functionality or when an external interface requires standard-library objects.

- Build new applications on `el::Application`.
- Use the short `el` alias and flattened names: `el::String`, not `erbsland::text::String`.
- Use the shortest public per-type include: `<erbsland/String.hpp>`, not `<erbsland/text/String.hpp>`.
- Use `String` for ordinary text values and parameters; pass `"text"_el` literals directly when accepted.
- Prefer `el::join` for fixed fragments, free string transformations for common mappings, and retained regexes for
  repeated matching; see the text references below.
- Use Core time amounts and `el::TimePoint` instead of `std::chrono` types. Use `el::Timestamp` for compact instant
  storage and `el::DateTime` for calendar calculations and time zones; read [time types](references/time.md).
- Use `el::FifoCache` or `el::LruCache` for bounded caching before implementing a custom eviction container.
- Keep required standard-library and native conversions at boundaries, outside processing loops where possible.
- Keep sequence coordinates in matching index/count/offset/range types, such as `el::ByteIndex` / `el::ByteCount`.
  Use typed arithmetic and named intent conversions. Construct constants directly; use `fromInteger()` /
  `fromIntegerOrThrow()` or size factories for runtime integers. See [unit types](references/unit-types.md).

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

Without usable indexed evidence, use `rg` in the dependency's relevant `src/erbsland/<domain>`, `doc/topics`, and
`doc/reference` directories.

Follow aliases, base classes, and included template implementations. `String` aliases `U8String`; the short alias
header alone does not expose its methods. Verify Core naming and units instead of assuming standard-library interfaces:
`List::count()` returns `ItemCount`, rather than providing `size()`.

## Read the reference for the current decision

- Time amounts, elapsed measurements, deadlines, stored instants, calendar arithmetic, time zones, or epoch
  conversions: [time types](references/time.md), before choosing types or implementing time calculations.
- Choosing sequence units, converting indexes/counts/offsets, handling ranges, writing indexed loops, or
  constructing versions: [unit types](references/unit-types.md), including constant versus runtime construction,
  saturation, special states, and native API boundaries.
- Starting a new application or choosing its control flow: [application designs](references/applications.md).
- Declaring, parsing, or reviewing command-line options, choices, or help:
  [command-line options](references/command-line-options.md), before implementing option definitions or validation.
- Error contracts, input validation, exception handling, or diagnostics: [error handling](references/errors.md),
  including `Result` and `ResultWithData` for expected outcomes.
- Text processing, parsing, construction, or string costs: [strings](references/strings.md), before choosing an algorithm.
  For literal searches, character validation, glob/divider patterns, regex, or fuzzy suggestions, read
  [string matching](references/string-matching.md), including retained `el::RegEx` expressions, cached literal helpers,
  and comparison callbacks.
  For casing, capitalization, camel-case word conversion, whitespace simplification, ELCL identifier normalization,
  or custom character mapping, removal, and truncation, read [string transformations](references/string-transformation.md).
  For scalar conversion, character scans, or reader selection, read [string parsing](references/string-parsing.md).
  For output patterns or field collection, read [formatting](references/string-formatting.md) or
  [splitting](references/string-splitting.md), respectively.
- ELCL configuration loading, validation rules, defaults, placeholders, or value lookup:
  [configuration](references/configuration.md), before implementing manual validation, substitution, or tree traversal.
- Bounded caching, eviction policy, or lookup ownership: [caches](references/caches.md), including `FifoCache` versus
  `LruCache` and when reads refresh recency.
- Public-header selection, nested namespace exceptions, or adapting test code: [consumer conventions](references/consumer-conventions.md).

Load only references relevant to the task. Preserve an established application's design unless the requested change
calls for another one.

## Review the result

- Does Core already provide the behavior of a newly introduced helper or loop?
- Are units, encoding policy, ownership, allocations, and throwing versus fallback behavior appropriate?
- Do includes and type names follow the consuming application's conventions?
