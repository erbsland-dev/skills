# Common and custom string transformations

Read this for case conversion, capitalization, camel-case word conversion, whitespace simplification, ELCL-style
identifier normalization, or custom mapping, removing, and stopping at decoded characters.
Paths below are relative to the Core dependency.

## Prefer the common transformation free functions

Include `<erbsland/StringTransformations.hpp>` and call `el::toAsciiLowercase(text)` or the appropriate helper below.
These are the shortest public operations for common transformations: prefer them over `transformed()` with predefined
`Char` callbacks, wrapper helpers, manual loops, or standard-string conversions. Keep `text.transformed(...)` for
custom character rules that the common functions do not cover.

The free functions accept concrete Core UTF-8, UTF-16, and UTF-32 strings and editors. They leave the input unchanged
and return a read-only string of the same encoding, including when the input is an editor. They do not accept literals,
standard strings, or type-erased `AnyString` / `AnyStringEditor` directly; use `el::String{"text"_el}` for a UTF-8 literal.
Unchanged results share source storage, including slices. Convert the result to an editor only when further editing
is needed.

## Casing and capitalization

| Intent | Expression |
| --- | --- |
| ASCII lowercase / uppercase | `el::toAsciiLowercase(text)` / `el::toAsciiUppercase(text)` |
| Unicode simple lowercase / uppercase | `el::toLowercase(text)` / `el::toUppercase(text)` |
| Unicode simple case folding | `el::toCaseFolded(text)` |
| Capitalize the complete input | `el::toCapitalized(text)` / `el::toAsciiCapitalized(text)` |
| Select capitalization at runtime | `el::toCapitalization(text, capitalization)` / `el::toAsciiCapitalization(text, capitalization)` |
| ELCL-style identifier normalization | `el::toIdentifierNormalized(text)` |

Choose ASCII mapping when the contract calls for ASCII casing: it leaves non-ASCII characters unchanged and avoids
Unicode database lookups. Choose Unicode mapping when non-ASCII letters must also change case. Unicode mappings here
are simple one-code-point mappings; they do not expand a character into several characters or apply language-specific
casing rules. Encoded byte length can still change. Case folding serves caseless matching and is distinct from
lowercasing. Unicode normalization forms are a separate operation, `normalized(NormalizationForm)`.

`toCapitalized()` uppercases the first code point and lowercases the remainder of the complete input:
`hELLO wORLD` becomes `Hello world`, and `!HELLO` becomes `!hello`. It does not capitalize each word separately.
The runtime selector accepts `el::Capitalization::Lowercase`, `Uppercase`, or `Capitalized`; include
`<erbsland/Capitalization.hpp>` when naming this enum.

`toIdentifierNormalized()` applies ASCII lowercase and changes each ordinary space (`U+0020`) to an underscore
(`U+005F`). It leaves tabs, other whitespace, punctuation, and non-ASCII characters unchanged. It neither trims nor
collapses spaces and does not validate an identifier.

For the common-function examples, include `<erbsland/String.hpp>` and `<erbsland/StringTransformations.hpp>`. Put
`using namespace el::text::literals;` inside the application's namespace in a `.cpp` file; the fragments are function
bodies.

```cpp
const auto text = el::String{"North ÄTHER"_el};
const auto asciiLower = el::toAsciiLowercase(text);          // "north Äther"
const auto unicodeLower = el::toLowercase(text);             // "north äther"
const auto identifier = el::toIdentifierNormalized(text);   // "north_Äther"
```

## Word conversion and whitespace simplification

| Intent | Unicode rules | ASCII rules |
| --- | --- | --- |
| Join words into camel case | `el::toCamelCaseFromWords(text)` | `el::toAsciiCamelCaseFromWords(text)` |
| Separate camel-case words | `el::toWordsFromCamelCase(text)` | `el::toAsciiWordsFromCamelCase(text)` |
| Trim and collapse whitespace | `el::toWhitespaceSimplified(text)` | `el::toAsciiWhitespaceSimplified(text)` |

Word joining defaults to Unicode or ASCII whitespace respectively. Pass a `CharSet` as the second argument for other
separators, such as `el::CharSet{U'_', U'-'}`; include `<erbsland/CharSet.hpp>` when doing so. Edge and repeated separators
produce no empty words. By default, joining preserves the first word and uppercases only the first code point of each
later word, preserving its remaining casing: `foo_HTTP_bar` with underscore separators becomes `fooHTTPBar`.
It does not force the first word to lowercase. Underscores and hyphens are not default separators.

Camel-case splitting defaults to a space and preserves casing: `myHTTPServer` becomes `my HTTP Server`. Pass a `Char`
as the second argument for another separator. Boundaries precede uppercase letters after lowercase letters or digits,
and the final uppercase letter of an acronym followed by a lowercase letter. ASCII variants use ASCII classification
as well as ASCII casing; use Unicode variants when non-ASCII letters must participate in word boundaries. Unicode
classification also recognizes titlecase letters and decimal digits.

