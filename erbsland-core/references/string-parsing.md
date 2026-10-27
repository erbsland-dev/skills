# Choose a string parsing or scanning tool

Read this before adding a character loop, constructing a `StringCharReader`, or converting a scalar from text.
Paths below are relative to the Core dependency; confirm unfamiliar signatures against its headers.
For collecting fields, read [splitting](string-splitting.md). For constructing output, read
[formatting and joining](string-formatting.md).

## Start with the operation, not a reader

| Task | Preferred tool | Why |
| --- | --- | --- |
| Convert a string containing one Boolean, integer, or float | `String::toBoolean`, `toInteger<T>`, `toFloat<T>`, or their `OrThrow` forms | Expresses whole-value conversion, including range and trailing-input checks |
| Locate a delimiter, substring, or first disallowed character | `find`, `findFirstOf`, `findFirstNotOf`, and corresponding last-match methods; see [matching](string-matching.md) | Returns the position directly without a custom scan |
| Inspect decoded characters with custom state or stop/error conditions | `String::forEach` | Direct sequential iteration with loop control; optional built-in code-point index |
| Resume from a native position, walk backward, or use a minimal cursor | `readCharAndAdvance`, `readCharAndRetreat`, `advance`, `retreat` | Operates on the existing string and a typed index without constructing a reader |
| Parse a grammar with lookahead, rollback, captures, transformed buffers, or integer tokens | One `StringCharReader`, passed by reference to consuming helpers | Supplies the parser state and operations that would otherwise need custom code |

Use a range-based loop for straightforward visitation when it is already clear. Prefer the higher-level search,
conversion, or splitting operation when it expresses the entire task. Do not introduce a reader merely because the
input happens to be text.

## Convert a complete integer directly to its target type

For strings such as `123`, `0xa45e`, or `+12`, call `text.toIntegerOrThrow<T>()` directly. Do not construct a reader,
parse into a larger integer, check `numeric_limits<T>`, and cast down. Conversion already checks the range of `T`.
Keep additional checks only for application limits narrower than the type, such as a port range excluding zero.

Use `toInteger<T>(fallback, options)` only when failure should mean that fallback; it returns the fallback for both
invalid syntax and overflow. `toIntegerOrThrow<T>(options)` preserves failure as an exception, so invalid text does
not silently become zero. The same choice applies to Boolean and floating-point conversion.

With `<erbsland/String.hpp>` and `<cstdint>`, in a function body with the literal namespace in scope:

```cpp
const auto decimal = el::String{"123"_el}.toIntegerOrThrow<std::uint16_t>();
const auto hexadecimal = el::String{"0xa45e"_el}.toIntegerOrThrow<std::uint16_t>();
const auto signedValue = el::String{"+12"_el}.toIntegerOrThrow<std::int16_t>();
```

For an existing `el::String`, call its method without constructing another string. Parsing `65536` as `std::uint16_t`
fails directly; do not clamp, wrap, or retry in a wider type to hide that failure.

### Match the integer grammar explicitly

Use `<erbsland/IntegerParseOptions.hpp>` and the relevant per-type headers when customizing options.
Start from `IntegerParseOptions::stringDefault()` for whole-string conversion, then change only what the grammar needs.
The default constructor and `parserDefault()` do not enable the same sign flags.

| Requirement | `IntegerParseOptions` choice |
| --- | --- |
| Automatic base detection | Default: recognize `0x`, `0b`, and `0o`; otherwise decimal |
| A fixed decimal/hexadecimal/binary/octal grammar | `setFixedBase(IntegerBase::...)`; fixed-base parsing rejects base prefixes |
| Leading signs | `stringDefault()` enables `AcceptMinusSign` and `IgnorePlusSign`; clear flags when forbidden |
| Group separators | Add `IntegerParseFlag::AllowSeparator`; `setSeparator()` changes the default apostrophe |
| Digit-count bounds | `setMinimumDigits()`, `setMaximumDigits()` with `CpLength` |
| Exact-width decimal or hex token | `fixedDecimal(count)` / `fixedHex(count)`; these also set `StopAtMaximum` |
| Accept a numeric prefix and ignore the suffix | Add `IgnoreTrailingChars` only when that is the input contract |

Whole-string conversion normally requires the complete input and does not trim whitespace. Trim only if the grammar
permits it. `StopAtMaximum` controls digit scanning; it does not by itself waive the whole-string trailing-input check.
When the suffix must be parsed too, use the existing reader's `readIntegerOrThrow<T>()` and continue from its position.
The string conversion API may use a reader internally; prefer it for its complete contract, not an assumed zero-allocation
implementation.

Sources: `src/erbsland/text/{IntegerParseOptions,IntegerParseFlag}.hpp`,
`src/erbsland/text/u8/U8String_integer.tpp`, `src/erbsland/text/impl/IntegerConversion.hpp`, and
`doc/topics/text_parsing_and_encoding/converting_text_and_scalar_values.rst`.

## Search before scanning

