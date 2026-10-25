# Lowercase, uppercase, and custom string transformations

Read this for case conversion, ELCL-style identifier normalization, or mapping, removing, and stopping at decoded
characters. Paths below are relative to the Core dependency.

## Lowercase and uppercase strings with `transformed()`

Core strings have no `toLower()` or `toUpper()` method. Use `text.transformed(...)` with a predefined transformation
from `el::Char`. This returns a new string value and leaves the source unchanged. The same operation accepts custom
mappings, so use it before writing a character loop or converting to `std::string` for case conversion.

| Intent | Expression |
| --- | --- |
| ASCII lowercase | `text.transformed(el::Char::toAsciiLowercase)` |
| ASCII uppercase | `text.transformed(el::Char::toAsciiUppercase)` |
| Unicode simple lowercase | `text.transformed(el::Char::toLowercase)` |
| Unicode simple uppercase | `text.transformed(el::Char::toUppercase)` |
| Unicode simple case folding | `text.transformed(el::Char::caseFolded)` |
| ELCL-style identifier normalization | `text.transformed(el::Char::toIdentifierNormalized)` |

Choose ASCII mapping when the contract calls for ASCII casing: it leaves non-ASCII characters unchanged and avoids
Unicode database lookups. Choose Unicode mapping when non-ASCII letters must also change case. Unicode mappings here
are simple one-code-point mappings; they do not expand a character into several characters or apply language-specific
casing rules. Encoded byte length can still change. Case folding serves caseless matching and is distinct from
lowercasing. Unicode normalization forms are a separate operation, `normalized(NormalizationForm)`.

`toIdentifierNormalized` applies ASCII lowercase and changes each ordinary space (`U+0020`) to an underscore
(`U+005F`). It leaves tabs, other whitespace, punctuation, and non-ASCII characters unchanged. It neither trims nor
collapses spaces and does not validate an identifier.

For the code fragments below, include `<erbsland/Char.hpp>` and `<erbsland/String.hpp>`. Put
`using namespace el::text::literals;` inside the application's namespace in a `.cpp` file; the fragments are function
bodies.

```cpp
const auto text = el::String{"North ÄTHER"_el};
const auto asciiLower = text.transformed(el::Char::toAsciiLowercase);       // "north Äther"
const auto unicodeLower = text.transformed(el::Char::toLowercase);         // "north äther"
const auto identifier = text.transformed(el::Char::toIdentifierNormalized); // "north_Äther"
```

Pass the static `Char` function directly, without calling it or wrapping it in a lambda.

## Replace, remove, and truncate with one custom mapping

The callback receives one decoded `el::Char` and returns an `el::Char`:

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

- An unchanged `String::transformed()` result shares the source storage; do not pre-scan merely to avoid a copy.
- The current UTF-8 implementation sizes changed output before writing it and copies an unchanged prefix directly.
  Combining mapping, removal, and truncation avoids intermediate strings from separate transformation stages.
- `endOfData()` stops processing the remaining suffix during sizing and writing. This can save work as well as
  shorten the output. The callback contract permits repeated calls; do not describe the operation as a single pass.
- ASCII mappings perform simple character checks without Unicode case data. Prefer them when their semantics match
  the task, while keeping Unicode mappings for text that requires them.

Use `transformed()` for per-character rules that emit at most one code point per input character. For replacing a
substring with arbitrary text, use `replacedAll()`; for a known range, use `slice()` or the appropriate truncation
operation. A single transformation does not need a `StringEditor` or a manual append loop.

## Detailed contracts

- Predefined mappings and signal factories: `src/erbsland/text/Char.hpp`.
- Unicode mapping behavior: `src/erbsland/text/Char_unicode.cpp`.
- Callback type and invocation contract: `src/erbsland/text/TransformCharacterFn.hpp`.
- String contract: `src/erbsland/text/u8/U8String.hpp`, `transformed()`.
- Allocation and early-stop behavior: `src/erbsland/text/u8/impl/U8StringTransformTools.cpp`.
- Consumer examples: `doc/topics/text_strings/transforming_strings.rst` and
  `doc/topics/text_strings/working_with_character_sets.rst`.
