---
name: erbsland-elcl
description: Learn the Erbsland Configuration Language (ELCL) and its usage. Use when writing/editing configuration files in the ELCL format.
license: Apache-2.0
metadata:
  author: erbsland-dev
  version: "1.0"
---
# Erbsland Configuration Language (ELCL) Compact Agent Reference (v1.0)
Scope: Erbsland Configuration Language (ELCL) syntax and semantics, excluding validation-rules DSL.  
Full documentation online: https://config-lang.erbsland.dev/

## General Document Format/Encoding/Structure
Details: [Document structure](references/document-structure.md)

- Encoding: UTF-8, optional BOM.
- File is line-based. Empty/comment-only lines are ignored.
- Reject invalid UTF-8, surrogates, and code points > `U+10FFFF`.
- Also reject control chars (0x00..0x1F, 0x7F..0x9F), except TAB (`0x09`), LF (`0x0A`), CR (`0x0D`).
- Line breaks: `LF` or `CRLF`; lone `CR` invalid.
- Max line length: 4000 bytes (including line break if present).
- Comments: `#` to end of line, only outside values.
- Spacing is spaces/tabs. For multi-line constructs, the indentation pattern must match exactly (empty lines exempt).
- Everything is case-insensitive, except text enclosed in quotes.
- Usual file extension: `.elcl`

Compact example:
```elcl
@version: "1.0"     # Optional ELCL version declaration.
[app.server]        # A section name, implicit 'app' namespace.
name: "My Server"   # A value assignment.
*[.interface]       # A section list, with relative name path resulting in -> 'app.server.interface'.
ip: "127.0.0.1"     
port: 8080
*[.interface]       # Second entry in 'app.server.interface'
ip: "0.0.0.0"
port: 80
```

## Names
Details: [Names and conflicts](references/names-and-conflicts.md)

- Names are case-insensitive.
- Names must start with a letter.
- Names can contain letters, digits, space, and underscore.
- Space and underscore are equivalent.
- No consecutive separators (`__`, double spaces, mixed runs).
- No trailing underscore.
- Max length: 100 chars.
- Normalization: spaces -> `_`, then lowercase.
- `this_name` == `This Name` == `THIS_NAME`

## Sections
Details: [Sections and name paths](references/sections-and-name-paths.md)

- A section is enclosed in square brackets (`[...]`).
- It contains a *name path*, names separated by dots.
- Spacing around names and dots is allowed.
- Must start in column 1.
- Absolute section starts with a name, e.g. `[main.server]`.
- Relative section starts with a dot, e.g. `[.connection]`, and resolves from the last absolute section.
- A document must not start with a relative section.
- After `@include`, relative section context is cleared; define an absolute section before using relative sections again.
- Any number of `-` chars as prefix/suffix `---[ section.name ]---` allowed.
- Asterisk (`*`) in front of `[` indicates a section list `*[name]`, or fancy `--*[ section.name ]*--`.

```elcl
[main]   # defines section 'main'
value: 123
------[ main . sub ]-------   # creates section 'main.sub'
value: 345
[   Main   .   Another   ]
value: 789
*[list]   # creates section list 'list'
value: 1
---*[ List ]*--- # adds a new entry to 'list'
value: 2
*[   list   ]*  # adds another entry to 'list'
value: 3
```

## Name Paths
Details: [Sections and name paths](references/sections-and-name-paths.md)

- A name path is a sequence of names separated by dots.
- It is used in sections to create the context for the following value assignments.
- No consecutive separators (`..`) and no trailing separator (`.`).
- Max path length: 10 elements.
- A name path must be unique and cannot be repeated in the same document, 
  therefore, all values for a given path must be defined in one location.
- Section paths and value paths share one namespace, so section/value collisions are invalid.
- Parent sections are implicitly created if they do not exist.
- Implicitly created sections can be defined later in the document. 
- Implicitly created sections must not be redefined as section list or scalar value.

