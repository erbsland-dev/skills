# Find Core functionality by task

Use this map before implementing a helper or standard-library workaround. It supplies search seeds, not a complete
API catalog. Confirm signatures, availability, and exact public include paths in the actual dependency.
Names use default consumer spelling; see [consumer conventions](consumer-conventions.md) for exceptions.

Documentation paths below are relative to the Core root. With Knowledge tools, use document paths returned by search
rather than assuming the index contains the same layout as the checkout.

## Text

Read [strings](strings.md) for string semantics, examples, and allocation choices.

| Task / useful search terms | Start with | Local documentation |
| --- | --- | --- |
| Trim, replace, remove, Unicode normalization | `el::String`, `el::CharSet`; `trimmed`, `replacedAll`, `removedAll`, `normalized` | `doc/topics/text_strings/` |
| Lowercase, uppercase, case folding, identifier normalization, custom mapping, filtering, truncation | `String::transformed` with `Char::toAsciiLowercase`, `toAsciiUppercase`, `toLowercase`, `toUppercase`, `toIdentifierNormalized`, `caseFolded`; `Char::noCodePoint()` / `Char::endOfData()`; read [transformations](string-transformation.md) | `doc/topics/text_strings/transforming_strings.rst`; `src/erbsland/text/Char.hpp` |
| Extract fields, slice, split, delimiter, retain separators | `el::ByteRange`, `el::CpRange`, `StringList::fromSplit`, `StringSplitter::forEach`; read [splitting](string-splitting.md) | `doc/topics/text_strings/slicing_and_splitting_strings.rst`; `doc/reference/text/matching_and_splitting.rst` |
| Concatenate fragments, join, format, construct efficiently | `String::fromJoined`, `StringList::join`, `el::StringEditor`, `el::AnyStringBuilder`, `el::StringFormat`; read [formatting](string-formatting.md) | `doc/topics/text_formatting/` |
| Parse Boolean, integer, float, complete value, overflow | `String::toBooleanOrThrow`, `toIntegerOrThrow<T>`, `toFloatOrThrow`; read [parsing](string-parsing.md) for target-type checks and options | `doc/topics/text_parsing_and_encoding/converting_text_and_scalar_values.rst` |
| Literal search, prefix/suffix, occurrence count, character validation | `startsWith`, `endsWith`, `contains`, `count`, `find` with `Char::compareAsciiFolded` or another comparator; `containsOneOf`, `containsOnly`, `findFirstOf` / `findFirstNotOf` / `findLastOf` / `findLastNotOf` with `CharSet`; read [matching](string-matching.md) | `doc/topics/text_strings/comparing_strings.rst`; `doc/topics/text_strings/working_with_character_sets.rst` |
| Lightweight glob, prefix/suffix pattern, extract middle | `el::StringPattern::fromDivider`, `fromGlob`, `fromPosixGlob`, `el::pattern`; read [matching](string-matching.md#stringpattern-choose-divider-or-glob-semantics) | `doc/topics/text_parsing_and_encoding/using_string_patterns.rst` |
| Regular expressions, captures, Unicode, stream matching | `el::RegEx::compile` / `lazyCompile`; read [matching](string-matching.md#use-the-erbsland-regular-expression-engine) for factory, syntax, ownership, and reuse | `doc/topics/re/syntax/`; `doc/reference/re/regular_expressions.rst` |
| Suggestions, edit distance, fuzzy matching | `el::fuzzy::Matcher` returns `el::fuzzy::Match`; read [matching](string-matching.md#fuzzy-suggestions) | `doc/topics/text_strings/fuzzy_matching.rst` |
| UTF conversion, encoding errors, Base64, Base32, IDNA, Punycode | `StringConverter`, encoding/decoding APIs, `text::base_n`, `punycode`; search by required encoding | `doc/topics/text_parsing_and_encoding/`; `doc/reference/text/encoding.rst` |
| Named-key parameters, aliases, compact option grammar | Search "named key parser" and the required syntax | `doc/reference/text/named_keys.rst` |
| Structured documents and HTML fragments | `el::TextDocument`, `el::html::HtmlParser`; search "document renderer" | `doc/topics/text_rendering/`; `doc/reference/text/documents_and_rendering.rst` |
| Template files, conditional/repeated sections, Jinja-like syntax | `el::text::render::Environment`, `Context`, and loaders; implements a defined Jinja-like subset | `doc/topics/text_rendering/rendering_text_layouts.rst`; `doc/topics/text_rendering/template_language_syntax.rst` |
| Named substitutions, filters, placeholder sources | `el::text::placeholder::Replacer`, `Source`, `Filter` | `doc/topics/text_placeholders/`; `doc/reference/text/placeholders.rst` |
| Character search, indexed iteration, sequential parsing, captures | `String::findFirstOf`, `forEach`, `readCharAndAdvance`, `el::StringCharReader`; read [parsing](string-parsing.md) to choose | `doc/reference/text/formatting_and_parsing.rst`; `doc/topics/text_strings/character_access_and_parsing.rst` |

## Time, files, and data

| Task / useful search terms | Start with | Local documentation |
| --- | --- | --- |
| Calendar date, daily clock, local zone, ISO output | `el::Date`, `el::Time`, `el::DateTime`, `el::TimeZone`, `el::TimeWithZone` | `doc/topics/time/overview.rst` |
| Record an instant, fixed duration, calendar changes | `el::Timestamp`, `el::Duration`, `el::TimeDelta`, `el::CalendarDelta` | `doc/topics/time/`; `doc/reference/time/` |
| Measure elapsed time, deadline, processing budget | `el::ElapsedTimer`, `el::TimePoint` | `doc/topics/time/measuring_elapsed_time.rst`; `doc/topics/time/working_with_monotonic_time_points.rst` |
| Paths, metadata, whole-file content, traversal, file operations | `el::Path`: `info()`, `content()`, `walker()`, `operations()` | `doc/topics/path/overview.rst` |
| Byte I/O, text I/O, encoding boundary, buffering, timeout | `el::ByteInputStream`, `el::ByteOutputStream`, `el::TextInputStream`, `el::TextOutputStream` | `doc/topics/stream/overview.rst` |
| Standard input/output, formatted console output | `el::io` helpers and standard-stream proxies | `doc/topics/stream/standard_streams.rst` |
| JSON, XML, BSON, CBOR, parse limits, value trees | `el::json`, `el::xml`, `el::bson`, `el::cbor` | `doc/topics/data/overview.rst`; `doc/reference/data/` |
| ELCL configuration, validation, defaults, placeholders, value paths | `el::conf::Parser`, `el::conf::vr::RulesBuilder`, `Rules`, `Value::getTextOrThrow`; read [configuration](configuration.md) | `doc/topics/conf/`; `doc/reference/conf/` |
| CLI options, choices, sets, help output | `el::Options`, `el::OptionSet`, `el::OptionChoices`, `el::OptionChoiceEditor`, `el::OptionManager`; read [command-line options](command-line-options.md) | `doc/topics/options/`; `doc/reference/options/command_line_options.rst` |

Choose monotonic time for elapsed measurements; calendar and civil-time types have different semantics.
For streams, distinguish timeout from normal completion and failure. Verify filesystem collision, symlink, and
throwing/nonthrowing behavior before choosing an operation.

## Infrastructure and other domains

| Task / useful search terms | Start with | Local documentation |
| --- | --- | --- |
| Bytes, shared immutable data, mutation, endian access, bit fields | `el::ByteBlock`, `el::ByteBlockEditor`, `el::ByteReader`, `el::ByteWriter`, `el::BitReader`, `el::BitWriter` | `doc/reference/mem/memory_and_byte_data.rst` |
| Lists, maps, sets, enum flags, results, coroutines | `el::List`, `el::Map`, `el::HashMap`, `el::Set`, `el::EnumFlags`, `el::Result`, `el::CoTask` | `doc/reference/util/utilities.rst` |
| Typed indexes, counts, offsets, ranges, versions | `el::ByteIndex`, `el::ByteLength`, `el::CpIndex`, `el::CpLength`, `el::ItemCount`, `el::Version`; read [unit types](unit-types.md) for sequence coordinates, arithmetic, and conversions | `doc/reference/unit/units_and_versions.rst` |
| Overflow handling, saturation, large integers | `el::BoundedInteger`, `el::SaturatingInteger`, `el::BigInteger`; search required arithmetic policy | `doc/reference/math/mathematics.rst` |
| Error handling, results, exceptions, diagnostic documents | `el::ApplicationError`, `el::Result`, `el::ResultWithData`, `el::DiagnosticHelper`; read [error handling](errors.md) for consumer patterns | `doc/topics/err/`; `doc/reference/err/errors_and_diagnostics.rst` |
| App startup, lifecycle, service parts | `el::Application`, `el::ApplicationPart`, `el::ApplicationPartManager`; read [application designs](applications.md) when choosing control flow | `doc/topics/core/choosing_an_application_design.rst` |
| Events, scheduling, timers, event threads | `el::EventLoop`, `el::EventTimer`, `el::EventSubscription`, `el::ManagedEventThread` | `doc/topics/event/`; `doc/reference/event/event_system.rst` |
| Logs, sinks, formatting, rotation | `el::LogManager`, `el::LogConfiguration`, `el::LogWriter` | `doc/topics/log/`; `doc/reference/log/logging.rst` |
| IP addresses, hostnames, connections, HTTP, URLs | `el::IpAddress`, `el::Host`, `el::Network`; search the protocol and operation | `doc/topics/network/`; `doc/reference/network/` |
| Environment, process information, subprocess capture | `el::EnvironmentVariables`, `el::ProcessInfo`, `el::Subprocess`, `el::sys_info` | `doc/reference/system/system_services.rst` |
| Random values, secure random bytes | `el::FastRandom`, `el::SecureRandom` | `doc/topics/random/`; `doc/reference/random/random_numbers.rst` |
| Hashing, HMAC, passwords, keys, TLS | `el::Hasher`, `el::Hmac`, `el::PasswordHasher`; search specific operation | `doc/topics/cryptology/`; `doc/reference/cryptology/` |
| Compression, decompression, ZIP archives | `el::ByteCompressor`, `el::ByteDecompressor`; search "ZIP archive" | `doc/topics/compression/`; `doc/reference/compression/` |
| Compiled resources, embedded files | `el::ResourceManager`, `el::Resources` | `doc/topics/resource/`; `doc/reference/resource/compiled_resources.rst` |
| Terminal output, input, retained buffers, layout | `el::cterm::Terminal`, `el::cterm::TerminalSession`, `el::cterm::CursorBuffer`, `el::cterm::GridLayout` | `doc/topics/cterm/`; `doc/reference/cterm/` |
| Coordinates, rectangles, axes, alignment | `el::block::Rectangle`, `el::block::Size`, `el::Axis`, `el::Alignment` | `doc/reference/block/`; `doc/reference/geometry/` |
| Display-text translation | `el::DisplayTextTranslator`, `el::DisplayTextMap` | `doc/reference/i18n/internationalization.rst` |
| Debugging helpers | Search the specific diagnostic task | `doc/reference/debug/debugging.rst` |

When no row fits, inspect `doc/topics/index.rst`, `doc/reference/index.rst`, and the relevant source domain before
concluding that Core lacks the functionality. Prefer a narrow search over loading the complete library into context.
