# Use typed indexes, counts, offsets, and ranges

Read this when choosing sequence coordinates, doing index arithmetic, iterating over typed ranges, converting
between Core units and an external API, or constructing versions. Keep values typed throughout processing; do
not extract raw integers just to do arithmetic or change an index into a count.

Time quantities use separate contracts. Paths in the verification section are relative to the Core dependency.

## Choose both the unit and the intent

| Intent | Meaning | Example |
| --- | --- | --- |
| `...Index` | A zero-based, non-negative position, or `noIndex()` for an absent position | `el::ByteIndex{3U}` |
| `...Count` | A non-negative amount, including zero, or `infinite()` | `el::ByteCount{4U}` |
| `...Offset` | Signed movement relative to a position | `el::ByteOffset{-2}` |
| `...Range` | An inclusive beginning and a length; the end is exclusive | `el::ByteRange{el::ByteIndex{3U}, el::ByteCount{4U}}` |

Use the family required by the data and API:

| Unit | Index | Count | Offset | Range |
| --- | --- | --- | --- | --- |
| Bytes, including UTF-8 storage | `ByteIndex` | `ByteCount` | `ByteOffset` | `ByteRange` |
| Decoded Unicode code points | `CpIndex` | `CpCount` | `CpOffset` | `CpRange` |
| Unsigned 16-bit values, including UTF-16 code units | `U16Index` | `U16Count` | `U16Offset` | `U16Range` |
| Source lines | `LineIndex` | `LineCount` | `LineOffset` | `LineRange` |
| Source columns | `ColumnIndex` | `ColumnCount` | `ColumnOffset` | `ColumnRange` |
| Collection items | `ItemIndex` | `ItemCount` | `ItemOffset` | `ItemRange` |
| Command-line and format arguments | `ArgumentIndex` | `ArgumentCount` | — | — |

Names in the table use the consumer prefix `el::` and per-type public headers such as `<erbsland/ByteCount.hpp>`.
Include each concrete type directly; a unit-tag header such as `<erbsland/ArgumentUnit.hpp>` declares only its tag.
Methods keep their semantic names: `List::count()` returns `ItemCount`, `String::length()` returns `ByteCount`,
and `range.length()` returns the matching count type.

A unit and its intent are both part of the type. A `ByteIndex` is neither a `ByteCount` nor a `CpIndex`.
Code-point positions do not measure grapheme clusters or display cells, and a UTF-16 code unit is not necessarily
a whole code point.

The generic positional types are `IntegerUnitIndex<Unit>`, `IntegerUnitCount<Unit>`, `IntegerUnitOffset<Unit>`,
and `IntegerUnitRange<Unit>`. Prefer the concrete aliases for existing families. Unit tags identify the coordinate
system; indexes and counts use unsigned 64-bit values,
offsets use signed 64-bit values, and the largest unsigned value is reserved for `noIndex()` or `infinite()`.

## Construct constants and runtime values deliberately

Numeric scalar constructors are `consteval`: `el::ByteCount{4U}` validates a constant at compile time.
They accept native integers up to 64 bits, excluding `bool`; floating-point values, negative indexes/counts,
and numeric sentinel representations are rejected. Offsets allow the full signed 64-bit range.
Default construction yields zero; copying an already typed value works at runtime.

Use `T::fromInteger(value)` for runtime integers with saturation into the finite domain, or
`T::fromIntegerOrThrow(value)` to reject an out-of-range value with `OverflowError`. Negative inputs to index/count
factories saturate to zero; oversized inputs saturate to the finite maximum. Indexes and counts also provide
`fromSizeT()` / `fromSizeTOrThrow()` for native sizes. These imports never create sentinel states; use the named
state factories. Choose exact conversion when clipping would lose required information.

```cpp
const auto constant = el::ByteCount{16U};
const auto nativeSize = externalBuffer.size();
const auto exactSize = el::ByteCount::fromSizeTOrThrow(nativeSize);
```

## Factories and state tests

Prefer named factories and predicates for special values and common comparisons.

| Kind | Useful factories | Useful predicates |
| --- | --- | --- |
| Index | `zero()`, `one()`, `minimum()`, `maximum()`, `noIndex()`, `end(count)`, `last(count)` | `isZero()`, `isOne()`, `isMinimum()`, `isMaximum()`, `isNoIndex()`, `isValid()`, `isWithin(count)` |
| Count | `zero()`, `one()`, `minimum()`, `maximum()`, `infinite()` | `isZero()`, `isOne()`, `isMinimum()`, `isMaximum()`, `isInfinite()`, `isFinite()` |
| Offset | `zero()`, `one()`, `minusOne()`, `minimum()`, `maximum()` | `isZero()`, `isOne()`, `isMinusOne()`, `isMinimum()`, `isMaximum()`, `isNegative()`, `isPositive()` |
| Range | `empty()`, `emptyAt(index)`, `all()`, `noRange()`, `fromLength(count)`, `fromBegin(index)` | `isEmpty()`, `isValid()`, `isNoRange()`, `isInfinite()`, `contains(index)`, `isWithin(count)` |

