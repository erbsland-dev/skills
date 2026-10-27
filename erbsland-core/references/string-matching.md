# Match text with the appropriate Core API

Read this before implementing text searches, character validation, wildcard patterns, regular expressions, or fuzzy
suggestions. `String` aliases `U8String`; the literal, character-set, pattern, and regex operations below also support
`U16String` and `U32String`. Keep the input's encoding and use its native coordinates.

## Choose the matching operation

| Intent | Start with |
| --- | --- |
| Test a literal prefix, suffix, or substring | `startsWith`, `endsWith`, `contains`, with an optional character comparator |
| Count literal occurrences or locate a substring | `count`, `find`; `find` also accepts a native start index |
| Validate allowed code points or detect a forbidden character | `containsOnly`, `containsOneOf` with a `CharSet` |
| Locate the first/last character inside or outside a set | `findFirstOf`, `findFirstNotOf`, `findLastOf`, `findLastNotOf` |
| Recognize a small prefix/suffix shape and extract its middle | `StringPattern::fromDivider` |
| Match a complete glob pattern | `StringPattern::fromGlob`; `fromPosixGlob` for its distinct compatibility dialect |
| Match structured text with alternatives, repetitions, and captures | `el::RegEx::compile` or `el::RegEx::lazyCompile` |
| Rank approximate candidate strings | `el::fuzzy::Matcher`, returning `el::fuzzy::Match` results |

Prefer these complete operations over hand-written character loops, temporary lowercase copies, or conversion to
`std::string`. For sequential grammar parsing, use [parsing and scanning](string-parsing.md); for delimiter-separated
fields, use [splitting](string-splitting.md).

## Use the Erbsland regular expression engine

Core includes a feature-rich, Unicode-aware regular expression engine: **`el::RegEx`**, included through
`<erbsland/RegEx.hpp>`. Use it for regex work in Core consumers instead of reaching for `std::regex` or adding PCRE.
Its Thompson NFA avoids the catastrophic backtracking behavior of backtracking engines and scales with subject
length for a fixed compiled pattern. Pattern size, captures, and input costs still matter.
`el::re::Settings` provides execution and complexity limits when the application needs them.

### Compile once and reuse the shared expression

- `el::RegEx::compile(pattern)` compiles and validates immediately and returns `el::re::RegExPtr`, a shared pointer.
  Call methods with `->`; do not construct `RegEx{pattern}` or copy the `std::regex` construction/matching model.
- `el::RegEx::lazyCompile(pattern)` retains the pattern and defers compilation until first use, reducing creation
  cost when an expression might never be used. It does not validate at creation. `compileNow()` triggers validation;
  `isCompiled()` inspects the state without triggering compilation.
- Expressions are immutable and safe to share across threads. Concurrent first use of a lazy expression coordinates
  compilation. Cache a fixed expression as `static const auto`, or retain a configuration-dependent expression with
  its owner, rather than recompiling inside a matching loop.
- Invalid patterns throw `el::re::RegExError`. With lazy compilation that error occurs at first use or `compileNow()`;
  a failed compilation leaves the expression uncompiled and a later use retries. Validate external patterns when
  accepting configuration if that is where errors should be reported.

With `<erbsland/RegEx.hpp>`, `<erbsland/re/Match.hpp>`, and `<erbsland/String.hpp>`, in a function body with
`using namespace el::text::literals;` supplied by the enclosing application namespace:

```cpp
static const auto cRecord = el::RegEx::compile(R"((?<name>[a-z]+):(?<value>[0-9]+))"_el);
const auto text = el::String{"sensor:42"_el};
if (const auto match = cRecord->fullMatch(text)) {
    const auto name = match->content("name"_el);
    const auto value = match->content("value"_el);
    // name and value share the subject's storage: "sensor" and "42".
}
```

Use `lazyCompile` in the same factory position when deferral is useful. `RegEx` is flattened into `el`, while its
supporting types such as `el::re::Match`, `RegExError`, `Flags`, `Settings`, and `Input` remain in `el::re`.

### Select the matching boundary deliberately

| Method | Contract |
| --- | --- |
| `match(text)` | Match at the beginning; trailing text may remain |
| `fullMatch(text)` | Require the entire subject to match; use for whole-value validation |
| `findFirst(text)` | Search for the first match anywhere |
| `findAll(text)` | Lazily yield matches through a generator |
| `collectAll(text)` | Eagerly collect matches into a list |

