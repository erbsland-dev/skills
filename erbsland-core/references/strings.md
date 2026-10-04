# Choose Core text operations

Read this for text processing, parsing, construction, or performance-sensitive string work.
Paths below are relative to the Core dependency.

## Ownership and mutation

- `String` aliases `U8String`: an owning, read-only UTF-8 value with copy-on-write backing storage.
  Use it for parameters, ordinary stored values, and copy-returning transformations.
  Method details are in `src/erbsland/text/u8/U8String.hpp` and its bases/implementations.
- Pass `"text"_el` directly when accepted. Only construct `el::String{"text"_el}` when an object or its methods are needed.
- `StringEditor` aliases `U8StringEditor`: a local mutable working value for dependent edits or construction.
  Constructing it from a nonempty read-only string or literal copies the text into editable storage; copying an
  existing editor can share storage until mutation. Return the finished result as `String`.
- Default to common aliases. Use UTF-16/UTF-32 types at the corresponding encoding boundaries, and `AnyStringBuilder`
  for output supporting multiple widths or direct value formatting.

### Literal scope

Include each public type used. Put `using namespace el::text::literals;` inside the application's namespace in a `.cpp`
file, rather than repeating it in every function or placing it at header scope.
Use `el::StringLiteral{"..."}` in headers/templates with a few literals or constexpr variables.
In a template function with more than five literals, place the literal using-directive inside that function.

## Choose the operation

| Intention | Search/use |
| --- | --- |
| Strip boundary characters | `trimmed()` with the appropriate `CharSet` and `StringSide` |
| Change or remove occurrences | `replacedFirst()`, `replacedAll()`, `removedFirst()`, `removedAll()` |
| Map decoded characters | `transformed()` with a `Char` mapping function |
| Normalize Unicode | `normalized(NormalizationForm)`; choose the form deliberately |
| Locate text or character sets | `find()`, `findFirstOf()`, related operations and comparison functions |
| Select a shared range | `slice()`, `first()`, `last()` with typed coordinates |
| Collect split fields | `StringList::fromSplit()` with a `CharSet` and explicit empty-field policy |
| Sequentially split large text | `StringSplitter` with a `Char` or `CharSet` separator |
| Recognize small wildcard shapes | `StringPattern`; check its limited syntax |
| Convert scalars from/to text | `toBoolean`, `toInteger<T>`, `toFloat<T>`, their `OrThrow` forms, and `from...` factories |
| Construct fixed fragments | `String::fromJoined({...})` |
| Join discovered fragments | `StringList::join()` |
| Format reusable structured output | `StringFormat::build()` or `appendTo()` |
| Parse characters sequentially | `readCharAndAdvance()`; `StringCharReader` for captures, buffers, and complex parsing |

