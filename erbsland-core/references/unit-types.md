# Use typed indexes, lengths, offsets, and ranges

Read this when choosing sequence coordinates, doing index arithmetic, iterating over typed ranges, or converting
between Core units and an external API. Keep values typed throughout processing; do not extract raw integers just
to do arithmetic or change an index into a length.

This reference covers the sequence-unit families below. Time quantities and version components have separate
contracts. Paths in the verification section are relative to the Core dependency.

## Choose both the unit and the intent

| Intent | Meaning | Example |
| --- | --- | --- |
| `...Index` | A zero-based, non-negative position, or `noIndex()` for an absent position | `el::ByteIndex{3U}` |
| `...Length` / `...Count` | A non-negative amount, including zero, or `infinite()` | `el::ByteLength{4U}` |
| `...Offset` | Signed movement relative to a position | `el::ByteOffset{-2}` |
| `...Range` | A start index and a length/count; the end is exclusive | `el::ByteRange{el::ByteIndex{3U}, el::ByteLength{4U}}` |

Use the family required by the data and API:

| Unit | Index | Length/count | Offset | Range |
| --- | --- | --- | --- | --- |
| Bytes, including UTF-8 storage | `ByteIndex` | `ByteLength` | `ByteOffset` | `ByteRange` |
| Decoded Unicode code points | `CpIndex` | `CpLength` | `CpOffset` | `CpRange` |
| UTF-16 code units | `U16DataIndex` | `U16DataLength` | `U16DataOffset` | `U16DataRange` |
| Source lines | `LineIndex` | `LineCount` | `LineOffset` | `LineRange` |
| Source columns | `ColumnIndex` | `ColumnCount` | `ColumnOffset` | `ColumnRange` |
| Collection items | `ItemIndex` | `ItemCount` | `ItemOffset` | `ItemRange` |

Names in the table use the consumer prefix `el::` and per-type public headers such as `<erbsland/ByteIndex.hpp>`.
The byte amount is `ByteLength`; do not invent `ByteCount` or a `Bytes...` prefix. `Length` and `Count` express the
same amount intent, but the public alias spelling depends on the family.

A unit and its intent are both part of the type. A `ByteIndex` is neither a `ByteLength` nor a `CpIndex`.
Use `List::count()` as an `ItemCount` and `String::length()` as a `ByteLength`. Code-point positions do not measure
grapheme clusters or display cells, and a UTF-16 code unit is not necessarily a whole code point.

## Factories and state tests

Prefer named factories and predicates for special values and common comparisons.

| Kind | Useful factories | Useful predicates |
| --- | --- | --- |
| Index | `zero()`, `one()`, `minimum()`, `maximum()`, `noIndex()` | `isZero()`, `isOne()`, `isMinimum()`, `isMaximum()`, `isNoIndex()`, `isValid()`, `isWithin(length)` |
| Length/count | `zero()`, `one()`, `minimum()`, `maximum()`, `infinite()` | `isZero()`, `isOne()`, `isMinimum()`, `isMaximum()`, `isInfinite()`, `isFinite()` |
| Offset | `zero()`, `one()`, `minusOne()`, `minimum()`, `maximum()` | `isZero()`, `isOne()`, `isMinusOne()`, `isMinimum()`, `isMaximum()`, `isNegative()`, `isPositive()` |
| Range | `empty()`, `emptyAt(index)`, `all()`, `noRange()`, `fromLength(length)` | `isEmpty()`, `isValid()`, `isInfinite()`, `contains(index)`, `isWithin(length)` |

Default construction yields zero for scalar units and an empty range at zero. `maximum()` is a finite value,
separate from `noIndex()` or `infinite()`. Use `index.isZero()` or `index == el::ByteIndex{10U}` instead of comparing
raw values. Ordinary ordering is available between values of the same type; use predicates to recognize sentinels.

`index.isValid()` only means it is not `noIndex()`. For element access, use `index.isWithin(length)`, which excludes
the end position. An empty range can be valid, and `noRange()` is both empty and invalid: `isEmpty()` cannot stand
in for `isValid()`.

## Arithmetic: keep the intent visible

| Operation | Result / preferred spelling |
| --- | --- |
| Index plus/minus matching length | Index: `index + length`, `index - length` |
| Index plus/minus matching offset | Index: `index + offset`, `index - offset` |
| Length/count plus/minus the same amount type | Length/count |
| Offset plus/minus the same offset type | Offset |
| Length/count multiplied, divided, or modulo an unsigned scalar | Same amount type: `length * 3U`, `length / 2U`, `length % 2U` |
| Offset multiplied, divided, or modulo a signed scalar | Same offset type: `offset * 3`, `offset / 2`, `offset % 2` |
| Range plus/minus a matching offset | Range with a moved start and unchanged length |

