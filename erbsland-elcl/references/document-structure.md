# ELCL Document Structure

Use this file when reading/writing whole documents and when fixing parser-facing syntax issues.

## Encoding and Line Rules

- Document encoding is UTF-8, optional UTF-8 BOM.
- Reject invalid UTF-8 sequences, UTF-16 surrogate code points, and code points above `U+10FFFF`.
- Allowed line breaks: `LF` or `CRLF`. A lone `CR` is invalid.
- Max line length is 4000 bytes (including line break when present).
- Newline at end of file is optional.

## Character and Comment Rules

- Only `TAB` (`U+0009`), `LF` (`U+000A`), and `CR` (`U+000D`) are allowed control characters in the document stream.
- Comments start with `#` and continue to end of line.
- Comments are ignored for parsing structure.
- `#` inside quoted/text-like payloads is data, not a comment.

## Spacing and Case

- Spacing is spaces and tabs.
- Section lines and value-name lines must start in column 1 (no leading indentation).
- Most keywords/literals are case-insensitive.
- Text content and text-name content are case-sensitive after escape resolution.

## Indentation Pattern Concept

- Multi-line constructs define an indentation pattern on the first continued line.
- Every continued non-empty line must match that exact spaces/tabs prefix.
- Empty lines are exempt from pattern matching.