Both word-conversion directions accept optional `el::Capitalization` after the separator, or as the second argument
when using default separators. Explicit capitalization applies to every word, including the first. These helpers
assume prepared text; they do not validate or sanitize identifiers.

```cpp
const auto words = el::String{"HTTP server name"_el};
const auto typeName = el::toCamelCaseFromWords(words, el::Capitalization::Capitalized); // "HttpServerName"
const auto heading = el::toWordsFromCamelCase(typeName); // "Http Server Name"
const auto key = el::toAsciiWordsFromCamelCase(typeName, U'_', el::Capitalization::Lowercase); // "http_server_name"
```

Whitespace simplification trims both ends and replaces each internal run with one separator, defaulting to a space;
pass a `Char` as the second argument to change it. Empty or whitespace-only input becomes empty. ASCII whitespace is
`U+0020` and `U+0009` through `U+000D`; Unicode additionally recognizes `U+0085` and Unicode space, line, and paragraph
separator categories. Zero-width space and byte-order marks are not whitespace under these rules. Output separators
for splitting and simplification must be valid Unicode scalar values.

```cpp
const auto spaced = el::String{"  north\t station  "_el};
const auto compact = el::toAsciiWhitespaceSimplified(spaced);        // "north station"
const auto separated = el::toAsciiWhitespaceSimplified(spaced, U'_'); // "north_station"
```

## Replace, remove, and truncate with one custom mapping

Use `text.transformed(...)` for custom per-character rules. Include `<erbsland/Char.hpp>` and `<erbsland/String.hpp>`
for this example. The callback receives one decoded `el::Char` and returns an `el::Char`:

| Return value | Effect |
| --- | --- |
| The input character | Keep it |
| Another character | Replace it |
| `el::Char::noCodePoint()` | Omit this character and continue with the next input character |
| `el::Char::endOfData()` | Stop before this character; omit it and the remaining suffix |

These signals are factory calls with parentheses. Neither signal is emitted into the result. In particular,
`noCodePoint()` removes only the current character, while `endOfData()` truncates the entire remaining input.

For example, this label rule lowercases ASCII, turns hyphens into underscores, removes ordinary spaces, and stops
at `#`. All four rules fit in one transformation:

```cpp
const auto raw = el::String{"Sensor-ID 07#ignored suffix"_el};
const auto label = raw.transformed([](const el::Char character) noexcept -> el::Char {
    if (character == U'#') {
        return el::Char::endOfData();
    }
    if (character == U' ') {
        return el::Char::noCodePoint();
    }
    if (character == U'-') {
        return el::Char{U'_'};
    }
    return character.toAsciiLowercase();
});
// label is "sensor_id07"; raw is unchanged.
```

With `#` as the first character, the result is empty. With no `#`, the mapping processes the whole input. A custom
mapping can reuse any predefined `Char` transformation for its remaining characters, as above.

The callback type is `Char (*)(Char) noexcept`. Use a named `noexcept` function or a captureless `noexcept` lambda
with an explicit `el::Char` return type. Keep it deterministic and free of observable side effects: Core may invoke
it more than once for the same input while sizing and writing the result. Do not count calls, mutate external state,
or depend on invocation order. For position-dependent or stateful processing, choose an iterator/reader or `forEach()`
with explicit output construction; see [parsing and scanning](string-parsing.md).

## Efficiency and operation choice

- Unchanged common transformations and `String::transformed()` share source storage; do not pre-scan merely to avoid a copy.
- The current UTF-8 `transformed()` implementation sizes changed output before writing it and copies an unchanged prefix directly.
  Combining mapping, removal, and truncation avoids intermediate strings from separate transformation stages.
- `endOfData()` stops processing the remaining suffix during sizing and writing. This can save work as well as
  shorten the output. The callback contract permits repeated calls; do not describe the operation as a single pass.
- ASCII mappings perform simple character checks without Unicode case data. Prefer them when their semantics match
  the task, while keeping Unicode mappings for text that requires them.

Use `transformed()` for custom per-character rules that emit at most one code point per input character. For replacing a
substring with arbitrary text, use `replacedAll()`; for a known range, use `slice()` or the appropriate truncation
operation. A single transformation does not need a `StringEditor` or a manual append loop.

## Detailed contracts

- Common free functions and capitalization enum: `src/erbsland/text/StringTransformations.hpp` and
  `src/erbsland/text/Capitalization.hpp`; defaults and examples in `doc/reference/text/strings.rst`.
- Common transformation behavior across string widths and editors: `test/unittest/src/text/StringTransformationsTest.cpp`.
- Predefined mappings and signal factories: `src/erbsland/text/Char.hpp`.
- Unicode mapping behavior: `src/erbsland/text/Char_unicode.cpp`.
- Callback type and invocation contract: `src/erbsland/text/TransformCharacterFn.hpp`.
- String contract: `src/erbsland/text/u8/U8String.hpp`, `transformed()`.
- Allocation and early-stop behavior: `src/erbsland/text/u8/impl/U8StringTransformTools.cpp`.
- Consumer examples: `doc/topics/text_strings/transforming_strings.rst` and
  `doc/topics/text_strings/working_with_character_sets.rst`.