For regular expressions, template rendering, and placeholder expansion, use [the capability map](capability-map.md#text).
A `CharSet` represents a defined set of characters, not a multi-character delimiter or a quoted-field grammar.
`StringList::fromSplit()` discards empty fields by default; `StringSplitter`'s default mode preserves them.
With a splitter, test `isAtEnd()` rather than whether `next()` is empty.

## Choose the allocation strategy

| Workflow | Choice and cost |
| --- | --- |
| Read-only slices and trims | Share backing storage without copying selected characters. Small slices can retain a large source allocation. |
| One transformation or independent derived values | Use `String` operations. Changed text generally needs result storage; unchanged `transformed()` returns the original value. Check storage reuse per operation. |
| Dependent mutations to one working value | Use a local `StringEditor`; balance its initial copy against saved intermediate results. Edits can invalidate earlier positions. |
| Fixed fragments | `String::fromJoined()` sizes and allocates final character storage once. |
| Fragments discovered in a loop | Collect lightweight handles in `StringList`, then `join()`; the list can grow, but joining sizes final character storage once. |
| An editor already needed for mutation | When final native length is known, `reserve()` once before appending, not before each append. UTF-8 capacity uses `ByteLength`. |
| Cross-width or formatted construction | `AnyStringBuilder` appends values without temporary strings, but can still grow/reallocate; pre-size it when capacity is known. |
| Formatting into an existing builder | Retain a parsed `StringFormat` and use `appendTo()` to avoid an intermediate formatted result. |

Keep Core text in Core types until an external API requires conversion. Describe character-storage costs separately
from container growth, static initialization, and stream output; avoiding a text copy does not promise zero allocations
for the entire operation.

## Coordinates and parsing

`String::length()` measures UTF-8 bytes. `CpIndex` and `CpLength` count decoded code points, not grapheme clusters or
terminal cells. `displayWidth()` is an approximate per-code-point measurement, not full layout/shaping.

Use byte positions returned by search/iteration directly. Repeated `charAt(CpIndex)` calls can scan from the start
and make a loop quadratic. Prefer iterators, `readCharAndAdvance()`, or `readCharAndRetreat()`; use code-point ranges
for small, naturally character-indexed values. Check not-found sentinels before creating ranges and preserve encoded
boundaries when slicing text. Use one `StringCharReader` for a complex parsing workflow.

Tolerant string decoding consistently yields replacement characters for malformed input. Use that default unless
the input contract requires rejection; validate once at the boundary with `isValidUtf8()` or select strict/throwing
conversion. Do not add repeated validation inside processing loops.

Scalar parsing normally consumes the complete string without trimming. Trim only when the input grammar permits it.
Use fallback forms when invalid text legitimately means a default, and `OrThrow` forms when invalid input must remain
distinguishable from a valid value. Check parse options for base, signs, overflow, and trailing input as needed.

## Examples

### Clean a label without unnecessary copies

```cpp
#include <erbsland/CharSet.hpp>
#include <erbsland/String.hpp>

namespace app {

using namespace el::text::literals;

/// Trim boundary markers and update an obsolete label component.
auto cleanLabel(const el::String &raw) -> el::String {
    static const auto trimChars = el::CharSet{" *"_el};
    return raw.trimmed(trimChars).replacedAll("old"_el, "new"_el);
}

}
```

`" *** station-old *** "_el` produces `station-new`; `"the-new-station"_el` stays unchanged.

### Use one editor for dependent changes

```cpp
#include <erbsland/String.hpp>
#include <erbsland/StringEditor.hpp>

namespace app {

using namespace el::text::literals;

/// Apply dependent changes to one local editable value.
auto updateLabel(const el::String &raw) -> el::String {
    auto editor = el::StringEditor{raw};
    editor.replaceAll("old"_el, "new"_el);
    editor.append(" ready"_el);
    return editor;
}

}
```

`"station-old"_el` produces `station-new ready`. The editor copies the input text once at construction and can grow
when appended text exceeds its capacity.

### Split, transform, sort, and join

```cpp
#include <erbsland/Char.hpp>
#include <erbsland/CharSet.hpp>
#include <erbsland/String.hpp>
#include <erbsland/StringList.hpp>

namespace app {

using namespace el::text::literals;

/// Normalize nonempty semicolon-separated fields and sort their output.
auto reassemble(const el::String &source) -> el::String {
    const auto fields = el::StringList::fromSplit(source, el::CharSet{U';'});
    auto result = el::StringList{};
    result.reserve(fields.count());
    for (const auto &field : fields) {
        result.append(field.trimmed().transformed(el::Char::toAsciiUppercase));
    }
    result.sort();
    return result.join(", "_el);
}

}
```

`"alpha;;   gamma;BETA;"_el` produces `ALPHA, BETA, GAMMA`. Splitting and trimming retain source storage; character
mapping creates result storage only when it changes a character. The two lists allocate handle arrays, and joining
allocates final character storage once.

### Construct the output shape directly

```cpp
#include <erbsland/String.hpp>
#include <erbsland/StringFormat.hpp>

namespace app {

using namespace el::text::literals;

/// Join a fixed set of label fragments.
auto directJoin(const el::String &name) -> el::String {
    return el::String::fromJoined({"station: "_el, name, " ready"_el});
}

/// Reuse a parsed pattern for structured labels.
auto formatting(const el::String &name, const int number) -> el::String {
    static const auto pattern = el::StringFormat{"station {}/{} ready"_el};
    return pattern.build(name, number);
}

}
```

`directJoin("north"_el)` produces `station: north ready`; `formatting("south"_el, 4)` produces `station south/4 ready`.
Core's named format specifications include text escaping; the compact `std::format`-style syntax is a supported
compatibility subset, not the full standard-library contract.

## Detailed contracts

Read the relevant topic, rather than every page:

- Ownership: `doc/reference/text/strings.rst` and `src/erbsland/text/String.hpp`.
- Transformations: `doc/topics/text_strings/transforming_strings.rst`.
- Editing/storage: `doc/topics/text_strings/editing_strings_in_place.rst` and
  `doc/topics/text_strings/managing_string_editor_storage.rst`.
- Slices/splitting: `doc/topics/text_strings/slicing_and_splitting_strings.rst` and
  `doc/reference/text/matching_and_splitting.rst`.
- Coordinates: `doc/topics/text_strings/character_access_and_parsing.rst`.
- Construction: `doc/topics/text_formatting/building_strings.rst`.
- Formatting: `doc/topics/text_formatting/using_string_format.rst`.
- Scalar conversion: `doc/topics/text_parsing_and_encoding/converting_text_and_scalar_values.rst`.
- Normalization: `doc/topics/text_strings/normalizing_strings.rst`.