## Value Assignments
Details: [Value assignments and lists](references/value-assignments-and-lists.md)

- A value assignment consists of a name and a value.
- Assignment starts with a name at column 1, followed by a colon `:` or `=` (equivalent), called value-separator.
- Spacing around value-separator is allowed.
- Value follows value-separator, in the same line, or with indentation on the next line.
- If the value starts on the next line, indentation is required.
- No empty/comment-only line is allowed between separator and first value line.
- Lists of values are allowed, on the same line, separated by commas (more formats in multi-line values below).
- Single-line lists must not have leading/trailing/double commas.
- Single-line lists must not contain multi-line value forms.

```elcl
[section]
value1: "text"   # same line using :
value2 = "text"  # same line using =
value3   :    # next line using :
    "text"
value4=       # next line using =
    "text"
```

## Value Types
Details: [Value types and parsing rules](references/value-types.md)

- Core types: Integer, Boolean, Text
- Standard types: Code, Float, Byte Counts, Multi-line Text/Code, Date/Time/DateTime, Byte Data
- Advanced types: Regular Expression, Time Delta
- Text escapes: `\\`, `\"`, `\$`, `\n`, `\r`, `\t`, `\uXXXX`, `\u{1..8 hex}`.
- Unknown escape sequences are invalid. Null (`\u0000`) is invalid.

```elcl
[core types]
integers1: 123, 0x12ab, 0b10101010  # integer in decimal, hex, binary
integers2: 123'456, 0x12ab'cdef, 0b1010'0000  # integer with separators
integers3: 0X12AB, 0B1010 # case-insensitive
booleans: Yes, no, on, OFF, Enabled, disabled, true, False # boolean literals (case-insensitive)
text1: "Any kind of text" # no unescaped control chars are allowed
text2: "Escapes: \\ \" \$ \n \r \t \u12af \u{20}" # *all* valid escape sequences (case-insensitive)
```

```elcl
[standard types]
code_text: `code \\\ no escapes` # no escapes are allowed in code text
float: 1.23e45, -inf, NaN  # floating point literals (case-insensitive)
byte_count: 123kb, 123 MiB  # byte count literals (case-insensitive, one space between digits and suffix is allowed)
date: 2021-01-01  # date in YYYY-MM-DD format
time: 12:34:56.123  # time in [T]?HH:MM[:SS[.fraction]] format (case-insensitive)
datetime: 2021-01-01t12:34:56.123, 2026-08-30 10:00  # date and time in YYYY-MM-DD[ T]HH:MM[:SS[.fraction]] format
time2: 12:34:56Z, 2021-01-01T12:34:56+01:00  # time/date-time with offsets,
byte_data: <01ab cdef 00 01   02>  # hex bytes, separated by spacing (case-insensitive)
```

```elcl
regex: /[a-z]+/  # regular expression.
time_delta: 123s, 123ms  # time-delta.
```

## Multi-line Values
Details: [Multi-line values](references/multiline-values.md)

- Multi-line values are allowed for text, code, byte-data, and regular expressions.
- Can start on the same line, or on the next line after value-separator.
- Open with matching delimiter: `"""`, triple backticks, `<<<`, or `///`.
- Indentation pattern must match exactly (empty lines exempt) and is removed from the content.
- Closing delimiter must use the same indentation pattern.
- Trailing spacing is removed from the content.
- For text/code/regex, line breaks normalize to `\n`. For byte-data, line breaks are layout-only.

```elcl
[multiline values]
text_multiline1: """
    the first line
    second line
    """
text_multiline2:
    """
    the first line
    second line
    """
code_multiline: ```
    code line 1
    code line 2
    ```
regex_multiline: ///
    a+
    ///
byte_data: <<<
    01 02 03
    04 05 06
    >>>
```

## Multi-line Lists
Details: [Value assignments and lists](references/value-assignments-and-lists.md)