The first three return a null shared pointer on mismatch. Test it before accessing captures. An unsuccessful
`findAll`/`collectAll` is empty. `content()` selects the whole match; `content(name)` or a `CaptureGroupIndex` selects
a capture. `begin`, `end`, and `range` expose positions in the original encoding, with an exclusive end.

String subjects produce `re::Match`, `re::Match16`, or `re::Match32` according to their width. Captured strings are
zero-copy slices that retain the source allocation; they remain valid after the original variable or match object
is destroyed. Match objects and generators can still allocate, and a small capture can retain a large allocation.
See [string ownership](strings.md#ownership-and-mutation) for the shared-storage model.

### Write Erbsland regex syntax

The language is close to PCRE, with stricter parsing and its own supported feature set. Read the dependency's
`doc/topics/re/syntax/overview.rst` for the expression being written; the published
[Erbsland regex syntax overview](https://re.erbsland.dev/syntax/overview.html) is also a useful guide. Core's actual
headers and bundled syntax documentation govern version differences from the standalone library.

Do not assume `std::regex`/ECMAScript or arbitrary PCRE syntax. Only documented escapes are accepted. In the current
language, backreferences and lookahead/lookbehind are unsupported. Prefer `(?<name>...)` for named captures and
`(?:...)` when a capture is unnecessary. Global flags such as `(?i)` belong at the beginning; use `(?i:...)` for a
local scope. Check character-class and anchor semantics instead of importing another engine's assumptions, especially
for `\d`, `\s`, `\w`, and line endings. Use C++ raw literals for patterns with backslashes so C++ escaping does not
obscure the regex language.

### Match streams or custom inputs without converting everything to a string

The matching methods also accept `el::TextInputStreamPtr`. The built-in stream adapter requires a seekable stream;
it copies captured text into owning results and restores the stream's logical position after capture extraction.
The zero-copy claim for string subjects does not apply to stream captures. Stream failures/timeouts propagate as
errors, not ordinary mismatches or end-of-input.

For an application-specific source, implement `el::re::Input`, `Input16`, or `Input32` and the `InputBase` contract.
The adapter supplies sequential characters, positions, and `createMatch`; it controls capture storage and decoding.
Read `doc/reference/re/regular_expressions.rst` before implementing one. Ordinary Core-string overloads already
decode malformed units tolerantly as U+FFFD; avoid an adapter when a built-in input suffices.

## Literal matching and comparison callbacks

`startsWith(text, compareFn)`, `endsWith(text, compareFn)`, `contains(text, compareFn)`, and
`count(text, compareFn)` compare decoded characters. `count` returns `ItemCount`, counts non-overlapping occurrences,
and returns zero for an empty needle. Use `find(text, compareFn)` when the position is needed, or
`find(text, start, compareFn)` to resume at an inclusive native index. An unsuccessful search returns `noIndex()`.
These APIs perform the search directly without constructing a regex or allocating transformed copies of both inputs.

The optional `CharCompareFn` is a `noexcept` function pointer returning `std::strong_ordering` from two `el::Char`
arguments, not a Boolean predicate. Equality is `std::strong_ordering::equal`.

- `el::Char::compareAsciiFolded`: case-insensitive ASCII matching; other code points remain distinct.
- `el::Char::compareCaseFolded`: Unicode simple case folding, comparing one code point at a time.
- `el::Char::compareIdentifier`: ASCII case folding plus equivalence of space and underscore.
- A custom function or captureless `noexcept` lambda with the same signature for an application-specific equivalence.

No normalization or locale collation is implicit. Simple folding cannot match multi-code-point expansions; choose
normalization or [transformations](string-transformation.md) separately when the text contract requires them.

With `<erbsland/ByteIndex.hpp>`, `<erbsland/Char.hpp>`, and `<erbsland/String.hpp>` and literal scope as above:

```cpp
const auto text = el::String{"SENSOR-17 sensor-18"_el};
const auto hasPrefix = text.startsWith("sensor-"_el, el::Char::compareAsciiFolded);
const auto occurrences = text.count("sensor-"_el, el::Char::compareAsciiFolded);
const auto next = text.find("sensor-"_el, el::ByteIndex{1}, el::Char::compareAsciiFolded);
```

Here `hasPrefix` is true, `occurrences` is 2, and `next` is byte index 10. The same comparison callback works with
`endsWith` and `contains`; `compare(other, compareFn)` supplies whole-string ordering or equality.

## Validate and locate code points with CharSet

A `CharSet` describes allowed or selected code points. It is not a substring, multi-character delimiter, or full
regular expression. Use `containsOneOf(set)` for any member, `!containsOneOf(set)` for no members, and
`containsOnly(set)` for all characters being allowed. Empty text satisfies `containsOnly` and fails `containsOneOf`;
add `!text.isEmpty()` when a value must be nonempty. `containsOnly(AsciiCategory)` also handles built-in ASCII classes.

Use `findFirstOf(set)` / `findLastOf(set)` to locate a member and `findFirstNotOf(set)` / `findLastNotOf(set)` to locate
a nonmember. These are the direct, efficient alternatives to custom decoding loops. Forward overloads accept an
inclusive `start`; reverse overloads accept an exclusive `end`. Check `isNoIndex()` before slicing or reporting a
position. Membership uses decoded code points; malformed units become U+FFFD under the ordinary tolerant policy.

### Construct small sets directly; retain larger sets

`CharSet` stores up to two ranges inline. Constructors taking one `Char`, one `CharRange`, or two `CharRange` values
are `constexpr` and cheap; small fixed sets do not need a cache for performance. Prefer direct range construction for
a known range instead of parsing a set pattern. `CharSet(U';')` selects the single-character constructor directly;
the braced `CharSet{U';'}` selects the initializer-list overload. The constructors being `constexpr` does not imply
that the complete `CharSet` type supports a `constexpr` variable in every supported C++ version.

Three or more stored ranges use shared storage. Cache an unchanged larger set as `static const` in repeated code,
or retain a configuration-dependent set with its owner. Parsing `fromPattern` and constructing Unicode-property
sets can also involve work beyond the final storage size; do not repeat that setup inside a hot loop.

With `<erbsland/CharRange.hpp>`, `<erbsland/CharSet.hpp>`, and `<erbsland/String.hpp>` and literal scope as above:

```cpp
const auto digits = el::CharSet{el::CharRange{U'0', U'9'}};
const auto letters = el::CharSet{el::CharRange{U'A', U'Z'}, el::CharRange{U'a', U'z'}};
static const auto cNameChars = el::CharSet::fromPattern("_a-zA-Z0-9"_el);
const auto text = el::String{"sensor_17!"_el};
const auto valid = !text.isEmpty() && text.containsOnly(cNameChars);
const auto invalidAt = text.findFirstNotOf(cNameChars);
```

Here `valid` is false and `invalidAt` is byte index 9. `fromPattern("a-z"_el)` describes a range without outer square
brackets; `CharSet{"a-z"_el}` instead collects the three literal characters. A leading `^` in the set pattern inverts
it. Keep this syntax distinct from `StringPattern` bracket sets and regex character classes.

### Preserve native positions

| Subject | Native index / length / range |
| --- | --- |
| `String`, `U8String` | `ByteIndex`, `ByteLength`, `ByteRange` |
| `U16String` | `U16DataIndex`, `U16DataLength`, `U16DataRange` |
| `U32String` | `CpIndex`, `CpLength`, `CpRange` |

Use returned positions directly, preserving encoded boundaries. UTF-8 byte offsets and UTF-16 code-unit offsets
are not decoded-character counts. With `<erbsland/ByteIndex.hpp>`, `<erbsland/ByteRange.hpp>`,
`<erbsland/CharSet.hpp>`, and `<erbsland/String.hpp>`:

```cpp
const auto text = el::String{"prefix[value]"_el};
const auto opening = text.findFirstOf(el::CharSet(U'['));
if (!opening.isNoIndex()) {
    const auto prefix = text.slice(el::ByteRange{el::ByteIndex::zero(), opening});
    // prefix is "prefix"; the source text is shared.
}
```

## StringPattern: choose divider or glob semantics

Include `<erbsland/StringPattern.hpp>` and create a reusable pattern through a named factory. All three factories
match decoded characters case-sensitively and accept UTF-8/16/32 pattern text. Retain parsed patterns across repeated
matches; they are immutable compiled values.

| Factory | Matching boundary and syntax |
| --- | --- |
| `fromDivider` | Without `*`, match a prefix. One `*` separates an anchored prefix and suffix with a non-overlapping middle. `*` alone and multiple dividers are invalid. |
| `fromGlob` | Match the entire string. Any number of `*` wildcards each match zero or more characters. |
| `fromPosixGlob` | Full-string compatibility glob with Python-style brackets: `[!a-z]` negation, literal backslashes and unmatched brackets. |

Divider and Core glob syntax support `?` for one code point, `[a-z]`, `[^a-z]`, and backslash escapes only for
`?`, `*`, `[`, `]`, and backslash. Glob stars include separators and newlines; this is not automatically a filesystem
traversal or path-component matcher. An empty divider never matches; an empty glob matches only empty text.
Invalid parsed patterns throw `el::ParseError`.

The divider checks its boundary parts without scanning the arbitrary middle. Use it to recognize or unwrap a small
envelope efficiently. With `<erbsland/StringPattern.hpp>` and `<erbsland/String.hpp>` and literal scope as above:

```cpp
static const auto cEnvelope = el::StringPattern::fromDivider("record:*;ok"_el);
const auto text = el::String{"record:alpha;ok"_el};
if (cEnvelope.matches(text)) {
    const auto middle = cEnvelope.trimmed(text);             // "alpha"
    const auto [prefix, payload] = cEnvelope.split(text);    // "record:", "alpha"
}
static const auto cFiles = el::StringPattern::fromGlob("report-*-??.txt"_el);
const auto accepted = cFiles.matches(el::String{"report-north-17.txt"_el});
```

For divider patterns, `trimmed` removes matching prefix/suffix parts and returns a shared slice. `split` returns
`{prefix, remainder}` for a prefix-only pattern, `{prefix, middle}` for both parts, and `{leading text, suffix}` for
a suffix-only pattern. A mismatch returns the original text from `trimmed`, or `{empty, text}` from `split`.
Test `matches` when an empty middle is a valid success. Glob `trimmed` returns empty on a match, and glob `split`
returns `{text, empty}`; neither extracts individual wildcard captures. Use regex captures when those are needed.

`trim(editor)` mutates an editor and reports whether text was removed. `length(text)` and `index(text)` expose the
selected native boundary; see the header for prefix versus suffix-only semantics. Typed factories use `el::pattern`
elements: `Divider` for `fromDivider`, `Star` for `fromGlob`, plus `Text`, `OneChar`, `Range`, and `Set`. Use the typed
form when explicit components are clearer than parsed syntax, and verify its limits against the dependency.

## Fuzzy suggestions

Use `<erbsland/fuzzy/Matcher.hpp>` for `el::fuzzy::Matcher`; **`el::fuzzy::Match` is the result**, not the search engine.
The matcher uses bounded Damerau-Levenshtein distance over decoded code points, with an adjacent transposition
counting as one edit. Set the distance and result limits for the suggestion task and pass candidates in preferred
tie order. It returns deduplicated results ordered by distance, preserving candidate order for ties.

With `<erbsland/fuzzy/Matcher.hpp>`, `<erbsland/fuzzy/Match.hpp>`, `<erbsland/Char.hpp>`, `<erbsland/CpLength.hpp>`,
`<erbsland/ItemCount.hpp>`, `<erbsland/String.hpp>`, and `<erbsland/StringList.hpp>`:

```cpp
auto matcher = el::fuzzy::Matcher{"senosr"_el};
matcher.setMaximumDistance(el::CpLength{1});
matcher.setMaximumResults(el::ItemCount{3});
matcher.setComparisonFn(el::Char::compareAsciiFolded);
const auto matches = matcher.findMatches(el::StringList{{"sensor"_el, "status"_el}});
```

The result contains `sensor` at distance 1. Each `Match` exposes `text()` / `candidate()` and `distance()`.
Use fuzzy matching for suggestions where approximate equality is intended, keeping exact validation rules explicit.

## Source contracts

Paths are relative to the Core dependency. Consult only the relevant contract:

- Regex: `src/erbsland/re/{RegEx,Match,Match16,Match32,InputBase}.hpp`,
  `doc/reference/re/regular_expressions.rst`, and `doc/topics/re/syntax/`.
- Strings and comparators: `src/erbsland/text/{u8/U8String,u16/U16String,u32/U32String,Char,CharCompareFn}.hpp`
  and `doc/topics/text_strings/comparing_strings.rst`.
- Character sets: `src/erbsland/text/{CharSet,CharRange}.hpp` and
  `doc/topics/text_strings/working_with_character_sets.rst`.
- Patterns: `src/erbsland/text/StringPattern.hpp`, `doc/reference/text/matching_and_splitting.rst`, and
  `doc/topics/text_parsing_and_encoding/using_string_patterns.rst`.
- Fuzzy matching: `src/erbsland/text/fuzzy/{Matcher,Match}.hpp` and `doc/topics/text_strings/fuzzy_matching.rst`.
