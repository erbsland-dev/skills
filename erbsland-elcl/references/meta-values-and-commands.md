# Meta Values and Commands

Use this file for `@version`, `@features`, `@include`, and `@signature`.

## Common Meta Rules

- Meta names start with `@` and use regular-name syntax.
- Meta values must be defined before first section, except `@include` which can appear between sections.
- Known core names: `@version`, `@features`, `@include`, `@signature`.
- Unknown meta names should be treated as errors unless parser-specific (`@parser_*`).

## Value Type Rules

- Core meta values use text values:
  - `@version` requires text like `"1.0"` (currently only valid version).
  - `@features` requires text with space-separated feature identifiers.
- Parser-specific meta values may use text, integer, or boolean.

## `@features` Identifiers

Feature groups:

- `core`, `minimum`, `standard`, `advanced`, `all`

Language features:

- `float`, `byte-count`, `multi-line`, `section-list`, `value-list`, `text-names`
- `date-time`, `code`, `byte-data`, `include`, `regex`, `time-delta`

Parser capabilities (not for `@features` language requirements):

- `validation`, `signature`

## `@include` Essentials

- Value is text, usually `"[file:]<path>"`.
- Can be used multiple times.
- Clears open section and last absolute section context in the including file.
- Included file is parsed as standalone syntax context.
- Included files share only the resulting value tree.
- Name conflicts are checked across the merged configuration.
- Path behavior:
  - relative path resolves from including file
  - `*` wildcard only in filename part
  - `**` must be a standalone path element for recursive matching
  - deterministic alphabetical processing of matches
- Include nesting limit: 5.

## `@signature` Essentials

- Must be on first line.
- Value is text signature payload.
- Signed content starts from line 2.
- Editing the configuration invalidates signature; remove or regenerate it.