Use a direct search when the task is locating text or a character from a set. The
[matching reference](string-matching.md#validate-and-locate-code-points-with-charset) covers `CharSet` validation,
first/last searches, native positions, a slicing example, and when to cache a set. Continue with a cursor or reader
only when the surrounding parsing operation needs its state.

## Use the indexed `forEach` overload instead of a second counter

The callback forms are `(el::Char) -> el::LoopStatus` and `(el::Char, el::CpIndex) -> el::LoopStatus`.
The second supplies the zero-based decoded code-point index; do not increment a separate character counter.
Return `Continue`, `Stop`, or `Error`. The call returns `LoopResult::Success`, `Stopped`, or `Error`, respectively
for complete traversal, requested early stop, or reported failure. Retain that result when completion matters.
Store application-specific failure details in captured state; the callback does not return an arbitrary result type.

For a positional rule, such as an ASCII identifier whose first character cannot be a digit, use
`<erbsland/Char.hpp>`, `<erbsland/CpIndex.hpp>`, `<erbsland/LoopStatus.hpp>`, `<erbsland/LoopResult.hpp>`, and
`<erbsland/String.hpp>`:

```cpp
const auto text = el::String{"item2!"_el};
auto invalidAt = el::CpIndex::noIndex();
const auto result = text.forEach([&invalidAt](const el::Char character, const el::CpIndex index) -> el::LoopStatus {
    const auto accepted = character.isAsciiLetter() || character == U'_' ||
        (!index.isZero() && character.isAsciiDigit());
    if (!accepted) {
        invalidAt = index;
        return el::LoopStatus::Error;
    }
    return el::LoopStatus::Continue;
});
```

Here `result` is `Error` and `invalidAt` is code-point index 5. Empty input invokes no callbacks and returns `Success`;
check emptiness separately if the grammar requires a nonempty value. `CpIndex` is not a UTF-8 byte offset, grapheme
index, or terminal column. Do not cast it to `ByteIndex` for slicing.

`forEach` avoids constructing a `StringCharReader`. Its callbacks use `std::function`, so do not promise that every
capture is allocation-free. Prefer the overload for its traversal and control-flow semantics without claiming an
unmeasured speedup for every workload.

Sources: `src/erbsland/text/ProcessCharacterFn.hpp`, `src/erbsland/text/u8/impl/U8StringTransformTools.cpp`, and
`src/erbsland/util/{LoopStatus,LoopResult}.hpp`.

## Keep a native cursor for lightweight sequential work

For UTF-8 `String`, start with `auto position = el::ByteIndex::zero()` and read using
`text.readCharAndAdvance(position)`. Save the position before the read when a token's start is needed.
`advance(position)` skips forward; `retreat(position)` moves backward by decoded characters. To read backward,
start at `text.indexAt(el::StringSide::Back)` and use `readCharAndRetreat(position)`.

These operations act directly on strings, without reader construction or a separate heap-backed cursor. Use the
string width's native index type for UTF-16/UTF-32 rather than assuming every string uses `ByteIndex`.
Never increment a byte index by one to advance one Unicode character. Repeated `charAt(CpIndex)` on UTF-8 can rescan
from the start and make traversal quadratic.

An exact-end read returns `Char::endOfData()` without moving; an invalid/past-end index returns `Char::noCodePoint()`.
Malformed encoding is decoded tolerantly as replacement characters. Keep cursor positions valid and stop on the
appropriate signal; a null character is data, not the end marker. If the input contract requires strict UTF-8,
validate once before parsing instead of adding repeated validation inside the loop.

Sources: `doc/reference/text/strings.rst` and `doc/topics/text_strings/character_access_and_parsing.rst`.

## Use one reader when the grammar benefits from it

`StringCharReader` allocates an internal backend. Create it once for the parsing operation and pass
`el::StringCharReader &reader` to helpers that consume the same input. Reconstructing it for each field loses the
shared cursor and repeats setup. Copy deliberately for independent lookahead, or use `save()` / `restore()` for a
trial branch on the same visible text.

Use `peek`, `readIf`, `advanceIf`, `readWhile`/`readUntil`, and `advanceWhile`/`advanceUntil` for grammar operations.
`startCapture()` / `takeCapture()` returns a shared slice as `AnyString`, preserving source encoding; use it for
unchanged token text. For decoded escapes or other rewritten content, use the reader's buffer operations such as
`appendToBuffer`, `appendCaptureToBuffer`, and `takeBuffer`. Capturing does not copy the selected source characters,
but converting to another encoding or assembling a buffer can allocate.

For an integer embedded in an already active parse, use the reader rather than capturing and reparsing a substring.
With `<erbsland/StringCharReader.hpp>`, `<erbsland/IntegerParseOptions.hpp>`, `<erbsland/IntegerParseFlag.hpp>`, and
`<erbsland/String.hpp>`:

```cpp
auto reader = el::StringCharReader{el::String{"12;next"_el}};
auto options = el::IntegerParseOptions::parserDefault();
options.addFlags(el::IntegerParseFlag::IgnoreTrailingChars);
const auto count = reader.readIntegerOrThrow<int>(options);
const auto hasSeparator = reader.readIf(U';');
```

This reads 12 and then the separator; the reader remains at `next`. Unlike whole-string conversion, the surrounding
parser owns suffix validation. Integer parsing restores the original position on failure. Reader `position()` is a
code-point index. A capture can retain a large source allocation, just like an ordinary string slice.

Sources: `src/erbsland/text/StringCharReader.hpp`, `src/erbsland/text/StringCharReader_integer.tpp`,
`doc/reference/text/formatting_and_parsing.rst`, and `doc/topics/text_strings/character_access_and_parsing.rst`.