Do not add two indexes or subtract one index from another. Use the distance/offset methods below. Unit values
cannot be multiplied or divided by other unit values; scalar operations preserve the unit rather than creating
a new dimension.

For indexes, `advance(length)`, `retreat(length)`, and `move(offset)` mutate the receiver; `advanced`, `retreated`,
and `moved` return a new value. `increment()` / `decrement()` mutate by one; `incremented()` / `decremented()`
return a new index. Lengths and offsets use `add` / `added` and `subtract` / `subtracted`. All three scalar intents
support `++` and `--`.

```cpp
auto position = el::ByteIndex{4U};
const auto next = position.incremented();                  // ByteIndex{5U}; position stays 4.
position.advance(el::ByteLength{10U});                     // position is 14.
const auto previous = position + el::ByteOffset{-2};       // ByteIndex{12U}.
const auto doubled = el::ByteLength{6U} * 2U;              // ByteLength{12U}.
```

### Saturation and exactness

Ordinary arithmetic saturates at the type's finite bounds: subtracting too much from a finite length produces
zero; advancing a finite index too far produces `maximum()`, never `noIndex()`. A no-index value stays no-index
when moved. Finite amount arithmetic does not overflow into infinity. Offsets saturate at their signed limits.

Special states have explicit rules. Adding an infinite amount yields infinity. Subtracting from infinity keeps
infinity, including infinity minus infinity; subtracting infinity from a finite amount yields zero. Infinite
amounts remain infinite under scalar multiplication, division, and modulo. Division or modulo by zero terminates;
saturation does not make a zero divisor valid.

Use normal saturating operations when clipping is the intended behavior; do not add manual overflow checks around
every operation. When an exact result is required, choose the available `OrThrow` operation, such as
`advancedOrThrow`, `moveOrThrow`, `addedOrThrow`, or `offsetToOrThrow`. These report `OverflowError` for unsupported
special states or an unrepresentable result according to their contract. Do not assume every operation has an
`OrThrow` variant.

Type bounds are not sequence bounds. A representable index can still be outside a buffer. Neither saturation nor
`advanceOrThrow()` proves an index is within a particular sequence. `wouldBeWithinAfterAdvance(step, bounds)`
(and the retreat/move variants) tests the **saturated** result against sequence bounds; it is not an overflow test.
Use `wouldAddSaturate`, `wouldSubtractSaturate`, or the specific conversion predicate when a separate branch is
needed to handle loss of exactness.

## Convert between intents with named operations

All conversions in this table stay within the same unit family.

| Needed value | Operation | Meaning / special state |
| --- | --- | --- |
| Length from zero to an index | `index.distanceFromZero()` | Distance, not the count including that element; no-index becomes infinity |
| Absolute distance between indexes | `first.absoluteDistanceTo(second)` | Non-negative length; either no-index makes it infinite |
| Signed movement from one index to another | `first.offsetTo(second)` | `second - first`, saturated to the offset limits; either no-index yields the maximum offset |
| Signed movement from zero | `index.offsetFromZero()` | Saturates if too large or no-index |
| Exclusive end of a length/count starting at zero | `el::ByteIndex::end(length)` | Finite length becomes that index; infinity becomes no-index |
| Magnitude of an offset | `offset.absoluteLength()` | Matching non-negative length; direction is discarded |
| Range starting at zero | `el::ByteRange::fromLength(length)` | Preserves the supplied length |

Use `offsetToOrThrow()` / `offsetFromZeroOrThrow()` when a saturated signed distance would be wrong. Unsigned
index space can be larger than the positive range of an offset. The corresponding predicates are
`wouldOffsetToSaturate()` and `wouldOffsetFromZeroSaturate()`.

```cpp
const auto begin = el::ByteIndex{3U};
const auto end = el::ByteIndex{8U};
const auto length = begin.absoluteDistanceTo(end);         // ByteLength{5U}.
const auto backwards = end.offsetTo(begin);               // ByteOffset{-5}.
const auto prefixLength = begin.distanceFromZero();       // ByteLength{3U}, not 4.
const auto prefixEnd = el::ByteIndex::end(prefixLength);   // ByteIndex{3U}.
```

For a nonempty finite sequence, the last element is one before its exclusive end. Check emptiness before using
`end(length).decremented()` as a last-element index: saturating decrement of zero still yields zero.

## Construct, bound, and move ranges

A range describes `[begin, end)`. Construct it with `{startIndex, length}` or `{beginIndex, endIndex}`. The latter
expects ordered endpoints. A reversed end currently produces an empty range, so validate ordering if reversal is
an input error. A no-index end produces an infinite length, not an absent range; check search results before
using them as endpoints.