`maximum()` is finite, separate from `noIndex()` or `infinite()`. Compare typed values directly rather than their
raw values. Ordinary ordering is available between values of the same type; use predicates to recognize sentinels.

`index.isValid()` only means it is not `noIndex()`. For element access, use `index.isWithin(count)`, which excludes
the end position. For ranges, `isNoRange()` tests an absent beginning, `isEmpty()` tests zero length, and
`isValid()` also checks endpoint representability. These tests are independent: a present range can be invalid.
Default ranges are valid and empty at zero; `noRange()` is empty and invalid.

## Arithmetic: keep the intent visible

| Operation | Result / preferred spelling |
| --- | --- |
| Index plus/minus matching count | Index: `index + count`, `index - count` |
| Index plus/minus matching offset | Index: `index + offset`, `index - offset` |
| Count plus/minus the same count type | Count |
| Offset plus/minus the same offset type | Offset |
| Count multiplied, divided, or modulo an unsigned scalar | Same count type: `count * 3U`, `count / 2U`, `count % 2U` |
| Offset multiplied, divided, or modulo a signed scalar | Same offset type: `offset * 3`, `offset / 2`, `offset % 2` |
| Range plus/minus a matching offset | Range with a moved beginning and unchanged length |

Do not add two indexes or subtract one index from another. Use the distance/offset methods below. Unit values
cannot be multiplied or divided by other unit values; scalar operations preserve the unit rather than creating
a new dimension. Scalars may be native integers or matching signed/unsigned Core saturating integer values.

For indexes, `advance(count)`, `retreat(count)`, and `move(offset)` mutate the receiver; `advanced`, `retreated`,
and `moved` return a new value. `increment()` / `decrement()` mutate by one; `incremented()` / `decremented()`
return a new index. Counts and offsets use `add` / `added` and `subtract` / `subtracted`. All three scalar intents
support `++` and `--`.

```cpp
auto position = el::ByteIndex{4U};
const auto next = position.incremented();                 // ByteIndex{5U}; position stays 4.
position.advance(el::ByteCount{10U});                     // position is 14.
const auto previous = position + el::ByteOffset{-2};       // ByteIndex{12U}.
const auto doubled = el::ByteCount{6U} * 2U;               // ByteCount{12U}.
```

### Saturation and exactness

Ordinary arithmetic saturates at the type's finite bounds: subtracting too much from a finite count produces
zero; advancing a finite index too far produces `maximum()`, never `noIndex()`. A no-index value stays no-index
when moved. Finite count arithmetic does not overflow into infinity. Offsets saturate at their signed limits.

Adding an infinite count yields infinity. Subtracting from infinity keeps infinity, including infinity minus
infinity; subtracting infinity from a finite count yields zero. Infinite counts remain infinite under scalar
multiplication, division, and modulo. Division or modulo by zero terminates.

Use normal saturating operations when clipping is intended. When an exact result is required, choose an available
`OrThrow` operation, such as `advancedOrThrow`, `moveOrThrow`, `addedOrThrow`, or `offsetToOrThrow`. These report
`OverflowError` for unsupported special states or an unrepresentable result according to their contract.
Do not assume every operation has an `OrThrow` variant.

Type bounds are not sequence bounds. Neither saturation nor `advanceOrThrow()` proves an index is within a buffer.
`wouldAdvanceSaturate(count)`, `wouldRetreatSaturate(count)`, and `wouldMoveSaturate(offset)` detect failure of exact
finite movement. `canAdvance(count, bounds)`, `canRetreat(count, bounds)`, and `canMove(offset, bounds)` additionally
require the exact destination to be within the supplied bounds. The starting index need not already be within them.

## Convert between intents with named operations

All conversions in this table stay within the same unit family.

| Needed value | Operation | Meaning / special state |
| --- | --- | --- |
| Count from zero to an index | `index.distanceFromZero()` | Distance, not the count including that element; no-index becomes infinity |
| Absolute distance between indexes | `first.absoluteDistanceTo(second)` | Non-negative count; either no-index makes it infinite |
| Signed movement from one index to another | `first.offsetTo(second)` | `second - first`, saturated to the offset limits; either no-index yields the maximum offset |
| Signed movement from zero | `index.offsetFromZero()` | Saturates if too large or no-index |
| Exclusive end of a count starting at zero | `el::ByteIndex::end(count)` | Finite count becomes that index; infinity becomes no-index |
| Final included index from a count | `el::ByteIndex::last(count)` | Zero or infinite counts produce no-index |
| Magnitude of an offset | `offset.absoluteLength()` | Matching non-negative count; direction is discarded |
| Count remaining after a position | `count.remainingFrom(index)` | Count from index to the exclusive bound; absent or out-of-bounds indexes yield zero |
| Range starting at zero | `el::ByteRange::fromLength(count)` | Preserves the supplied count |

