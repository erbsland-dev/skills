# Value Types and Important Parsing Rules

Use this file for choosing value forms and avoiding ambiguous/invalid literals.

## Core Types

- Integer: decimal, hex (`0x`), binary (`0b`), optional sign, optional `'` separators.
- Boolean literals (case-insensitive): `true/false`, `yes/no`, `on/off`, `enabled/disabled`.
- Text: double-quoted, supports escapes `\\`, `\"`, `\$`, `\n`, `\r`, `\t`, `\uXXXX`, `\u{1..8 hex}`.

## Standard Types

- Float:
  - Decimal only
  - `inf`, `nan` supported (case-insensitive)
  - Decimal point or exponent required to be a float
- Byte count:
  - Decimal integer plus optional space plus unit
  - Units: `kb/mb/gb/tb/pb/eb/zb/yb` and binary forms `kib/.../yib`
- Date: `YYYY-MM-DD` (year `0001..9999`, Gregorian validity required)
- Time: `[t]HH:MM[:SS[.fraction]]` with optional `Z` or `+/-HH[:MM]`
- DateTime: `YYYY-MM-DD[ T]HH:MM[:SS[.fraction]][offset]`
- Code text:
  - Single-line with `` `...` ``
  - Multi-line with triple backticks
  - No escape processing in code text
- Byte data:
  - Single-line `<...>` with optional `<hex: ...>`
  - Multi-line `<<< ... >>>` with optional `<<<hex`

## Advanced Types

- Regular expression:
  - Single-line `/.../`
  - Multi-line `/// ... ///`
  - Parser primarily needs to handle `\/` and `\\` correctly
- Time delta:
  - Decimal integer + optional space + unit
  - Required support at least: seconds, minutes, hours, days, weeks
  - Optional units include `ns`, `us`/`µs`, `ms`, months, years

## Integer/Float Limits

- Integer digit limits (64-bit baseline): decimal 19, hex 16, binary 64.
- Float allows digit separators in integral/fractional parts.
- Float total digits (integral + fractional) must stay within spec limits.