- `index()` and `length()` expose the typed components; `endIndex()` returns the exclusive end.
- `endIndex()` is no-index for an invalid range, an infinite range, or an end beyond representable index space.
- `contains(index)` tests membership; `isWithin(bounds)` tests whether the whole range fits the sequence.
  A valid empty range at the sequence end fits, even though that index is not an element within the sequence.
- `clampedTo(bounds)` clips to a sequence length. An out-of-bounds start becomes empty at the sequence end;
  an invalid range stays invalid. Use it when clipping is intended; use `isWithin` when invalid input must be rejected.
- `all()` starts at zero with infinite length. Bound it with `clampedTo(actualLength)` before enumerating it.
- `advance` / `advanced`, `retreat` / `retreated`, and `move` / `moved` shift the start without changing the length.
  `withOrigin(origin)` interprets the current start relative to another origin. Recheck sequence bounds after moving.

For example, `el::ByteRange{el::ByteIndex{2U}, el::ByteLength{5U}}.clampedTo(el::ByteLength{4U})` starts at byte 2
and has length 2; its exclusive end is byte 4.

## Iterate without raw counters

Use a typed index and a finite bound:

```cpp
const auto length = el::ByteLength{10U};
for (auto index = el::ByteIndex::zero(); index.isWithin(length); ++index) {
    // Process the byte at index.
}
```

For a range, `forEach()` supplies the matching typed index. The callback may return `void`, or `el::LoopStatus`
for explicit loop control; the method returns `el::LoopResult`.

```cpp
const auto range = el::ByteRange::fromLength(el::ByteLength{10U});
const auto result = range.forEach([&](el::ByteIndex index) -> el::LoopStatus {
    // Process the byte at index.
    return el::LoopStatus::Continue;
});
```

A valid empty range succeeds without invoking the callback; an invalid range returns `LoopResult::Error`.
Keep the result when callback stop/error outcomes matter. Use a finite range bounded to the actual sequence.
An infinite bound with saturating `++` can keep a manual loop at `maximum()` forever; similarly, decrementing zero
does not advance a reverse loop toward termination.

For text, byte iteration is suitable for byte work. Use string character iteration or readers for decoded code
points; see [string parsing and scanning](string-parsing.md). `uncheckedAdvance()` and `uncheckedIncrement()` are
reserved for measured low-level loops with already-proven finite inputs and result bounds.

## Cross unit and external API boundaries deliberately

Different units need a semantic conversion, not a cast of the stored number. For UTF-8, use the string's
`indexAt(CpIndex)` and `toCharIndex(ByteIndex)` where appropriate; they inspect the text and can require a scan.
A raw rewrap from `CpIndex` to `ByteIndex` does not perform encoding conversion. See [string coordinates](strings.md#coordinates-and-parsing).

At native or standard-library interfaces:

- Use `fromSizeT()` on indexes or lengths/counts to import `std::size_t`; oversized inputs saturate to the largest
  finite value. Use `fromSizeTOrThrow()` when truncating the represented position or size would be wrong.
- Use `toSizeT()` to export an index or length/count, or `toSizeTOrThrow()` for a representability check. Check
  `isNoIndex()` / `isInfinite()` first when the target requires a real position or size: the throwing export checks
  integer width, not sentinel meaning.
- Direct raw-value constructors accept the underlying representation, including sentinel values. They are not a
  general checked conversion from arbitrary signed or wider integers. Validate external signed values before
  converting them to unsigned sizes.
- Use `toRawValue()` only when the interface genuinely needs the underlying integer. Explain non-obvious unit,
  origin, scale, or sentinel translations at that boundary, and keep the conversion outside processing loops when possible.

## Other useful helpers and verification

`Index::fromGrid(x, y, width, height)` computes a row-major position using one matching unit family. It returns
no-index for invalid coordinates/dimensions or unrepresentable arithmetic. `index.flipped(length)` mirrors a valid
position within a finite sequence. Establish `index.isWithin(length)` and a finite length before calling `flipped`;
it does not validate an out-of-bounds finite index for you.

Confirm unfamiliar methods and version differences in:

- `src/erbsland/unit/IntegerUnitIndex.hpp`: movement, distances, offsets, grid helpers, native conversions.
- `src/erbsland/unit/IntegerUnitAmount.hpp`: lengths/counts, infinity, scalar arithmetic.
- `src/erbsland/unit/IntegerUnitOffset.hpp`: signed movement and magnitude.
- `src/erbsland/unit/IntegerUnitRange.hpp`: endpoints, clipping, origin changes, iteration.
- Concrete `..._fwd.hpp` aliases and `...Unit.hpp` definitions: family names and representation widths.
- `doc/reference/unit/units_and_versions.rst` and `test/unittest/src/unit/IntegerUnitTest.cpp`: contracts and edge cases.

Use these as behavioral evidence and keep application code on public includes and consumer names.