`remainingFrom()` on an infinite count yields infinity for a valid index and zero for no-index.
Use `offsetToOrThrow()` / `offsetFromZeroOrThrow()` when a saturated signed distance would be wrong.
Unsigned index space can be larger than the positive range of an offset.

```cpp
const auto begin = el::ByteIndex{3U};
const auto end = el::ByteIndex{8U};
const auto length = begin.absoluteDistanceTo(end);         // ByteCount{5U}.
const auto backwards = end.offsetTo(begin);               // ByteOffset{-5}.
const auto prefixLength = begin.distanceFromZero();       // ByteCount{3U}, not 4.
const auto prefixEnd = el::ByteIndex::end(prefixLength);   // ByteIndex{3U}.
const auto last = el::ByteRange{begin, end}.lastIndex();   // ByteIndex{7U}, in the range's coordinates.
```

Prefer `Index::last(count)` or a container's `lastIndex()` when available to manually decrementing an exclusive
end. They return no-index for an empty sequence; saturating decrement of zero would incorrectly yield zero.
Use `range.lastIndex()` for the final absolute index of a finite, nonempty, valid range.

## Construct, bound, and move ranges

A range describes `[begin, end)`. Construct it with `{startIndex, count}` or `{beginIndex, endIndex}`, including
compact slicing such as `text.slice({begin, end})`. `fromBeginEnd()` has the same semantics: reversed finite
endpoints produce `emptyAt(begin)`, an absent beginning produces `noRange()`, and a valid beginning with a no-index
end produces an open-ended range. `fromBeginEndOrThrow()` rejects an absent beginning or reversed finite endpoints
with `OutOfRangeError`, while accepting an open endpoint. Check search results before using them as endpoints.

`fromBegin(begin)` directly preserves its argument and supplies infinite length. It performs no validation;
a no-index beginning produces an absent range with infinite length.

- `beginIndex()` and `length()` expose the typed components; `endIndex()` returns the exclusive end.
- `isValid()` requires a valid beginning and, for finite ranges, an exclusive endpoint at most `Index::maximum()`.
  An empty range at the maximum is valid; a one-element finite range beginning there is invalid.
- `endIndex()` is no-index for an invalid or infinite range. `lastIndex()` is no-index for an empty, infinite,
  or invalid range. Open-ended ranges include the maximum representable index.
- `contains(index)` tests membership. `contains(range)` requires two valid ranges and includes empty endpoints;
  a finite range cannot contain an open-ended one. `isWithin(bounds)` tests complete fit in a sequence, allowing
  an empty range at the sequence end even though that position is not an element.
- `clampedTo(bounds)` clips to a sequence length. An out-of-bounds beginning becomes empty at the sequence end;
  an invalid range becomes `noRange()`. Use `isWithin` when invalid input must be rejected instead of clipped.
- `intersection(other)` returns the common range, an empty range at the later beginning when disjoint, or
  `noRange()` if either range is invalid.
- `remainingFrom(index)` returns a suffix clamped to this range: before its beginning it returns the whole range,
  and at or beyond a finite end it returns `emptyAt(end)`. Invalid input yields `noRange()`.
- `advance` / `advanced`, `retreat` / `retreated`, and `move` / `moved` preserve length. Saturating movement also
  keeps valid ranges valid by bounding their beginning; throwing variants reject invalid input or saturation.
- `toAbsolute(origin)` and `relativeTo(origin)` translate coordinates exactly, preserving length. They return
  `noRange()` for invalid input, overflow, or subtraction below zero. Recheck sequence bounds after translation.

For example, `el::ByteRange{el::ByteIndex{2U}, el::ByteCount{5U}}.clampedTo(el::ByteCount{4U})` starts at byte 2
and has length 2; its exclusive end is byte 4.

## Iterate without raw counters

Use `count.forEach()` to visit typed indexes from zero to a finite count, or `range.forEach()` for a subrange.
Both capture their original bounds and use raw increments internally, avoiding repeated saturation checks.
Callbacks accept the matching typed index and return `void` or `el::LoopStatus`; the method returns `el::LoopResult`.

```cpp
const auto range = el::ByteRange::fromLength(el::ByteCount{10U});
const auto result = range.forEach([&](el::ByteIndex index) -> el::LoopStatus {
    // Process the byte at index.
    return el::LoopStatus::Continue;
});
```

