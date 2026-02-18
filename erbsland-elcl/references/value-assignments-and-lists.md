# Value Assignments and Value Lists

Use this file for assignment syntax and list formatting.

## Assignment Syntax

- Value line starts with a regular name, text name, or meta name at column 1.
- Separator is `:` or `=` (equivalent), with optional spacing around it.
- Value may be:
  - on the same line
  - on the next line, but then indentation is required

## Next-Line Rules

- If the value starts on the next line, it must be indented with at least one space/tab.
- No empty/comment-only line is allowed between separator and first value line.

## Single-Line Lists

- Comma creates a list: `value: 1, 2, 3`
- Optional spacing around commas.
- No leading comma, trailing comma, or double commas.
- Multi-line value forms (`"""`, triple-backtick code blocks, `<<< >>>`, `///`) are not allowed inside single-line lists.

## Multi-Line Lists

- Must start on the line after the separator.
- Each entry is an indented line starting with `*`.
- Indentation pattern must be consistent for list entries.
- No empty/comment-only lines between entries.
- Each entry can hold a single-line value or a single-line comma list (matrix style).
- Multi-line value forms are not allowed as list items.
