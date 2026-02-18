# Multi-Line Values

Use this file when writing/editing `"""`, triple-backtick code, `<<< >>>`, or `///`.

## General Rules

- Multi-line values can start on the same line as the separator or on the next line.
- Opening delimiter line may include trailing spacing/comments, then must end with line break.
- The first continued line defines the indentation pattern.
- All non-empty continued lines must match that exact spaces/tabs prefix.
- Closing delimiter must appear at the same indentation pattern.
- Line endings normalize to `\n` in resulting text-like values.

## Type-Specific Delimiters

- Multi-line text: `""" ... """`
- Multi-line code text: triple-backtick block (optional language id after opening delimiter)
- Multi-line byte-data: `<<< ... >>>` (optional `hex` format id after opening delimiter)
- Multi-line regex: `/// ... ///`

## Whitespace Behavior

- Indentation pattern is removed from each continued line.
- Trailing spacing at line ends is removed for multi-line text, byte-data, and regex forms.
- Leading/trailing wrapper spacing around content is removed by multiline spacing rules.
- Empty lines are preserved as empty lines in resulting text-like values.

## Escape/Content Notes

- Multi-line text: same escape rules as single-line text.
- Multi-line code: no escape processing; backticks are allowed in content.
- Multi-line byte-data: hex bytes only; comments are allowed in content lines.
- Multi-line regex: escape sequences are mostly pass-through to regex payload.