- Lists of values can be multi-line.
- Multi-line lists must start on the line after the value-separator.
- Each entry must start on a new line, with indent, prefixed by an Asterisk (`*`).
- Entry indentation pattern must match exactly.
- Empty/comment-only lines between entries are invalid.
- Multi-line values are not valid as list entries.
- Multi-line lists allow value lists to form a matrix.

```elcl
[multiline lists]
integers:
    * 123
    * 456
    * 789
matrix:
    * 1, 2, 3
    * 4, 5, 6
    * 7, 8, 9
```

## Text Names
Details: [Text names](references/text-names.md)

A special name syntax, to allow using text as key.
- Syntax: same as single-line text (`"..."` with text escape rules).
- Compared by Unicode code points after escape resolution (no Unicode normalization required).
- Regular name and text name are never equal (`text` != `"text"`).
- In one section, do not mix regular and text names.
- Section lists cannot use text names.
- Sections with text names cannot have subsections.

```elcl
[text names]
"hello": 123
"world": 456
[section."hello"]
value: 123
[section."world"]
value: 456
```

## Meta Values and Commands
Details: [Meta values and commands](references/meta-values-and-commands.md)

- Special values that are not part of the value tree.
- Core meta values use single-line text values; parser-specific meta values may also use integer/boolean values.
- Prefixed with `@`.
- `@version` (core): defines ELCL language version.
- `@features` (core): text with space-separated feature identifiers. Known groups: `core`, `minimum`, `standard`, `advanced`, `all`.
- `@features` (core): known features: `float`, `byte-count`, `multi-line`, `section-list`, `value-list`, `text-names`, `date-time`, `code`, `byte-data`, `include`, `regex`, `time-delta`.
- Unknown/unsupported feature identifiers must fail.
- `@include` (standard): includes another document.
- `@signature` (advanced): used for document signature
  - always in the first line
  - programmatically created
  - must be removed when a configuration is edited

## Notes
### `@version` (core)
- Optional.
- Must appear before first section.
- At most once per document.
- Text format `<digit>.<digit>`, currently only `"1.0"` valid.

### `@include` (feature `include`)
- May appear multiple times before/between/after sections.
- Value must be text source descriptor.
- Include command closes any open section and clears last absolute section context.
- Included documents are parsed as standalone documents; they do not inherit parser state context except shared value tree.
- Name conflicts are checked across the merged result as if one document.
- Usual source format: `@include: "[file:]<path>"`.
- Usual path behavior:
  - Relative paths resolve from including file.
  - `*` wildcard only in filename part.
  - `**` standalone path element for recursive matching.
  - Process matched files in deterministic alphabetical order.

## Limits
Details: [Limits and authoring](references/limits-and-authoring.md)

- Max line length: 4000 bytes.
- Max name length: 100 chars.
- Max name-path length: 10 elements.
- Max include nesting: 5 documents.
- Integer max digits: dec 19, hex 16, bin 64 (64-bit baseline).

## ELCL is a Human Centric Format
Details: [Limits and authoring](references/limits-and-authoring.md)

- ELCL is designed to be human-readable and editable.
- For this reason, it gives freedom with spacing and comments and is mostly case-insensitive.
- When not instructed otherwise, make sure your configuration is designed for humans.
  - Group sections logically
  - Use empty lines to separate sections.
  - Use comments to explain complex values.
  - Names like `default_port` are easier to read as `Default Port`.
  - Separating elements in a name path with spacing is easier to read.

Example for a human-readable configuration:
```elcl
--------------[ App . Server ]-----------------------------------------------
Name          : "MyServer"        # The public name of the server.
Default Port  : 8080              # The port to listen on, >1024 recommended.

*[ . Service ]*
Name          : "web"
Welcome       :
    """
    Welcome to the Server!
    Please read the documentation.
    """

*[ . Service ]*
Name          : "serial"
Enabled       : No                # Disable serial service by default.

--------------[ Core ]-------------------------------------------------------
Threads       : 10                # The number of worker threads to use.
Timeout       : 30 seconds        # The timeout for requests.
```
