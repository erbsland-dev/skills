# Text Names

Use this file when keys/section elements are quoted text (`"..."`) instead of regular names.

## Definition

- Text name syntax is single-line text syntax.
- Escape sequences are resolved before comparison.
- No Unicode normalization is required by parser.

## Comparison Rules

- Text names are compared by Unicode code points after escape resolution.
- Regular names and text names are always different types:
  - `text` is never equal to `"text"`.

## Structural Restrictions

- Do not mix regular names and text names in the same section.
- Text names must be unique in a section.
- Section lists cannot use text names.
- Sections with text names cannot have subsections.

## Practical Guidance

- Use text names only when exact human text must be the key.
- Prefer regular names for normal config keys to keep paths and tooling simple.