`Continue` proceeds, `Stop` returns `LoopResult::Stopped`, and `Error` returns `LoopResult::Error`.
Empty valid iteration succeeds without calling the callback; exceptions from callbacks propagate.
An infinite count or invalid range returns `LoopResult::Error` without calling the callback.
Open-ended range iteration visits representable indexes through the maximum without wrapping, but normally bound
it with `clampedTo(actualLength)` so iteration covers real elements. Keep stop/error results when they matter.

When a manual loop is needed, keep a typed index and a finite bound. Saturating `++` under an infinite bound can
keep the loop at `maximum()` forever; decrementing zero does not advance a reverse loop toward termination.
For text, use byte iteration for byte work and character iteration or readers for decoded code points;
see [string parsing and scanning](string-parsing.md).

### Unchecked operations with proven bounds

Unchecked index movement and count arithmetic, such as `uncheckedAdvance()` and `uncheckedAdd()`, require finite
inputs and an exactly representable finite result. Use them in low-level loops only when enclosing checks already
prove those bounds; prefer `forEach()` when it fits.

`T{rawValue, T::Unchecked{}}` stores a raw representation directly, including sentinel representations. It does
not check signedness, narrowing, or range before conversion to `T::Value`. Use it only when the caller has already
proved the intended representation; keep checked factories at external boundaries.

## Cross unit and external API boundaries deliberately

Different units need a semantic conversion, not a cast of the stored number. For UTF-8, use the string's
`indexAt(CpIndex)` and `toCharIndex(ByteIndex)` where appropriate; they inspect the text and can require a scan.
A raw rewrap from `CpIndex` to `ByteIndex` does not perform encoding conversion.
See [string coordinates](strings.md#coordinates-and-parsing).

At native or standard-library interfaces:

- Import integers using the [runtime factories](#construct-constants-and-runtime-values-deliberately).
- Use `toSizeT()` to export an index or count, or `toSizeTOrThrow()` for a representability check. Check
  `isNoIndex()` / `isInfinite()` first when the target requires a real position or size: the throwing export checks
  integer width, not sentinel meaning.
- Use `toRawValue()` only when the interface genuinely needs the underlying integer. Explain non-obvious unit,
  origin, scale, or sentinel translations at that boundary, and keep the conversion outside processing loops when possible.

## Version components

`Major`, `Minor`, `Revision`, and `BuildNumber` are distinct `VersionComponent` aliases, each selected by a
`VersionPart` enumerator. Each stores an unsigned 16-bit field over the full range 0–65535, without sentinel states.
They are separate from the positional index/count/offset families and from ratio-scaled `IntegerAmount` quantities.

Numeric component constructors and the raw numeric `Version` constructor validate constants at compile time.
For runtime values, create typed components with `fromInteger()` / `fromIntegerOrThrow()` and pass those to
`Version`. Typed components may appear in any order; omitted components default to zero and duplicates are rejected
at compile time. `fromInteger()` clips to 0–65535; use `fromIntegerOrThrow()` when out-of-range input is an error.
With `<erbsland/Major.hpp>`, `<erbsland/Minor.hpp>`, and `<erbsland/Version.hpp>`:

```cpp
const auto major = el::Major::fromIntegerOrThrow(runtimeMajor);
const auto version = el::Version{major, el::Minor{2U}};
```

Components support comparison, hashing, formatting, saturating increment/decrement, and addition/subtraction of
unsigned scalar counts. `addedOrThrow()` / `subtractedOrThrow()` and their mutating variants reject results outside
0–65535.

## Other useful helpers and verification

`Index::fromGrid(x, y, width, height)` computes a row-major position using one matching unit family. It returns
no-index for invalid coordinates/dimensions or unrepresentable arithmetic. `index.mirroredWithin(count)` mirrors
a valid position within a finite count, returning no-index for an infinite count or an out-of-bounds position.

Confirm unfamiliar methods and version differences in:

- `src/erbsland/unit/IntegerUnitIndex.hpp`: movement, distances, offsets, grid helpers, native conversions.
- `src/erbsland/unit/IntegerUnitCount.hpp`: counts, infinity, remaining counts, scalar arithmetic, iteration.
- `src/erbsland/unit/IntegerUnitOffset.hpp`: signed movement and magnitude.
- `src/erbsland/unit/IntegerUnitRange.hpp`: endpoints, clipping, coordinate translations, iteration.
- Concrete `..._fwd.hpp` aliases and `...Unit.hpp` definitions: family names and unit tags.
- `src/erbsland/unit/VersionComponent.hpp` and `Version.hpp`: component construction and version contracts.
- `doc/reference/unit/units_and_versions.rst` and the unit suites in `test/unittest/src/unit`: contracts and edge cases.

Use these as behavioral evidence and keep application code on public includes and consumer names.
