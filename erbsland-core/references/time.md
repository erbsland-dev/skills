# Choose Core time types

Read this when choosing time quantities, measuring elapsed time, storing instants, or doing calendar and time-zone
work. Prefer Core types over `std::chrono`; keep required native/standard-library conversions at external boundaries.
Use public headers such as `<erbsland/TimePoint.hpp>` and consumer names such as `el::TimePoint`.

## Choose by meaning

| Need | Type and reason |
| --- | --- |
| A quantity in a named unit, such as a delay or timeout | `el::Nanoseconds`, `Microseconds`, `Milliseconds`, `Seconds`, `Minutes`, `Hours`, `Days`, `Weeks`, `Months`, `Years` from `<erbsland/TimeAmounts.hpp>`; typed conversion, comparison, and arithmetic |
| Elapsed measurement or a deadline | `el::TimePoint`: efficient, platform-appropriate steady-clock time; `el::ElapsedTimer` for a timer abstraction |
| Compact, fast storage and comparison of dated instants | `el::Timestamp`: UTC instant with nanosecond precision, without display-zone metadata |
| Calendar calculations, local fields, time-zone conversion | `el::DateTime`: UTC storage plus display-zone information |
| Fast, precise signed interval | `el::TimeDelta`: signed 64-bit nanoseconds, approximately ±292 years |
| Larger-range interval for date/time calculations | `el::Duration`: signed whole seconds; subsecond precision is discarded |
| A change containing months or years | `el::CalendarDelta`: independent, non-normalized components, including fixed units, months, and years |
| Separate date, clock time, or clock time with a zone | `el::Date`, `el::Time`, `el::TimeWithZone` |
| An individual calendar or clock field | `el::Year`, `el::Month`, `el::Day`, `el::Hour`, `el::Minute`, `el::Second` |

## Keep amounts typed

Use `el::Milliseconds{250}` rather than an unlabelled integer or a chrono duration. Amounts accept signed counts,
including runtime values; do not apply the sequence index/count construction rules to them. Arithmetic and comparison
require matching amount types. Convert explicitly with `converted<T>()`, then calculate without extracting raw counts:

```cpp
const auto delay = el::Milliseconds{250};
const auto budget = el::Seconds{2}.converted<el::Milliseconds>();
const auto remaining = budget - delay * 3;
```

Fixed amounts from nanoseconds through weeks share a conversion group. Coarser conversions truncate toward zero;
finer conversions can overflow. Ordinary conversions and arithmetic saturate. `convertedOrThrow<T>()` rejects
overflow, but still truncates fractions. `toRawValue()` only extracts the current unit's count; it does not convert units.

`Months` and `Years` have distinct calendar dimensions: neither converts to fixed units or to the other through
the amount conversion API. Keep them in `CalendarDelta`. `Days{1}` is a fixed 86,400 seconds, which does not guarantee
the same local clock reading across a daylight-saving transition.

Singular fields and plural amounts have different meanings: `Day{15}` selects a day of the month, while `Days{15}`
is a quantity. Use the field types to build or inspect dates/times and the amount types to describe changes.

### Use time literals for constants

Include `<erbsland/time/Literals.hpp>` and import `using namespace el::time::literals;` in the local scope where
needed. Prefer these readable literals for constant amounts: `_ns`, `_us`, `_ms`, `_s`, `_m`, and `_h` produce
`Nanoseconds`, `Microseconds`, `Milliseconds`, `Seconds`, `Minutes`, and `Hours`, respectively.

```cpp
using namespace el::time::literals;
const auto delay = 250_ms;
const auto budget = (2_s).converted<el::Milliseconds>();
const auto deadline = el::TimePoint::inFuture(el::TimeDelta{delay});
```

Literals create Core amounts and follow the same conversion and arithmetic rules. Use integer spellings such as
`1500_ms` for fractional seconds; `1.5_s` is unsupported. Use explicit amount construction for days, weeks, months,
and years. Keep the underscore in the suffix; `250ms` belongs to `std::chrono_literals`.

## Measure with TimePoint

Use `TimePoint::now()`, `TimePoint::inFuture(delta)`, comparisons, and delta arithmetic instead of choosing a platform
clock or manually manipulating ticks. Default construction is the clock epoch, not the current time.

```cpp
const auto start = el::TimePoint::now();
const auto deadline = el::TimePoint::inFuture(el::TimeDelta{el::Milliseconds{250}});
const auto elapsed = start.timeDeltaToNow();
const auto remaining = el::TimePoint::now().timeDeltaTo(deadline);
const auto expired = el::TimePoint::now() >= deadline;
```

`start.timeDeltaTo(end)` is `end - start`, positive when the target is later. Adding or subtracting a `TimeDelta`
shifts a point. A steady-clock point is for elapsed time and deadlines; use `Timestamp` for persisted event dates
and epoch conversions. Nanosecond delta resolution does not promise nanosecond hardware clock accuracy.

## Store instants, calculate calendars, choose the display zone

Prefer `Timestamp` for memory-efficient collections, copying, comparison, and fixed-interval arithmetic.
Use `DateTime` for calendar manipulation and time-zone work. Both provide date/time and epoch conversion tools;
look up the needed operation before writing manual tick, calendar, or offset arithmetic.

`DateTime` stores its date and time internally as **UTC**, even when displayed in another zone. This keeps instant
calculations and conversion to/from `Timestamp` efficient. Formatting, extracting local fields, and resolving zones
add work; a zoned display does not imply that every calculation requires conversion from local time.
`utcDate()` / `utcTime()` access stored UTC fields; `date()` / `time()` access display-local fields. Keep paired
fields in the same view. Use `parts()` when extracting several local components together.

- `el::Timestamp::fromDateTime(value)` copies the UTC instant and discards display metadata.
- `timestamp.toDateTime()` produces a UTC `DateTime`. Both conversions return optionals; `OrThrow` forms reject
  invalid input. Choose the display zone again with `toTimeZone(zone)` when needed.
- `toTimeZone(zone)` changes the display zone while preserving the instant. Prefer a named `TimeZone` when rules
  vary by date; a fixed numeric offset does not encode those rules.
- Default `Timestamp` and `DateTime` values are invalid. Check validity or use the appropriate checked API.
- For epoch interchange, choose both the tick unit and `TimeEpoch` explicitly. The default Core epoch is not POSIX.
  Check the specific conversion's range, signed-input, precision, and failure contracts.

Choose delta precision deliberately. `Timestamp::timeDeltaTo()` returns a precise `TimeDelta`, but widely separated
valid timestamps may exceed its range; `durationTo()` returns a whole-second `Duration` with a larger range.
`Timestamp` subtraction returns `TimeDelta`, while `DateTime` subtraction returns `Duration`.
Use `CalendarDelta` for month/year changes; inspect the operation's month-end, application-order, and zone semantics
when implementing recurring local schedules. A calendar change is not a guessed number of seconds.

Use `Date`, `Time`, or `TimeWithZone` for partial values instead of inventing a dummy date or storing a full instant.
A `TimeWithZone` needs a date to resolve a named zone to a UTC offset and identify an instant.

## Verify the operation in the dependency

Search the relevant type under `src/erbsland/time/`; amount mechanics live in `src/erbsland/unit/IntegerAmount.hpp`.
For examples, use the matching `doc/topics/time/working_with_time_amounts.rst`, `working_with_monotonic_time_points.rst`,
`writing_time_literals.rst`, `working_with_timestamps.rst`, `working_with_datetime.rst`, `working_with_time_deltas.rst`,
or `working_with_durations.rst`.
These paths are relative to Core. Confirm unfamiliar signatures and version differences against the actual headers.
